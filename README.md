# Project template: spec-driven development on herdr

A starting point for a coding project where features are specified before they are built, by agent
CLIs (Claude Code, Codex, ...) working in parallel, one git worktree per feature, under
[herdr](https://herdr.dev): a terminal workspace manager that opens worktrees as workspaces and runs agent
CLIs in tabs, which the launcher drives through herdr's CLI. Everything harness-related lives in `harness/`;
`AGENTS.md` is the map every agent reads; the rest of the repo is your project.

## Prerequisites
- git and Python 3.9+ (the launcher is stdlib-only); [uv](https://docs.astral.sh/uv/) for the shipped `test_cmd`
- [herdr](https://herdr.dev/docs/install/): `curl -fsSL https://herdr.dev/install.sh | sh`
- The agent CLIs named in `harness/config.json`, logged in
- A read of what the roles' permissions mean, `harness/help/config.md` (the whole trust model:
  `harness/docs/sdd.md` §10)

## New project
1. GitHub's "Use this template", or `git clone <this repo> myproject && cd myproject && rm -rf .git &&
   git init -b main && git add -A && git commit -m "start from template"`. `LICENSE` is the template's:
   replace or delete it.
2. Edit `harness/config.json`: agent per role, gates, commands, files to copy into worktrees. Every
   key is explained in `harness/help/config.md`.
3. Open the repo in herdr as a workspace, start your agent in it (the driver's prompts reach only an agent
   started inside herdr) and ask for a feature. Before the first one it runs `doctor` (the setup, tools,
   config, and `test_cmd` on the main checkout, so a red baseline is not blamed on the first implementer),
   fills in `AGENTS.md` §1 with you, and waits for your yes; from then on `start` checks the setup itself.
   A repo with no code yet is fine: the first feature sets the project up. `doctor --models` also sends each
   Claude Code and Codex role one tiny prompt with its model, so a wrong model id fails there (a few tokens).

## Daily flow
You, to the agent on `main`: "Build email+password login with a cookie session." It runs `start login`
(worktree, branch, `harness/features/login/`, a herdr workspace for the worktree, and the driver, a launcher
process with no model, in its own tab), and is free again. The driver runs the spec author, spec reviewer,
implementer and reviewer in their own tabs, commits what each wrote, merges `main` in and tests before every
review of the code, and sends what is missing back. With the shipped config (`gates` set to `human`) it stops
twice, for you to read the spec and to merge, and prompts your agent with `harness: login ...`:
```
python3 harness/harness.py status          # needs you: spec_ready → read spec.md, then
python3 harness/harness.py approve login   # the driver continues
python3 harness/harness.py status          # needs you: reviewed
python3 harness/harness.py merge login
```
Type those, or tell the agent "approved" / "merge it". Feedback at a gate: `drive login "<feedback>" --tab`
(`--tab` runs the driver in the feature's workspace and returns). Set a gate to `auto` and the driver runs
it itself (`harness/help/config.md`, "Fully automatic"). Several features run at once, one driver each;
`merge` is local (`harness/docs/sdd.md` §10). Each feature's state shows in herdr's sidebar once you add two
lines to its config (`harness/help/config.md`).

## Layout
```
AGENTS.md                      the map (read natively by Claude Code, Codex)
harness/
  harness.py                   the launcher: --help lists every command
  selftest.py                  the launcher's own tests, only for changes to harness/: python3 harness/selftest.py (ruff.toml: its lint settings)
  config.json                  agent per role, gates, commands, files copied into worktrees
  help/config.md               every key of config.json
  docs/sdd.md                  the rules: lifecycle, file formats, EARS, traceability, what check enforces
  roles/*.md                   job of orchestrator, spec_author, spec_reviewer, implementer, reviewer
  features/_template/          copied by start
  features/<slug>/             spec.md, tasks.md, spec_review.md, log.md, review.md, questions.md
```
