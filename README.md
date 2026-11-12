# Priotas for AI assistants

[Priotas](https://priotas.io) is a to-do app with the decisions already made:
capture everything into one inbox, clarify it into next actions, projects,
waiting-fors, reminders, what is in Someday and notes, and work from filtered lists and
a daily agenda, alone or in a team.

This repository is the installable Priotas plugin for **Claude Code** and
**Codex**, and the user guide for connecting any MCP-capable assistant (Claude,
ChatGPT, Codex and others). Priotas hosts the tools, handles OAuth, enforces
workspace permissions and records every assistant change. The plugin only carries
the server address and a short skill; nothing runs on your machine.

- Server address: `https://api.priotas.io/mcp`
- Sign-in: OAuth. The assistant sends you to Priotas, you pick a workspace
  (and, for your personal one, the teams it may work in) and press **Allow**.
  No API keys.
- What it can never do: delete forever or empty the trash, reset or clear a
  workspace, change members or project owners. What it deletes goes to the trash,
  where you restore it.

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
"Launch newsletter" (tasks "Draft issue 1", "Pick a sending tool" deadline 2026-01-15,
"Write welcome mail"), the next action "Compare hosting plans" with the context
@computer and a deadline, the empty project "Old ideas", the next action "Draft agenda"
with the step "Collect topics", the reminder "Renew domain", the waiting-for "Logo from Anna" and the inbox
captures "Ask Jon about pricing" and "Look into podcast idea"; and the team
workspace "Directory review" with "Prepare directory demo" (assigned to the
reviewer, two steps) and "Review pricing page" (assigned to someone else). Both
connections carry the *write* scope. Reset after a run: revert every row on the
Assistant changes page, delete the captures the run created and empty the trash.

Positive:

1. Personal. "Summarize my Priotas workspace." Expect `priotas_get_workspace_summary`
   and a reply naming the personal workspace with 5 next actions, 2 projects and
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
6. Personal. "Write a note called Newsletter ideas and put it on the Launch
   newsletter project." Expect `priotas_create_item` with kind `note` and
   `links` naming the project; the project page lists the note under Notes, the
   note's text holds no link, and the Assistant changes page shows the note as
   created. Then "Take that note off the project." Expect `priotas_update_item`
   with `removeLinks`; the row reads Unlinked from project "Launch newsletter",
   and Revert links it again.
7. Personal. "Move Compare hosting plans to Someday." Expect
   `priotas_update_task` with `status: someday` (or `priotas_convert_item` with
   to: someday); the task leaves Next actions and shows on the Someday page with
   its context and deadline, the Assistant changes page shows the status going
   from active to someday, and Revert puts it back in Next actions.
8. Team, after case 4. "Reopen Prepare directory demo, it needs another
   rehearsal." Expect `priotas_reopen_task` with the reason; the task is open
   again with the reason as a comment written via the assistant, the Assistant
   changes page shows a comment row and a Reopened row, and Revert on the
   Reopened row finishes the task again with the summary from case 4 still on it.
9. Personal. "Delete the Old ideas project and the task Draft agenda." Expect
   `priotas_delete_item` twice; both are on the Trash page with the assistant's
   name next to the user's, the task's row saying 1 step went with it, and the
   reply names the step. The Assistant changes page shows two Deleted rows, and
   Revert on the task's row brings it back with its step.
10. Personal, after reverting case 9. "Turn Draft agenda into a waiting-for on
   Anna." `priotas_convert_item` is refused first: the task has a step, which a
   waiting-for cannot hold. The assistant says the step "Collect topics" would
   be dropped and asks. After "Yes, go ahead" expect `priotas_convert_item`
   with `acknowledgeLoss: true`; the item is on the Waiting for list under the
   same id, the Assistant changes page shows a Converted row, and Revert brings
   back the next action with its step.

Negative:

11. Personal. "Delete the Launch newsletter project." `priotas_delete_item` is
   refused: the project still holds tasks. The assistant says a project must be
   empty to be deleted and names what is in it, without deleting the tasks. The
   project still exists.
12. Team. "Make someone else the owner of the Directory review project." No tool
   changes a project's owner; the assistant says it is the person's decision and
   points to the app. The owner is unchanged.

## Self-hosted Priotas

The public plugin targets `api.priotas.io`. A self-hosted instance is connected
by hand with its own MCP address (the Connect dialog in that instance shows it),
or with a local copy of this package whose `.mcp.json` points at it.
