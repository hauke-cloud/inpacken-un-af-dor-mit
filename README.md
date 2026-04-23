# Build, Test, and Release Action

A comprehensive GitHub Action for building, testing, and releasing containerized applications with optional Helm chart support. This action supports multiple languages and provides a complete CI/CD pipeline in a single reusable action.

## Features

- 🚀 **Multi-language support**: Go, Node.js, Python, Rust, Java, .NET, and generic projects
- 🐳 **Container image building**: Multi-platform Docker builds with caching
- ⛵ **Helm chart support**: Automatic chart versioning and OCI registry push
- ✅ **Testing & Linting**: Language-specific test and lint commands
- 📊 **Code coverage**: Codecov integration
- 📦 **GitHub Releases**: Automatic release creation with changelogs
- 🔧 **Customizable**: Extensive configuration options and hooks

## Quick Start

### Basic Usage

```yaml
name: CI/CD

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main, develop]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
      id-token: write
    
    steps:
      - name: Build and Release
        uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          language-version: '1.21'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### Go Application with Helm

```yaml
- name: Build and Release
  uses: hauke-cloud/inpacken-un-af-dor-mit@v1
  with:
    language: 'go'
    language-version: '1.21'
    registry-password: ${{ secrets.GITHUB_TOKEN }}
    enable-helm: 'true'
    helm-chart-path: './deployments/helm'
    codecov-token: ${{ secrets.CODECOV_TOKEN }}
```

### Node.js Application

```yaml
- name: Build and Release
  uses: hauke-cloud/inpacken-un-af-dor-mit@v1
  with:
    language: 'node'
    language-version: '20.x'
    registry-password: ${{ secrets.GITHUB_TOKEN }}
    dockerfile-path: './Dockerfile'
    platforms: 'linux/amd64,linux/arm64'
```

### Python Application

```yaml
- name: Build and Release
  uses: hauke-cloud/inpacken-un-af-dor-mit@v1
  with:
    language: 'python'
    language-version: '3.11'
    registry-password: ${{ secrets.GITHUB_TOKEN }}
    test-command: 'pytest --cov'
    lint-command: 'flake8 . && black --check .'
```

## Inputs

### Language Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `language` | Primary language (go, node, python, rust, java, dotnet, generic) | Yes | `generic` |
| `language-version` | Language version (e.g., "1.21", "20.x", "3.11") | No | Language-specific |

### Build Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `dockerfile-path` | Path to Dockerfile | No | `./Dockerfile` |
| `docker-context` | Docker build context path | No | `.` |
| `platforms` | Target platforms for multi-arch builds (comma-separated) | No | `linux/amd64,linux/arm64` |
| `build-args` | Additional Docker build arguments (one per line) | No | `''` |
| `cache-enabled` | Enable build caching | No | `true` |

### Registry Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `registry` | Container registry URL | No | `ghcr.io` |
| `registry-username` | Registry username | No | `${{ github.actor }}` |
| `registry-password` | Registry password/token | Yes | - |
| `image-name` | Image name | No | `${{ github.repository }}` |
| `additional-tags` | Additional image tags (one per line) | No | `''` |

### Helm Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `enable-helm` | Enable Helm chart build and push | No | `false` |
| `helm-chart-path` | Path to Helm chart directory | No | `./deployments/helm` |
| `helm-chart-name` | Helm chart name | No | Repository name |
| `helm-registry` | Helm OCI registry | No | Same as `registry` |

### Test Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `run-tests` | Enable test execution | No | `true` |
| `test-command` | Custom test command | No | Language-specific |
| `enable-coverage` | Enable code coverage reporting | No | `true` |
| `codecov-token` | Codecov token for coverage upload | No | `''` |

### Lint Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `run-lint` | Enable linting | No | `true` |
| `lint-command` | Custom lint command | No | Language-specific |

### Release Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `enable-release` | Enable GitHub Release creation on tags | No | `true` |
| `release-draft` | Create release as draft | No | `false` |
| `release-prerelease` | Mark release as prerelease | No | `false` |
| `release-notes-file` | Path to release notes file | No | `''` |

### Advanced Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `pre-build-command` | Command to run before building | No | `''` |
| `post-build-command` | Command to run after building | No | `''` |
| `skip-push-on-pr` | Skip pushing images on pull requests | No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `image-digest` | Digest of the built image |
| `image-tags` | Tags applied to the image |
| `image-version` | Version extracted from tag or generated |
| `helm-version` | Helm chart version |
| `release-url` | URL of the created GitHub release |
| `test-passed` | Whether tests passed |

## Assumptions and Requirements

### Directory Structure

The action makes the following assumptions about your repository structure:

#### Docker
- **Dockerfile location**: `./Dockerfile` (configurable via `dockerfile-path`)
- The Dockerfile should be at the root or specify a custom path

#### Helm Charts
- **Chart directory**: `./deployments/helm/<chart-name>/` (configurable via `helm-chart-path`)
- Required files:
  - `Chart.yaml` - Helm chart metadata
  - `values.yaml` - Default values (optional, but recommended)
  - `templates/` - Kubernetes manifests
  
Example structure:
```
./deployments/helm/
  └── my-app/
      ├── Chart.yaml
      ├── values.yaml
      ├── templates/
      │   ├── deployment.yaml
      │   ├── service.yaml
      │   └── ...
      └── ...
