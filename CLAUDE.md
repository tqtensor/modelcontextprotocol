# Upstream sync playbook

This fork tracks `perplexityai/modelcontextprotocol`. The workflow at `.github/workflows/upstream-sync.yml` opens/updates a single issue (label `upstream-sync`, typically #1) whenever upstream lands new commits. When that issue is open, follow the steps below.

## Rules of engagement

- **Do not merge, push to `main`, or close the sync issue without explicit per-step approval from the user.** "Let's address that" is approval to investigate and propose a plan — not to execute. Show the plan, wait for go-ahead, then execute one step at a time.
- This repo prefers **rebase merge**, never merge commits.
- The sync ships as a **PR against the fork** — do not push directly to `main`. Even though the auto-generated issue body literally says `git push origin main`, ignore that line.
- No AI attribution (`Co-Authored-By: Claude…`, etc.) in commits or PR bodies.

## Workflow

1. Read the open `upstream-sync` issue:
   ```bash
   gh issue list --repo tqtensor/modelcontextprotocol --label upstream-sync
   gh issue view <n> --repo tqtensor/modelcontextprotocol
   ```
2. Inspect divergence:
   ```bash
   git fetch upstream main
   git log upstream/main ^main --oneline
   git diff main...upstream/main --stat
   ```
3. Decide what to bring in. Apply this fork's established filters (confirm with the user before deviating):
   - **No install badges** (Cursor / VS Code / Kiro / …). They hardcode `@perplexity-ai/mcp-server` + `PERPLEXITY_API_KEY` and would launch the upstream server, not this fork.
   - **No npm version badge.** The fork installs from GitHub, not npm.
   - **Keep the separate "VS Code" section** with the `servers` / `type: stdio` schema. Don't accept upstream's consolidation that folds VS Code into the `mcpServers` table.
   - Adapt all commands and JSON to fork conventions: package `github:tqtensor/modelcontextprotocol`, env var `OPENROUTER_API_KEY`.
4. Pick a strategy and confirm with the user:
   - **Single-commit PR (preferred when most upstream content is filtered out).** Branch from `main`, make only the changes you actually want, open one PR. Loses upstream commit attribution but yields the cleanest history.
   - **Rebase then PR.** `git rebase upstream/main`, resolve conflicts per the filters above, push the branch, open the PR. Use this only when most upstream commits should land verbatim.
5. Open the PR **against the fork**, not upstream. `gh pr create` defaults to the parent repo from a fork checkout, so always pass `--repo` and `--head` explicitly:
   ```bash
   gh pr create --repo tqtensor/modelcontextprotocol \
     --base main --head tqtensor:<branch> \
     --title "<conventional commit title>" \
     --body "$(cat <<'EOF'
   ## Summary
   - …
   Closes #<sync-issue-number>.

   ## Test plan
   - [ ] …
   EOF
   )"
   ```
   **Verify the printed URL starts with `tqtensor/`** before declaring the PR opened. If it starts with `perplexityai/`, close it immediately and re-open against the fork.
6. Include `Closes #<n>` in the PR body so the sync issue auto-closes on rebase-merge.
7. Leave the actual rebase-merge to the user via the GitHub UI.

## Pitfalls observed in past sessions

- Pushing the merge directly to `origin/main` instead of opening a PR. Caused a force-push reset.
- Opening the PR against `perplexityai/modelcontextprotocol` because `gh pr create` was run without `--repo`. The PR was externally visible upstream before being closed.
- Treating "let's address the issue" as authorization for the full pipeline (merge + push + close). It's not. Each remote-visible step needs its own approval.
