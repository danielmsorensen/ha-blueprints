# Home Assistant blueprints

[![Validate](https://github.com/danielmsorensen/ha-blueprints/actions/workflows/validate.yaml/badge.svg)](https://github.com/danielmsorensen/ha-blueprints/actions/workflows/validate.yaml)

Blueprints I use at home. Each one is validated in CI against the current
stable and beta Home Assistant releases.

| Blueprint | What it does | Import |
| --- | --- | --- |
| [Notify people at home](#notify-people-at-home) | Script: notify the phones of whoever is home, or clear a notification everywhere | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdanielmsorensen%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fscript%2Fnotify%2Fpeople_at_home.yaml) |
| [Calendar chore task sync](#calendar-chore-task-sync) | Chore task (e.g. bins) from a calendar, on to-do lists + phones, completed everywhere at once | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdanielmsorensen%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fhousehold%2Ftask_sync.yaml) |

---

## Notify people at home

[`blueprints/script/notify/people_at_home.yaml`](blueprints/script/notify/people_at_home.yaml)

A script that sends a notification to the given people's phones, optionally
only those at home. Phones are found from the mobile app device trackers
assigned to each person (**Settings → People**), so there's nothing to map.

### Inputs (set once on the script)

| Input | Default | Notes |
| --- | --- | --- |
| Persistent | on | Android: can't be swiped away. iOS: time-sensitive (breaks through Focus) |
| Show on Android Auto | off | Adds `car_ui`. The HA app must be in the Android Auto launcher |
| iOS critical alert | off | Loud, ignores silent mode. Reportedly the only way onto CarPlay |
| Android channel | Household tasks | Tune sound/vibration per channel on the phone |

### Fields (passed by the caller)

`people` (required), `message`, `title`, `tag`, `action_id`, `action_title`,
`only_home` (default true), `clear` (default false: removes the tag's
notification from every listed person's phone).

On iOS, turn on **Settings → Notifications → Announce Notifications** for the
HA app to have Siri read time-sensitive notifications in CarPlay.

## Calendar chore task sync

[`blueprints/automation/household/task_sync.yaml`](blueprints/automation/household/task_sync.yaml)

Every day at the start time it checks the calendars for events **tomorrow**.
If there are any, it:

1. Adds one task naming them all (e.g. "Garden and Recycling") to each to-do
   list, due today, such as a local HA list and Google Tasks.

   Tasks made this way don't show on a Nest display: the Google Tasks API only
   stores a date, and Nest seems to surface only tasks with a time.
2. Notifies the people at home, with a button to mark it completed.

Then until it's done:

- Anyone arriving home re-notifies everyone at home.
- Tapping the button on any phone, or ticking the task off on any list,
  completes it on every list and clears every phone.
- At the expire time on the event day, an unfinished task is removed and
  phones are cleared.

Because it reads whatever the calendar says for tomorrow, moved or one-off
collection days just work.

### Requirements

- Home Assistant 2024.10 or later.
- A script created from [Notify people at home](#notify-people-at-home).
- To-do lists that support descriptions and due dates. Local To-do and Google
  Tasks both do.

### Inputs

| Input | Example | Notes |
| --- | --- | --- |
| Calendars | The bin calendars from Waste Collection Schedule | |
| Start time (day before) | `15:00` | Create + notify |
| Expire time (event day) | `09:00` | Remove unfinished task |
| Task key | `bins` | Unique per automation |
| Task name | `{{ events }}` | `events` = tomorrow's event names joined |
| To-do lists | `todo.household_tasks`, `todo.google_tasks` | |
| People | `person.dan`, `person.ning` | |
| Notify script | `script.notify_people_at_home` | |
| Notification title / Button label | `Bin Night` / `Mark completed` | |
| Refresh lists every (minutes) | `2` | Speeds up Google Tasks sync. 0 = off |

### How it tracks the task

Each task's description holds a marker such as `[ha:bins]`, so the title can
be anything. You'll see the marker in Google Tasks. If last time's task is
still open, it's replaced rather than duplicated.

Google Tasks is normally polled every 30 minutes. While a task may be open
(start time the day before until expire time), the lists are refreshed every
few minutes so ticking it off in Google Tasks clears phones quickly.

### Updating

In Home Assistant: **Settings → Automations & scenes → Blueprints**, then
**⋮ → Re-import blueprint**. Automations and scripts using it pick up the
change.

## License

MIT
