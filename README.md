# claude-code-demo

A minimal repository for demonstrating Claude Code as an automated reviewer in
GitHub Actions. The application is a single static HTML page; the value being
shown is the pull-request workflow, not the app itself.

On every pull request, a workflow runs `anthropics/claude-code-action@v1` and
posts a review comment that checks three things:

1. **PR title** follows Conventional Commits (`feat:`, `fix:`, `chore:`,
   `docs:`, `refactor:`, `test:`).
2. **PR description** is non-empty and explains both *what* changed and *why*.
3. **Code quality** in the diff — bugs, missing error handling, hardcoded
   values that belong in configuration, and basic accessibility issues.

## Repository contents

| Path                                  | Purpose                                          |
| ------------------------------------- | ------------------------------------------------ |
| `index.html`                          | The demo page (textarea + character counter).    |
| `.github/workflows/claude-review.yml` | The Claude review workflow.                      |
| `.github/PULL_REQUEST_TEMPLATE.md`    | Prompts contributors for what / why / how to test. |

## Setup

These steps need to be done once per repository (or once per organization, if
you install the app at the org level).

1. **Install the Claude GitHub App.** Go to
   [github.com/apps/claude](https://github.com/apps/claude) and install it on
   the account or organization that owns this repository. Grant access to this
   repository (or "All repositories" if you prefer).
2. **Add the API key as a repository secret.** In GitHub, go to
   *Settings → Secrets and variables → Actions → New repository secret*. Name
   it `ANTHROPIC_API_KEY` and paste in a key from the Anthropic Console.
3. **Confirm Actions are enabled.** *Settings → Actions → General* should allow
   workflows to run and allow GitHub Actions to *create and approve pull
   request comments* (this is the default for most repos).

That's it. The next pull request opened against this repo will trigger a
review.

