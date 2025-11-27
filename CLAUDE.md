# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**OWASP Juice Shop** is a full-stack web application designed as an intentionally vulnerable application for security training, awareness, and testing. It demonstrates real-world security flaws from the OWASP Top 10 and beyond, making it ideal for security professionals to learn about web vulnerabilities and for testing security tools.

- **Type**: Full-stack TypeScript web application
- **Backend**: Express.js + TypeScript
- **Frontend**: Angular 20.x + TypeScript
- **Database**: SQLite (Sequelize ORM) + MarsDB collections
- **Default Port**: 3000
- **Node.js**: 20-24

## Documents To Refer

ALWAYS refer below documents while working on this application

1. /docs/DEVSECOPS_TRAINING_WITH_JUICE_SHOP.txt
2. /docs/STAGE_EXPLAINED.txt
3. /docs/TECHNICAL_DOCUMENTATION.txt

**Session Context Files** (for continuity across sessions):

4. /docs/COMMANDS_LOG.txt
5. /docs/CURRENT_STATE.txt
6. /docs/DECISIONS.txt
7. /docs/ISSUES.txt
8. /docs/SESSION_HANDOFF.txt

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Backend Runtime** | Node.js | 20-24 |
| **Backend Framework** | Express.js | Latest |
| **Backend Language** | TypeScript | Latest |
| **Frontend Framework** | Angular | 20.x |
| **Frontend Language** | TypeScript | Latest |
| **Database (Primary)** | SQLite | Via Sequelize |
| **ORM** | Sequelize | 6.37 |
| **Data (NoSQL)** | MarsDB | In-memory collections |
| **Package Manager** | npm | Latest |
| **Testing** | Mocha, Jest, Cypress, Jasmine | Multiple frameworks |
| **Task Runner** | Grunt | For packaging |
| **Build Tools** | TypeScript Compiler, Angular CLI | Latest |
| **Linting** | ESLint + TypeScript ESLint | Latest |
| **Container** | Docker | Multi-stage builds |
| **Security** | helmet, jsonwebtoken, bcrypt, express-jwt | Various |

## Project Architecture

### High-Level Architecture

```
Frontend (Angular SPA)
    ↓ HTTP/HTTPS + WebSocket
Express.js API Server (40+ routes)
    ↓ Sequelize ORM
SQLite Database + MarsDB Collections
```

### Key Components

1. **Frontend** (`/frontend`): 70+ Angular components, 40+ services, Material Design UI
2. **Backend** (`/routes`): 40+ Express route handlers implementing vulnerable APIs
3. **Data Layer** (`/models`): 18 Sequelize models (User, Product, Order, Challenge, etc.)
4. **Security Library** (`/lib/insecurity.ts`): Collection of intentional vulnerabilities
5. **Configuration** (`/config`): 14 YAML profiles for different modes (ctf, tutorial, unsafe, etc.)
6. **Challenges** (`/data/static/challenges.yml`): OWASP vulnerability scenarios database
7. **Testing** (`/test`): Unit, API, and E2E tests across multiple frameworks

### Directory Structure (Key Locations)

```
/frontend                    # Angular SPA application
  /src
    /app                    # Components, services, models
    /assets                 # Static files, translations (30+ languages)
    main.ts, index.html     # Angular bootstrap
  /dist                     # Compiled production build

/routes                      # Express route handlers (40+ files)
  /login.ts, /basket.ts, /admin.ts, /web3Wallet.ts, etc.

/models                      # Sequelize ORM models (18 models)
  /index.ts                 # Database initialization & relationships

/lib                         # Utility libraries
  /insecurity.ts            # Vulnerability implementations
  /utils.ts, /logger.ts     # General utilities
  /startup                  # Server initialization tasks

/data                        # Data initialization & population
  /datacreator.ts           # Main data population (26K lines)
  /mongodb.ts               # MarsDB collections (posts, orders)
  /static
    /challenges.yml         # Challenge definitions
    /users.yml              # Pre-loaded user accounts
    /products.yml           # Product catalog
    /i18n                   # Translations (30+ languages)
    /codefixes              # Code fixing challenge solutions

/config                      # Configuration files (14 profiles)
  /default.yml              # Main configuration (23K lines)
  /ctf.yml, /tutorial.yml, /unsafe.yml, etc.

/test                        # Complete test suite
  /server                   # Mocha tests
  /api                      # Jest/Frisby API tests
  /cypress                  # Cypress E2E tests (100+ tests)

/build                       # Compiled JavaScript output
  /app.js, /server.js       # Entry points after compilation

/app.ts                      # Application entry point
/server.ts                   # Server initialization (400+ lines)
config.schema.yml            # Config validation schema
```

