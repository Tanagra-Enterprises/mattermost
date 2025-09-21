# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Claude Code Sandbox Environment

**IMPORTANT: This is a supplemental development tool, NOT part of the Mattermost application codebase.**

The `.claude/` directory contains configuration for Claude Code's containerized sandbox environment. This tooling exists purely to provide Claude with a consistent execution environment and is separate from the core Mattermost application.

### Sandbox Technical Stack
- **Base Image**: Node.js on Alpine Linux
- **Build Tools**: Complete C/C++ build toolchain (gcc, g++, make)
- **Native Module Support**: Python3, node-gyp, and native library dependencies
- **Canvas Dependencies**: Cairo, Pango, JPEG, PNG, SVG libraries for HTML5 Canvas support
- **Go Runtime**: Matches Mattermost server requirements
- **Isolated User**: Non-root claude user (UID 1001) with restricted permissions

### Sandbox Configuration Files
- `.claude/Dockerfile` - Container image definition (development tool only)
- `.claude/docker-compose.yaml` - Container orchestration (development tool only)
- `.claude/settings.local.json` - Claude Code settings (development tool only)

### Session Context
- **THIS SESSION IS RUNNING INSIDE THE CONTAINER** - Launched by running `./claude`
- All commands and operations happen within this containerized sandbox
- Changes to `.claude/Dockerfile` require container rebuilds to take effect
- The sandbox provides Claude with a consistent environment independent of the host system

# Mattermost

## Architecture Overview

Mattermost is an open-source collaboration platform built on a Go backend and React frontend, designed for team communication and workflow automation.

