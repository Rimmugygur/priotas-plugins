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
names an area for new work, file it there with `areaId`.

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
on the user's own rows these are refused too: a change of assignee, a
move to another project or area, a change of blockers (`blockedBy`,
`blocks`), a move to Someday, reopening a finished task, completing a
project, marking work reviewed, following, pinning, making a task a step,
board columns, editing a comment and deleting. A call that asks for one of
those with other fields is refused whole, so send the other fields on their
own. When a call says
this assistant isn't allowed to work in a team, do not retry it and do not look
for another way in: tell the user they can allow that team in Priotas under
Account, Connected assistants, and carry on with what does not need it.

Contexts and note folders are the user's own names for where work is done
and where notes are filed. Never guess one: `priotas_list_contexts` gives each
context's id and how many open next actions carry it, and
`priotas_list_note_folders` gives the folder tree with each folder's path.
Use an id from there as a `contextId` or a note's `folderId`. Create one
(`priotas_create_context`, `priotas_create_note_folder`) only when the user
asks for it; a name that already exists is refused and the error names the
one to use. `priotas_update_context` renames a context, and
`priotas_update_note_folder` renames a folder or moves it under another.
Deleting either stays with the user.

A note is linked to the items it is about, where an item is `{kind, id}` and
`kind` is `task`, `project`, `waiting`, `reminder`, `note` or `area`. There are
two ways, and they can hold the same item:

- Attach it. `links` on `priotas_create_item` (kind `note`), and `addLinks` and
  `removeLinks` on `priotas_update_item`, each a list of `{kind, id}`. Nothing
  is written in the note's text, and the links the note already has stay,
  the person's included. This is what "put this note on the Launch project" or
  "file it under the Home area" asks for.
- Mention it. `[[kind:id|Title]]` in `bodyMd` shows as a link in the text and
  links the note too. Use it when the sentence itself refers to the item. These
  links follow the text: take the wiki link out and that link goes. A link
  inside a code block or inline code is only text.

Every link in a result carries its `source`: `manual` (attached) or `body` (from
the text). `removeLinks` takes off an attached link only. If the text names the
item too, the note stays linked and the result says "Still linked from the
note's text": to unlink it fully, edit the body, and say so to the user rather
than calling `removeLinks` again. To find the notes on an item, call
`priotas_list_items` with kind `note`, `linkedKind` and `linkedId`. Notes and
the items they link live in the personal workspace: a team item's id is not
found.

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
A clarify into a task takes the rest in the same call: `areaId` to file it on
an area, `parentTaskId` to make it the next step of a task, `blockedBy` and
`blocks`, and `repeat`. If one of those is refused after the capture was
clarified, the error says so: fix that part with `priotas_update_task`, do
not clarify again.
In a team, a capture the person has started to clarify can carry
`clarifyDraft.assignee` in `priotas_list_inbox`: a member's `userId`, `me`,
or `none` when they chose that nobody holds it. Absent means they have not
decided. When you clarify that capture for them, keep their choice: pass the
`userId` as `assigneeId`, use `assignToMe` for `me`, and leave it unassigned
for `none`.
Never move work back to the inbox: that is the person's decision. Planning and review requests
can be answered with reads; they do not by themselves authorize changing dates,
assignments or statuses.

Someday is a place a task or a project waits in, out of the flow. When the
user asks for it, move a task there or back with `status` (`someday` or
`active`) on `priotas_update_task`, in the same call as a move to a project
or an area if they want that too; `plannedDate` is then the date to look at
it again. `priotas_create_task` takes `status: someday` for a task that
starts there. A project moves with `status` on `priotas_update_project`; in
Someday it takes its tasks out of everyone's lists, so in a team say that
before you do it. A status on a done task or on a capture is refused: reopen
the one, clarify the other. Suggest a move to Someday in a review; make it
when the user says yes.

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

