# Home Assistant blueprints

[![Validate](https://github.com/danielmsorensen/ha-blueprints/actions/workflows/validate.yaml/badge.svg)](https://github.com/danielmsorensen/ha-blueprints/actions/workflows/validate.yaml)

Blueprints I use at home. Each one is validated in CI against the current
stable and beta Home Assistant releases.

| Blueprint | What it does | Import |
| --- | --- | --- |
| [Household task sync](#household-task-sync) | Chore task on to-do lists + phones, completed everywhere at once | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdanielmsorensen%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fhousehold%2Ftask_sync.yaml) |

---

## Household task sync

[`blueprints/automation/household/task_sync.yaml`](blueprints/automation/household/task_sync.yaml)

On a trigger of your choice (for example, the evening before bin day), it:

1. Adds a task to one or more to-do lists, such as a local HA list and Google
   Tasks (which can show on a Nest Display).
2. Sends an actionable notification with a **Done** button.

When the task is done, by tapping **Done** on any phone or ticking it off on
any of the lists, it marks it complete on every list and clears the
notification from every phone.

### Requirements

- Home Assistant 2024.10 or later.
- To-do lists that support descriptions and due dates. Local To-do and Google
  Tasks both do.
- A notify action that reaches everyone. For more than one phone, use a notify
  group; see [`examples/notify_group.yaml`](examples/notify_group.yaml).

### Inputs

| Input | Example | Notes |
| --- | --- | --- |
| Create trigger | Calendar `calendar.waste_collection_schedule`, event start, offset `-5:00:00` | 7pm the night before an all-day event |
| Task name | `Put out {{ trigger.calendar_event.summary }}` | Templates render at creation time |
| Task key | `bins` | Must be unique per automation |
| To-do lists | `todo.house_chores`, `todo.google_tasks` | |
| Notify service | `notify.household_phones` | Full action name |
| Notification title / Done label | `Bin night` / `Bins out` | Optional |
| Due in (days) | `0` | 0 = today |

### How it tracks the task

Each task's description holds a marker such as `[ha:bins]`. The blueprint
uses this to find the task again, so the title can be anything. You'll see the
marker in Google Tasks. If last time's task is still open when the trigger
fires, it's replaced instead of duplicated.

The Google Tasks integration polls Google periodically, so a task ticked off in
Google can take a few minutes to clear the phones.

### Updating

In Home Assistant: **Settings → Automations & scenes → Blueprints**, then
**⋮ → Re-import blueprint**. Automations using it pick up the change.

## License

MIT
