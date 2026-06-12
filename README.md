# gh2gh-source

Source repository for testing GitHub-to-GitHub replication using `git clone --mirror` and `git push --mirror` from a GitHub Actions workflow.

Destination: [shashanklmurthy/gh2gh-destination](https://github.com/shashanklmurthy/gh2gh-destination)

## Mirror workflow

The [Mirror to destination](.github/workflows/mirror-to-destination.yml) workflow runs on demand (`workflow_dispatch`). It:

1. Clones this repository as a bare mirror
2. Pushes the full mirror (all branches, tags, and refs) to the destination repository

**Warning:** `git push --mirror` overwrites the destination repository to match the source exactly. Treat the destination as a mirror, not a place for independent commits.

## Setup (one time)

### 1. Create a Personal Access Token (PAT)

You need a token that can **push** to the destination repository.

**Fine-grained PAT (recommended)**

- Resource owner: your account
- Repository access: only `gh2gh-destination`
- Permissions:
  - **Contents** → Read and write
  - **Workflows** → Read and write (required because this repo includes `.github/workflows/`)

**Classic PAT**

- Scopes: **`repo`** and **`workflow`**
- GitHub rejects pushes that create or update workflow files unless the token has the `workflow` scope, even if `repo` is enabled

### 2. Add the PAT as a repository secret

In **gh2gh-source** on GitHub:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `MIRROR_PAT`
4. Value: your PAT

The workflow uses the built-in `GITHUB_TOKEN` to read this repository and `MIRROR_PAT` to push to the destination.

### 3. Run the workflow

1. Open the **Actions** tab in this repository
2. Select **Mirror to destination**
3. Click **Run workflow**

After it completes, [gh2gh-destination](https://github.com/shashanklmurthy/gh2gh-destination) should match this repository (same commits, branches, and tags).

## What you do not need

- **Deploy keys** — mirror push updates all refs; a single-repo deploy key is awkward for this pattern
- **A PAT on the destination repo** — the secret lives on the source repo and pushes outward
- **GitHub App** — optional for org/production use; a PAT is enough for personal repos

## Local testing

```bash
git clone --mirror https://github.com/shashanklmurthy/gh2gh-source.git mirror.git
cd mirror.git
git push --mirror https://github.com/shashanklmurthy/gh2gh-destination.git
```

Use a credential helper or embed a PAT in the URL locally; do not commit tokens.