**Official Documentation Links:**
- [Developer Setup Guide](https://developers.mattermost.com/contribute/developer-setup/)
- [Mattermost Developer Documentation](https://developers.mattermost.com/)
- [GitHub Repository](https://github.com/mattermost/mattermost)

### Core Architecture

- **Server** (`/server`): Go 1.24+ backend providing:
  - REST API endpoints (`/channels/api4`)
  - WebSocket API (`/channels/wsapi`) for real-time features
  - Plugin system with dedicated plugin API (`/public/plugin`)
  - Enterprise/Community edition separation (`/enterprise`)
  - Background job processing (`/channels/jobs`)
  - Database abstraction layer (`/channels/store`)

- **Webapp** (`/webapp`): React 17/TypeScript frontend featuring:
  - Workspace-based npm build system with 6 workspaces
  - Redux state management (`/platform/mattermost-redux`)
  - Shared component library (`/platform/components`)
  - Main application (`/channels`) with 300+ React components
  - Custom ESLint plugin (`/platform/eslint-plugin`)

- **API Documentation** (`/api`): OpenAPI 3.0 specifications with generated client SDKs
- **E2E Tests** (`/e2e-tests`): Comprehensive test coverage using Cypress and Playwright
- **Tools** (`/tools`): Development utilities including mmgotool for code generation

### Key Components

**Server Structure** (`/server`):
- `/cmd`: Entry points and CLI tools (mattermost, mmctl)
- `/channels/`: Core business logic layer
  - `/api4/`: REST API endpoints (v4)
  - `/app/`: Application business logic and services
  - `/store/`: Data access layer with SQL/NoSQL implementations
  - `/jobs/`: Background job processing system
  - `/web/`: HTTP handlers and middleware
  - `/wsapi/`: WebSocket API for real-time communication
  - `/utils/`: Server-side utilities and helpers
- `/platform/`: Shared platform services
- `/enterprise/`: Enterprise-only features (separate licensing)
- `/public/`: Public APIs, models, and plugin interfaces
  - `/model/`: Data models and API structures
  - `/plugin/`: Plugin API and hooks
  - `/pluginapi/`: Helper API for plugin development

**Webapp Structure** (`/webapp`):
- `/channels/`: Main web application
  - `/src/components/`: 300+ React components organized by feature
  - `/src/actions/`: Redux action creators
  - `/src/reducers/`: Redux reducers
  - `/src/selectors/`: Redux selectors for state access
  - `/src/utils/`: Client-side utilities
  - `/src/i18n/`: Internationalization files
  - `/src/sass/`: SCSS stylesheets
- `/platform/`: Shared workspace packages
  - `/types/`: TypeScript type definitions
  - `/client/`: API client library
  - `/components/`: Reusable UI components
  - `/mattermost-redux/`: Redux store, actions, and selectors
  - `/eslint-plugin/`: Custom ESLint rules for Mattermost
- **Workspace Management**: Uses npm workspaces for monorepo dependency management

## Common Development Commands

### Server (Go) - Run from `/server`
**Building & Running:**
- `make build` - Build server binary
- `make run` / `make run-server` - Run server with file watching
- `make run-haserver` - Run High Availability server setup
- `make package` - Build distribution packages
- `make debug-server` - Run server in delve debugger

**Testing:**
- `make test-server` - Run unit tests with coverage
- `make test-server-race` - Run tests with race condition detection
- `make test-server-ee` - Run enterprise edition tests
- `make test-server-quick` - Run fast subset of tests
- `make test-compile` - Compile tests without running

**Code Quality:**
- `make check-style` - Run all style/lint checks (includes golangci-lint, vet, modernize)
- `make golangci-lint` - Run golangci-lint specifically
- `make vet` - Run go vet
- `make modernize` - Run modernize linter

**Dependencies & Setup:**
- `make setup-go-work` - Set up Go workspace
- `make golang-versions` - Install Go versions for compatibility testing
- `make mocks` - Generate mock files
- `make prepackaged-plugins` - Download and prepare plugins

**Additional Commands:** See [Server Makefile](https://github.com/mattermost/mattermost/blob/master/server/Makefile) for complete list

### Webapp (React/TypeScript) - Run from `/webapp`
**Building & Running:**
- `npm run build` - Production build
- `npm run run` - Development build with file watching
- `npm run dev-server` / `make dev` - Webpack dev server with hot reload
- `npm run stats` - Generate webpack bundle analysis

**Testing:**
- `npm run test` / `make test` - Run Jest unit tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:debug` - Run tests with debugging options
- `npm run test:updatesnapshot` - Update Jest snapshots
- `npm run test-ci` - Run tests with coverage for CI

**Code Quality:**
- `npm run check` / `make check-style` - Run ESLint and Stylelint
- `npm run check:eslint` - Run ESLint only
- `npm run check:stylelint` - Run Stylelint only  
- `npm run fix` / `make fix-style` - Auto-fix linting issues
- `npm run check-types` / `make check-types` - TypeScript type checking

**Utilities:**
- `npm run clean` - Clean build artifacts and dependencies
- `npm run mmjstool` - Run Mattermost JS tooling

### E2E Testing - Run from `/e2e-tests`
**Setup Variables** (create `.ci/env` file):
```bash
SERVER=onprem|cloud
TEST=cypress|playwright|none
ENABLED_DOCKER_SERVICES="postgres inbucket minio openldap elasticsearch keycloak"
TEST_FILTER="smoke|full"
```

**Running Tests:**
- `make` - Run default E2E test suite
- Set `TEST=cypress` for Cypress tests
- Set `TEST=playwright` for Playwright tests
- Set `TEST_FILTER=smoke` for smoke tests only

### Docker Development - Run from `/server`
- `make start-docker` - Start PostgreSQL, MinIO, and other services
- `make stop-docker` - Stop Docker services
- `make update-docker` - Update and restart Docker services
- `make clean-docker` - Remove Docker containers and volumes

## Development Workflow

### Prerequisites
- **Node.js**: 18.10+ with npm 9.0+ or 10.0+ (use NVM for version management)
- **Go**: 1.21+ (current: 1.24.5 specified in `/server/.go-version`)
- **Docker**: For running PostgreSQL, MinIO, and other services
- **System Tools**: make, git, jq (for E2E testing)

**Reference:** [Official Setup Requirements](https://developers.mattermost.com/contribute/developer-setup/)

### Initial Setup
1. **Clone and Install Dependencies**:
   ```bash
   cd /workspace/webapp && npm install  # Installs all workspace dependencies
   cd /workspace/server && go mod download
   ```

2. **Start Development Services**:
   ```bash
   cd /workspace/server && make start-docker  # PostgreSQL, MinIO, etc.
   ```

3. **Configure Environment** (optional):
   - Server config: `/server/config/config.json` or environment variables
   - E2E testing: Create `/e2e-tests/.ci/env` file

### Development Process
**Full Stack Development:**
```bash
# Terminal 1: Start server
cd /workspace/server && make run-server

# Terminal 2: Start webapp with hot reload
cd /workspace/webapp && make dev
```

**Server-Only Development:**
```bash
cd /workspace/server && make run-server
# Server runs on :8065, serves static webapp from dist/
```

**Webapp-Only Development:**
```bash
# If server is already running elsewhere
cd /workspace/webapp && npm run dev-server
# Webpack dev server on :9005, proxies API to :8065
```

### Common Development Tasks
- **Add new API endpoint**: Modify `/server/channels/api4/`
- **Add new React component**: Add to `/webapp/channels/src/components/`
- **Update database schema**: Create migration in `/server/channels/db/migrations/`
- **Add new plugin hook**: Modify `/server/public/plugin/`

### Troubleshooting
- **Port conflicts**: Server default :8065, webapp dev server :9005
- **Docker issues**: Run `make clean-docker` then `make start-docker`
- **Node version issues**: Use `nvm use` if `.nvmrc` present
- **Go workspace issues**: Run `make setup-go-work` in `/server`
- **Dependency conflicts**: Clear caches with `npm run clean` (webapp) or `go clean -modcache` (server)

## Testing Strategy

### Unit Testing
**Server (Go)**:
- Uses Go's built-in testing framework (`*_test.go` files)
- Test files located alongside source code
- Mock interfaces in `/server/public/plugin/plugintest/`
- Run with `make test-server` (all tests) or `go test ./...` (specific packages)

**Webapp (React/Jest)**:
- Jest with React Testing Library
- Test files: `*.test.tsx` alongside components
- Snapshot testing for UI components (`.snap` files)
- Mock API calls using platform client mocks
- Configuration in each workspace's `jest.config.js`

### Integration Testing
**Server Integration**:
- Full integration tests in `/server/tests/`
- Uses testcontainers for database testing
- Tests API endpoints with full stack
- Enterprise tests verify licensing and premium features

**API Testing**:
- Tests in `/server/channels/api4/` verify REST endpoints
- WebSocket testing for real-time features
- Plugin API testing for extension points

### E2E Testing
**Cypress** (`/e2e-tests/cypress/`):
- Full workflow testing (login, channels, messaging, etc.)
- Page Object Model pattern
- Supports both community and enterprise features
- Custom commands in `/cypress/support/`
- Test files follow `*_spec.ts` pattern
- Run with `npm run cypress:run` or `npm run cypress:open`

**Playwright** (`/e2e-tests/playwright/`):
- Focused scenario testing
- Parallel execution support
- Cross-browser testing capabilities
- Being added as new framework alongside Cypress

**Documentation:** [E2E Testing Guide](https://developers.mattermost.com/contribute/more-info/webapp/e2e-testing/)

### Test Patterns & Conventions
**Webapp Test Patterns**:
```typescript
// Component testing
import {render, screen} from '@testing-library/react';
import Component from './component';

test('should render correctly', () => {
    render(<Component prop="value" />);
    expect(screen.getByText('expected text')).toBeInTheDocument();
});
```

**Server Test Patterns**:
```go
func TestFunctionName(t *testing.T) {
    // Arrange
    // Act  
    // Assert
    require.NoError(t, err)
    assert.Equal(t, expected, actual)
}
```

### Test Data & Mocking
- **Server**: Uses testlib for common test utilities
- **Webapp**: Redux store mocking via platform packages
- **API**: Mock server responses using nock or MSW
- **E2E**: Test data seeding via API calls in test setup

## Important Notes

### Code Conventions & Standards
- **React Version**: React 17.0.2 (not React 18) - important for dependency compatibility
- **TypeScript**: Strict mode enabled across all webapp workspaces
- **Node Version**: 20.11 (see `.nvmrc`) with npm workspaces
- **Go Version**: 1.21+ (current: 1.24.5) with Go modules and workspaces
- **ESLint**: Custom Mattermost ESLint plugin with workspace-specific configs

**Reference:** [Server Workflow Guide](https://developers.mattermost.com/contribute/more-info/server/developer-workflow/)

### Architecture Constraints
- **Enterprise/Community Split**: Enterprise features in `/server/enterprise/` with build-time separation
- **Plugin System**: Extensible via server-side and webapp plugins with dedicated APIs
- **Database**: PostgreSQL primary, MySQL legacy support
- **Internationalization**: react-intl for webapp, server-side locale support
- **Styling**: SCSS with Bootstrap 3.4.1 base (legacy), component-scoped styles

### Development Gotchas
- **Workspace Dependencies**: Always run `npm install` from `/webapp` root, not individual workspaces
- **Hot Reload**: Webpack dev server proxies API to :8065, ensure server is running
- **Docker Ports**: PostgreSQL :5432, MinIO :9000, Elasticsearch :9200, server :8065, webapp dev :9005
- **Enterprise Features**: Many features require `MM_LICENSE` environment variable
- **Build Optimization**: Webapp build can be memory-intensive, increase Node.js heap if needed
- **Plugin Development**: Server restarts required for Go plugin changes, webapp plugins hot-reload

### File Organization Patterns
- **Test Files**: Co-located with source files (`component.tsx` + `component.test.tsx`)
- **API Endpoints**: RESTful structure under `/server/channels/api4/`
- **Redux**: Actions, reducers, selectors organized by domain in `/platform/mattermost-redux/`
- **Components**: Feature-based organization in `/webapp/channels/src/components/`
- **Utilities**: Shared utilities in respective `/utils/` directories

### Performance Considerations
- **Bundle Size**: Webapp uses code splitting, check with `npm run stats`
- **Database**: Use proper indexes, consider read replicas for scale
- **WebSocket**: Real-time features use WebSocket, fallback to polling
- **Caching**: Redis recommended for session storage and caching in production

## Additional Resources

### Official Documentation
- **Main Documentation**: [docs.mattermost.com](https://docs.mattermost.com/)
- **Developer Documentation**: [developers.mattermost.com](https://developers.mattermost.com/)
- **API Documentation**: [api.mattermost.com](https://api.mattermost.com/)
- **Webapp Development**: [Webapp Developer Guide](https://developers.mattermost.com/contribute/more-info/webapp/)
- **Desktop Architecture**: [Desktop App Architecture](https://developers.mattermost.com/contribute/more-info/desktop/architecture/)

### Testing Resources
- **E2E Testing**: [E2E Testing Documentation](https://developers.mattermost.com/contribute/more-info/webapp/e2e-testing/)
- **E2E Cheatsheets**: [E2E Testing Cheatsheets](https://developers.mattermost.com/contribute/more-info/webapp/e2e-cheatsheets/)
- **Test Guidelines**: [General Test Guidelines](https://developers.mattermost.com/contribute/more-info/getting-started/test-guideline/)

### Repository Links
- **Main Repository**: [github.com/mattermost/mattermost](https://github.com/mattermost/mattermost)
- **Server Makefile**: [Server Makefile Commands](https://github.com/mattermost/mattermost/blob/master/server/Makefile)
- **Developer Documentation Repo**: [mattermost-developer-documentation](https://github.com/mattermost/mattermost-developer-documentation)