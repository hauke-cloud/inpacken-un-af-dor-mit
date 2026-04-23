# Assumptions and Requirements

This document outlines all assumptions made by the Build, Test, and Release action regarding repository structure, file locations, and configurations.

## Table of Contents

- [Directory Structure](#directory-structure)
- [Docker Requirements](#docker-requirements)
- [Helm Requirements](#helm-requirements)
- [Language-Specific Requirements](#language-specific-requirements)
- [CI/CD Requirements](#cicd-requirements)
- [Versioning Assumptions](#versioning-assumptions)

---

## Directory Structure

### Default Repository Layout

The action assumes a standard repository structure:

```
.
├── Dockerfile                          # Container image definition (default location)
├── .github/
│   └── workflows/
│       └── *.yml                       # GitHub Actions workflows
├── deployments/                        # Deployment configurations
│   └── helm/                           # Helm charts (default location)
│       └── <chart-name>/
│           ├── Chart.yaml              # Required for Helm
│           ├── values.yaml             # Recommended
│           └── templates/              # Kubernetes manifests
├── src/ or pkg/ or cmd/                # Source code (language-specific)
├── tests/ or test/                     # Test files
└── README.md
```

**Customizable paths:**
- `dockerfile-path` - Default: `./Dockerfile`
- `docker-context` - Default: `.`
- `helm-chart-path` - Default: `./deployments/helm`

---

## Docker Requirements

### Dockerfile Location

**Default**: `./Dockerfile` at repository root

**Customizable**: Set `dockerfile-path` input to any location

**Assumptions**:
- Dockerfile exists and is valid
- Dockerfile can be built without additional external dependencies
- Build context includes all necessary files

### Docker Build Context

**Default**: `.` (repository root)

**Customizable**: Set `docker-context` input

**Requirements**:
- All files referenced in Dockerfile must be within the build context
- `.dockerignore` file (if present) is respected

### Build Arguments

**Automatically provided**:
```dockerfile
ARG VERSION=dev
ARG COMMIT=unknown
ARG DATE=unknown
ARG BRANCH=unknown
```

**Usage in Dockerfile**:
```dockerfile
FROM golang:1.21 AS builder

# These are automatically passed by the action
ARG VERSION
ARG COMMIT
ARG DATE

# Use them in your build
RUN echo "Building version ${VERSION}"

LABEL org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.revision="${COMMIT}" \
      org.opencontainers.image.created="${DATE}"
```

### Multi-Platform Builds

**Default platforms**: `linux/amd64,linux/arm64`

**Requirements**:
- Base images must support target platforms
- Cross-compilation configured (if needed)
- QEMU and Docker Buildx are set up automatically

**Example Dockerfile for multi-platform Go**:
```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.21 AS builder

ARG TARGETOS
ARG TARGETARCH

WORKDIR /app
COPY . .

RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} \
    go build -o app .

FROM alpine:latest
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]
```

---

## Helm Requirements

### Chart Structure

**Required when `enable-helm: 'true'`**

**Expected location**: `./deployments/helm/<chart-name>/`

**Required files**:
```
deployments/helm/<chart-name>/
├── Chart.yaml              # REQUIRED - Chart metadata
├── values.yaml             # RECOMMENDED - Default values
├── templates/              # REQUIRED - Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl        # Recommended
│   └── ...
├── .helmignore             # Optional
└── README.md               # Recommended
```

### Chart.yaml Format

**Minimum required**:
```yaml
apiVersion: v2
name: my-app
description: A Helm chart for my application
type: application
version: 0.1.0        # Will be updated automatically
appVersion: "1.0"     # Will be updated automatically
```

**The action automatically updates**:
- `version` field - Set to git tag version or generated version
- `appVersion` field - Set to match version

### values.yaml Assumptions

**Image configuration** (if present, will be updated):
```yaml
image:
  repository: ghcr.io/owner/repo
  pullPolicy: IfNotPresent
  tag: ""  # Will be updated to match version
```

**The action updates**:
- `tag` field - Set to the built image version

### Chart Naming

**Default**: Repository name (e.g., `my-repo` → chart name `my-repo`)

**Customizable**: Set `helm-chart-name` input

**Chart directory must match**: `<helm-chart-path>/<helm-chart-name>/`

### Helm Registry

**Default**: Same as container registry (e.g., `ghcr.io`)

**Customizable**: Set `helm-registry` input for different registry

**Push location**: `oci://<registry>/<owner>/charts/<chart-name>`

**Example**:
- Registry: `ghcr.io`
- Owner: `myorg`
- Chart: `myapp`
- Version: `1.2.3`
- **Result**: `oci://ghcr.io/myorg/charts/myapp:1.2.3`

---

## Language-Specific Requirements

### Go

**Required files**:
- `go.mod` - Module definition (at repository root)
- `go.sum` - Dependency checksums

**Optional files**:
- `Makefile` - Custom build targets
- `.golangci.yml` - Linter configuration

**Assumptions**:
- Go modules are used (not GOPATH)
- Tests use standard `testing` package
- Code follows standard Go project layout

**Default commands**:
- Install: `go mod download`
- Test: `go test -v -race -coverprofile=coverage.out -covermode=atomic ./...`
- Lint: `gofmt -s -l . && go vet ./...`

**Recommended Dockerfile**:
```dockerfile
FROM golang:1.21 AS builder

ARG VERSION
ARG COMMIT
ARG DATE

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-X main.Version=${VERSION} -X main.Commit=${COMMIT}" -o app .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]
```

---

### Node.js

**Required files**:
- `package.json` - Project metadata and dependencies
- `package-lock.json` - Dependency lock file (for `npm ci`)

**Optional files**:
- `.eslintrc.*` - Linter configuration
- `jest.config.js` - Test configuration
- `.prettierrc` - Code formatter configuration

**Assumptions**:
- npm is used as package manager
- Scripts defined in package.json:
  - `test` - Run tests
  - `lint` - Run linter
  - `build` - Build application (if needed)

**Default commands**:
- Install: `npm ci`
- Test: `npm test` (if script exists)
- Lint: `npm run lint` (if script exists)

**Recommended Dockerfile**:
```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./

CMD ["node", "dist/index.js"]
```

---

### Python

**Required files**:
- `requirements.txt` - Production dependencies

**Optional files**:
- `requirements-dev.txt` - Development dependencies
- `setup.py` or `pyproject.toml` - Package configuration
- `pytest.ini` or `pyproject.toml` - Test configuration
- `.flake8` - Linter configuration

**Assumptions**:
- pip is used as package manager
- pytest is used for testing
- Application follows standard Python project structure

**Default commands**:
- Install: `pip install -r requirements.txt`
- Test: `pytest --cov=. --cov-report=xml`
- Lint: `flake8 . && black --check .`

**Recommended Dockerfile**:
```dockerfile
FROM python:3.11-slim AS builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /app .

CMD ["python", "app.py"]
```

---

### Rust

**Required files**:
- `Cargo.toml` - Package manifest
- `Cargo.lock` - Dependency lock file

**Assumptions**:
- Standard Rust project structure
- Tests use `cargo test`
- Code follows Rust conventions

**Default commands**:
- Install: `cargo fetch`
- Test: `cargo test --all-features`
- Lint: `cargo fmt -- --check && cargo clippy -- -D warnings`

**Recommended Dockerfile**:
```dockerfile
FROM rust:1.75 AS builder

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs && cargo build --release
RUN rm -rf src

COPY src ./src
RUN cargo build --release

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/app /app
CMD ["/app"]
```

---

### Java

**Required files**:
- `pom.xml` - Maven project file (for Maven)
- `build.gradle` - Gradle build file (for Gradle)

**Assumptions**:
- Maven or Gradle is used
- Standard Java project structure
- Tests use JUnit or similar framework

**Default commands**:
- Test: `mvn test` (Maven)
- Test: `gradle test` (Gradle)

**Recommended Dockerfile (Maven)**:
```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package -DskipTests

FROM eclipse-temurin:17-jre-alpine
COPY --from=builder /app/target/*.jar /app.jar
CMD ["java", "-jar", "/app.jar"]
```

---

### .NET

**Required files**:
- `*.csproj` - Project file
- `*.sln` - Solution file (optional, for multi-project)

**Assumptions**:
- .NET SDK is used
- Standard .NET project structure
- Tests use xUnit, NUnit, or MSTest

**Default commands**:
- Test: `dotnet test`

**Recommended Dockerfile**:
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder

WORKDIR /app
COPY *.csproj .
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=builder /app/out .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

---

## CI/CD Requirements

### GitHub Token Permissions

**Minimum required permissions**:
```yaml
permissions:
  contents: read     # Read repository contents
  packages: write    # Push to GitHub Container Registry
```

**For releases**:
```yaml
permissions:
  contents: write    # Create GitHub releases
  packages: write    # Push to GHCR
```

**For OIDC authentication** (optional):
```yaml
permissions:
  id-token: write    # Generate OIDC tokens
```

### Registry Authentication

**GitHub Container Registry (default)**:
- Username: `${{ github.actor }}` (automatic)
- Password: `${{ secrets.GITHUB_TOKEN }}` (provided by GitHub)

**Other registries** (Docker Hub, ECR, etc.):
- Username: Set via `registry-username` input
- Password: Set via `registry-password` input (use secrets)

### Triggering Events

**Recommended workflow triggers**:
```yaml
on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main, develop]
```

**Behavior by event**:
- `push` to branches - Build and push images
- `push` tags - Build, push, create release
- `pull_request` - Build and test only (no push by default)

---

## Versioning Assumptions

### Git Tag Format

**Semantic versioning tags**:
- `v1.0.0` - Recommended format
- `v1.0.0-alpha.1` - Prerelease
- `v1.0.0-rc.1` - Release candidate

**Image tags generated**:
- From `v1.2.3`:
  - `1.2.3`
  - `1.2`
  - `1` (if not v0.x.x)
  - `latest` (if main/master branch)

**Non-tag versions**:
- Branch builds: `0.0.0-dev-<sha>`
- Example: `0.0.0-dev-a1b2c3d`

### Version Extraction

**Priority**:
1. Git tag (if on tag): `v1.2.3` → version `1.2.3`
2. Generated: `0.0.0-dev-<7-char-sha>`

**Used in**:
- Container image tags
- Helm chart version
- Docker build args
- Release version

---

## Testing Assumptions

### Test File Locations

**Go**: `*_test.go` files alongside source
**Node.js**: `test/` or `__tests__/` directories
**Python**: `tests/` or `test/` directory
**Rust**: `tests/` directory or inline tests
**Java**: `src/test/java/` directory
**.NET**: `*.Tests` projects

### Coverage Files

**Expected output locations**:
- **Go**: `coverage.out` (at root)
- **Python**: `coverage.xml` (at root)
- **Node.js**: `coverage/` directory
- **Rust**: `target/coverage/` directory

**These are automatically uploaded to Codecov if token is provided**

---

## Environment Variables

### Automatically Set

The action relies on GitHub's automatic environment variables:

- `GITHUB_REF` - Git reference (branch/tag)
- `GITHUB_SHA` - Commit SHA
- `GITHUB_REPOSITORY` - Owner/repo name
- `GITHUB_ACTOR` - Username triggering the workflow
- `GITHUB_TOKEN` - Authentication token (if provided)

### Not Set by Default

Users must provide:
- `CODECOV_TOKEN` - For coverage uploads (optional)
- Custom registry credentials (for non-GHCR registries)

---

## File Presence Validation

### Checked Before Execution

- Dockerfile existence (at specified path)
- Helm chart directory (if Helm enabled)
- Chart.yaml (if Helm enabled)

### Not Checked (Assumed Present)

- Language-specific dependency files (go.mod, package.json, etc.)
- Test files
- License files

**These cause action to skip the step if missing**

---

## Build Caching

### Enabled by Default

- Docker build cache (GitHub Actions cache)
- Language-specific caches (Go modules, npm, pip, cargo)

### Cache Keys

Automatically managed based on:
- Language and version
- Lock files (go.sum, package-lock.json, etc.)
- Dockerfile contents

**To disable**: Set `cache-enabled: 'false'`

---

## Resource Limits

### GitHub Actions Limits

The action respects GitHub Actions limits:
- **Build time**: 6 hours per job (default timeout)
- **Artifact size**: 10 GB per repository
- **Cache size**: 10 GB per repository

### Recommendations

- Use multi-stage builds to reduce image size
- Enable caching to speed up builds
- Use `.dockerignore` to exclude unnecessary files
- For large builds, consider self-hosted runners

---

## Summary Checklist

Before using this action, ensure:

- [ ] Dockerfile exists at expected location
- [ ] Helm chart structure is correct (if using Helm)
- [ ] Language-specific files present (go.mod, package.json, etc.)
- [ ] GitHub token has required permissions
- [ ] Git tags follow semantic versioning (for releases)
- [ ] Test and lint commands work locally
- [ ] Build context includes all necessary files
- [ ] Multi-platform builds supported by base images

---

## Overriding Assumptions

Most assumptions can be overridden using action inputs:

```yaml
- uses: hauke-cloud/inpacken-un-af-dor-mit@v1
  with:
    # Override defaults
    dockerfile-path: './docker/prod.Dockerfile'
    docker-context: './app'
    helm-chart-path: './charts'
    test-command: 'make test-all'
    lint-command: 'make lint-all'
    platforms: 'linux/amd64'
```

**When in doubt**: Check the action outputs and logs for specific errors about missing files or incorrect paths.