A step is a task inside another task. `parentTaskId` on `priotas_update_task`
makes an existing task the last step of that task, and an empty string takes
a step out again. A step is where its task is, so joining takes the task's
project, area and place (which can drop its label) and `parentTaskId` is not
combined with `projectId`, `areaId` or `status`. `afterId` or `beforeId`
places a step among its sibling steps, and any other task in the manual order
of its list. Steps are worked in order: a later step waits for the ones before
it. A task cannot be made a step of a capture, since that would move it back
to the inbox: clarify the capture first.

A project's board columns are the labels its tasks carry. `columns` on
`priotas_update_project` replaces the whole list, in order, each with an
optional `limitCount`: read the current list first (`priotas_get_item` on the
project, `columns`) and send it back with your change. A column you leave out
is dropped from the board and its tasks keep the label. `renameColumn`
(`from`, `to`) renames one column and every task that carries it. Renaming
onto a name that already exists would merge two columns and is refused: tell
the user to do that in Priotas.

To bring a finished task back, call `priotas_reopen_task` with `reasonMd`: why
it comes back, which is posted as a comment for whoever finished it. It keeps
its assignee. A reopened step reopens its task. Skip one occurrence of a
repeating task with `priotas_skip_occurrence`; tick a reminder with `done` on
`priotas_update_item` (a repeating reminder moves on to its next occurrence
instead, and the result says which happened).

Complete a project with `priotas_finish_project` and a summary of what was
achieved. It does not finish the project's tasks, so read them first
(`priotas_get_item`) and tell the user what is still open before you complete
it. `status: active` on `priotas_update_project` reopens a completed project.

`priotas_mark_reviewed` takes a finished task off the user's To review list
(`priotas_list_review_queue`). Use it when the user has looked the work over
and says it is fine, never to clear the list for them. `priotas_set_follow`
follows or unfollows a team task or project for the user, with `reviews` to
put its finished work on their To review list; it sets the user's own follow
only. `priotas_set_pin` pins a team project to the user's personal Projects
list or takes it off. `priotas_update_comment` replaces the text of a comment
the user wrote; someone else's comment is refused.

`priotas_convert_item` changes an item's kind in place. When the new kind
cannot hold something the item has (steps, blockers, comments, an assignee, a
repeat rule, a field), the call is refused and lists what would be dropped.
Tell the user exactly what is on that list and ask. Only after they say yes,
call again with `acknowledgeLoss: true`; never set it on your own, and never
recreate the item to get around the refusal. What the conversion drops is kept
for 30 days: Revert on the user's Assistant changes page brings the item back
as it was, and tells them if something could not return. Four refusals are
final, with or without `acknowledgeLoss`: into a note (a note does not convert
back, so only a lossless conversion is allowed), a capture in the inbox that
would lose something (clarifying it is the user's then), a project that still
holds work, and a team the user did not allow. For those, direct the user to
Change type in Priotas.

To delegate a personal task, convert it to a waiting-for with
`priotas_convert_item` (`to: waiting`, `who`); a team task is delegated by
assigning it, never by converting. Its planned date becomes the follow-up date when you
leave `followUpDate` out; pass a date to set another, or an empty string for no
follow-up date.

Delete only what the user asked you to delete, with `priotas_delete_item`
(kind `task`, `project`, `waiting`, `reminder`, `note`, `area`, or `inbox` for
a capture). Nothing is destroyed: the item goes to the trash with everything
on it (a task's steps, comments, files and blockers), waits there 30 days, and
the user restores it from the Trash page or from Assistant changes. Say what
went with it; the result lists it. A project or an area must be empty first:
when the delete is refused for that, say what is still in it and ask, do not
delete its tasks to get there. In a team, deleting a project takes the
workspace owner or an admin. `priotas_list_trash` lists what is in the trash
and `priotas_restore_item` brings one back by its `trashId`; if the result has
`notes`, tell the user what could not be put back. You cannot delete forever
or empty the trash: those are the user's, in Priotas.

Only report a change after the tool confirms it. If a write has an uncertain
outcome, read the affected item before retrying, especially for captures and
creation. Users can inspect and revert assistant changes in Priotas under
Account → Assistants → Assistant changes. A revert goes back the way the app
would: reverting a reopen finishes the task again, and the page says what
stayed as it was (a parent task that stays open, a task that no longer
repeats).
