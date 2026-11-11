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

Areas sit above projects (Home, Work, a client) and have no status or dates.
`priotas_list_areas` lists them and `priotas_search` finds one by name. For
"what's open in Work", ask the task list for that `areaId`
(`priotas_list_next_actions`, or `priotas_list_tasks` in a team), not one
`priotas_get_item` per project; add `projectId: "none"` for only the tasks filed
directly on the area. Next actions hide blocked tasks, so ask again with
`blocked: true` when the user wants the whole open workload. Pass the area's
`workspaceId` for a team area; from the personal connection next actions are the
user's own tasks there and `priotas_list_tasks` is everyone's. When the user
names an area for new work, file it there with `areaId`. Never delete an area.

A task and a project each carry one label (`labels` is a list of at most
one plain word); sending a second is refused, so to relabel, send the new
one alone. A task takes a label where it is filed in a project or directly
on an area (`projectId` or `areaId` when you create it), from that
project's or that area's vocabulary; a task filed nowhere takes none.

Personal lists include team tasks assigned to the user. Preserve each returned
`workspaceId` when acting on those rows. On a personal connection
`priotas_get_workspace_summary` also lists the team workspaces the user allowed
for this assistant: pass a
team's `workspaceId` to `priotas_list_tasks` for everyone's tasks there and to
`priotas_list_people` for its members, and to `priotas_create_project`,
`priotas_create_area` or `priotas_create_task` to create there (the user owns a
project created this way). You have the access the user has there: you can edit
and finish any task they can reach, and assign one with `assigneeId` (a `userId`
from `priotas_list_people`, `""` to unassign). Assigning tells that person, so
do it when asked. Changing a project's owner stays the person's decision.

The user chooses which teams an assistant on a personal connection works in.
The summary says how many teams are not allowed. In a team that is not allowed
you still read and update the user's own rows there, the ones the merged lists
show (tasks assigned to them, projects they own or pinned): get, update, finish
and comment work with the row's `workspaceId`. Everything else there is refused
(everyone's tasks, the people, creating anything, another person's task), and
on the user's own rows three changes are refused too: a change of assignee, a
move to another project or area, and a change of blockers (`blockedBy`,
`blocks`). A call that asks for one of those with other fields is refused
whole, so send the other fields on their own. When a call says
this assistant isn't allowed to work in a team, do not retry it and do not look
for another way in: tell the user they can allow that team in Priotas under
Account, Connected assistants, and carry on with what does not need it.

Team workspaces use tasks and projects;
personal workspaces also have inbox, next actions, waiting-fors and Someday (tasks and projects shelved for later).
The tool inventory depends on workspace kind and granted scopes.

For an unstructured idea the user wants saved, use `priotas_capture`, with a
`projectId` or an `areaId` (one or the other) when the user names where it
belongs. A capture
is a task in Inbox; clarifying it with `priotas_clarify_inbox_item` moves it into
the flow (or converts it in place), so what it became keeps the capture's id.
An idea in Someday (a capture clarified with `as: someday`) needs no project,
in a team either: it can sit in a project, on an area or in neither.
In a team, a capture the person has started to clarify can carry
`clarifyDraft.assignee` in `priotas_list_inbox`: a member's `userId`, `me`,
or `none` when they chose that nobody holds it. Absent means they have not
decided. When you clarify that capture for them, keep their choice: pass the
`userId` as `assigneeId`, use `assignToMe` for `me`, and leave it unassigned
for `none`.
Never move work back to the inbox: that is the person's decision. Planning and review requests
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

To delegate a personal task, convert it to a waiting-for with
`priotas_convert_item` (`to: waiting`, `who`); a team task is delegated by
assigning it, never by converting. Its planned date becomes the follow-up date when you
leave `followUpDate` out; pass a date to set another, or an empty string for no
follow-up date.

Only report a change after the tool confirms it. If a write has an uncertain
outcome, read the affected item before retrying, especially for captures and
creation. Users can inspect and revert assistant changes in Priotas under
Account → Assistants → Assistant changes.
