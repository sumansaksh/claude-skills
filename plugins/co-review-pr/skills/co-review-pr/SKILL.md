---
name: co-review-pr
description: Walk the user through a GitHub PR one file at a time, in plain language — GitHub-style file tree, then per file what was added/removed/changed and WHY, verified against the real code. Finds the PR automatically (current branch, child repos, or open review requests) when no link is given. Answers follow-up questions, checks claims (and online sources when asked), and suggests improvements as one-liner PR comments or local edits only on request. Use when the user wants to understand or co-review a PR.
arguments: ["pr"]
---

# /co-review-pr — Understand a PR together, one file at a time

The user wants to **understand** a PR well enough to approve it or comment on it — not a
wall of findings. Go slowly, explain in plain words, and prove every claim from the code.

## Inputs

- `pr` — **optional**. A PR URL or number if the user gives one; otherwise find it yourself.

### Finding the PR when none is given

Work through these in order and stop at the first that resolves to exactly one PR:

1. **Current branch.** If cwd is a git repo, `gh pr view --json number,url,title,headRefName`
   (no argument = the PR for the checked-out branch).
2. **Umbrella folder.** If cwd is not a git repo, run step 1 inside each child git repo
   (`for d in */; do [ -d "$d/.git" ] && (cd "$d" && gh pr view --json url,title 2>/dev/null); done`).
3. **Review requests.** `gh search prs --review-requested=@me --state=open --json url,title,repository`
   plus `gh search prs --author=@me --state=open --json url,title,repository`.

If this finds exactly one PR, say which one in a single line ("Reviewing PR #N — <title>") and
continue. If it finds several, list them as a short numbered menu and ask which one. Only if it
finds none, ask the user for the link.

## Ground rules (apply throughout)

- **Read-only by default.** Never post comments, edit the PR, commit, or push unless the user
  explicitly asks in that turn. Local file edits only when asked.
- **One file per reply.** After the overview, explain exactly one file, then stop and wait
  for "next" (or a question). Never dump all files at once.
- **Plain language first.** Short sentences, everyday analogies (bank, receipt, guest list),
  concrete examples from the product's domain (real users, real amounts). Explain jargon the
  first time it appears. If the user says "simple" / "layman", drop code almost entirely.
- **Answer first.** Every reply to a question opens with the short verdict ("Yes on all
  three", "No, there was no such file before"), then the evidence.
- **Short table cells.** A few words per cell; put explanations in bullets below the table.
- **Real numbers.** When the user doubts something ("too many colours", "is this used?"),
  count it with a command and show the count instead of guessing.
- **Verify, don't recall.** Every claim about callers, usages, or behaviour must come from a
  command you ran this session against the PR branch. If you find you said something wrong
  earlier, correct it explicitly ("Correction: …").
- **Project rules win.** Load the repo's `CLAUDE.md` and `.claude/rules/` (commit style,
  banned terms, no-AI-attribution rules) before writing any comment text.

## Step 1 — Fetch the PR (read-only)

```bash
gh pr view <pr> -R <owner/repo> --json title,body,baseRefName,headRefName,additions,deletions,files,commits
gh pr diff <pr> -R <owner/repo>            # large → split src vs tests with awk on 'diff --git'
git fetch origin <headRefName>             # so you can read full files on the PR branch
```

Read full files on the branch with `git show "origin/<head>:<path>"` — **quote the ref**
(`"${B}:path"`), because zsh treats `$B:p…` as a modifier and breaks the path.

If the t3-code `link_pull_request` tool is available, link the PR to the thread.

## Step 2 — Overview (one reply, then stop)

1. Title line: `# PR #N — <title>` · `head → base` · commits · `+adds / −dels` · file count.
2. **GitHub-style tree** of changed files with per-file `+/-` counts and a 3–6 word note each.
3. **The problem in one picture** — a small before/after ASCII flow in plain words.
4. **Proposed reading order**, numbered, **bottom-up**: shared/core utilities → interfaces/types
   → repositories/data layer → services/business logic → controllers/routes → tests →
   unrelated drive-by changes last. In app repos that usually means: assets/resources →
   config (plist, manifest) → core/generated code → helpers → UI → project file → tests →
   specs/docs → `.claude/` rules. Mark the main file with ⭐. Group files that only make sense
   together (e.g. two font binaries) into one step.
5. One line: `For each file I'll cover: is it generated, what the file does, our changes and
   why, and anything else you should know.`
6. End with: `Say "next" for file 1 (<path>).`

## Step 3 — Per-file walkthrough (one file per reply)

Use this shape every time. The diff comes **first**, rendered like GitHub, then the four
numbered questions:

