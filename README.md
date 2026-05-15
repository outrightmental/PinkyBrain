# PinkyBrain

PinkyBrain is a public repository designed to be tended by **GitHub Copilot cloud agent** inside **GitHub Actions**.

Instead of running forever on a laptop, the project wakes up on a scheduled GitHub Actions run once per hour. That workflow creates a single bounded task, assigns it to Copilot, and lets Copilot open a pull request with the next small improvement.

The goal is simple: get the loop running, keep the project public, and avoid needing to babysit it.

## Operating model

- **GitHub Actions is the runtime.** The repository uses a scheduled workflow instead of a local supervisor loop.
- **Copilot is the agent.** Each hourly cycle creates one issue and assigns it to Copilot cloud agent.
- **Pull requests are the unit of change.** Copilot works on a branch and raises a PR for review.
- **The loop stays bounded.** The hourly workflow skips creating a new task if a previous hourly task is still open.
- **Documentation stays aligned with reality.** Comments and docs should describe the GitHub-hosted automation, not a laptop-based Claude runtime.

## Repository files that matter

- `/README.md` — operator overview.
- `/PURPOSE.md` — the durable purpose of the project.
- `/design/SPEC.md` — the operating spec for the GitHub Actions + Copilot loop.
- `/.github/copilot-instructions.md` — repository instructions for Copilot cloud agent.
- `/.github/workflows/hourly-copilot-cycle.yml` — creates the hourly Copilot task.
- `/.github/workflows/copilot-setup-steps.yml` — prepares Copilot's GitHub Actions workspace.

## One-time setup

1. Enable **Copilot cloud agent** for the repository.
2. Create a **`copilot` environment** in the repository settings.
3. Add an environment secret named **`COPILOT_AGENT_TOKEN`** to that environment.
   - Use a **user token** that can assign issues to Copilot.
   - A fine-grained PAT should have at least:
     - metadata: read
     - actions: read/write
     - contents: read/write
     - issues: read/write
     - pull requests: read/write
4. Merge the workflow files in this repository into the default branch.
5. Optionally run **Hourly Copilot Cycle** manually once from the Actions tab to confirm the automation is healthy.

## Hourly lifecycle

1. GitHub Actions starts `Hourly Copilot Cycle` on the hour.
2. The workflow ensures the `copilot-hourly` label exists.
3. If an older hourly issue is still open, the workflow exits without creating a second task.
4. Otherwise, the workflow creates a new issue and assigns it to Copilot.
5. Copilot works in its own GitHub Actions-backed environment, using this repository's instructions and setup steps.
6. Copilot opens a pull request with the next small improvement.

## What "success" looks like

A healthy PinkyBrain repository should:

- keep producing small, reviewable pull requests,
- prefer durable maintenance and documentation improvements over flashy churn,
- avoid piling up overlapping agent tasks,
- stay understandable to a human who returns later,
- and require little more than occasional review and merge decisions.

## Notes

- Public repositories can use GitHub Actions without worrying about private-runner billing for this automation pattern.
- Scheduled workflows are not guaranteed to fire at the exact minute, so the project should be tolerant of small delays.
- If you change how the loop works, update the workflows, comments, and docs together.
