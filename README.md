# ash-registry

This repository servies a dual purpose:
1. It is a registry for Rust crates that are compatible with the Ash Framework.
2. It provides a GitHub Action to automate the process of publishing Rust crates to this registry.

## 1. For Crate Consumers

### Step 1: Add the Registry to Your Cargo Configuration
To use crates hosted here, you only need to configure your local Cargo client once.
Add the following to your `.cargo/config.toml`:

```toml
[registries.ash-registry]
index = "https://github.com/AABelkhiria/ash-registry"

[net]
git-fetch-with-cli = true
```

### Step 2: Add the Registry to Your Crate's `Cargo.toml`
In your crate's `Cargo.toml`, specify the registry:
```toml
[dependencies]
my-lib = { version = "1.2.3", registry = "ash-registry" }
```

## 2. For Crate Authors
To publish your library (e.g., `my-lib`) to this registry, you use the custom GitHub Action defined in this repository.

### Pre-requisites
In your library's GitHub repository (e.g., `my-lib`), you must configure a secret to allow its workflow to push updates to ash-registry.

<details>
<summary>Create a Personal Access Token (PAT)</summary>

1. Go to your GitHub Developer Settings.
2. Generate a new classic token.
3. Give it a descriptive name (e.g., ash-registry-writer).
4. Set an expiration date.
5. Grant it the repo scope. This is all it needs.
</details>

<details>
<summary>Create a Repository Secret</summary>

1. In your `my-lib` repository, go to `Settings` > `Secrets and variables` > `Actions`.
2. Create a New repository secret named `REGISTRY_TOKEN`.
3. Paste the PAT you just created as the value.
</details>

### The Publishing Workflow
In your library's repository (`my-lib`), create a workflow file at `.github/workflows/publish.yml` with the following content. This workflow will trigger automatically whenever you push a new version tag.

```yaml
# my-lib/.github/workflows/publish.yml

name: Publish to ash-registry

on:
  push:
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+*' # e.g., v1.2.3, v1.2.3-alpha.1

jobs:
  publish:
    runs-on: ubuntu-latest
    # This permission is required for the action to create a GitHub Release
    permissions:
      contents: write
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Install Rust Toolchain
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          override: true
      
      - name: Publish via Custom Action
        uses: AABelkhiria/ash-registry@v1
        with:
          # The default GITHUB_TOKEN is used to create the release
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # The PAT is used to push the index commit to ash-registry
          registry-token: ${{ secrets.REGISTRY_TOKEN }}
```
