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

- Path: `blueprints/automation/<category>/<name>.yaml`.
- Every blueprint sets `author`, `source_url` (the GitHub `blob/main` URL of
  that file) and `homeassistant.min_version`.
- Use current syntax: `triggers:`/`trigger:`, `actions:`/`action:`,
  `conditions:`/`condition:`. Inject a trigger-selector input with
  `- triggers: !input x`. It can't carry an `id`, so tell it apart from the
  blueprint's own triggers by checking that `trigger.id` isn't one of theirs.
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

1. Add the YAML under `blueprints/automation/<category>/`.
2. Add at least one `use_blueprint` instance to `test/config/automations.yaml`,
   setting every input, plus one using only the defaults.
3. Add a row and a section to `README.md`, including a My Home Assistant import
   badge (URL-encode the `blob/main` URL).
4. Open a PR.

## Open items

- Check that a Google Task created through the API (date only, no time) shows
  on the Nest Display. If it doesn't, consider an optional TTS announcement
  input (`tts.speak` to a `media_player`).
- task_sync: a list's open count dropping for an unrelated item makes it send
  a harmless extra `clear_notification`. Could be tightened if it becomes noisy.
