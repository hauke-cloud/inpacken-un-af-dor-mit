<!-- llm-readme-management spec=1 commit=460d188ddea3674ce11971b59f460d9368fbc2c1 template=default model=qwen3.6-35b-a3b digest=598d66067ca0 generated=2026-09-08T20:32:37Z -->
<a href="https://hauke.cloud" target="_blank"><img src="https://img.shields.io/badge/home-hauke.cloud-brightgreen" alt="hauke.cloud" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud" target="_blank"><img src="https://img.shields.io/badge/github-hauke.cloud-blue" alt="hauke.cloud Github Organisation" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud/llm-readme-management" target="_blank"><img src="https://img.shields.io/badge/template-default-orange" alt="Repository type - default" style="display: block;" /></a>


# Inpacken Un Af Dor Mit


<img src="https://raw.githubusercontent.com/hauke-cloud/.github/main/resources/img/organisation-logo-small.png" alt="hauke.cloud logo" width="109" height="123" align="right">


<llm header>

This repository contains a reusable composite GitHub Action that orchestrates complete CI/CD pipelines for multi-language projects. It automates linting, testing, building multi-platform container images, and packaging Helm charts within your workflows. Developers and DevOps engineers use this action to standardize build processes across Go, Node.js, Python, Rust, Java, and .NET repositories with a single configuration step.

</llm>


## :book: Description

<llm description>

You need a consistent, repeatable CI/CD pipeline across projects written in different languages without maintaining separate workflow configurations for each repository. This composite GitHub Action solves that by orchestrating the entire build process in a single step. It detects your project language, configures the appropriate runtime, and executes dependency installation, linting, and testing using sensible defaults or your custom commands.

The action handles containerization and deployment artifacts automatically:
- Builds and pushes multi-platform Docker images to an OCI registry
- Packages and pushes Helm charts as OCI artifacts when enabled
- Uploads code coverage reports to Codecov
- Generates changelogs and creates GitHub Releases on tag pushes

</llm>


## 🚀 Getting started

<llm getting_started hint="Assume nothing about the ecosystem beyond what the analysis names. If the repository has no build step, say what a reader does with it instead.">

1. Clone the repository and navigate into it.
```bash
git clone https://github.com/hauke-cloud/inpacken-un-af-dor-mit.git
cd inpacken-un-af-dor-mit
```

2. Create a `.github/workflows/ci.yml` file in your project to define the pipeline.
```yaml
name: CI
on: push
jobs:
  run:
    runs-on: ubuntu-latest
    permissions: packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: hauke-cloud/inpacken-un-af-dor-mit@v1
        with:
          language: go
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

3. Add your project's source files and a `Dockerfile` to the repository root.
```bash
touch Dockerfile && git add . && git commit -m "Add workflow and Dockerfile"
```

4. Push the workflow to GitHub to trigger the lint, test, build, and push steps.
```bash
git push origin main
```

</llm>


## :airplane: Usage

<llm usage>

You consume this repository by adding a single step to any GitHub Actions workflow. The action validates your project layout, sets up the appropriate runtime, and runs linting, testing, image building, and optional Helm packaging in sequence.

- **Run a standard CI/CD pipeline** for a supported language. Provide the `language` input and a registry token. The action automatically detects defaults for dependency installation, linting, and

</llm>


## 📄 License

This Project is licensed under the GNU General Public License v3.0

- see the [LICENSE](LICENSE) file for details.


## :coffee: Contributing

To become a contributor, please check out the [CONTRIBUTING](CONTRIBUTING.md) file.


## :email: Contact

For any inquiries or support requests, please open an issue in this
repository or contact us at [contact@hauke.cloud](mailto:contact@hauke.cloud).
