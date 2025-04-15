
# Build .NET Project - GitHub Action

This GitHub Action restores, builds, and optionally uploads a .NET project using the dotnet CLI. It's designed for monorepos or multi-project repositories and supports matrix builds across different .NET SDK versions and project configurations.

## Inputs

| Name                | Description                                           | Required | Default  |
|---------------------|-------------------------------------------------------|----------|----------|
| project-name        | Name of the .NET project. Used for logs/artifacts.    | Yes      | -        |
| app-dir             | Relative path to the .csproj file.                    | Yes      | -        |
| build-configuration | Build configuration (Release, Debug, etc.).           | No       | Release  |
| upload_build        | Whether to upload the build output (true or false).   | No       | false    |

## What This Action Does

1. Checks out the source code
2. Validates that the .csproj file exists
3. Restores NuGet packages with dotnet restore
4. Builds the project using dotnet build with diagnostic logging
5. Parses build logs and annotates warnings/errors
6. Optionally uploads the build artifacts

## Output Artifacts

If upload_build: true, an artifact folder named dotnet_build_output-<project-name> is created and uploaded. This includes all compiled files for that project.

## Example Workflow: Matrix Build for Multiple .NET Projects

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
        dotnet-version: [ '7.0.x' ]
        project:
          - project-name: my-app-dotnet-1
            app-dir: my-app-dotnet-1/app-dotnet.csproj
          - project-name: my-app-dotnet-2
            app-dir: my-app-dotnet-2/app-dotnet.csproj

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup .NET ${{ matrix.dotnet-version }}
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ matrix.dotnet-version }}

      - name: Build ${{ matrix.project.project-name }}
        uses: ./.github/actions/dotnet-build-action
        with:
          project-name: ${{ matrix.project.project-name }}
          app-dir: ${{ matrix.project.app-dir }}
          build-configuration: Release
          upload_build: true
```

## Directory Structure Example

```
.github/
├── workflows/
│   └── build-multiple-dotnet.yml
└── actions/
    └── dotnet-build-action/
        └── action.yml
my-app-dotnet-1/
  └── app-dotnet.csproj
my-app-dotnet-2/
  └── app-dotnet.csproj
```

## References

- [GitHub Actions using matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs
- [actions/checkout](https://github.com/actions/checkout)
- [GitHub Actions setup custom .NET](https://github.com/actions/setup-dotnet)
- [GitHub Actions Building and testing .NET](https://docs.github.com/en/enterprise-cloud@latest/actions/use-cases-and-examples/building-and-testing/building-and-testing-net)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [.NET CLI Documentation](https://learn.microsoft.com/en-us/dotnet/core/tools/)





## Notes

- Ensure the relative paths (app-dir) are correct from the repo root.
- This setup uses a composite action, referenced locally via ./.github/actions/dotnet-build-action.
- You can version and publish this action to a public repository for reuse across multiple projects.