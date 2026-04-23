# Contributing to Build, Test, and Release Action

Thank you for your interest in contributing! This document provides guidelines and information for contributors.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Actions Workflow                       │
└─────────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  action.yml (Composite Action)                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Validation │  │   Language   │  │   Pre-Build  │          │
│  │   & Setup    │─▶│   Setup      │─▶│   Hooks      │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                           ▼                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Linting    │◀─│  Dependencies│  │   Testing    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│         ▼                                   ▼                    │
│  ┌──────────────┐                   ┌──────────────┐           │
│  │   Coverage   │                   │   Docker     │           │
│  │   Upload     │                   │   Build      │           │
│  └──────────────┘                   └──────────────┘           │
│                                           ▼                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Post-Build  │  │     Helm     │  │   Release    │         │
│  │    Hooks     │  │   Package    │  │   Creation   │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ▼
         ┌────────────────────────────────────────┐
         │  Outputs (digest, tags, version, etc.) │
         └────────────────────────────────────────┘
```

## Action Flow

1. **Validation & Setup**
   - Validates input parameters
   - Checks for required files (Dockerfile, Helm charts)
   - Sets up environment

2. **Language Setup**
   - Detects language from input
   - Installs language runtime (Go, Node.js, Python, etc.)
   - Configures language-specific caching

3. **Dependencies**
   - Installs project dependencies
   - Uses language-specific package managers

4. **Pre-Build Hooks**
   - Executes custom commands before build
   - Useful for code generation, asset compilation

5. **Linting**
   - Runs language-specific linters
   - Supports custom lint commands
   - Can be disabled via input

6. **Testing**
   - Executes test suites
   - Collects code coverage
   - Supports custom test commands

7. **Coverage Upload**
   - Uploads coverage to Codecov
   - Only if token provided

8. **Docker Build**
   - Sets up QEMU and Buildx
   - Authenticates to registry
   - Builds multi-platform images
   - Pushes to registry (except on PRs by default)

9. **Post-Build Hooks**
   - Executes custom commands after build
   - Useful for verification, scanning

10. **Helm Package**
    - Updates Chart.yaml with version
    - Packages Helm chart
    - Pushes to OCI registry

11. **Release Creation**
    - Generates changelog from git history
    - Creates GitHub Release on tags
    - Includes image and Helm info

## Project Structure

```
.
├── action.yml              # Main action definition
├── README.md               # User documentation
├── ASSUMPTIONS.md          # Requirements and assumptions
├── EXAMPLES.md             # Usage examples
├── QUICKREF.md             # Quick reference guide
├── CHANGELOG.md            # Version history
├── CONTRIBUTING.md         # This file
├── LICENSE                 # MIT License
└── .github/
    └── workflows/
        └── ci.yml          # Original workflow for reference
```

## How to Contribute

### Reporting Issues

1. Check existing issues to avoid duplicates
2. Use issue templates when available
3. Provide:
   - Action version
   - Runner OS and version
   - Language and version
   - Minimal reproducible example
   - Expected vs actual behavior
   - Relevant logs

### Suggesting Features

1. Open an issue with `[Feature Request]` prefix
2. Describe the use case
3. Explain why existing inputs/features don't work
4. Provide example usage

### Submitting Pull Requests

1. **Fork and Clone**
   ```bash
   git clone https://github.com/YOUR-USERNAME/inpacken-un-af-dor-mit.git
   cd inpacken-un-af-dor-mit
   ```

2. **Create a Branch**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/your-fix-name
   ```

3. **Make Changes**
   - Follow existing code style
   - Use descriptive variable names
   - Add comments for complex logic
   - Update documentation

4. **Test Your Changes**
   - Test with multiple languages
   - Test with different configurations
   - Verify backward compatibility
   - Test on ubuntu-latest runner

5. **Update Documentation**
   - Update README.md if adding features
   - Update ASSUMPTIONS.md if changing requirements
   - Add examples to EXAMPLES.md
   - Update CHANGELOG.md

6. **Commit Changes**
   ```bash
   git add .
   git commit -m "feat: add support for xyz"
   # or
   git commit -m "fix: resolve issue with abc"
   ```

   Use conventional commit format:
   - `feat:` - New feature
   - `fix:` - Bug fix
   - `docs:` - Documentation only
   - `style:` - Code style (formatting)
   - `refactor:` - Code refactoring
   - `test:` - Adding tests
   - `chore:` - Maintenance tasks

