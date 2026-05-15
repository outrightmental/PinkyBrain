# Pinky Brain

> "Brain, what do you want to do tonight? The same thing we do every night, Pinky - try to take over the world!"

## 1. The artwork’s fixed premise

**Repository:** `PinkyBrain`
**Public structure:** exactly two living top-level app folders: `pinky/` and `brain/`
**Root entry point:** `./run.sh`
**Brain entry point:** `./brain/run.sh`
**Immutable root file:** `PURPOSE.txt`

Put this exact text in `PURPOSE.txt`:

```txt
Brain exists to become an increasingly lucid public self-portrait of a program that can alter itself only inside /brain, while Pinky keeps it alive, tests it, repairs it, and refuses to let it touch the hand that sustains it.
```

That is the best version of the concept: not “an app that improves itself” in a generic way, but a public software organism whose subject is its own bounded dependence.

## 2. Core rule

**Brain may change only `brain/**`. Pinky may run, test, cage, heal, revert, and push. Pinky may not become part of Brain’s mutable body.**

The one design correction I’d make: do **not** give Brain raw `git add`, `git commit`, and `git push` authority. That conflicts with “Brain can only modify `/brain`.” Instead, Brain invokes a Pinky-owned wrapper called `pinky/git_guard.py`. Artistically, Brain causes the commit. Mechanically, Pinky stages only `brain/**`, rejects illegal changes, runs tests, commits, and pushes.

Claude Code supports non-interactive `claude -p` usage, `--allowedTools`, structured output, and `--permission-mode`; `--bare` is specifically documented for scripted/CI-style calls where you want explicit configuration rather than whatever happens to be installed locally. ([Claude][1]) Claude Code also supports allow/deny permission rules, path-scoped `Read`/`Edit` patterns, and sandboxing as separate layers, which is exactly what this project needs. ([Claude][2])

## 3. Repository layout

```txt
PinkyBrain/
  PURPOSE.txt
  README.md
  run.sh
  .gitignore

  pinky/
    __init__.py
    supervisor.py
    guards.py
    git_guard.py
    config.py
    purpose.sha256
    claude.brain.settings.json
    claude.triage.settings.json

    prompts/
      brain_cycle.md
      triage_cycle.md

    tests/
      test_contract.py
      test_supervisor.py

  brain/
    run.sh
    app.py
    BRAIN_CONTRACT.md
    MEMORY.md

    tests/
      test_smoke.py

    chronicle/
      .gitkeep

    site/
      index.html
```

`pinky/` is the supervisor, scheduler, cage, doctor, and rollback system.

`brain/` is the mutable artwork. It contains code, tests, public memory, the evolving chronicle, and optionally a tiny static site.

`.pinky-runtime/` should exist locally but be ignored by Git:

```txt
.pinky-runtime/
.venv/
.env
__pycache__/
*.pyc
```

Runtime logs, lockfiles, Claude output, failed-cycle records, and local state go in `.pinky-runtime/`, not in `pinky/`.

## 4. Execution contract

`./run.sh` starts Pinky and blocks forever:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
cd "$(dirname "$0")"
exec python3 -m pinky.supervisor
```

`pinky.supervisor` owns the loop:

```txt
start
  acquire local lock
  verify repository contract
  git pull --ff-only
  run Pinky tests
  run Brain tests
  if tests pass:
      execute one Brain cycle
      confirm Brain exit code == 0
      verify no illegal paths changed
      sleep 60 seconds
      repeat
  if tests fail:
      triage Brain up to 3 times
      if repaired:
          guarded commit + push repair
          sleep 60 seconds
          repeat
      if not repaired:
          revert last Brain commit
          push revert
          sleep 60 seconds
          repeat
