# orchestrator

The teamlead set on top of the developer set: drive several Claude Code agents from
[herdr](https://herdr.dev) panes, one role per pane, from one orchestrating session.
Three things live here — the `orchestrator` skill, five roles, six wrappers over herdr —
and they ship together as one Claude Code plugin.

## What is here

**The `orchestrator` skill** in `skills/` — the role that hands work out, watches it and accepts it. Its
protocols cover hiring, chaining, status, unblocking, evidence and handoff. It carries no
lead's standing rules of its own: it reads them from `~/.agents/standards/orchestrator.md`,
or the project's `.agents/standards/orchestrator.md`, the same slot the development skills
read code standards from. Absent both, it works to the defaults written in the protocols.

**Five roles** in `agents/` — `back`, `front`, `reviewer`, `tester`, `researcher` — as
native Claude Code agent definitions. Each names the skill that is its craft
(`implement-task`, `review-mr`, `clickup`) and pins its model.
The plugin registers them as `orchestrator:back`, `orchestrator:front` and so on; `recruit`
and `.roster` also accept the bare name.

**Six wrappers** over herdr in `bin/`. Claude Code puts a plugin's `bin/` on the PATH of
its Bash tool, so they are there for the orchestrating session and everything it starts —
not in a terminal of your own:

| Tool | What it does | Needs |
|---|---|---|
| `recruit` | Hire an agent into its own herdr pane, optionally on a fresh git worktree | herdr |
| `roster` | Show who is hired and what each one is doing | herdr |
| `fire` | Dismiss an agent and close its pane | herdr |
| `tell` | Send text to an agent — safer than `herdr agent prompt`, which can paste without submitting and lose the message | herdr |
| `await` | Wait for an agent and tell apart the three states herdr shows identically: finished, asked a question, drifted | herdr |
| `await-mr` | The same wait, tied to a merge request appearing | herdr, glab |

None of them does anything without herdr installed and its server running.

## Before you install

Three things a developer does not need and a teamlead does:

- **herdr**, from https://herdr.dev. The wrappers drive its panes and the installer treats
  its absence as a stop, not a warning. Its server has to be started from a real terminal,
  never from a shell Claude Code spawned — the skill explains why in its first constraint.
- **glab**, authenticated against your GitLab (`brew install glab`, then `glab auth login`).
  Only `await-mr` and the merge-request parts of the roles need it.
- **The developer set**: the ClickUp CLI from
  [claude-work-tools](https://github.com/TechTechWizard/claude-work-tools) and the six
  skills from [dev-skills](https://github.com/TechTechWizard/dev-skills). The roles call those
  skills by name, so install the developer set first, by its own READMEs.

## Install

In Claude Code:

```
/plugin marketplace add RuslanAktaev/orchestrator
/plugin install orchestrator@orchestrator
```

or from a shell: `claude plugin marketplace add RuslanAktaev/orchestrator`, then
`claude plugin install orchestrator@orchestrator`. Start a new session afterwards.

Nothing is cloned and nothing is linked: the plugin system loads the skill and the roles and
puts the wrappers on the PATH. The plugin's one hook, on session start, does its work once
per plugin version and is silent the rest of the time:

- it removes what the old two-step installation left behind — the links `install.sh` made in
  `~/.local/bin` and `~/.claude/agents`, and the copy of the skill `npx skills` made in
  `~/.agents/skills` and `~/.claude/skills`, with its lock-file entry. Only links into a clone
  of this repository and a skill whose `SKILL.md` names this repository are touched; the
  clone itself stays, and the message says where it is;
- it reports a missing herdr, glab or clickup.

## Check that it worked

In a new Claude Code session:

```
/skills                  # lists orchestrator:orchestrator
! herdr status server    # running — start it with `herdr` in a real terminal if not
```

and ask the session to run `recruit --roles` — five roles, `orchestrator:back` to
`orchestrator:tester`. The skill checks the same things before its first hire and stops
with the step to repeat if one is missing.

## Update

```sh
claude plugin marketplace update orchestrator
claude plugin update orchestrator@orchestrator
```

then start a new session. An update arrives only when `version` in
`.claude-plugin/plugin.json` changed — bump it with every change meant to reach people. A role that is already running keeps the definition it started
with — re-hire to apply an update.

## Your standing rules go into the slot

Capacity ceiling, model tier, a project's board route, a shared test dataset that testers
must not clobber — the skill reads these from a file named `orchestrator.md` in
`~/.agents/standards/` (yours) or `<project>/.agents/standards/` (the project's, which wins
for what it covers). Plain Markdown, any structure; write what you would otherwise repeat to
the orchestrator every session, with the reasons. Nothing breaks without it.

## Roles: where `recruit` finds them

A role is a native Claude Code agent definition, and `recruit` accepts it from either of
two places: a file in `~/.claude/agents/<role>.md` — where roles of your own go — or an
installed plugin that ships an `agents/` directory, listed as `<plugin>:<role>`, which is
how this repository's five arrive. `recruit --roles` prints every role it can see and where each one
comes from. A bare name works when exactly one definition answers to it; when two do,
`recruit` refuses and prints both spellings.

The same two spellings are valid keys in a project's `.roster`, one line per role:
`<role> <path-relative-to-project>`, with `.` meaning the project root.

## License

MIT, see [LICENSE](LICENSE).
