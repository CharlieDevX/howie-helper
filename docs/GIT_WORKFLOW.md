# Howie Helper — Git Workflow

A simple, repeatable git workflow for work with Claude Code, set up so other
people can use the live app without you disturbing them while you work. You don't
run git commands yourself — you tell Claude Code what you want and it runs them.
This doc is the playbook so the steps stay consistent.

> **⛔ The one hard rule: nothing crosses into `main` or `stable` except through a
> Pull Request that Charlie reviews and merges on GitHub.** Claude Code never runs
> a local `git merge` into `main`/`stable` and never pushes directly to them. It
> works on a feature/fix branch, pushes that branch, and opens a PR.

## The big picture (three layers)

- **`stable` = what people actually use.** GitHub Pages deploys the `stable`
  branch. This is the version running on everyone's phone. It **only** changes
  when a release PR is merged — so you can tinker all day without affecting anyone
  mid-shift.
- **`main` = your working/integration version.** Finished, tested features land
  here **via PR**. It's your "next release in progress." Not live until you
  promote it.
- **Feature branches = your workbench.** Every new tool or change happens on its
  own branch off `main`. Experiment freely; if it breaks, nothing downstream is
  touched.

```
stable (LIVE — what people run) ──●───────────────────●────▶  (changes only when a release PR merges)
                                   ▲  release PR ───────▲
main (your dev version) ──●────────●──────────●─────────●──▶
                           \      /  PR merged when a feature is done
   feature/...      ●──●──●─●    (build + test here)
```

The key idea: **two safety gaps.** A broken feature can't hurt `main`, and an
unfinished `main` can't hurt the `stable` version other people are using.

## The everyday loop (building)

For each new tool or change:

1. **Start a branch.** "Make a new branch off main for the prep tracker."
   → `feature/prep-tracker`.
2. **Build + commit.** "Build it per the spec, committing as you go."
3. **Test it.** "Serve it locally so I can try it on my phone."
4. **Push + open a PR into main.** "Looks good — push it and open a PR into main."
   → Charlie reviews and merges on GitHub. (Still not live.)
5. **Clean up.** "Delete the branch." (After the PR is merged.)

You can repeat this as much as you want. None of it reaches other users yet —
they're on `stable`. And nothing lands on `main` until Charlie merges the PR.

## Releasing (making it live for everyone)

When `main` is in a state you're happy to put in front of people:

> "Open a release PR from main into stable." (a.k.a. "cut a release")

→ Claude Code pushes and opens a PR from `main` into `stable`; Charlie reviews and
merges it on GitHub; GitHub Pages redeploys; within a minute everyone's app
updates. This is the **only** moment the live version changes, and it's always
deliberate. (No local merge into `stable`, no direct push — the release is the
merged PR.)

Tip: cut a release at a calm time (not mid-rush), so if anything looks off you
have room to fix it.

## Branch names

- `feature/<thing>` for new tools/features — `feature/nightly-inventory`,
  `feature/drawer-cash`, `feature/home-screen`.
- `fix/<thing>` for bug fixes — `fix/pop-loose-counter`.

One branch = one focused piece of work.

## Commit messages

Short, present-tense, says what changed (Claude Code handles these):

- `Add Drawer Cash placeholder tile to home screen`
- `Fix Pop loose-crate total math`

Lots of small commits beat one giant one — easier to undo.

## Pushing = your backup

"Push" copies commits up to GitHub. Push often — at least when you stop for the
day. Pushed = backed up off your computer. A good habit: "commit and push" when
wrapping up. (Pushing your feature branch does **not** change what users run —
only a merged release PR into `stable` does.)

## If a release goes wrong

Because users are on `stable`, a bad release is fixable fast. On GitHub, Charlie
can **revert the release PR** (or merge a quick revert PR):

> "Open a PR to revert the last release on stable."

→ Rolls the live version back to the previous good one while you sort out the fix
on `main`. Nobody's stuck on a broken version.

## Safety net / undo

Handy things to ask Claude Code anytime:

- "What branch am I on / what's the status?"
- "Show me what changed."
- "Undo my uncommitted changes."
- "Revert the last commit on this branch." (Reverting something already merged into
  `main`/`stable` goes through a revert PR Charlie merges.)

## First-time setup (one time — ask Claude Code)

If the repo's already started, just ask for the parts you're missing:

1. In the **app project folder**: "Initialize a git repo, add a `.gitignore`, and
   make the first commit on `main`." (Already done if Claude Code is building.)
2. "Create a `stable` branch from `main`."
3. "Create a **private** GitHub repo and push both `main` and `stable`."
4. "Turn on **GitHub Pages**, deploying from the **`stable`** branch" — that gives
   the app its phone URL, serving the stable version.

> Note: if Pages is currently set to deploy from `main`, switch it to `stable` so
> your day-to-day work on `main` stops auto-publishing to users.

## Two-folder reminder

- **This folder (`HungryHowiesCoWorkFiles`)** = planning + specs. Not the code repo.
- **The app project folder** = the git repo Claude Code builds in.

(Optional: keep a copy of the specs in the repo under `/docs` for reference.)
