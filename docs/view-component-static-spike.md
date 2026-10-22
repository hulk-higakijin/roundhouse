# ViewComponent static spike (triage only, no support claim)

Status: `view_component` stays `Unknown`. This spike only improves
attribution so a dispatch failure on its surface is labelled as the
tool's coverage gap, not the author's error. No diagnostic was
removed, so no emitted program is claimed to run (see AGENTS.md
invariant 6).

## Census input

- `view_component` 4.17.0, `lib` holds 45 Ruby files, about 4900 lines.
- Core: `base.rb` (862), `slotable.rb` (458), `compiler.rb` (206),
  `template.rb` (221) — about 1750 lines combined.

## Dynamic surface (grep, `lib/view_component`)

Total 63 hits for the transpiler's rejected constructs:

- `autoload` about 20 (entry-point fan-out)
- `define_method` about 20 (`slotable.rb`, `compiler.rb`, `template.rb`)
- `method_missing` 4 (`base.rb`, `inline_template.rb`,
  `serializable/proxy.rb`, `slot.rb`)
- `class_eval` 2 (`template.rb`, `base.rb` inheritance hook)
- `instance_variable_get` / `set` 10+ (`@virtual_path` juggling in
  `base.rb`, plus test helpers)

Each hit is a lowering gap the transpiler must be taught. Full-gem
support belongs upstream, never per-app.

## Narrowing plan (not implemented here)

Model only the static subset an app actually calls, as plain methods
with static types:

1. Subclass `ViewComponent::Base` with an explicit `def call`
   returning a static string (no slots, no `define_method`).
2. Render via an explicit static call site (no
   `"components/#{name}"` interpolation, no `constantize` / `send`).
3. Verify with `roundhouse check --continue` (census shows
   `0 unknown` for the narrowed surface plus `0 unresolved on known
   receivers`) and against the running binary.

Tapioca/Sorbet RBI output is a type inventory only. It does not
provide the lowering plus runtime implementation the emitter needs.

## Change in this spike

- `src/gems.rs` `SURFACES`: added `view_component` author-facing
  surface (`render_in`, `with_collection`,
  `with_collection_parameter`, `with_content_areas`, `with_slot`,
  `renders_one`, `renders_many`) for attribution only.
