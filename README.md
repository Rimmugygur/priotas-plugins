# Priotas for AI assistants

[Priotas](https://priotas.io) is a to-do app with the decisions already made:
capture everything into one inbox, clarify it into next actions, projects,
waiting-fors, reminders, someday items and notes, and work from filtered lists and
a daily agenda, alone or in a team.

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

## User guide

Everything a person needs to know, from connecting each client to what an
assistant can and cannot do, changes and revert, sessions, formatting and staying
signed in, lives in the Priotas docs:

- [Assistants](https://priotas.io/docs/assistants): connecting Claude, Claude Code,
  Codex and ChatGPT, workspaces and scopes, the change log and revert.
- [How Priotas works](https://priotas.io/how-it-works): the method in five
  minutes, with links into the docs.
- [Docs home](https://priotas.io/docs/getting-started).

Short version for Claude Code:

```text
/plugin marketplace add Rimmugygur/priotas-plugins
/plugin install priotas@priotas
```

Then `/mcp`, select `priotas`, **Authenticate**. Use the plugin or a manual
`claude mcp add`, not both, or Claude sees every tool twice.

### Codex desktop

With the [Codex CLI](https://developers.openai.com/codex/cli/) installed, add this
marketplace from a terminal on the machine running Codex:

```bash
codex plugin marketplace add Rimmugygur/priotas-plugins
```

Restart Codex desktop, open **Plugins**, find **Priotas** in the added marketplace
and install it. Alternatively, run `codex plugin add priotas@priotas`.
Complete the connection prompt (or choose **Authenticate** for the plugin's MCP
connection), sign into Priotas, choose your workspace and press **Allow**.
Start a new task and ask **“Summarize my Priotas workspace.”**

Repeat setup on each machine. Windows and WSL can have separate configuration;
run the command in the environment hosting Codex. If the command is unavailable,
install or update the CLI. Disable any previous manual Priotas MCP connection or
other Priotas plugin installation before using this copy to avoid duplicate tools.

This installs the tools **and** the Priotas workflow skill from our GitHub
marketplace; it does not require a public OpenAI directory listing. For manual
MCP setup or self-hosted deployments, see the
[Codex setup guide](https://priotas.io/docs/assistants#codex).

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
"Write welcome mail"), the next action "Compare hosting plans" with the context
@computer and a due date, the reminder "Renew domain", the waiting-for "Logo from Anna" and the inbox
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
8. Personal. "Turn Compare hosting plans into a someday idea." `priotas_convert_item`
   is refused as lossy (a someday item holds neither the context nor the due date);
   the assistant reports the losses and points to Change type in the app. The
   task is unchanged.

## Self-hosted Priotas

The public plugin targets `api.priotas.io`. A self-hosted instance is connected
by hand with its own MCP address (the Connect dialog in that instance shows it),
or with a local copy of this package whose `.mcp.json` points at it.
