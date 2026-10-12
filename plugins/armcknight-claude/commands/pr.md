---
description: "Review, commit, open a PR described from the Linear issue, then iterate until CI is green."
argument-hint: "[--autonomous] [--draft]"
---

First: perform /armcknight-claude:review

Then:
- Decide about screenshots on every run, from the branch as it is now. Do not skip this step because the code is already committed, a review already ran, or an earlier PR for this branch exists, open or closed: the earlier work may predate this step.
- If the change is something an end user can see — a new feature, a changed screen or flow, or a fix to a visible bug — and the repo has a UI-test target that takes screenshots (for example an iOS app whose CI passes `ui-screenshots-scheme` to `armcknight/workflows`' `ios-ci.yml`), add or extend a UI test there that shows it:
    - Launch the app into seeded, deterministic state the way the existing UI tests do, go to the changed UI, and assert what the change is meant to do, so the test also proves the change.
    - Attach a screenshot at each step a reviewer should see, with the target's existing screenshot helper (an `XCTAttachment` kept with `.keepAlways`) and a short, descriptive name. For a fix, capture the state that used to be wrong.
    - Put each new test in its own `XCTestCase` class, or change an existing class only when the change belongs there: CI runs every test class in a file the PR adds or changes and comments their screenshots on the PR.
    - If an existing test that the PR does not change already goes to the changed UI — common for a fix — name it instead of writing a new one, in a hidden marker in the PR description: `<!-- ui-screenshots: <class>/<method> -->`, with IDs separated by spaces or commas. Also use the marker to narrow a changed class down to the methods that show the change.
    - Run the new test locally and confirm it passes before continuing.
- Commit any outstanding work in the working index.
- Open the pull request, supplying the title and description:
    - Use `direnv exec linctl` to grab context from the original issue to write a top-line problem/solution statement in the PR description.
    - Use the [conventional commit](https://www.conventionalcommits.org/en/v1.0.0/) format, scoping to specific components when possible like `fix(AutofixStatusView): <message>` or `feat(seer setup): <message>` for the PR title.
    - Write the description from the branch as it is now. Do not copy an earlier PR's description, even one for the same commit: it may lack the screenshot decision.
    - The description should summarize the solution strategy and pertinent implementation details.
    - State the screenshot decision in the description. If you added, changed or named a UI test for screenshots, say which one, and that CI will comment its screenshots on the PR; include the `ui-screenshots` marker when you name tests there. CI reads the description of the push that started it, so write it before pushing again, not after. Otherwise, say in one sentence why no screenshot test applies: the change is not user-facing, or the repo has no screenshot UI-test target.
    - Note the linear issue ID in the description.
    - If `$ARGUMENTS` contains `--autonomous`, use `gh pr create` (no `--web`) so no browser is opened. Otherwise use `gh pr create --web`.
    - If `$ARGUMENTS` contains `--draft`, add `--draft` to the `gh pr create` invocation so the PR is opened in draft state.
- Mark the associated linear issue, if there is one, as "In Review" status using `linctl`.
- After opening the PR, keep an eye on any CI checks and iterate to fix breakages until they're all green.
- Once CI passes, if the repo is owned by `getsentry` (check with `gh repo view --json owner --jq .owner.login`), add a comment to the PR with `gh`: `@sentry review` to initiate a Seer review. Wait until the action completes, then check for any comments left by Seer on the PR, and address them in a new commit. In a repo with a different owner, do not comment `@sentry review`.
- Check the PR for review comments left by humans, not bots like cursor or sentry. Find the PR author with `gh pr view --json author --jq .author.login`.
    - Address comments from the PR author in a new commit.
    - Flag comments from any other human to me for manual handling.
    - Do not reply to any human comment.
- Do another round of CI iteration until all checks pass.