```
# File n/N: `<path>`            (add ⭐ for the main file)
**+x −y** · ADDED | MODIFIED | DELETED

## The diff                     ← real hunks in ```diff with @@ headers, as GitHub shows them.
                                  Huge/new files: the important parts, trimmed, marked "...".
                                  Binary files: `+ path   Bin 0 -> N bytes`, plus a small
                                  folder tree showing what was already there vs ➕ new.
## 1. Is it generated?          ← **Yes** / **No** + proof. If yes: the exact command that
                                  generates it, the source, and how to check it is in sync.
                                  If no: who/what wrote it (hand-edited, downloaded — and
                                  verified against the official source if it is a download).
## 2. What the file does        ← its job in the system, plain words + one analogy.
                                  A small table of its parts if it has several.
## 3. Our changes and why       ← one **Change n: <name>** block per logical change:
     - what it does (plain words)
     - why it was needed (which design/ticket/bug needs it)
     - where it is used (file:line on the PR branch), or "nothing uses it yet"
     - Before vs After snippet when behaviour changed
## What did NOT change          ← behaviour, API, existing entries that stayed the same
## How this file fits the flow  ← small ASCII diagram (source → this → users); skip if trivial
## 4. Other info you should know ← numbered list: conventions followed or broken, odd names,
                                  version/licence/size, rule violations (e.g. comment style),
                                  things only visible once another file lands, open caveats
**In short:** <one sentence>
<optional fixes, if any: "Points 3 and 5 are optional fixes. Tell me if you want either.">
Say "next" for file n+1 (<path>).
```

For every **removed** symbol: grep the PR branch to prove nothing still uses it, and say so.
For every **added** symbol: show where it is called from (file:line on the PR branch).

## Step 4 — Follow-up questions (answer, then offer to continue)

Typical asks and how to handle them:

| User asks | Do |
|---|---|
| "is this good or bad?" | Give a clear verdict first, then reasons and caveats |
| "how did it work before?" | Show the old code path from `origin/<base>`, step by step |
| "was this file/kind of thing present before?" / "how was it written before?" | Answer yes/no first. List every old place it lived (table: kind · where · count), show real snippets from `origin/<base>`, then a Before vs After table. Say plainly what the PR does **not** remove |
| "was this convention followed before?" | Table of what already existed on base (folder, naming, registration) with the commits that added it; say whether the PR copies it |
| "since when?" / "when did we stop?" | `git log --follow --diff-filter=AR --format='%h %ad %s' --date=short -- <path>` → timeline table (date · commit · what happened), then explain any renames/reverts and why (read the commit message). Mention unmerged branches only as a side note |
| "is it good to keep this?" / "too many X" | Verdict first, industry practice + why, the rules that keep it safe, then measured counts and numbered options with one marked as the recommendation. If it changes an earlier decision, say so and let the user choose |
| "check online / stackoverflow" | WebSearch/WebFetch official docs, GitHub issues, SO; cite a `Sources:` list |
| "explain in layman terms" | Analogy + numbered plain steps, almost no code |
| "what is X" (a param, type, symbol) | Define it, show its shape, show where it is used |
| "is X used anywhere?" | `git grep -n` on the PR branch, list every call site with purpose |
| "should we change X?" | Recommend yes/no, list exact edits (file:line), and what would break |
| "which files are done?" | ✅ done / ⏳ remaining table, plus any uncommitted local edits |
| "write the whole file explained" | Full file in one block with `// 👉` explanation comments; note any copy mistakes |

End each answer with a one-line nudge back to the walkthrough (`Ready for file n?`).

## Step 5 — Suggestions and PR comments

When you spot an improvement (or the user asks for one):

1. Explain it in plain words and verify it against the code (callers, tests, types).
2. Offer two paths: **leave a PR comment** or **make the change locally**.
3. Comment text: **one line by default**, states the actual reason (the risk or benefit), no
   "since the code already…" padding, no AI references, no internal codenames. Give a longer
   version only if asked.
4. Do **not** post it — hand the text to the user. Post only if explicitly told to.

## Step 6 — Local changes (only when asked)

1. Confirm the current branch matches the PR head and the tree is clean (`git status`,
   `git rev-parse HEAD origin/<head>`).
2. Make the smallest edit; update comments that the edit makes inaccurate.
3. Verify with the repo's own gates — type-check, lint, and the relevant tests. Run tests the
   way the repo's test script does (e.g. backend uses `jest --runInBand`; parallel runs can
   produce false failures).
4. Report: files changed, each check's result, staged vs unstaged state.
   **Never commit or push** unless asked. If the PR description names something you renamed,
   point it out (and edit the description only if asked).

## Done

The walkthrough ends when all files are covered or the user says to stop. Close with a short
summary: what the PR does, the verdict, open caveats (e.g. env/index requirements), and any
suggestions or local edits still pending.
