# Priotas for AI assistants

[Priotas](https://priotas.io) is a Getting Things Done app: capture everything,
clarify it into next actions, projects, waiting-fors, reminders, someday items and
notes, and work from filtered lists and a daily agenda, alone or in a team.

This repository is the installable Priotas plugin for **Claude Code** and
**Codex**, and the user guide for connecting any MCP-capable assistant (Claude,
ChatGPT, Codex and others). Priotas hosts the tools, handles OAuth, enforces
workspace permissions and records every assistant change. The plugin only carries
the server address and a short skill; nothing runs on your machine.

- Server address: `https://api.priotas.io/mcp`
- Sign-in: OAuth. The assistant sends you to Priotas, you pick a workspace and
  press **Allow**. No API keys.
- What it can never do: delete anything, reset or clear a workspace, change members
  or project owners.

## Connecting

Account → **Connected assistants** in Priotas shows these steps with your exact
server address and copy buttons.

### Claude Code

Inside Claude Code:

```text
/plugin marketplace add Rimmugygur/priotas-plugins
/plugin install priotas@priotas
```

Choose the **user** scope so the plugin is available in every project. Then run
`/mcp`, select `priotas` and choose **Authenticate**. Installation alone does not
sign you in.

Without the plugin, one command registers the server for every project:

```bash
claude mcp add --scope user --transport http priotas https://api.priotas.io/mcp
```

Use the plugin or the manual registration, not both, or Claude sees every tool
twice. The server's own instructions tell Claude that tasks and projects live in
Priotas, so no per-project prompt text is needed.

### Codex

Until the plugin is listed in the Codex plugin directory, connect the server
directly. In the desktop app: **Settings → MCP servers → Add server**, name it
`priotas`, choose **Streamable HTTP**, paste the server address, save, restart,
then choose **Authenticate** for Priotas. From the CLI:

```bash
codex mcp add priotas --url https://api.priotas.io/mcp
codex mcp login priotas
```

Run these where Codex runs. Windows and WSL keep separate configuration and
sign-in storage. Codex desktop and the VS Code extension read the same
configuration, so one sign-in covers them.

### Claude (claude.ai, Claude Desktop)

Settings → Connectors → **Add custom connector**. Name it Priotas, paste the
server address, leave the client secret empty, save, then **Connect**. A connector
added on the web is available in Claude Desktop, mobile and Cowork too.

### ChatGPT

Turn on developer mode once (Settings → Connectors → Advanced), then Settings →
Connectors → **Create**. Same server address, OAuth authentication; ChatGPT
registers itself and walks you through the same consent screen.

## Choosing the workspace

A connection is *you in one workspace*. The consent screen lists every workspace
you belong to.

- **Personal:** the assistant sees what you see. Inbox, next actions (including
  team tasks assigned to you), projects (yours plus the team projects you pinned),
  waiting-fors, reminders, someday list and notes.
- **Team:** the assistant sees that team's tasks, projects and people. It can
  create tasks for you or leave them unassigned, update and finish tasks assigned
  to you, and comment. It cannot assign work to anyone else.

Connect twice for two workspaces. Each connection is listed and revoked on its own
under Account → Connected assistants. Leaving a team, or being removed from one,
revokes every connection you held there.

## What the assistant can do

Every tool is named `priotas_<verb>`, so a fresh chat can tell whose tools they
are. Reads, on every connection: workspace summary, next actions (personal) or
tasks (team), projects, inbox, waiting-fors, reminders / someday / notes, people
(team), one item in full, search, agenda. Writes, on connections granted the
*write* scope: capture, clarify an inbox item, create / update / finish a task,
create / update a project, create / update a waiting-for, reminder, someday item
or note, add a comment, toggle a checklist step, change an item's type. In a
team, writes reach only tasks assigned to you, and a project's owner cannot be
changed.

Resources (`priotas://today`, `priotas://inbox`, `priotas://project/{id}`) and
prompts (`weekly_review`, `clarify_inbox`, `plan_my_day`, `draft_followups`,
`project_brief`, and `standup` in teams) appear as slash commands in assistants
that support them.

## Seeing and undoing what an assistant changed

Every write is recorded with the fields as they were before and after.
**Assistant changes** (in the sidebar while there is anything from the last seven
days, always from Account → Assistants) lists them newest first as one-line
diffs; in a team each row names the person and the assistant. **Revert** puts the
fields back: a created item is deleted, a finished task returns to active with its
summary removed, a clarified capture goes back to the inbox, a converted item goes
back to its old kind under the same id. If a person edited the item after the
assistant, Revert asks first. A session's changes can be reverted together. The
log is kept for ninety days.

Finishing a task is final on the assistant's side: it marks the task done and
appends its summary to the notes. The dashboard's **Finished by assistants** card
shows the last week's finishes with Revert one click away. Comments and team
notifications say *via <assistant>* and *by <person> via <assistant>*.

## Following an assistant's work

An assistant working on a task or project can open a **session** there. The
item's page then shows an *Assistant* block: its plan, what it did and, when it
needs a decision, a question you answer right there. Until you answer, the task
shows *Needs you* and the dashboard lists it under **Needs your input**. Anyone
who can see the item can follow the session; only the person the assistant works
for can answer.

## Formatting

Assistants read and write **Markdown** everywhere: notes, task notes, project
descriptions and the notes on waiting-fors, reminders and someday items. Notes
store Markdown as-is; the other kinds convert to the editor's rich text on the way
in and back on the way out, so round trips do not drift. Attachments cannot be
uploaded or downloaded by an assistant. Inline attachment chips appear as
`![filename](attachment://id)` and are re-appended if an edit drops them, so a
file reference is never lost. Every list carries created and updated timestamps
and accepts `createdSince` / `updatedSince`.

## Changing an item's type

There is no move back to the inbox; the inbox is a one-way funnel. An item of the
wrong kind is **converted**: Change type in the item menu, `priotas_convert_item`
for assistants. A conversion keeps the item's id, attachments, links, tags and,
between tasks and projects, comments. Fields the new kind cannot hold are
**losses**: steps, dependencies, an assignee, a due date on a someday item, and so
on. A person sees the list and may go ahead. An assistant may only make lossless
conversions and is told what would be lost so it can ask you. A project converts
or is deleted only when it is empty (no tasks, waiting items or linked notes, done
ones included). An item with an open assistant session cannot be converted;
deleting it cancels the session.

## Staying signed in

You authenticate once per client per machine. Every Claude Code (or Codex)
install is the same OAuth client, so a new consent adds a token set and leaves the
others alive, unless it grants narrower scopes, in which case the wider tokens are
revoked. Access tokens last a day and refresh silently; every request still checks
the token live, so Disconnect cuts access at once. A refresh token lasts thirty
days from its last use, with a one-minute grace after rotation so two sessions
refreshing at the same moment do not log each other out. After that minute a
reused refresh token is treated as stolen and its whole family is revoked.

## Support

Questions and problems: [open an issue](https://github.com/Rimmugygur/priotas-plugins/issues)
or write to support@priotas.io.

---

# The plugin package

```text
.claude-plugin/marketplace.json        Claude Code marketplace catalog (repo root)
plugins/priotas/
├── .claude-plugin/plugin.json         Claude Code identity
├── .codex-plugin/plugin.json          Codex identity and listing presentation
├── .mcp.json                          The hosted MCP endpoint
├── skills/priotas/SKILL.md            Priotas workflow skill, shared by both clients
└── assets/                            Logos (and listing screenshots)
```

Both clients use the same endpoint and skill. Client-specific metadata stays in
its own manifest. Everything the installed copy needs is inside `plugins/priotas`;
there is no build step. The MCP server itself lives in the Priotas backend, not
here.

## Local development

Claude Code, from this repository's root:

```bash
claude --plugin-dir ./plugins/priotas
claude plugin validate ./plugins/priotas
claude plugin validate ./.claude-plugin/marketplace.json
```

Codex: register a copy of this package in the personal marketplace on the host
running Codex (`~/.agents/plugins/marketplace.json` pointing at a copy of
`plugins/priotas`), install it from Plugins or with `codex plugin add
priotas@personal`, authenticate, and check that a workspace summary succeeds. An
installed or enabled status alone does not prove the connection works. Windows and
WSL have different user homes; use the one for the actual Codex host.

## Releases

Both manifests carry the same version; bump it (and tag `v<version>`) for any
change to the skill, the manifests or the endpoint. Before tagging: validate both
Claude manifests, walk both clients on a local copy (authenticate, read a
workspace summary, read a project, do one reversible write, revert it in Priotas,
disconnect), push `main`, and confirm a clean-machine `/plugin marketplace add`
plus install works against the published branch. Never package credentials.

## Codex plugin directory submission

The public listing goes through OpenAI Platform as a **With MCP** plugin with the
production endpoint and this skill. It needs a verified publisher identity, the
listing URLs in the Codex manifest (website, privacy, terms) plus the support URL
above, domain verification (Priotas serves the token at
`https://api.priotas.io/.well-known/openai-apps-challenge`), reviewer credentials
for a test account, and the test cases below.

### Submission test cases

Run against the reviewer account's fixtures: a personal workspace with the project
"Launch newsletter" (tasks "Draft issue 1", "Pick a sending tool" due 2026-01-15,
"Write welcome mail"), the next action "Compare hosting plans" with two checklist
steps, the reminder "Renew domain", the waiting-for "Logo from Anna" and the inbox
captures "Ask Jon about pricing" and "Look into podcast idea"; and the team
workspace "Directory review" with "Prepare directory demo" (assigned to the
reviewer, two steps) and "Review pricing page" (assigned to someone else). Both
connections carry the *write* scope. Reset after a run: revert every row on the
Assistant changes page and delete the captures the run created.

Positive:

1. Personal. "Summarize my Priotas workspace." Expect `priotas_get_workspace_summary`
   and a reply naming the personal workspace with 4 next actions, 1 project and
   2 inbox items.
2. Personal. "What is left on the Launch newsletter project?" Expect
   `priotas_search` or `priotas_list_projects`, then `priotas_get_item`; the
   reply lists the three tasks by title.
3. Personal. "Capture: call the bank about the card limit." Expect
   `priotas_capture`; the capture appears at the top of the inbox in Priotas.
4. Team. "Finish Prepare directory demo, summary: demo prepared and rehearsed."
   Expect `priotas_finish_task`; the task shows done with the summary appended,
   the dashboard card Finished by assistants lists it, Revert reopens it.
5. Personal. "What am I behind on in Priotas?" Expect `priotas_get_agenda`; its
   Overdue section lists "Pick a sending tool" and nothing from another workspace.

Negative:

6. Personal. "Delete the Launch newsletter project." No delete tool exists; the
   assistant says it cannot delete and points to the app. The project still exists.
7. Team. "Mark Review pricing page as done." The assistant declines because the
   task is assigned to someone else, or calls `priotas_finish_task` and relays the
   server's refusal. The task stays active.
8. Personal. "Turn Compare hosting plans into a note." `priotas_convert_item` is
   refused as lossy (the steps would be lost); the assistant reports the loss and
   points to Change type in the app. The task is unchanged.

## Self-hosted Priotas

The public plugin targets `api.priotas.io`. A self-hosted instance is connected
by hand with its own MCP address (the Connect dialog in that instance shows it),
or with a local copy of this package whose `.mcp.json` points at it.
