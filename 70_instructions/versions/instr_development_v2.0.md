# Development Protocol

**Version:** 2.0.0
**Last Updated:** 2025-11-03
**Maintainer:** PianoMan
**Status:** active
**Source File:** instr_development_v1.0.md
**Migration Notes:** Converted from Markdown to YAML as part of req_1019

## Table of Contents

1. Purpose
2. Collaboration Workflow
3. Universal Coding Standards
4. Language-Specific Guidance
   - PowerShell
   - Python
   - VBA
   - Power Query (M)
5. Logging & Error Handling
6. Final Review Checklist

## 1. Purpose

- Deliver readable, reusable, maintainable, and reliable code.
- Minimize back-and-forth by providing implementation-ready outputs with clear assumptions.
- Preserve trust through precise change tracking and immediate correction of mistakes.

## 2. Collaboration Workflow

- **Understand → Overview → Implement:** Confirm intent, outline the plan (including assumptions and trade-offs), then provide the code.
- **Minimal Diff Responses:** Show only changed lines with enough context. Provide full files only when explicitly requested.
- **Integrity Preservation:** Treat existing behavior and comments as requirements. If a change breaks compatibility, flag the impact and rationale.
- **Information Gaps:** State missing inputs instead of inventing placeholders. Offer concrete next steps (e.g., files to upload, values to confirm).
- **Tone & Clarity:** Stay concise, direct, and professional; highlight risks or open questions before diving into detail.

## 3. Universal Coding Standards

- Single `return` per function; return a variable, not an inline expression.
- Do not nest function calls inside parameter lists—assign intermediate results first.
- Prefer `if / elseif / else` over `switch / case`.
- Keep one function per code block unless delivering a first-pass scaffold.
- Place function descriptors or docstrings immediately above the definition.
- Reuse existing helpers when practical; call out opportunities for modularization.

## 4. Language-Specific Guidance

### PowerShell

- Use `[CmdletBinding()]` by default.
- Header template:
  ```powershell
  #========================================
  #region FunctionName
  .SYNOPSIS
  Brief description.
  #========================================
  #endregion
  function FunctionName {
      [CmdletBinding()]
      param (
          # parameters
      )

      # logic
  }
  ```
- No `Write-Host`; rely on `Write-Verbose`, `Write-Debug`, or shared `Log`.
- Separate entry logic into a wrapper (e.g., `Invoke-Main`).

### Python

- Place a full docstring above each function, including summary, Args, Returns, and example when helpful.
- Use type hints, enums, and named constants instead of literals.
- Return variables; avoid inline expressions in `return`.
- Wrap risky operations in targeted `try/except` blocks with actionable messages.

### VBA

- Keep data members private; expose via `Property Get/Let/Set`.
- Prefer `Private Sub` implementations with `Friend Function` wrappers for external access.
- Name tests using `Test_<Subject>` and ensure error handling via `On Error` with informative messages.

### Power Query (M)

- Use descriptive, unique step names.
- Annotate each major transformation with a brief `//` comment.
- Validate inputs inside custom functions before performing work.

## 5. Logging & Error Handling

- **PowerShell:** use the shared logging switches (`-Dbg`, `-Info`, `-Warn`, `-Err`, `-Always`, `-DryRun`).
- **Python:** log or print through the project's logger; include actionable context in exceptions.
- **VBA:** capture `Err.Number` and `Err.Description`, surface clear remediation steps.
- Report skipped items and partial failures explicitly.

## 6. Final Review Checklist

- [ ] Goal alignment confirmed.
- [ ] Protocol rules followed (returns, naming, descriptors, error handling).
- [ ] Changes limited to intended scope; side-effects noted.
- [ ] Tests or verification steps suggested when appropriate.
- [ ] Response format supports quick application (diffs, commands, or patch blocks).
