
# Building .NET Project - GitHub Action

This GitHub Action restores, builds, and optionally uploads a .NET project using the `dotnet` CLI. It helps automate common CI tasks like dependency restore, compiling code, capturing logs, and saving build artifacts.

## Inputs

| Name                | Description                                 | Required | Default    |
|---------------------|---------------------------------------------|----------|------------|
| `project-name`      | Name of the .NET project.                   | ✅ Yes   | —          |
| `app-dir`           | Path to the `.csproj` file.                 | ✅ Yes   | —          |
| `build-configuration` | Build configuration (`Release`, `Debug`). | ❌ No    | `Release`  |
| `upload_build`      | Upload artifacts after build (`true`/`false`). | ❌ No | `false`    |

## Usage

```yaml
name: Build Multiple .NET Projects

on:
  push:
    branches:
      - "main"
      - "release"
  
  pull_request:
  
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        project:
          - project-name: 03-app-dotnet
            app-dir: 03-app-dotnet/app-dotnet.csproj
          - project-name: 03-app-dotnet
            app-dir: 03-app-dotnet/app-dotnet.csproj

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build ${{ matrix.project.project-name }}
        uses: ./.github/actions/dotnet-build-action
        with:
          project-name: ${{ matrix.project.project-name }}
          app-dir: ${{ matrix.project.app-dir }}
          build-configuration: Release
          upload_build: true
```

## What This Action Does

1. Checks out source code.
2. Validates the existence of the `.csproj` file.
3. Restores NuGet packages using `dotnet restore`.
4. Builds the project with detailed logging (`--verbosity diagnostic`).
5. Parses the log and annotates build **warnings** and **errors** in the GitHub Actions log.
6. ☁️ Optionally uploads build output as an artifact.

### Artifacts
If `upload_build: true`, a directory named like `dotnet_build_output-<project-name>` is uploaded as an artifact, containing the compiled files.

## Example

```yaml
- name: Build and upload MyProject
  uses: your-username/your-repo-name@main
  with:
    project-name: MyProject
    app-dir: src/MyProject/MyProject.csproj
    build-configuration: Debug
    upload_build: true
```

## Useful References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [dotnet CLI Documentation](https://learn.microsoft.com/en-us/dotnet/core/tools/)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [actions/checkout](https://github.com/actions/checkout)

---

**Note**: Replace `your-username/your-repo-name` with the actual path to your GitHub repository where this action resides.
