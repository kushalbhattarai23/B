# GitHub Actions Powered

## Bidirectional Repo Mirror

Automatically sync two GitHub repositories in both directions.\
No scripts, no manual steps, no infinite loops. Just push and go.

Repo A\
Repo B

------------------------------------------------------------------------

## How It Works

### Fully Automatic

Zero manual intervention. Push to one repo, the other updates instantly
via GitHub Actions.

### Loop Prevention

Smart commit message filtering with \[skip-sync\] prevents infinite sync
cycles.

### History Preserved

All commits, authors, and messages are preserved exactly as-is across
both repos.

### Bidirectional

Changes flow both ways --- push to A or B, and the other stays perfectly
in sync.

### Near-Instant

GitHub Actions triggers within seconds of a push event. Sync completes
in under a minute.

### No Code Required

One-time YAML setup. After that, every team member gets automatic sync
with zero effort.

------------------------------------------------------------------------

## Architecture

The system uses GitHub Actions workflows triggered on push events, with
a commit-message guard to prevent infinite loops.

Developer pushes to Repo A (main branch)\
GitHub Actions triggers sync-to-b.yml\
Checks: does commit message contain \[skip-sync\]?

├─ Yes → Workflow exits. No sync. Loop broken ✓\
└─ No → Pushes commits to Repo B via PAT

Repo B receives push → its workflow checks \[skip-sync\]\
Since it's the same commit, --force-with-lease makes it a no-op → loop
prevented

------------------------------------------------------------------------

## Loop Prevention Strategy

The biggest risk with bidirectional sync is an infinite loop:\
A pushes to B, which triggers B to push back to A, and so on forever.

This system uses two layers of protection:

1.  Commit message filter:\
    Workflows skip if the commit contains \[skip-sync\]. You can tag
    automated commits with this flag.

2.  Force-with-lease:\
    If the target branch already has the exact same commits,
    --force-with-lease makes the push a no-op, naturally breaking the
    loop.

------------------------------------------------------------------------

## Setup Guide

Four steps to fully automated bidirectional sync.

### 1. Create a Personal Access Token (PAT)

Generate a GitHub PAT with repo access for cross-repository pushes.

Go to GitHub → Settings → Developer settings → Personal access tokens →
Fine-grained tokens\
Create a token with 'Contents: Read and write' permission for both
repos\
Copy the token --- you'll need it in the next step

### 2. Add the PAT as a Secret in Both Repos

Store the token securely so GitHub Actions can authenticate.

In Repo A: Settings → Secrets → Actions → New secret named TARGET_PAT\
In Repo B: Settings → Secrets → Actions → New secret named TARGET_PAT\
Paste the same PAT value in both repos

### 3. Add the Workflow Files

Create GitHub Actions workflow YAML files in each repository.

In Repo A: create .github/workflows/sync-to-b.yml with the A→B workflow\
In Repo B: create .github/workflows/sync-to-a.yml with the B→A workflow\
Update the repo URLs in the YAML to match your actual repositories

### 4. Test the Sync

Make a small change and watch it propagate automatically.

Push a commit to Repo A's main branch\
Check Repo B --- the same commit should appear within \~30 seconds\
Try pushing to Repo B and verify it syncs back to Repo A

------------------------------------------------------------------------

## .github/workflows/sync-to-b.yml

``` yaml
name: Sync A → B

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest
    if: ${{ !contains(github.event.head_commit.message, '[skip-sync]') }}
    steps:
      - name: Checkout Repo A
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Push to Repo B
        run: |
          git remote add target https://x-access-token:${{ secrets.TARGET_PAT }}@github.com/YOUR_ORG/repo-b.git
          git push target main --force-with-lease
          echo "✅ Synced A → B"
```

------------------------------------------------------------------------

## .github/workflows/sync-to-a.yml

``` yaml
name: Sync B → A

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest
    if: ${{ !contains(github.event.head_commit.message, '[skip-sync]') }}
    steps:
      - name: Checkout Repo B
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Push to Repo A
        run: |
          git remote add target https://x-access-token:${{ secrets.TARGET_PAT }}@github.com/YOUR_ORG/repo-a.git
          git push target main --force-with-lease
          echo "✅ Synced B → A"
```

------------------------------------------------------------------------

Built with GitHub Actions • No servers • No cron jobs • Just YAML
