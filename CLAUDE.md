# Notes for Claude

Public repo of Home Assistant blueprints. Dan imports them into his live HA
instance through each blueprint's `source_url` and updates them from his phone
(HA Companion app → Blueprints → ⋮ → Re-import). Anything merged to `main` can
reach his house, so keep `main` working.

## Workflow

- Work on a branch and open a PR. Don't push to `main`.
- CI (`.github/workflows/validate.yaml`) must pass: `yamllint --strict` and
  HA's `check_config` against `test/config/` on stable (and beta, which is
  allowed to fail). `check_config` exits 0 on a broken blueprint instance and
  only logs `ERROR`, so the workflow greps its output. Keep that grep.
- In the PR description, write what changed and how Dan should test it on his
  real setup. CI only checks the schema, not behaviour.

## Conventions

- Path: `blueprints/<domain>/<category>/<name>.yaml` (domain `automation` or
  `script`).
- Every blueprint sets `author`, `source_url` (the GitHub `blob/main` URL of
  that file) and `homeassistant.min_version`.
- Use current syntax: `triggers:`/`trigger:`, `actions:`/`action:`,
  `conditions:`/`condition:`. Inputs used in trigger templates go in
  `trigger_variables`.
- Phone notifications go through the `people_at_home` script blueprint. It
  finds each person's phone from their mobile_app device trackers
  (`notify.mobile_app_<slug of device registry name>`); don't add notify
  action inputs.
- Only render templates that depend on `trigger.*` in the branch for that
  trigger. Accessing `trigger.calendar_event.summary` under another trigger
  raises an error.
- Don't rely on wildcard or title matching for to-do items. Find items with
  `todo.get_items` (response is `{entity_id: {items: [...]}}`) and act on them
  by `uid`.
- Keep personal entity IDs out of blueprints. Put them in `test/config` or the
  README examples.
- Never break the inputs of a published blueprint (renames, removals). Dan's
  live automations would stop working after a re-import. Add new inputs with
  defaults.

## Adding a blueprint

1. Add the YAML under `blueprints/<domain>/<category>/`.
2. Add at least one `use_blueprint` instance to `test/config/automations.yaml`
   (or `scripts.yaml`), setting every input, plus one using only the
   defaults.
3. Add a row and a section to `README.md`, including a My Home Assistant import
   badge (URL-encode the `blob/main` URL).
4. Open a PR.

## Open items

- Confirm on the first live run: date-only Google Task shows on the Nest
  Display; iOS `clear_notification` by tag works; Android Auto `car_ui` shows.
- CarPlay only shows critical alerts (community reports, unverified).
- 2026-11-24: `todo.google_tasks` once stopped matching service calls while
  still showing a state (calls silently skip it). Integration reload fixed it.
  Cause unknown; watch for a missing Google task.
- task_sync with a single list: an unrelated item ticked off while no task is
  open sends a harmless extra `clear_notification`.