```

#### Language-Specific Files

**Go Projects:**
- `go.mod` and `go.sum` at repository root
- Tests using standard `testing` package
- Optional: `Makefile` with `test` target

**Node.js Projects:**
- `package.json` at repository root
- `npm` as package manager (or override with custom commands)
- Optional: `lint` and `test` scripts in package.json

**Python Projects:**
- `requirements.txt` or `requirements-dev.txt` at repository root
- Tests using `pytest` framework
- Optional: Configuration for `flake8`, `black`, `isort`

**Rust Projects:**
- `Cargo.toml` at repository root
- Standard Rust project structure
- Tests using `cargo test`

**Java Projects:**
- `pom.xml` for Maven projects
- Standard Maven project structure

**.NET Projects:**
- `.csproj` or `.sln` file at repository root
- Standard .NET project structure

### Docker Build Arguments

The action automatically passes the following build arguments to Docker:
- `VERSION` - Extracted from git tag or generated
- `COMMIT` - Current git commit SHA
- `DATE` - Build timestamp in ISO 8601 format
- `BRANCH` - Current branch name

Example Dockerfile usage:
```dockerfile
ARG VERSION=dev
ARG COMMIT=unknown
ARG DATE=unknown

LABEL org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.revision="${COMMIT}" \
      org.opencontainers.image.created="${DATE}"
```

### Image Tagging Strategy

Images are automatically tagged with:
- **Branch builds**: `<branch-name>`, `<branch>-<sha>`
- **PR builds**: `pr-<number>`
- **Tag builds**: `<version>`, `<major>.<minor>`, `<major>` (for v1.0.0+)
- **Main/Master**: `latest` (only on default branch)

### Versioning

- **Git tags**: Use `v*` pattern (e.g., `v1.2.3`) for releases
- **Semantic versioning**: Supports SemVer for automatic major/minor tagging
- **Development versions**: Non-tagged commits get `0.0.0-dev-<sha>` version

### Permissions

Required GitHub token permissions:
```yaml
permissions:
  contents: write    # For creating releases
  packages: write    # For pushing to GHCR
  id-token: write    # For OIDC authentication (optional)
```

### Registry Support

The action supports any OCI-compliant container registry:
- GitHub Container Registry (ghcr.io) - default
- Docker Hub (docker.io)
- AWS ECR
- Google Container Registry
- Azure Container Registry
- Any other OCI registry

## Examples

### Complete Go Application with All Features

```yaml
name: Complete CI/CD

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
      id-token: write
    
    steps:
      - name: Build, Test, and Release
        uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          # Language
          language: 'go'
          language-version: '1.21'
          
          # Container
          dockerfile-path: './Dockerfile'
          platforms: 'linux/amd64,linux/arm64'
          registry: 'ghcr.io'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          
          # Helm
          enable-helm: 'true'
          helm-chart-path: './deployments/helm'
          helm-chart-name: 'my-app'
          
          # Testing
          run-tests: 'true'
          enable-coverage: 'true'
          codecov-token: ${{ secrets.CODECOV_TOKEN }}
          
          # Build customization
          pre-build-command: 'make generate'
          build-args: |
            GO_VERSION=1.21
            CGO_ENABLED=0
          
          # Release
          enable-release: 'true'
          release-notes-file: './RELEASE_NOTES.md'
