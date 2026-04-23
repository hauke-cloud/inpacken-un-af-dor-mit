# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-04-23

### Added

- Initial release of Build, Test, and Release action
- Multi-language support (Go, Node.js, Python, Rust, Java, .NET, Generic)
- Container image building with multi-platform support
- Helm chart packaging and OCI registry push
- Automatic GitHub Release creation with changelogs
- Language-specific test and lint commands
- Code coverage reporting via Codecov
- Pre/post-build command hooks
- Extensive customization options
- Docker build caching via GitHub Actions cache
- Semantic versioning support with automatic image tagging
- Support for multiple container registries (GHCR, Docker Hub, ECR, etc.)

### Features

#### Language Support
- Go with automatic module download and testing
- Node.js with npm package management
- Python with pip and pytest support
- Rust with cargo toolchain
- Java with Maven support
- .NET with dotnet CLI support
- Generic language support with custom commands

#### Container Building
- Multi-architecture builds (amd64, arm64)
- Automatic build argument injection (VERSION, COMMIT, DATE, BRANCH)
- Docker Buildx integration
- Build caching for faster builds
- Custom Dockerfile paths and build contexts

#### Helm Charts
- Automatic chart versioning from git tags
- OCI registry push support
- Chart.yaml and values.yaml automatic updates
- Support for custom Helm registries

#### Testing & Quality
- Automatic test execution with language-specific defaults
- Custom test command support
- Code coverage collection and Codecov upload
- Linting with language-specific tools
- Custom lint command support

#### Release Management
- Automatic GitHub Release creation on tags
- Generated changelogs from git history
- Support for release notes files
- Draft and prerelease options
- Release URL output for downstream jobs

#### Outputs
- `image-digest` - Container image digest
- `image-tags` - Applied image tags
- `image-version` - Extracted version
- `helm-version` - Helm chart version
- `release-url` - GitHub Release URL
- `test-passed` - Test execution status

### Documentation

- Comprehensive README with examples
- ASSUMPTIONS.md documenting all requirements
- EXAMPLES.md with 13 real-world scenarios
- QUICKREF.md for quick lookup
- Inline documentation in action.yml

### Supported Registries

- GitHub Container Registry (ghcr.io)
- Docker Hub (docker.io)
- AWS Elastic Container Registry (ECR)
- Google Container Registry (gcr.io)
- Azure Container Registry (azurecr.io)
- Any OCI-compliant registry

### Compatibility

- GitHub Actions runner: ubuntu-latest, ubuntu-22.04, ubuntu-20.04
- Docker Buildx: v0.10+
- Helm: v3.8+
- Git: v2.0+

## [Unreleased]

### Planned Features

- Support for additional package managers (pnpm, yarn)
- Integration with GitHub Container Scanning
- SBOM (Software Bill of Materials) generation
- Sigstore/Cosign image signing
- Automated security scanning with Trivy
- Support for GitLab Container Registry
- Buildah as alternative to Docker
- Custom notification webhooks
- Artifact uploading to releases

---

[1.0.0]: https://github.com/hauke-cloud/inpacken-un-af-dor-mit/releases/tag/v1.0.0
[Unreleased]: https://github.com/hauke-cloud/inpacken-un-af-dor-mit/compare/v1.0.0...HEAD
