# PinkyBrain specification

## 1. Goal

PinkyBrain should operate as a low-maintenance public repository that uses GitHub Actions for scheduling and GitHub Copilot cloud agent for implementation work.

The desired behavior is:

- GitHub Actions wakes the project once per hour.
- The workflow creates one bounded task.
- Copilot works that task in GitHub's hosted environment.
- Copilot opens a pull request with a small improvement.
- The system avoids creating overlapping hourly tasks.

## 2. Core architecture

### Runtime

GitHub Actions is the runtime. This project should no longer be documented as a laptop process, daemon, or local Claude loop.

### Agent

GitHub Copilot cloud agent is the coding agent. It should receive repository context from the issue body, repository documentation, and `.github/copilot-instructions.md`.

### Scheduler

A scheduled workflow, `/.github/workflows/hourly-copilot-cycle.yml`, runs once per hour and may also be triggered manually.

### Workspace preparation

`/.github/workflows/copilot-setup-steps.yml` customizes the Copilot cloud agent environment before Copilot starts work.

## 3. One-time repository setup

The operator should do this once:

1. Enable Copilot cloud agent for the repository.
2. Create a repository environment named `copilot`.
3. Add an environment secret named `COPILOT_AGENT_TOKEN`.
4. Keep the workflow files on the default branch so both the scheduler and Copilot setup steps can run.

The token must be a user token that is allowed to assign issues to Copilot cloud agent.

## 4. Hourly workflow contract

The hourly workflow should:

1. Run on `cron: '0 * * * *'` and on manual dispatch.
2. Use concurrency so overlapping scheduler runs do not create duplicate work.
3. Ensure the `copilot-hourly` label exists.
4. Check for an existing open hourly issue.
5. Exit cleanly if an hourly issue is already open.
6. Otherwise create a new issue and assign it to Copilot.
7. Provide Copilot with instructions to make one small, durable improvement and open a pull request.

## 5. Task shape for Copilot

Each scheduled task should bias Copilot toward work that keeps the repository healthy over time.

Preferred work:

- clarifying documentation,
- simplifying repository structure,
- improving workflows,
- tightening automation safety,
- adding lightweight validation,
- and making the project easier to leave unattended.

Avoid:

- large rewrites,
- adding external infrastructure,
- introducing paid dependencies or services,
- changing repository settings from code,
- and opening more than one line of work per hourly cycle.

## 6. Copilot repository instructions

Repository-level instructions should tell Copilot to:

- choose the smallest worthwhile improvement,
- keep changes reviewable,
- update related comments and docs when behavior changes,
- preserve the hourly automation,
- and avoid depending on manual follow-up unless absolutely necessary.

## 7. Validation expectations

When Copilot changes workflows or automation, it should at least validate syntax and keep workflow comments accurate.

Because the repository currently has no application code or test suite, the main safety mechanism is keeping the automation simple, explicit, and well-documented.

## 8. Human maintenance model

The human operator should only need to:

- enable Copilot and add the required token once,
- occasionally review and merge pull requests,
- refresh the token if it expires,
- and update the high-level purpose if the project direction changes.

That is the design target: something that can keep moving without constant supervision.