7. **Push and Create PR**
   ```bash
   git push origin feature/your-feature-name
   ```
   
   Then open a Pull Request on GitHub

## Development Guidelines

### Adding Language Support

To add support for a new language:

1. **Add Setup Step**
   ```yaml
   - name: Set up NewLang
     if: inputs.language == 'newlang'
     uses: actions/setup-newlang@v1
     with:
       version: ${{ inputs.language-version || 'default' }}
   ```

2. **Add Dependency Installation**
   ```yaml
   - name: Install dependencies (NewLang)
     if: inputs.language == 'newlang' && inputs.run-tests == 'true'
     shell: bash
     run: |
       echo "::group::Installing NewLang dependencies"
       # Install commands here
       echo "::endgroup::"
   ```

3. **Add Lint Step**
   ```yaml
   - name: Run linting (NewLang)
     if: inputs.run-lint == 'true' && inputs.language == 'newlang'
     shell: bash
     run: |
       # Linting commands
   ```

4. **Add Test Step**
   ```yaml
   - name: Run tests (NewLang)
     if: inputs.run-tests == 'true' && inputs.language == 'newlang'
     shell: bash
     run: |
       # Test commands
   ```

5. **Update Documentation**
   - Add to README.md supported languages
   - Add to ASSUMPTIONS.md language requirements
   - Add example to EXAMPLES.md

### Adding New Inputs

1. **Define in action.yml**
   ```yaml
   new-input:
     description: 'Description of the input'
     required: false
     default: 'default-value'
   ```

2. **Use in Steps**
   ```yaml
   - name: Use new input
     shell: bash
     run: |
       if [ "${{ inputs.new-input }}" == "value" ]; then
         # Do something
       fi
   ```

3. **Document**
   - Add to README.md inputs table
   - Add to QUICKREF.md if commonly used
   - Add example usage

### Testing Guidelines

**Manual Testing Checklist:**

- [ ] Test with Go project
- [ ] Test with Node.js project
- [ ] Test with Python project
- [ ] Test with custom Dockerfile path
- [ ] Test with Helm enabled
- [ ] Test with tests disabled
- [ ] Test with custom commands
- [ ] Test on tag push
- [ ] Test on branch push
- [ ] Test on pull request
- [ ] Test with multi-platform builds
- [ ] Test with different registries

**Test Repository Setup:**

Create test repositories with:
- Simple application in target language
- Dockerfile
- Optional Helm chart
- GitHub Actions workflow using the action

## Code Style

### Shell Scripts

```bash
# Use bash for shell steps
shell: bash

# Group output
echo "::group::Descriptive Name"
# commands
echo "::endgroup::"

# Error handling
if [ condition ]; then
  echo "::error::Error message"
  exit 1
fi

# Warnings
echo "::warning::Warning message"

# Set outputs
echo "key=value" >> $GITHUB_OUTPUT
```

### YAML Formatting

```yaml
# Use 2-space indentation
- name: Step name
  if: condition
  shell: bash
  run: |
    command1
    command2
```

### Documentation

- Use clear, concise language
- Include code examples
- Use tables for structured data
- Add comments for complex logic
- Keep README.md under 15k words

## Release Process

1. **Update Version**
   - Update CHANGELOG.md
   - Update version references in README.md

2. **Create Tag**
   ```bash
   git tag -a v1.1.0 -m "Release v1.1.0"
   git push origin v1.1.0
   ```

3. **Update Major Version Tag**
   ```bash
   git tag -fa v1 -m "Update v1 to v1.1.0"
   git push origin v1 --force
   ```

4. **Create GitHub Release**
   - Copy CHANGELOG.md content
   - Mark as latest release
   - Attach any artifacts

## Support

- 📧 Email: [maintainer email]
- 💬 Discussions: GitHub Discussions
- 🐛 Issues: GitHub Issues
- 📖 Docs: README.md and related docs

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Code of Conduct

### Our Pledge

We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards

**Positive behavior includes:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints
- Gracefully accepting constructive criticism
- Focusing on what is best for the community

**Unacceptable behavior includes:**
- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Other conduct which could reasonably be considered inappropriate

## Recognition

Contributors will be recognized in:
- GitHub contributors list
- CHANGELOG.md for significant contributions
- README.md acknowledgments section (for major features)

## Questions?

Don't hesitate to ask questions by opening an issue with the `question` label.

---

Thank you for contributing! 🎉
