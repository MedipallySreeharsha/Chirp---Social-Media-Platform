# Chirp - Social Media Platform

A modern, full-stack social media platform built with a monorepo architecture using Turbo, featuring separate applications for users and administrators, along with a robust API backend.

## 📋 Overview

Chirp is a comprehensive social media solution featuring:
- **User Client**: Interactive web application for social networking
- **Admin Client**: Administrative dashboard for platform management
- **API Server**: Powerful backend service with protocol buffer support
- **Shared Packages**: Code sharing and utilities across the monorepo

## 🏗️ Project Structure

This is a **monorepo** managed with `pnpm` and `Turbo`, organized as follows:

```
chirp-monorepo/
├── apps/
│   ├── @chirp/client-user        # User-facing web application
│   ├── @chirp/client-admin       # Admin dashboard application
│   └── @chirp/api                # Backend API server
├── packages/
│   ├── @chirp/proto              # Protocol Buffer definitions
│   └── [shared utilities & types]
├── tooling/                       # Build tools and configurations
└── turbo.json                     # Turbo monorepo configuration
```

## 🚀 Quick Start

### Prerequisites
- **Node.js**: v18 or higher
- **pnpm**: v9.15.0 (as specified in package.json)

### Installation

```bash
# Install dependencies
pnpm install

# Generate protocol buffer files
pnpm proto:generate

# Generate database migrations
pnpm db:generate

# Run database migrations
pnpm db:migrate

# Seed the database (optional)
pnpm db:seed
```

### Development

```bash
# Run all development servers
pnpm dev

# Run specific application
pnpm dev:user      # User client only
pnpm dev:admin     # Admin client only
pnpm dev:api       # API server only
```

### Building

```bash
# Build all applications
pnpm build

# Build with Turbo's caching and optimizations
pnpm build
```

## 📦 Available Scripts

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start all development servers |
| `pnpm dev:user` | Start user client dev server |
| `pnpm dev:admin` | Start admin client dev server |
| `pnpm dev:api` | Start API server dev server |
| `pnpm build` | Build all applications |
| `pnpm lint` | Run linting checks |
| `pnpm lint:fix` | Fix linting issues automatically |
| `pnpm format` | Format code with Biome |
| `pnpm typecheck` | Run TypeScript type checking |
| `pnpm test` | Run all tests |
| `pnpm test:unit` | Run unit tests |
| `pnpm test:integration` | Run integration tests |
| `pnpm test:e2e` | Run end-to-end tests |
| `pnpm proto:generate` | Generate Protocol Buffer types |
| `pnpm db:generate` | Generate database schema |
| `pnpm db:migrate` | Run database migrations |
| `pnpm db:seed` | Seed the database |
| `pnpm clean` | Clean build artifacts and cache |

## 🛠️ Technology Stack

### Core
- **Node.js**: JavaScript runtime
- **TypeScript**: Type-safe JavaScript
- **pnpm**: Fast, disk-efficient package manager
- **Turbo**: High-performance build system

### Development & Quality
- **Biome**: Fast JavaScript toolchain (linting & formatting)
- **Vitest**: Fast unit testing framework
- **Playwright**: End-to-end testing
- **Faker.js**: Test data generation

### Architecture
- **Protocol Buffers**: Efficient data serialization for APIs
- **Monorepo**: Unified workspace for multiple applications

## 📝 Development Guidelines

### Code Quality
- **Linting**: Runs via `pnpm lint` (using Biome)
- **Formatting**: Runs via `pnpm format` (using Biome)
- **Type Checking**: Runs via `pnpm typecheck` (using TypeScript)

### Testing
- **Unit Tests**: `pnpm test:unit`
- **Integration Tests**: `pnpm test:integration`
- **E2E Tests**: `pnpm test:e2e` (uses Playwright)

### Database
The project uses a database with migrations managed by Turbo:
- Generate schema: `pnpm db:generate`
- Run migrations: `pnpm db:migrate`
- Seed data: `pnpm db:seed`

## 🤝 Contributing

1. Ensure your code follows the linting and formatting standards
2. Run `pnpm typecheck` to verify type safety
3. Add tests for new features
4. Submit pull requests with clear descriptions

## 📄 License

This project is licensed under the **ISC License**.

## 🔗 Related Repositories & Resources

- [Turbo Documentation](https://turbo.build)
- [Protocol Buffers](https://developers.google.com/protocol-buffers)
- [Biome Documentation](https://biomejs.dev)
- [Vitest Documentation](https://vitest.dev)

## 📞 Support

For questions or issues, please open an issue in the repository.

---

**Created**: 2026-09-25  
**Repository**: [MedipallySreeharsha/Chirp---Social-Media-Platform](https://github.com/MedipallySreeharsha/Chirp---Social-Media-Platform)
