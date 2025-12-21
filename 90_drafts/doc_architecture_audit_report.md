# Documentation Architecture Audit (req_3006)

## Summary statistics
- Docs in `ai_general/docs` (.md/.yml): 234 (per task commands)
- Condensed files: 62
- `_latest`-named entries: 40 total → 38 symlinks, 2 real files (`30_protocols/protocol_condensed_resolution_latest.yml`, `30_protocols/protocol_at_self_wake_latest.yml`)
- Symlink targets: no outdated pointers found; see missing list below for absent links

## Duplicate findings
- Chat pipeline packet duplicated between `10_architecture/systems/chat_pipeline` and `60_playbooks/chat_pipeline` (README, PIPELINE.md, BULK_IMPORT_202512.md, SCRIPTS.md, LIBRARIAN_TASK.md). `30_protocols/chat_pipeline/PIPELINE.md` is a different variant (not identical).
- Condensed copies identical to full versions for the architecture set: `10_architecture/architecture_latest.{yml,condensed.yml}`, `ai_communication_architecture_latest.{md,condensed.md}`, `cli_orchestration_latest.{md,condensed.md}`, `daemon_architecture_latest.{md,condensed.md}`, and `taxonomy_document_types_latest.{md,condensed.md}` → condensation not applied.
- `_registry_instructions` and `instr_pointer_loading` each exist in both `70_instructions` and `70_instructions/claude` plus versions (4 identical files per topic) — could consolidate via a single source + symlinks.
- Task template v1.1 is duplicated between `80_templates/task_template_v1.1.md` and `80_templates/tmp/task_template_v1.1.md`; older v1.0 remains alongside.
- `10_architecture/compare_test/architectural_layer_model_OLD.md` is identical to `10_architecture/architectural_layer_model.md` (likely a stale backup).

## Missing symlinks
- `40_specs/spec_message_insert_v1.{md,condensed.md}` — no `_latest`.
- `40_specs/cli_specialized_agent_roles_v2.{md,condensed.md}` — no `_latest`; only unversioned files exist.
- `70_instructions/claude/instr_memory_slot_protocol_v2.yml` and `CHANGELOG_codex_guidelines_v2.1.md` — no `_latest`; consumers likely hit unversioned copies.
- `70_instructions/versions/index_instructions_v1.0.yml` — no `_latest`.
- `80_templates/task_template_v1.1.{md,yml}` (and the same under `80_templates/tmp/`) — no `_latest`; v1.0 also present.
- Drafts/migrations: `90_drafts/_archive/cli_coordination_install_v1.md`, `90_drafts/to_migrate_comms_docs/cli_controls/cli_controls_architecture_v1.yml`, `90_drafts/to_migrate_comms_docs/docs - old dir from claude_cli/NOTIFICATION_PATTERN_v1.1.md`, `.../SETUP_GUIDE_v03-OLD.md` — no `_latest` (lower priority).
- `_latest` entries that are real files (not symlinks): `30_protocols/protocol_condensed_resolution_latest.yml`, `30_protocols/protocol_at_self_wake_latest.yml`.

## Manifest gaps
- Manifest: `ai_general/docs/70_instructions/claude/_knowledge_manifest.yml`.
- Coverage: 68 referenced of 234 md/yml docs → 166 missing.
- Missing by tier: 70_instructions (52), 90_drafts (27), 10_architecture (24), 60_playbooks (15), 80_templates (14), 30_protocols (13), 40_specs (7), 20_registries (6), 50_schemas (5), plus root `README.md` and `doc_regression_test_{questions,ANSWERS}.yml`.
- Notable omissions: architecture condensed/latest files, chat_pipeline packet, templates (task/execution/report), registries/indexes, schemas, specs (`spec_message_insert*`, `cli_specialized_agent_roles*`), instruction variants (index, file conventions versions, memory_slot_protocol), and all drafts.

## Recommendations
1. Add `_latest` symlinks for spec_message_insert, cli_specialized_agent_roles, task_template v1.1 (and prune v1.0 if deprecated), instr_memory_slot_protocol v2, CHANGELOG_codex_guidelines v2.1, index_instructions, and the migration/draft items if still active.
2. Convert `_latest` real files in `30_protocols` to symlinks or rename to drop `_latest` if versioning is not intended.
3. Deduplicate chat_pipeline docs by picking a canonical location (e.g., `60_playbooks` or `30_protocols`) and linking from others; remove identical backups like `architectural_layer_model_OLD.md` if unneeded.
4. Regenerate condensed variants for the architecture set so condensed copies differ meaningfully (architecture, daemon_architecture, ai_communication_architecture, cli_orchestration, taxonomy_document_types).
5. Expand `_knowledge_manifest.yml` coverage, prioritizing registries, schemas, templates, specs (including spec_message_insert/cli_specialized_agent_roles), and active drafts; ensure manifest pointers use `_latest` symlinks.