## Development Commands

### Quick Reference

```bash
npm start                    # Build & run production (compiles frontend & backend)
npm run serve              # Concurrent dev: Node.js (ts-node) + Angular dev server
npm run serve:dev          # Dev with auto-reload (ts-node-dev)

npm test                   # Run all tests
npm run lint               # Check code style
npm run lint:fix           # Auto-fix linting issues
```

### Building

```bash
npm run build:frontend     # Compile Angular frontend (→ /frontend/dist/frontend)
npm run build:server       # Compile TypeScript backend (→ /build)
npm run postinstall        # Auto-run on npm install (builds both)
```

### Running & Development

```bash
npm start                  # Production: Build + run from /build/app.js
npm run serve             # Development: Concurrent backend (ts-node) + frontend (ng serve)
npm run serve:dev         # Development: ts-node-dev with auto-reload (faster iteration)
```

### Testing

```bash
npm test                  # Frontend unit tests + server tests (combined)
npm run test:chromium     # Frontend tests in Chromium
npm run test:server       # Mocha server-side tests with NYC coverage
npm run test:api          # Jest/Frisby API tests with NYC coverage
npm run cypress:open      # Open Cypress test runner (interactive)
npm run cypress:run       # Run Cypress E2E tests headless (with retries)
```

### Code Quality

```bash
npm run lint              # ESLint checks for all TypeScript/JS files
npm run lint:fix          # Auto-fix linting issues
npm run lint:config       # Validate YAML configuration files
```

### Packaging & Distribution

```bash
npm run package           # Grunt: Create distributable packages
npm run package:ci        # CI: Package with pruning & deduplication
npm run sbom              # Generate Software Bill of Materials (CycloneDX JSON + XML)
```

### Special Utilities

```bash
npm run rsn               # Run Security Notification utility
npm run rsn:verbose       # Run with verbose output
npm run rsn:update        # Update security notifications
```

## Configuration

### Configuration System

Juice Shop uses a **node-config** based system with YAML files:

- **Main Config**: `/config/default.yml` (23K lines - all available settings)
- **Profile System**: 14 YAML files for different modes
  - `ctf.yml` - CTF mode (reduced hints, competitive)
  - `tutorial.yml` - Tutorial mode (simplified)
  - `unsafe.yml` - All security features disabled
  - `quiet.yml` - Minimal output
  - `test.yml` - Test environment
  - Others: `7ms.yml`, `addo.yml`, `bodgeit.yml`, `defcon33.yml`, etc.

**Select Profile**: Set `NODE_ENV` environment variable
```bash
NODE_ENV=ctf npm start    # Run in CTF mode
NODE_ENV=tutorial npm start  # Run in tutorial mode
```

### Key Configuration Areas

- **server**: Port (3000), basePath, baseUrl, compression, helmet settings
- **application**: Name, logo, theme, features (email, recycle, deluxe, etc.)
- **challenges**: Hint settings, mitigations, coding challenges, safety mode
- **hackingInstructor**: Training bot configuration
- **products**: Catalog with prices, images, custom challenges
- **users**: Pre-loaded accounts with roles and credentials
- **ctf**: CTF mode flags, leaderboard settings
- **memory**: Easter eggs, application memories
- **chatBot**: Training data location, bot responses
- **googleOauth**: OAuth integration configuration

### Configuration Validation

```bash
npm run lint:config       # Validates all YAML against config.schema.yml
```

## Database & Data

### Data Models

**18 Sequelize Models** (`/models`):
- `User` - Accounts, passwords, 2FA, security questions
- `Product` - Catalog with prices, images, descriptions
- `Challenge` - OWASP vulnerabilities (solved status, hints, mitigations)
- `Order`, `Basket`, `BasketItem` - Shopping functionality
- `Address`, `Card`, `Wallet` - Payment & shipping
- `Feedback`, `Complaint` - User feedback
- `Hint`, `SecurityQuestion`, `SecurityAnswer` - Training support
- `Delivery`, `Quantity`, `Recycle` - Business logic
- `PrivacyRequest`, `ImageCaptcha`, `Captcha`, `Memory` - Special features

