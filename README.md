# claude-skills

Skills for [Claude Code](https://docs.claude.com/en/docs/claude-code), shared as a plugin marketplace.

## Install

```
/plugin marketplace add sumansaksh/claude-skills
/plugin install co-review-pr@claude-skills
```

## Skills

| Skill | What it does |
|-------|--------------|
| `co-review-pr` | Walks you through a GitHub PR one file at a time |

### co-review-pr

Helps you understand a PR well enough to approve it or comment on it.

1. Finds the PR: from a link, the current branch, child repos, or your open review requests.
2. Shows a GitHub-style file tree and a reading order.
3. Explains one file per reply: the diff, if it is generated, what the file does, what changed and why.
4. Answers follow-up questions and checks every claim against the code.
5. Suggests one-line PR comments or local edits, but only when you ask. It never posts, commits, or pushes on its own.

Usage: `/co-review-pr:co-review-pr [PR link or number]`, or ask Claude to "co-review this PR".

Requires the [GitHub CLI](https://cli.github.com/) (`gh`), logged in.

## License

MIT
