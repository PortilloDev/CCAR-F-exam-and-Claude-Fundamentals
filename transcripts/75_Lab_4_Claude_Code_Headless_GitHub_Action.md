# Lecture 75: Lab 4: Claude Code Headless as GitHub Action

**Section:** Claude Code in a Team  
**Lecture ID:** 58217523

---

## Transcript

### Lab objective

Set up Claude Code to run automatically as a GitHub Action that reviews PRs and posts inline comments.

---

### Steps

#### 1. Duplicate the template repository

Fork or duplicate the provided template repo. This gives you a clean starting point with a basic project structure and a pre-configured workflow skeleton.

#### 2. Set the OAuth token as a GitHub secret

```
Repository → Settings → Secrets and variables → Actions → New repository secret

Name: ANTHROPIC_API_KEY
Value: <your Anthropic API key>
```

The workflow references it as `${{ secrets.ANTHROPIC_API_KEY }}`.

#### 3. Create a pull request

Make a code change on a feature branch and open a PR against `main`. This is the trigger for the action.

#### 4. The Claude Code Action runs

The workflow fires on `pull_request` events. Claude Code:
1. Checks out the repository.
2. Reads the diff (`git diff origin/main...HEAD`).
3. Reviews against the criteria in the prompt.
4. Posts findings as inline PR review comments.

---

### Workflow file (`.github/workflows/claude-review.yml`)

```yaml
name: Claude Code PR Review

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          prompt: |
            Review this pull request for:
            1. Security vulnerabilities (injection, auth bypass, secrets)
            2. Unhandled error paths
            3. Missing input validation
            Post a review comment for each finding with the file and line number.
          permission-mode: dontAsk
          allowed-tools: "Read,Grep,Bash(git diff)"
          post-inline-comments: true
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### What to observe

- After opening the PR, the Action appears in the PR's "Checks" tab.
- When complete, inline comments appear in the "Files changed" tab anchored to specific lines.
- Each comment names the file, line, and finding — no need to parse JSON manually when using `post-inline-comments: true`.

---

### Key permissions

```yaml
permissions:
  contents: read        # read the repo
  pull-requests: write  # post review comments
```

Without `pull-requests: write`, the action can read the code but cannot post comments.