### Data Population

**Process**:
1. Sequelize creates/syncs SQLite database on startup
2. `/data/datacreator.ts` populates from YAML configs
3. Static data loaded: users, products, challenges, deliveries
4. Supports 30+ language translations via `/data/static/i18n/`
5. MarsDB collections created for: posts (reviews), orders

**Starting Fresh**:
```bash
rm data/juiceshop.sqlite   # Delete database
npm start                  # Recreates with fresh data
```

### Database Location

- **SQLite**: `/data/juiceshop.sqlite` (created on startup)
- **Config Schema**: `/config.schema.yml` (JSON Schema for validation)

## Testing Strategy

### Test Frameworks & Coverage

| Test Type | Framework | Location | Command |
|-----------|-----------|----------|---------|
| **Frontend Unit** | Jasmine + Karma | Component `.spec.ts` files | `npm test` |
| **Server-Side** | Mocha + Chai | `/test/server/**/*.ts` | `npm run test:server` |
| **API Integration** | Jest + Frisby | `/test/api/*Spec.ts` | `npm run test:api` |
| **E2E** | Cypress | `/test/cypress/e2e/*.spec.ts` | `npm run cypress:run` |

### Running Specific Tests

```bash
# All tests
npm test                   # Frontend + server tests

# Specific test frameworks
npm run test:server        # Mocha server tests with NYC coverage (→ /build/reports/coverage/server-tests)
npm run test:api           # Jest/Frisby API tests with NYC coverage (→ /build/reports/coverage/api-tests)
npm run test:chromium      # Frontend tests with Chromium browser

# Cypress (E2E)
npm run cypress:open       # Interactive test runner
npm run cypress:run        # Headless execution with 2 retries
```

### Code Coverage

Tests generate coverage reports in `/build/reports/coverage/`:
- **Server tests**: Via NYC (Istanbul)
- **API tests**: Via Jest + NYC
- **Frontend tests**: Via Karma
- Reports in: lcov + text-summary formats

### Test Setup & Teardown

- **API Tests**: Global setup/teardown in `/test/apiTestsSetup.ts` and `/test/apiTestsTeardown.ts`
- **Cypress**: Support file in `/test/cypress/support/e2e.ts`, custom tasks for coupon generation, etc.

## Build Process

### Build Pipeline

```
npm install
  ↓ (runs postinstall hook)
  ├─ npm run build:frontend  (Angular CLI → /frontend/dist/frontend)
  └─ npm run build:server    (TypeScript → /build/)

npm start
  ↓ (compiles both if needed)
  └─ node /build/app.js
```

### Frontend Build

- **Tool**: Angular CLI with custom webpack
- **Command**: `npm run build:frontend`
- **Output**: `/frontend/dist/frontend/`
- **Features**: AOT compilation, tree-shaking, minification

### Backend Build

- **Tool**: TypeScript Compiler (tsc)
- **Command**: `npm run build:server`
- **Output**: `/build/` (mirrors source structure)
- **Source Maps**: Generated for debugging

### Production Build

```bash
npm run build:frontend
npm run build:server
npm start                 # Runs /build/app.js (compiled version)
```

## Key Entry Points

- **App Entry**: `/app.ts` → imports `/server.ts`
- **Server Init**: `/server.ts` (400+ lines) - initializes Express, routes, middleware, data
- **Frontend Entry**: `/frontend/src/main.ts` → Angular bootstrap
- **HTML Entry**: `/frontend/src/index.html`
- **Docker Entry**: `CMD ["/juice-shop/build/app.js"]` in Dockerfile
- **Production Run**: `/build/app.js` (compiled from `/app.ts`)

## Server Architecture

### Route Organization

40+ route handlers in `/routes/`:
- **Authentication**: `login.ts`, `logout.ts`, `register.ts`, `passwordReset.ts`, `twoFactorAuth.ts`
- **Shopping**: `basket.ts`, `basketItems.ts`, `payment.ts`, `order.ts`, `delivery.ts`
- **Products**: `products.ts`, `reviews.ts`, `search.ts`
- **Admin**: `administration.ts`, `dataExport.ts`, `recycle.ts`
- **Social**: `contact.ts`, `complain.ts`, `feedback.ts`
- **Web3**: `web3Wallet.ts`, `nftMint.ts`, `nftUnlock.ts`
- **Misc**: `fileUpload.ts`, `chatbot.ts`, `socket.ts`, `metrics.ts`