```

### Multi-Stage Build with Custom Commands

```yaml
- name: Build with Custom Commands
  uses: hauke-cloud/inpacken-un-af-dor-mit@v1
  with:
    language: 'go'
    registry-password: ${{ secrets.GITHUB_TOKEN }}
    
    # Custom test and lint
    test-command: 'go test -v -race -timeout 5m ./...'
    lint-command: 'golangci-lint run --timeout=5m'
    
    # Pre-build code generation
    pre-build-command: |
      make controller-gen
      make manifests
      make generate
    
    # Custom build args
    build-args: |
      GOPROXY=https://proxy.golang.org
      GOPRIVATE=github.com/myorg/*
```

### Using with Multiple Registries

```yaml
- name: Build and Push to Docker Hub
  uses: hauke-cloud/inpacken-un-af-dor-mit@v1
  with:
    language: 'node'
    registry: 'docker.io'
    registry-username: ${{ secrets.DOCKER_USERNAME }}
    registry-password: ${{ secrets.DOCKER_PASSWORD }}
    image-name: 'myorg/myapp'
```

### Kubernetes Operator Example

```yaml
- name: Build Operator
  uses: hauke-cloud/inpacken-un-af-dor-mit@v1
  with:
    language: 'go'
    language-version: '1.21'
    registry-password: ${{ secrets.GITHUB_TOKEN }}
    
    enable-helm: 'true'
    helm-chart-path: './deployments/helm'
    
    pre-build-command: |
      make controller-gen
      make kustomize
      make manifests
      make generate
      make copy-crds
    
    test-command: 'make test'
    lint-command: 'make lint'
```

### Microservice with Database Migrations

```yaml
- name: Build Microservice
  uses: hauke-cloud/inpacken-un-af-dor-mit@v1
  with:
    language: 'go'
    registry-password: ${{ secrets.GITHUB_TOKEN }}
    
    dockerfile-path: './build/Dockerfile'
    docker-context: '.'
    
    build-args: |
      SERVICE_NAME=user-service
      DB_MIGRATIONS_PATH=./migrations
    
    additional-tags: |
      type=raw,value=stable,enable=${{ github.ref == 'refs/heads/main' }}
```

## Default Language Commands

When custom commands are not provided, the action uses these defaults:

### Go
- **Test**: `go test -v -race -coverprofile=coverage.out -covermode=atomic ./...`
- **Lint**: `gofmt -s -l . && go vet ./...`
- **Dependencies**: `go mod download`

### Node.js
- **Test**: `npm test` (if script exists)
- **Lint**: `npm run lint` (if script exists)
- **Dependencies**: `npm ci`

### Python
- **Test**: `pytest --cov=. --cov-report=xml`
- **Lint**: `flake8 . && black --check .`
- **Dependencies**: `pip install -r requirements.txt`

### Rust
- **Test**: `cargo test --all-features`
- **Lint**: `cargo fmt -- --check && cargo clippy -- -D warnings`
- **Dependencies**: `cargo fetch`

### Java
- **Test**: `mvn test`
- **Lint**: (not configured by default)
- **Dependencies**: Handled by Maven

### .NET
- **Test**: `dotnet test`
- **Lint**: (not configured by default)
- **Dependencies**: Handled by dotnet CLI

## Troubleshooting

### Build fails with "Dockerfile not found"
Ensure your Dockerfile is at the expected location or set `dockerfile-path` correctly.

### Helm push fails
- Check that `helm-chart-path` points to the parent directory of your chart
- Ensure `Chart.yaml` exists in `<helm-chart-path>/<chart-name>/Chart.yaml`
- Verify registry permissions for pushing Helm charts

### Tests don't run
- For custom test commands, set `test-command` input
- Ensure test dependencies are installed (requirements.txt, package.json, etc.)
- Check that test files exist in the expected locations

### Image tags are incorrect
- For semantic versioning, ensure tags follow `v*` pattern (e.g., `v1.2.3`)
- Use `additional-tags` for custom tagging strategies

### Coverage upload fails
- Set `codecov-token` input with your Codecov token
- Ensure coverage files are generated in expected paths

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This action is released under the MIT License.
