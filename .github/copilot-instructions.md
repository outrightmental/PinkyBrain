# Copilot instructions for PinkyBrain

This repository is intentionally small and should stay low-maintenance.

## Working model

- GitHub Actions runs the project once per hour.
- Copilot cloud agent should make one small, durable improvement per task.
- Prefer changes that make the repository easier to understand, safer to automate, or easier to leave alone.

## Change boundaries

- Keep pull requests narrow and reviewable.
- Update documentation and workflow comments whenever behavior changes.
- Prefer improving existing files over adding new moving parts.
- Do not introduce paid services, external infrastructure, or credentials-in-code.
- Do not create overlapping automation loops.

## Priority order

1. Keep the hourly automation healthy.
2. Keep the repository self-explanatory.
3. Reduce maintenance burden.
4. Add only the smallest useful improvement.

If a task is open-ended, choose the smallest high-leverage change and stop once it is complete.