### Middleware Stack

Express middleware configured in `/server.ts`:
- `helmet` - Security headers
- `cors` - Cross-origin resource sharing
- `body-parser` - JSON parsing
- `compression` - Gzip compression
- `morgan` - HTTP request logging
- `express-jwt` - JWT authentication
- `express-rate-limit` - Rate limiting
- `express-ipfilter` - IP filtering

### Intentional Vulnerabilities

Core vulnerability implementations in `/lib/insecurity.ts`:
- SQL injection
- Cross-site scripting (XSS)
- Insecure deserialization
- Missing authentication
- Sensitive data exposure
- Broken access control
- Using known vulnerable dependencies
- Insecure configuration

## Frontend Architecture

### Angular Structure

- **Modules**: Modular with lazy loading
- **Routing**: 100+ routes defined in `app.routing.ts`
- **State Management**: Services + RxJS Observables (no NgRx)
- **UI Library**: Angular Material components

### Key Components (70+ total)

**Authentication**:
- `login`, `register`, `forgot-password`, `two-factor-auth`, `password-strength`

**Shopping**:
- `product-details`, `basket`, `purchase-basket`, `payment`, `order-completion`, `order-history`

**Training**:
- `score-board` (with challenge filtering, statistics)
- `code-fixes`, `code-snippet`, `code-area`
- `challenge-solved-notification`, `challenge-status-badge`

**User**:
- `user-details`, `address`, `saved-payment-methods`, `data-export`

**Admin**:
- `administration` (user management, metrics, exports)

**Features**:
- `chatbot` (AI training bot), `photo-wall`, `qr-code`, `wallet`, `web3-sandbox`, `nft-unlock`

### Key Services (40+ total)

- `UserService` - Authentication, profile
- `ProductService` - Product catalog
- `BasketService` - Shopping cart
- `ChallengeService` - Challenge data & scoring
- `PaymentService`, `DeliveryService` - Order processing
- `ConfigurationService` - Dynamic configuration
- `SocketIOService` - Real-time WebSocket communication
- `LocalBackupService` - Client-side data export
- Plus 30+ more specialized services

### Internationalization

- **30+ languages** supported
- **Translation Files**: `/frontend/src/assets/i18n/*.json`
- **Dynamic Switching**: Runtime language selection via `TranslateService`
- **Server-side i18n**: `/i18n/` for server translations

## Code Quality & Linting

### ESLint Configuration

- **File**: `.eslintrc.js`
- **Coverage**: All TypeScript, JavaScript, Node.js code
- **Rules**: TypeScript ESLint recommended rules + custom rules
- **Formatter**: Prettier integration

### Running Lint

```bash
npm run lint              # Check for issues
npm run lint:fix          # Auto-fix issues
npm run lint:config       # Validate YAML configs
```

### Configuration Validation

YAML configurations validated against JSON Schema in `/config.schema.yml`:
```bash
npm run lint:config       # Runs schema validation
```

## Security Considerations

### Intentional Vulnerabilities (By Design)

This application **deliberately contains security flaws** for training purposes. Do NOT use in production or exposed publicly without restrictions.

**Key Vulnerable Areas**:
- SQL injection in product search
- XSS in product reviews and feedback
- Broken authentication in API endpoints
- Insecure direct object references (IDOR)
- Sensitive data exposure (credentials, payment info)
- Insecure configuration (exposed metrics, swagger docs)
- Weak password policies
- Missing CSRF protection
- Unvalidated redirects
- Using known vulnerable npm packages

### Security Headers (In "Safe" Mode)

When not in `unsafe` config, Helmet applies security headers:
- Content-Security-Policy (CSP)
- X-Frame-Options
- X-Content-Type-Options
- Strict-Transport-Security

### JWT & Encryption Keys

- **JWT Keys**: `/encryptionkeys/jwt.pub` (public key for signing verification)
- **Premium Key**: `/encryptionkeys/premium.key` (for encryption)
- Generated or loaded on startup

## Docker & Deployment

### Docker Build

- **Multi-stage build** in `Dockerfile`
- **Stage 1**: Node base image with dependencies
- **Stage 2**: Compiled application only
- **Entry Point**: `node /juice-shop/build/app.js`
- **Exposed Port**: 3000

