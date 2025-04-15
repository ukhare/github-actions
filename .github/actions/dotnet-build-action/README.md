# Build .NET Project - GitHub Action

This GitHub Action restores, builds, and optionally uploads a .NET project using the dotnet CLI. It's designed to work with monorepos or repositories that may or may not have pre-existing .NET projects.

It also supports **creating test projects dynamically** when no project exists (e.g., in actions testing or validation scenarios), using a matrix strategy across multiple SDK versions and project configurations.

---

## Inputs

| Name                | Description                                           | Required | Default  |
|---------------------|-------------------------------------------------------|----------|----------|
| project-name        | Name of the .NET project. Used for logs/artifacts.    | Yes      | -        |
| app-dir             | Relative path to the .csproj file.                    | Yes      | -        |
| build-configuration | Build configuration (Release, Debug, etc.).           | No       | Release  |
| upload_build        | Whether to upload the build output (true or false).   | No       | false    |

---

## What This Action Does

1. Checks out the source code
2. Validates the `.csproj` path
3. Restores NuGet packages using `dotnet restore`
4. Builds with `dotnet build` and `diagnostic` verbosity
5. Parses logs to annotate warnings/errors
6. Optionally uploads build artifacts for the project

---

## Output Artifacts

If `upload_build: true`, an artifact folder named `dotnet_build_output-<project-name>` is created and uploaded.

---

## Example Workflow

### Default Mode (No existing .NET project)

When triggered by `push` or `pull_request`, this setup dynamically **creates** a testable console project using `dotnet new console` and builds it:

```yaml
name: Building Multiple .NET Projects (includes conditional test)

on:
  push:
    branches: [ "main", "release" ]

  pull_request:

  workflow_dispatch:
    inputs:
      run_build:
        description: 'Run the test for an existing dotnet project in repo'
        required: true
        default: 'false'

jobs:
  build:
    name: build-${{ matrix.project.project-name }}
    runs-on: [self-hosted, medium-amd64]

    strategy:
      matrix:
        dotnet-version: ['7.0.x']
        project:
          - project-name: my-app-dotnet-01
            app-dir: my-app-dotnet-01
          - project-name: my-app-dotnet-02
            app-dir: my-app-dotnet-02

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup .NET ${{ matrix.dotnet-version }}
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ matrix.dotnet-version }}

      - name: Create .NET Project - ${{ matrix.project.project-name }}
        if: github.event_name != 'workflow_dispatch'
        shell: bash
        run: |
          rm -rf "${{ matrix.project.app-dir }}"
          dotnet new console -n "${{ matrix.project.project-name }}" -o "${{ matrix.project.app-dir }}"

      - name: Build ${{ matrix.project.project-name }}
        uses: ./.github/actions/dotnet-build-action
        with:
          project-name: ${{ matrix.project.project-name }}
          app-dir: ${{ matrix.project.app-dir }}/${{ matrix.project.project-name }}.csproj
          build-configuration: Release
          upload_build: true
```

---

## Directory Structure Example

```
.github/
├── workflows/
│   └── build-dotnet.yml
└── actions/
    └── dotnet-build-action/
        └── action.yml
```

- With normal push, pull_request new projects will be created and build as defined in matrix.
- If any project exisst in repository then it can be tested with workflow_dispatch run_build: true
- This workflow can also be optimized to build existing projects by switching the workflow_dispatch logic to push, pull_request 
  - as this test workflow is created keeping in mind that no existing dotnet project already exists in 'devops-ci-workflow-shared' repository as of now.

---

## Notes

- Test projects are created automatically unless explicitly running a build for existing projects via `workflow_dispatch`.
- This is ideal for validating the action itself or for building real apps in your repo.
- Action is defined locally as a composite action at `./.github/actions/dotnet-build-action`.

## References

- [GitHub Actions using matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs
- [actions/checkout](https://github.com/actions/checkout)
- [GitHub Actions setup custom .NET](https://github.com/actions/setup-dotnet)
- [GitHub Actions Building and testing .NET](https://docs.github.com/en/enterprise-cloud@latest/actions/use-cases-and-examples/building-and-testing/building-and-testing-net)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [.NET CLI Documentation](https://learn.microsoft.com/en-us/dotnet/core/tools/)