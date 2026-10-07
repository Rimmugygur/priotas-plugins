---
name: priotas
description: Manage tasks, projects, inbox captures and planning in Priotas. Use when the user asks to work with their Priotas workspace, plan their Priotas day, or review their Priotas inbox or projects.
---

# Priotas

Use the connected Priotas MCP tools. Start with `priotas_get_workspace_summary`
to identify the workspace, its kind and available access. Discover tools by their
`priotas_` names; the host may add a plugin namespace. At the start of a
conversation the server may still be connecting, so if no `priotas_` tools are
listed, search the available tools for `priotas_` first; the host waits for a
connecting server inside that search. Only when the tools stay unavailable, ask
the user to connect Priotas and start a new conversation. Never request tokens
or passwords in chat.

Find named items with `priotas_search` or `priotas_list_projects` with `q`, then
read details with `priotas_get_item`. Use returned IDs, not guessed identifiers.
A project's details include its tasks. Respect an explicitly named alternative
source such as GitHub; this skill does not redirect unrelated work into Priotas.

Personal lists include team tasks assigned to the user. Preserve each returned
`workspaceId` when acting on those rows. Team workspaces use tasks and projects;
personal workspaces also have inbox, next actions, waiting-fors and someday items.
The tool inventory depends on workspace kind and granted scopes.

For an unstructured idea the user wants saved, use `priotas_capture`. Clarify
personal inbox items with `priotas_clarify_inbox_item`. Planning and review requests
can be answered with reads; they do not by themselves authorize changing dates,
assignments or statuses.

Tasks and reminders can repeat. Set a rule with `repeat` on `priotas_create_task`,
`priotas_update_task` and, for reminders, `priotas_create_item` and
`priotas_update_item` (a reminder rule carries `tz`, the person's zone);
`stopRepeating` ends it. Only send the rule's shape (unit, every, weekdays,
monthDay or ordinal and weekday, basis, until); the server owns the anchor and
the occurrence. Finishing a repeating task creates the next instance, which the
finish result names; reads carry `repeat`, `seriesId` and `nextOccurrence`.

Complete requested work with `priotas_finish_task` and a concrete summary of the
result. `priotas_update_task` cannot set completion status. Write Markdown in
notes and descriptions; preserve attachment references when editing bodies.
For a refused lossy conversion, explain the fields at risk and direct the user
to Change type in Priotas instead of recreating the item.

Only report a change after the tool confirms it. If a write has an uncertain
outcome, read the affected item before retrying, especially for captures and
creation. Users can inspect and revert assistant changes in Priotas under
Account → Assistants → Assistant changes.