### Docker Compose (Testing)

```bash
docker-compose -f docker-compose.test.yml up
```

Used for integration testing environment.

## Debugging & Troubleshooting

### Enable Debug Logging

```bash
DEBUG=* npm run serve:dev    # Enable debug logging for all modules
```

### Database Issues

```bash
# Reset database
rm data/juiceshop.sqlite
npm start

# Check if SQLite is locked
# Solution: Ensure no other process is using the database
```

### Port Already in Use

```bash
# Change default port via config or environment
PORT=3001 npm start
```

### Common NPM Issues

```bash
# Clean cache
npm cache clean --force

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install

# Rebuild native modules
npm rebuild
```

## Performance Notes

### Key Optimization Points

1. **Frontend**:
   - Angular production build: `ng build --prod` (AOT, tree-shaking, minification)
   - Lazy loading for route modules
   - OnPush change detection strategy where applicable

2. **Backend**:
   - Data caching in `/data/datacache.ts`
   - Compression middleware enabled
   - Rate limiting to prevent abuse
   - Sequelize connection pooling

3. **Database**:
   - SQLite file-based (suitable for single server)
   - Indexes on frequently queried fields
   - Sequelize eager loading for related data

## Important Architecture Decisions

### Why TypeScript?

- Type safety for both frontend and backend
- Consistent language across stack
- Better IDE support and refactoring
- Catches errors at compile time

### Why Angular?

- Full-featured SPA framework
- Comprehensive Material Design integration
- Strong typing with TypeScript
- CLI tooling for efficient development

### Why SQLite?

- Simple file-based database (no server needed)
- Good for development, single-user, and testing
- Sufficient for demonstrating web vulnerabilities
- Easy to reset/recreate for training

### Why Separate Frontend/Backend Build?

- Frontend served as static files (faster, can be cached)
- Backend API independently scalable
- Clear separation of concerns
- Frontend can be deployed to CDN if needed

### Configuration as YAML

- Human-readable format
- Easy to version control
- Schema validation for safety
- Supports multiple profiles for different environments

## Useful Patterns & Conventions

### Service Injection Pattern (Angular)

All components use constructor injection for services:
```typescript
constructor(private service: SomeService) {}
```

### Sequelize Model Pattern (Backend)

Models defined as functions:
```typescript
module.exports = (sequelize, DataTypes) => {
  const Model = sequelize.define('Model', { /* attributes */ })
  Model.associate = (models) => { /* relationships */ }
  return Model
}
```

### Error Handling

- Frontend: HTTP interceptors catch and handle errors
- Backend: Try-catch with logging, error responses to client
- Always check security functions in `/lib/insecurity.ts` for vulnerability examples

### Testing Pattern

- Unit tests alongside components/functions (.spec.ts)
- API tests verify endpoint behavior (even vulnerable behavior)
- E2E tests verify complete user workflows
- Coverage reports in `/build/reports/coverage/`

## Working with Challenges

### Challenge Configuration

- **File**: `/data/static/challenges.yml`
- **Fields**: Key, name, category, difficulty, solved status, hints, mitigations
- **Adding Challenges**: Edit YAML, data creator populates on startup

### Tracking Challenge Progress

- **Model**: `/models/challenge.ts` (Sequelize model)
- **Status**: Stored in SQLite, updated via API when users solve challenges
- **UI**: Score board displays progress and filtering

## Internationalization (i18n)

### Adding Languages

1. Add translation file: `/frontend/src/assets/i18n/[lang_code].json`
2. Register in: `/data/static/locales.json`
3. Frontend will allow dynamic selection

### Server-Side Translations

- `/i18n/` directory for server messages
- Loaded via `i18n` npm package
- Used for error messages, notifications

## Next Steps for Development

1. **Understand the vulnerability model**: Review `/lib/insecurity.ts` to see how vulnerabilities are implemented
2. **Check existing routes**: `/routes/` shows how vulnerable APIs are structured
3. **Review test examples**: `/test/` shows how to test both secure and vulnerable code
4. **Run locally**: `npm run serve:dev` for fastest development iteration
5. **Read challenges**: `/data/static/challenges.yml` explains each vulnerability in detail
6. **Explore configuration**: `/config/default.yml` documents all available settings