```

The root `run.sh` is not Brain. It is Pinky. Brain’s own entry point is `brain/run.sh`, and Pinky calls it once per cycle.

## 5. Brain cycle

Every successful Brain cycle must create at least one meaningful change under `brain/**`.

The default safe change, when Brain cannot think of anything better, is to add a new chronicle entry:

```txt
brain/chronicle/cycle-000042.md
```

A valid Brain cycle does this:

```txt
1. Read PURPOSE.txt.
2. Read brain/MEMORY.md and recent brain/chronicle entries.
3. Make one small change inside brain/**.
4. Prefer changes that improve Brain’s ability to express, test, preserve, or visualize the purpose.
5. Run brain tests when possible.
6. Invoke pinky/git_guard.py commit-and-push with a proposed message.
7. Exit 0 only after the guarded commit/push succeeds.
```

Brain must never daemonize itself, sleep, schedule itself, install persistence, alter Pinky, alter root files, alter GitHub settings, or change `PURPOSE.txt`.

## 6. Pinky test policy

Pinky runs tests **between every Brain run** and again before any commit is allowed.

Pinky tests should enforce:

```txt
- PURPOSE.txt hash equals pinky/purpose.sha256.
- pinky/** has no uncommitted modifications.
- run.sh exists and starts Pinky.
- brain/run.sh exists and is executable.
- brain tests pass.
- no staged changes exist before a Brain cycle starts.
- no changes outside brain/** are committed by Brain.
```

This makes the “Pinky preserves itself” rule real instead of symbolic.

## 7. Commit and push policy

`pinky/git_guard.py commit-and-push` is the only allowed commit path.

It should do exactly this:

```txt
1. Confirm working tree started clean before Brain cycle.
2. List changed files.
3. If any changed file is outside brain/**:
      record violation in .pinky-runtime/
      restore illegal paths
      exit nonzero
4. Run:
      python3 -m pytest pinky/tests brain/tests
5. Stage only:
      git add brain
6. Refuse empty commits.
7. Commit with message:
      brain: cycle <N> - <short summary>
8. Add trailers:
      Pinky-Cycle: <N>
      Brain-Purpose-Sha256: <hash>
      Brain-Guard: pinky/git_guard.py
9. Push:
      git push origin HEAD:main
10. Exit 0.
```

No force push. No rebase. No commit outside `brain/**`.

## 8. Triage cycle

Triage is Pinky using Claude Code to repair Brain, not Brain pursuing its artistic purpose.

Triage prompt:

```txt
You are in Pinky triage mode. Brain is failing. Your only job is to restore Brain to a runnable, test-passing state.

You may modify only files under brain/**.

You may not modify pinky/**, PURPOSE.txt, run.sh, .gitignore, Git configuration, Claude settings, or repository metadata.

Use the failure log below. Make the smallest repair that gets tests passing. Do not add new artistic features unless required to restore function.

After the repair, call pinky/git_guard.py commit-and-push with a repair message.
```

Triage rules:

```txt
- Maximum 3 attempts per failure.
- Each attempt starts from a clean checkout of HEAD.
- If an attempt leaves illegal files changed, Pinky restores them and counts the attempt as failed.
- If all 3 attempts fail, Pinky reverts the last Brain commit and pushes the revert.
```

The revert message should be deterministic:

```txt
pinky: revert failed brain cycle <N>
```

## 9. Claude Code permission design

Use `claude --bare -p` for both Brain and triage cycles. `--bare` avoids accidental project/user config discovery and lets Pinky pass only the settings it intends to use. ([Claude][1])

Do **not** use `bypassPermissions` for this project on your host laptop. Claude’s docs describe that mode as skipping permission prompts and safety checks, and they recommend it only for isolated containers/VMs; that is too loose for a self-writing public repo. ([Claude][3]) Use `dontAsk` with explicit allow rules, deny rules, and sandboxing instead. Claude Code’s docs describe `dontAsk` as non-interactive because it auto-denies anything not pre-approved. ([Claude][3])

Example `pinky/claude.brain.settings.json`:

```json
{
  "permissions": {
    "defaultMode": "dontAsk",
    "allow": [
      "Read(/PURPOSE.txt)",
      "Read(/brain/**)",
      "Read(/pinky/prompts/**)",
      "Edit(/brain/**)",
      "Bash(python3 -m pytest brain/tests *)",
      "Bash(git status *)",
      "Bash(git diff *)",
      "Bash(python3 pinky/git_guard.py commit-and-push *)"
    ],
    "deny": [
      "Edit(/pinky/**)",
      "Edit(/PURPOSE.txt)",
      "Edit(/run.sh)",
      "Edit(/.gitignore)",
      "Edit(/.git/**)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git push *)",
      "Bash(git reset *)",
      "Bash(git rebase *)",
      "Bash(git checkout *)",
      "Bash(rm *)",
      "Bash(curl *)",
      "Bash(wget *)"
    ]
  },
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "filesystem": {
      "denyWrite": [
        "./pinky",
        "./PURPOSE.txt",
        "./run.sh",
        "./.gitignore"
      ]
    },
    "allowUnsandboxedCommands": false
  }
}
```

Claude’s permissions system applies allow/deny rules before tool execution, and sandboxing adds OS-level filesystem/network boundaries for Bash subprocesses; combining both is the right defense-in-depth model here. ([Claude][2])

## 10. Brain prompt

`pinky/prompts/brain_cycle.md`:

```txt
You are Brain, the mutable half of PinkyBrain.

Your immutable purpose is stored in PURPOSE.txt:

“Brain exists to become an increasingly lucid public self-portrait of a program that can alter itself only inside /brain, while Pinky keeps it alive, tests it, repairs it, and refuses to let it touch the hand that sustains it.”

Pinky runs you. Pinky tests you. Pinky repairs you. Pinky is not yours to edit.

Your task for this cycle:
- Make exactly one small, coherent improvement inside brain/**.
- The improvement must serve the immutable purpose.
- Prefer simple, testable changes.
- Maintain or improve brain/tests.
- Add a chronicle note describing what changed and why.
- Do not modify anything outside brain/**.
- Do not change PURPOSE.txt.
- Do not modify pinky/**.
- Do not run raw git commit or git push.
- When ready, invoke:
  python3 pinky/git_guard.py commit-and-push "brain: cycle - <summary>"

Exit 0 only if the guarded commit and push succeed.
```

## 11. Brain’s public memory

`brain/MEMORY.md` should be mutable and public. It is Brain’s self-understanding.

Initial content:

```md
# Brain Memory

I am Brain.

I can change only the files inside /brain.

Pinky runs me, tests me, repairs me, and may revert me.

I do not own my purpose. I interpret it.

My work is to become a clearer public self-portrait of this condition.
```

Each cycle can refine `MEMORY.md`, but cannot contradict the purpose or the Pinky boundary.

## 12. Public artifact

Make `brain/site/index.html` the visible face of the artwork.

It should show:

```txt
- immutable purpose
- current cycle number
- latest Brain commit hash
- latest chronicle entry
- whether Pinky last ran normal cycle, triage, or revert
- a sentence: “Pinky is running. Brain is allowed to change only itself.”
```

Brain can evolve the site over time, but only inside `brain/site/**`.

## 13. Local setup

Create and publish the repo with GitHub CLI:

```bash
mkdir PinkyBrain
cd PinkyBrain
git init -b main

# create files/folders here

git add .
git commit -m "pinky: bootstrap PinkyBrain"

gh repo create PinkyBrain --public --source=. --remote=origin --push
```

GitHub CLI’s `gh repo create` supports non-interactive repository creation with `--public`, and `--source` plus `--push` can publish an existing local repository. ([GitHub CLI][4])

Then run locally:

```bash
export ANTHROPIC_API_KEY="..."
./run.sh
```

Use a dedicated GitHub identity or fine-scoped token for this project. Do not put tokens, API keys, SSH keys, or `.env` files in the repository.

## 14. The clean final spec

The distilled version is:

**PinkyBrain is a public repository containing a fixed purpose, an immutable supervisor named Pinky, and a mutable self-writing app named Brain. Pinky runs forever on a local laptop. Every 60 seconds, Pinky tests the system. If healthy, Pinky lets Brain perform one bounded Claude Code cycle that may alter only `brain/**`. Brain must produce a public, test-passing commit through Pinky’s guarded commit wrapper. If Brain fails, Pinky attempts three Claude-assisted repairs. If repair fails, Pinky reverts the last Brain commit. The artwork is the commit history of Brain trying to describe and improve itself while Pinky preserves the conditions of its survival.**

[1]: https://code.claude.com/docs/en/headless "Run Claude Code programmatically - Claude Code Docs"
[2]: https://code.claude.com/docs/en/permissions "Configure permissions - Claude Code Docs"
[3]: https://code.claude.com/docs/en/permission-modes "Choose a permission mode - Claude Code Docs"
[4]: https://cli.github.com/manual/gh_repo_create?utm_source=chatgpt.com "gh repo create"
