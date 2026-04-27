# claude-code-demo

A small repo we use to show what Claude Code looks like wired into GitHub Actions
as an automated PR reviewer. The app under review is a static HTML page with a
textarea and a character counter. That part doesn't matter. The point is the
review workflow.

When someone opens a pull request, a GitHub Action runs Claude against the diff
and posts a review comment. The prompt tells Claude to check three things:

- The PR title follows Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`,
  `refactor:`, `test:`)
- The PR description isn't empty and actually says what changed and why
- The code in the diff looks reasonable: no obvious bugs, no missing error
  handling, no hardcoded values that should be config, no glaring accessibility
  problems

The interesting file is `.github/workflows/claude-review.yml`. The whole review
policy lives in the prompt block there, in plain English, version-controlled
with the rest of the repo. If you want to change what Claude looks for, you edit
the prompt and merge it like any other code change.

## Running the demo

The setup (installing the GitHub app, adding the API key secret) is already
done on this repo, so you can go straight to running it. The whole thing takes
about five minutes.

### 1. Show the workflow file first

Before triggering anything, open `.github/workflows/claude-review.yml` on GitHub
and walk through it. Two things to point out:

The `prompt:` block is where the team's review standards live. It's just text.
Anyone on the team can read it, propose changes, and merge them like any other
code change.

The trigger is `pull_request` with `opened, synchronize, reopened`. That means
it runs on every new PR and every push to an open PR, but not on direct pushes
to main.

### 2. Open a deliberately bad PR

From your terminal, in this repo:

```
git checkout -b demo/bad-pr
# edit index.html — change the heading text, or anything else visible
git add index.html
git commit -m "update stuff"
git push -u origin demo/bad-pr
gh pr create --title "update stuff" --body ""
```

You're feeding Claude a PR that breaks all three rules at once: bad title, empty
description, and a code change in the diff. That's the point. You want to see
all three checks fire.

### 3. Watch the Action run

Open the Actions tab on the repo. The "Claude PR Review" job kicks off within a
few seconds and usually finishes in 30 to 90 seconds.

When it's done, go back to the PR. There will be a review comment from
`claude[bot]`. It should:

- Flag the title and suggest a Conventional Commits version
- Point out that the description is empty
- Comment on whatever you actually changed in the diff
- End with a one-line verdict: APPROVE, REQUEST CHANGES, or COMMENT

This is the moment in the demo. Read the comment out loud. The reviewer didn't
write any of those checks by hand — they came from the prompt in the workflow
file you showed in step 1.

### 4. Fix the PR and watch it re-run

Now make the PR good. Edit the title and body:

```
gh pr edit --title "feat: update demo page heading" --body "$(cat <<'EOF'
## What changed
Updated the heading text on the demo page.

## Why
Wanted something more descriptive for the team.

## How to test
Open index.html in a browser and confirm the new heading shows up.
EOF
)"
```

Title and body changes alone do not re-trigger the workflow. To re-run it, push
a small follow-up commit (any change), or re-run the failed job from the Actions
tab. The new review should pass, or come close.

### 5. The takeaway

The line worth landing: the prompt is the team's review checklist, kept in the
repo, applied to every PR before a human reviewer opens it. It doesn't replace
human review. It catches the obvious stuff so the human reviewer can spend their
time on the parts that actually need judgment.

The same pattern works on a real codebase. The prompt gets longer (testing
requirements, architecture notes, security checks for the parts that need them),
but the shape is the same as what's here.

## Setting this up on another repo

Three steps:

1. Install the Claude GitHub App from `github.com/apps/claude` on the repo or
   org that owns the codebase.
2. Add `ANTHROPIC_API_KEY` as a repository secret under Settings → Secrets and
   variables → Actions. The key comes from the Anthropic Console.
3. Copy `.github/workflows/claude-review.yml` into the target repo. Edit the
   prompt to match that team's review standards.

That's the whole setup.

## What's in this repo

- `index.html` — the static demo page (textarea, character counter)
- `.github/workflows/claude-review.yml` — the review workflow and prompt
- `.github/PULL_REQUEST_TEMPLATE.md` — what / why / how to test sections
