# Engineering Audit

## Credentials and Trust

The original password format was `SHA-256(password + "salt")`: one fast digest with the same embedded salt for every account. A database disclosure permits inexpensive offline guessing and identical hashes reveal reused passwords. This is critical for a service with reusable credentials. New passwords use scrypt with a random 16-byte per-account salt. Login verifies legacy 64-hex digests in constant time and replaces the hash with scrypt only after the correct password is supplied. No plaintext reset or forced migration is needed; dormant accounts retain their legacy digest until their next successful login.

JWT signing used a public, hard-coded fallback key when `GRPC_JWT_SECRET` was unset in the API and both client-side server adapters. Browser session cookies had the same problem through public `SESSION_SECRET` fallbacks. An attacker could sign an arbitrary user ID or elevated role into an API token or construct a valid browser session, enabling account impersonation and privilege escalation. This is critical authentication bypass. The API refuses startup without a configured JWT secret of at least 32 characters; API token and cookie issuers likewise fail closed. The former public constants are explicitly rejected. Replacing session-cookie keys invalidates existing cookies, so affected sessions will need to sign in again. Rotate any production secret independently of this code change.

## SQL Query Counts

Counts are measured using Drizzle's query logger in the new API tests. Setup/fixture inserts are excluded. The profile case requests the page as a different authenticated user.

| Operation | Before | After | Before pattern |
| --- | ---: | ---: | --- |
| Home feed, 10 posts | 32 | 4 | Following query + post query + 3 per-post queries |
| User profile | 5 | 1 | User query + follower/following/post counts + requester follow lookup |
| Bookmarks, 10 posts | 41 | 3 | Bookmark query + 4 per-post queries |

`getPostMetrics` is the reusable batch pattern for feed/bookmark counts and liked-state: it initializes zero-valued results for requested IDs, then groups likes and comments by post ID in two queries. Bookmark page details are joined in the page query. Profile metrics are correlated subqueries in the user query. These are application query-count reductions; database indexing and pagination policy are unchanged.

## gRPC Errors and Tracing

Thrown errors now map centrally: authentication to `UNAUTHENTICATED`, authorization to `PERMISSION_DENIED`, missing records to `NOT_FOUND`, conflicts to `ALREADY_EXISTS`, invalid input to `INVALID_ARGUMENT`, and expired edit windows or invalid state transitions to `FAILED_PRECONDITION`; unexpected failures become sanitized `INTERNAL`. Existing protobuf response envelopes are preserved because clients consume their `success`/`error` fields. Consequently, handlers that catch errors still return their established envelopes or legacy fallback values rather than changing RPC status.

A gRPC interceptor creates a UUID trace ID per call, propagates it through asynchronous handler/service execution using `AsyncLocalStorage`, returns it as `x-trace-id` response metadata/trailers, and emits structured completion logs containing method, peer, status, duration, and trace ID. Unexpected errors are logged with the trace ID and details while clients receive a generic internal message.

Handler error strategy catalogue:

| Service | Current handler behavior |
| --- | --- |
| Auth | Register/login return failure envelopes; `getCurrentUser` throws; invalid `validateSession` returns `valid: false`; logout succeeds client-side. |
| Posts | Create/update/delete return failure envelopes; reads ignore invalid optional tokens and let service errors throw. |
| Comments | Create/delete return failure envelopes; comment reads ignore invalid optional tokens and let service errors throw. |
| Likes | Toggle methods return failure envelopes; status methods swallow errors as `liked: false`. |
| Follows | Toggle returns a failure envelope; status/count methods swallow errors as false/zero. |
| Feed | Home-feed authentication and service errors throw; explore ignores invalid optional tokens and propagates service errors. |
| Search | Optional invalid post-search token is ignored; search failures throw; user search is public. |
| Users | Profile reads ignore invalid optional tokens but propagate lookup failures; profile updates return a failure envelope. |
| Admin | Read/list methods throw auth, authorization, and service errors; mutations return failure envelopes. |
| Notifications | Read methods swallow errors as empty/zero; mutations return failure envelopes. |
| Bookmarks | Toggle returns a failure envelope; status/list methods swallow errors as false/empty. |

## Tests and Isolation

API service unit tests currently cover auth, comments, follows, likes, and posts. They do not directly cover admin, bookmarks, feed, mentions, notifications, search, or users services. Existing gRPC handler tests cover auth, admin, comments, likes, notifications, and posts; follows, feed, bookmarks, search, and users handler paths remain untested. Error coverage exists in selected auth/comment/post/admin tests but is not a systematic per-handler matrix. New tests cover legacy credential migration, rejection of the former JWT key, gRPC error mapping, API query budgets, notification response/fallback contracts, and output values for feed/profile/bookmarks.

API tests mock the database with an in-memory database per test file and clear tables before each test; helper-created records use generated IDs. This encourages focused service tests and avoids cross-file database state. Client unit test setups both run Testing Library cleanup after each test. The E2E suites use shared seeded accounts and a shared API database; their helpers provide unique IDs but cannot reset the separate API process. Parallel browser workers could race on shared state, so user and admin E2E workers are serialized. The global setup only warms the dev server; stateful E2E assertions should continue to use unique records and avoid depending on mutations from other tests.

User E2E suites cover auth, comments, feed, mentions, notifications, posts, profiles, search, and bookmarks. Admin E2E suites cover auth, audit logs, dashboard, users, reports, posts, navigation, and moderation workflow. The root E2E command prepares an isolated database and runs the suites sequentially; CI installs Chromium and invokes the same runner.

## CI and Turbo

Pull requests install from the frozen lockfile, then run affected-package typechecks, Biome lint on changed source files, affected unit tests/builds, and both browser E2E suites. Turbo's `...[origin/<base>]` filter includes changed packages and their dependents. `dev` remains persistent and uncached; lint no longer waits for dependency builds; database operations and integration/E2E tests are not cached because they have side effects. Build/typecheck/test retain dependency build ordering for generated and workspace artifacts.
