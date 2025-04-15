
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
name: Build .NET Project

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Use .NET Build Action
        uses: your-username/your-repo-name@main
        with:
          project-name: MyApp
          app-dir: src/MyApp/MyApp.csproj
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
