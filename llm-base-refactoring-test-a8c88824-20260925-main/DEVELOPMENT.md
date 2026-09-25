# Development Setup

1. Install Node.js 22 and enable Corepack.
2. Install the pinned workspace dependencies:

   ```sh
   corepack enable
   pnpm install
   ```

3. Configure `DATABASE_URL` for a LibSQL database, `GRPC_JWT_SECRET` for API tokens, and `SESSION_SECRET` for browser cookies. Use two distinct private random secrets of at least 32 characters; the API refuses to start without the JWT secret.
4. Initialize and seed the database:

   ```sh
   pnpm db:migrate
   pnpm db:seed
   ```

5. Start all applications with `pnpm dev`, or use `pnpm dev:api`, `pnpm dev:user`, and `pnpm dev:admin` individually.
6. Validate changes with `pnpm typecheck`, `pnpm lint`, `pnpm test`, and `pnpm build`.
7. Install Chromium once with `pnpm exec playwright install chromium`, then run both browser suites with `pnpm test:e2e`. The runner uses a temporary database and seeds it automatically.
8. Enable the staged-file pre-commit check with `git config core.hooksPath .githooks`.

The API listens on gRPC port `50051` and health-check HTTP port `3001`. User and admin clients use ports `3000` and `3002`.
