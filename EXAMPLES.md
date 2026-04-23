# Example Workflows

This directory contains example workflows demonstrating various use cases for the Build, Test, and Release action.

## Basic Examples

### 1. Go Application

```yaml
name: Go Application CI/CD

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
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          language-version: '1.21'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Node.js Application

```yaml
name: Node.js Application CI/CD

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'node'
          language-version: '20.x'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          dockerfile-path: './Dockerfile'
```

### 3. Python Application

```yaml
name: Python Application CI/CD

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'python'
          language-version: '3.11'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          test-command: 'pytest -v'
```

## Advanced Examples

### 4. Kubernetes Operator with Helm

```yaml
name: Kubernetes Operator Release

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:

env:
  GO_VERSION: '1.21'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
      id-token: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          # Language
          language: 'go'
          language-version: ${{ env.GO_VERSION }}
          
          # Container Build
          dockerfile-path: './Dockerfile'
          docker-context: '.'
          platforms: 'linux/amd64,linux/arm64'
          
          # Registry
          registry: 'ghcr.io'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          
          # Helm
          enable-helm: 'true'
          helm-chart-path: './deployments/helm'
          helm-chart-name: 'my-operator'
          
          # Testing
          run-tests: 'true'
          test-command: 'make test'
          enable-coverage: 'true'
          codecov-token: ${{ secrets.CODECOV_TOKEN }}
          
          # Linting
          run-lint: 'true'
          lint-command: 'make lint'
          
          # Pre-build
          pre-build-command: |
            make controller-gen
            make kustomize
            make manifests
            make generate
            make copy-crds
          
          # Build Args
          build-args: |
            GO_VERSION=${{ env.GO_VERSION }}
            CGO_ENABLED=0
          
          # Release
          enable-release: 'true'
```

### 5. Microservices Monorepo

```yaml
name: Microservices Build

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  api-service:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          dockerfile-path: './services/api/Dockerfile'
          docker-context: './services/api'
          image-name: ${{ github.repository }}/api-service
          registry-password: ${{ secrets.GITHUB_TOKEN }}
  
  worker-service:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          dockerfile-path: './services/worker/Dockerfile'
          docker-context: './services/worker'
          image-name: ${{ github.repository }}/worker-service
          registry-password: ${{ secrets.GITHUB_TOKEN }}
  
  web-frontend:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'node'
          language-version: '20.x'
          dockerfile-path: './services/web/Dockerfile'
          docker-context: './services/web'
          image-name: ${{ github.repository }}/web-frontend
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### 6. Multi-Registry Deployment

```yaml
name: Multi-Registry Release

on:
  push:
    tags: ['v*']

jobs:
  build-and-push-ghcr:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          registry: 'ghcr.io'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          enable-helm: 'true'
  
  build-and-push-dockerhub:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          registry: 'docker.io'
          registry-username: ${{ secrets.DOCKER_USERNAME }}
          registry-password: ${{ secrets.DOCKER_PASSWORD }}
          image-name: 'myorg/myapp'
          run-tests: 'false'  # Already tested in GHCR job
          run-lint: 'false'
          enable-release: 'false'  # Only release once
```

### 7. Rust Application with Cargo

```yaml
name: Rust Application

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'rust'
          language-version: 'stable'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          test-command: 'cargo test --all-features --verbose'
          lint-command: 'cargo fmt --check && cargo clippy -- -D warnings'
          platforms: 'linux/amd64'
```

### 8. Full-Stack Application

```yaml
name: Full-Stack Application

on:
  push:
    branches: [main, staging]
    tags: ['v*']

jobs:
  backend:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          dockerfile-path: './backend/Dockerfile'
          docker-context: './backend'
          image-name: ${{ github.repository }}/backend
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          enable-helm: 'true'
          helm-chart-path: './deployments/helm'
          helm-chart-name: 'backend'
  
  frontend:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'node'
          language-version: '20.x'
          dockerfile-path: './frontend/Dockerfile'
          docker-context: './frontend'
          image-name: ${{ github.repository }}/frontend
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          test-command: 'npm run test:ci'
          lint-command: 'npm run lint && npm run type-check'
```

### 9. Custom Build with Pre/Post Hooks

```yaml
name: Custom Build Pipeline

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          
          # Pre-build: Generate code, download tools
          pre-build-command: |
            echo "Generating code..."
            go install github.com/golang/mock/mockgen@latest
            go generate ./...
            make proto
            make swagger
          
          # Custom test with race detector and timeout
          test-command: |
            go test -v -race -timeout 10m -coverprofile=coverage.out ./...
            go test -v -race -tags=integration ./tests/integration/...
          
          # Advanced linting
          lint-command: |
            golangci-lint run --timeout=10m
            go vet ./...
            staticcheck ./...
          
          # Post-build: Run security scan
          post-build-command: |
            echo "Running security scan..."
            trivy image ${{ steps.build.outputs.image-tags }}
          
          # Additional build arguments
          build-args: |
            BUILDKIT_INLINE_CACHE=1
            GO_VERSION=1.21
            ALPINE_VERSION=3.19
```

### 10. Development vs Production Builds

```yaml
name: Environment-Specific Builds

on:
  push:
    branches: [main, develop]
    tags: ['v*']

jobs:
  development:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'node'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          platforms: 'linux/amd64'  # Single platform for dev
          additional-tags: |
            type=raw,value=dev-latest
          build-args: |
            NODE_ENV=development
            API_URL=https://dev-api.example.com
  
  production:
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'node'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          platforms: 'linux/amd64,linux/arm64'  # Multi-platform for prod
          enable-helm: 'true'
          enable-release: 'true'
          build-args: |
            NODE_ENV=production
            API_URL=https://api.example.com
```

### 11. Private Registry with ECR

```yaml
name: AWS ECR Deployment

on:
  push:
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActions
          aws-region: us-east-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          registry: ${{ steps.login-ecr.outputs.registry }}
          registry-username: AWS
          registry-password: ${{ steps.login-ecr.outputs.docker_password_ACCOUNT_ID_dkr_ecr_REGION_amazonaws_com }}
          image-name: 'my-app'
```

### 12. Scheduled Builds

```yaml
name: Nightly Builds

on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM daily
  workflow_dispatch:

jobs:
  nightly:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          additional-tags: |
            type=raw,value=nightly
            type=raw,value=nightly-{{date 'YYYYMMDD'}}
          run-tests: 'true'
          enable-release: 'false'
```

## Matrix Builds

### 13. Multi-Version Testing

```yaml
name: Multi-Version Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        go-version: ['1.20', '1.21', '1.22']
    
    permissions:
      packages: write
    
    steps:
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: 'go'
          language-version: ${{ matrix.go-version }}
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          image-name: ${{ github.repository }}-go${{ matrix.go-version }}
          run-tests: 'true'
```

## Tips and Best Practices

1. **Use specific versions**: Pin action versions (e.g., `@v1.0.0` instead of `@v1`)
2. **Minimize permissions**: Only grant required permissions to jobs
3. **Cache dependencies**: Enable caching for faster builds (enabled by default)
4. **Use matrix builds**: Test against multiple language versions
5. **Separate concerns**: Use different jobs for dev and prod builds
6. **Secure secrets**: Use GitHub Secrets for sensitive data
7. **Tag properly**: Use semantic versioning for releases
8. **Test PRs**: Run tests on pull requests without pushing images
9. **Enable coverage**: Track code coverage trends over time
10. **Document releases**: Use release notes files for important changes
