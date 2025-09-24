# Checkout with GitHub App Token

A composite GitHub Action that combines GitHub App authentication with repository checkout, providing seamless access to private repositories and submodules.

## Features

- 🔐 Authenticates using GitHub App credentials
- 📦 Full support for private submodules
- ⚡ All `actions/checkout` options available
- 🔄 Token management with configurable scope
- 🛠️ Drop-in replacement for standard checkout when using GitHub Apps

## Usage

### Basic Example

```yaml
- uses: shousper/checkout-with-github-app-token@v1
  with:
    app-id: ${{ vars.APP_ID }}
    private-key: ${{ secrets.PRIVATE_KEY }}
```

### With Submodules

⚠️ **Important**: When using submodules, you **must** specify the `owner` parameter (and optionally `repositories`). This is because submodules are external repositories that require the token to have access beyond just the current repository.

```yaml
- uses: shousper/checkout-with-github-app-token@v1
  with:
    app-id: ${{ vars.APP_ID }}
    private-key: ${{ secrets.PRIVATE_KEY }}
    owner: ${{ github.repository_owner }}  # Required for submodules!
    submodules: recursive
```

### Multiple Repositories Access

```yaml
- uses: shousper/checkout-with-github-app-token@v1
  with:
    app-id: ${{ vars.APP_ID }}
    private-key: ${{ secrets.PRIVATE_KEY }}
    owner: ${{ github.repository_owner }}
    repositories: |
      repo1
      repo2
      repo3
```

## Inputs

### GitHub App Authentication

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `app-id` | GitHub App ID | ✅ | - |
| `private-key` | GitHub App private key | ✅ | - |
| `owner` | Repository owner for token scope | ❌ | `''` (current repo only) |
| `repositories` | Specific repositories to grant access to | ❌ | `''` |
| `skip-token-revoke` | Don't revoke token after job completes | ❌ | `false` |
| `github-api-url` | GitHub API URL for GitHub Enterprise | ❌ | `${{ github.api_url }}` |

### Checkout Options

All standard `actions/checkout@v5` options are supported:

| Input | Description | Default |
|-------|-------------|---------|
| `repository` | Repository name with owner | `${{ github.repository }}` |
| `ref` | Branch, tag or SHA to checkout | Default branch |
| `submodules` | Checkout submodules (`true` or `recursive`) | `false` |
| `fetch-depth` | Number of commits to fetch | `1` |
| `path` | Relative path to place the repository | `./` |
| `lfs` | Whether to download Git-LFS files | `false` |
| `clean` | Whether to clean the workspace | `true` |
| `sparse-checkout` | Patterns for sparse checkout | - |
| `persist-credentials` | Configure token in git config | `true` |

[View all checkout options →](https://github.com/actions/checkout#usage)

## Outputs

| Output | Description |
|--------|-------------|
| `token` | The GitHub App installation access token |
| `installation-id` | GitHub App installation ID |
| `app-slug` | GitHub App slug |

## Setup

### 1. Create a GitHub App

1. Go to Settings → Developer settings → GitHub Apps → New GitHub App
2. Configure with minimal permissions:
   - **Repository permissions**: Contents (Read), Metadata (Read)
   - **Where can this GitHub App be installed**: Your preference
3. After creation, note the App ID
4. Generate a private key and save it

> **Security Consideration**: The GitHub App will have access to all repositories it's installed on. Consider creating separate apps per team or project to maintain proper access isolation and follow the principle of least privilege.

### 2. Install the App

1. Go to the app's page and click "Install App"
2. Select the repositories that need access
3. Include both the main repository and any private submodules

### 3. Configure Secrets

In your repository settings:

1. Add a repository variable `APP_ID` with your App ID
2. Add a repository secret `PRIVATE_KEY` with the private key contents

## Examples

### Checkout Different Repository

```yaml
- uses: shousper/checkout-with-github-app-token@v1
  with:
    app-id: ${{ vars.APP_ID }}
    private-key: ${{ secrets.PRIVATE_KEY }}
    repository: myorg/another-repo
    ref: develop
```

### Sparse Checkout

```yaml
- uses: shousper/checkout-with-github-app-token@v1
  with:
    app-id: ${{ vars.APP_ID }}
    private-key: ${{ secrets.PRIVATE_KEY }}
    sparse-checkout: |
      src
      tests
```

### Full History

```yaml
- uses: shousper/checkout-with-github-app-token@v1
  with:
    app-id: ${{ vars.APP_ID }}
    private-key: ${{ secrets.PRIVATE_KEY }}
    fetch-depth: 0
```

## Why Use This Action?

The standard `GITHUB_TOKEN` provided by GitHub Actions has limitations:
- Cannot access private repositories outside the current one
- Cannot checkout private submodules
- Limited to the current repository's scope

This action solves these problems by using GitHub App authentication, which can be granted access to multiple repositories while maintaining security through scoped permissions.

## License

MIT