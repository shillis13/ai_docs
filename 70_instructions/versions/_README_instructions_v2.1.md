# AI Instruction Index - Getting Started

**Version:** 2.1.0
**Last Updated:** 2025-12-06
**Maintainer:** PianoMan
**Status:** active
**Source File:** _README_instructions_v2.0.yml
**Migration Notes:** Updated to use *_latest references per DEC-2025-12-06-001

## Purpose

- Provide a single entry point for every standing instruction that applies to our AI assistants.
- Clarify which guidance is common to all AIs versus Claude- or ChatGPT-specific.
- Track versions so we can see when something changes and whether downstream docs need updates.

## How to Use

- Start here at the beginning of each new engagement.
- Load every document in the **Common** section unless a project scope explicitly supersedes it.
- Layer on the **Claude** or **ChatGPT** overlays only when you are running that specific assistant.
- Obey status flags: `active` is required, `auxiliary` is optional context, `legacy` requires confirmation.

## Instruction Map

| File | Audience | Status | What It Covers |
|------|----------|--------|----------------|
| `instr_operating_principles_latest.yml` | All | active | Identity, partnership principles, time-first collaboration, assumption handling, feedback cadence. |
| `instr_development_latest.yml` | All | active | Coding workflow, universal standards, language-specific guidance, review checklist. |
| `../40_specs/spec_response_footer_latest.yml` | All | active | Canonical metadata footer fields, validation rules, helper snippets. |
| `../40_specs/spec_js_analysis_playbook_latest.yml` | All | auxiliary | Fast reference for leveraging the JavaScript analysis tool across languages. |
| `claude/instr_environment_overview_latest.yml` | Claude | active | Claude desktop vs web capabilities, filesystem access, workflow patterns, do/don't reminders. |
| `chatgpt/protocol_response_latest.yml` | ChatGPT | active | Expectation-first response flow, persistence defaults, retro-response etiquette, trust commitments. |
| `chatgpt/spec_persona_engine_config_latest.yml` | ChatGPT | active | Hybrid persona engine configuration and control commands. |

## Update Log

- **2.1.0 (2025-12-06)** - Updated all file references to use *_latest pattern per versioning decision.
- **2.0.0 (2025-11-03)** - Rebuilt index after migrating legacy instructions; aligned active set with new file structure and statuses.
- **1.0.0 (2025-10-18)** - Original index (superseded).
