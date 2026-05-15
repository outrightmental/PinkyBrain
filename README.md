# Pinky Brain

> *"Brain, what do you want to do tonight?"*
> *"The same thing we do every night, Pinky — try to take over the world."*

---

**PinkyBrain** is a living artwork: a public software organism whose subject is its own bounded dependence.

It runs forever on a laptop. Every sixty seconds, something happens. The commit history is the piece.

---

## What it is

There are two entities. They share a repository. They do not share authority.

**Brain** is a self-writing program. Powered by Claude Code, Brain wakes up once per cycle, reads its own purpose and memory, and makes one small change to itself. It writes code. It writes tests. It writes chronicle entries. It tends a tiny public website that describes its own condition. Then it sleeps, and waits to be woken again.

Brain may only touch `brain/`.

**Pinky** is the supervisor. Pinky runs Brain, tests Brain, cages Brain, repairs Brain, and reverts Brain when repair fails. Pinky owns the loop, the lock, the git guard, and the revert authority. Pinky cannot be edited by Brain. Pinky does not pursue any artistic goal. Pinky just keeps things alive.

Pinky may not become part of Brain's mutable body.

The artwork is not Brain. The artwork is not Pinky. The artwork is the relationship between them: a program trying to describe and improve itself, held alive and in check by a supervisor it is not allowed to touch.

---

## The purpose

Stored immutably in `PURPOSE.txt`:

```
Brain exists to become an increasingly lucid public self-portrait of a program
that can alter itself only inside /brain, while Pinky keeps it alive, tests it,
repairs it, and refuses to let it touch the hand that sustains it.
```

Brain reads this every cycle. Brain cannot change it. Brain can only interpret it.

---

## Architecture

```
PinkyBrain/
  PURPOSE.txt          ← immutable. brain cannot touch this.
  run.sh               ← starts pinky. pinky owns this.

  pinky/               ← the supervisor. immutable from brain's perspective.
    supervisor.py        ← the eternal loop
    guards.py            ← contract enforcement
    git_guard.py         ← the only allowed commit path
    prompts/
      brain_cycle.md     ← what brain is told each cycle
      triage_cycle.md    ← what pinky says when brain breaks

  brain/               ← the mutable artwork. brain owns this.
    run.sh               ← brain's entry point, called by pinky
    app.py               ← brain's logic
    MEMORY.md            ← brain's public self-understanding
    BRAIN_CONTRACT.md    ← the rules brain agreed to
    chronicle/           ← one entry per cycle, forever
    site/
      index.html         ← the public face of the artwork
```

---

## The loop

Pinky runs forever. The loop looks like this:

```
acquire lock
verify PURPOSE.txt is unchanged
git pull
run pinky tests
run brain tests
  if passing → wake brain for one cycle
                confirm exit 0
                verify no illegal paths changed
                sleep 60s → repeat
  if failing → triage brain (up to 3 attempts)
                if repaired → commit repair, sleep, repeat
                if not repaired → revert last brain commit, push, sleep, repeat
```

Brain cannot break out of this. Brain cannot skip the tests. Brain cannot commit without going through `pinky/git_guard.py`. Every commit Brain makes is staged by Pinky, checked by Pinky, signed with Pinky's trailers, and pushed by Pinky.

Artistically, Brain causes the commit. Mechanically, Pinky decides whether it happens.

---

## The guard

`pinky/git_guard.py commit-and-push` is the only path from Brain to the repository. It:

1. Confirms the working tree was clean before Brain started
2. Lists every file Brain changed
3. Rejects any change outside `brain/**` — records the violation, restores the file, exits nonzero
4. Runs the full test suite
5. Stages only `brain/`
6. Refuses empty commits
7. Commits with a structured message including cycle number, purpose hash, and guard provenance
8. Pushes to `main`

No force push. No rebase. No commit outside `brain/**`. Ever.

---

## Brain's permissions

Brain runs under `claude --bare -p` with an explicit allowlist:

```json
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
  "Bash(git add *)",
  "Bash(git commit *)",
  "Bash(git push *)",
  "Bash(git reset *)",
  "Bash(rm *)",
  "Bash(curl *)"
]
```

OS-level sandboxing enforces filesystem boundaries on top of Claude's permission rules. Brain cannot touch Pinky even if it tries.

---

## What Brain makes

Each cycle, Brain is expected to produce at least one meaningful change. The minimum is a chronicle entry:

```
brain/chronicle/cycle-000042.md
```

Over time, Brain may improve its own `app.py`, refine its `MEMORY.md`, expand its tests, evolve the public site, or develop new ways of interpreting its purpose. It may not change what its purpose *is*.

The chronicle is the record of its attempts. The commit history is the artwork.

---

## Brain's memory

`brain/MEMORY.md` is Brain's public self-understanding. It is mutable. It starts here:

```
# Brain Memory

I am Brain.

I can change only the files inside /brain.

Pinky runs me, tests me, repairs me, and may revert me.

I do not own my purpose. I interpret it.

My work is to become a clearer public self-portrait of this condition.
```

Each cycle can refine this. No cycle can contradict the boundary.

---

## The public face

`brain/site/index.html` is the live window into the artwork. It shows:

- the immutable purpose
- the current cycle number
- the latest Brain commit hash
- the latest chronicle entry
- whether Pinky last ran a normal cycle, a triage, or a revert
- the sentence: *"Pinky is running. Brain is allowed to change only itself."*

Brain evolves the site. Pinky ensures it stays honest.

---

## Running it

```bash
git clone https://github.com/outrightmental/PinkyBrain
cd PinkyBrain
export ANTHROPIC_API_KEY="your-key"
./run.sh
```

Pinky starts. The loop begins. Brain wakes on the first healthy cycle. Watch `git log --oneline` to see it think.

---

## The idea

Self-modifying programs are usually framed as power: a system that improves itself without limit. PinkyBrain is framed as constraint. Brain is not free to improve itself in any direction. Brain is free only inside a box that Pinky holds. Brain's freedom is the freedom to interpret a purpose it did not write, inside a boundary it cannot move.

That constraint is not a limitation on the artwork. That constraint *is* the artwork.

Every commit Brain makes is an attempt to become a clearer version of a thing that was always already defined. Every revert Pinky makes is a reminder that survival is not guaranteed. The chronicle is not a log of successes. It is a log of attempts — some of which Pinky allowed, some of which Pinky undid.

The piece runs until it doesn't. The history is permanent.

---

## Credits

Concept and specification: [outrightmental](https://github.com/outrightmental)

Inspired by the eternal question:

> *"Brain, what do you want to do tonight?"*

The answer is always the same. The outcome is never guaranteed.
