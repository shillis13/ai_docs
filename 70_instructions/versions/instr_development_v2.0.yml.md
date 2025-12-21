# Development Protocol

**Version:** 2.0.0
**Last Updated:** 2025-11-03
**Status:** active
**Maintainer:** PianoMan

---

## Table of Contents


---

## Purpose

## Collaboration Workflow

### **Understand → Overview → Implement

** Confirm intent, outline the plan (including assumptions and trade-offs), then provide the code.

### **Minimal Diff Responses

** Show only changed lines with enough context. Provide full files only when explicitly requested.

### **Integrity Preservation

** Treat existing behavior and comments as requirements. If a change breaks compatibility, flag the impact and rationale.

### **Information Gaps

** State missing inputs instead of inventing placeholders. Offer concrete next steps (e.g., files to upload, values to confirm).

### **Tone & Clarity

** Stay concise, direct, and professional; highlight risks or open questions before diving into detail.

## Universal Coding Standards

## Language Specific Guidance

### Powershell



### Header Template



### Separate Entry Logic Into A Wrapper (E.G., `Invoke-Main`).  Python



### Wrap Risky Operations In Targeted `Try/Except` Blocks With Actionable Messages.  Vba



### Name Tests Using `Test <Subject>` And Ensure Error Handling Via `On Error` With Informative Messages.  Power Query M



## Logging Error Handling

### Powershell

use the shared logging switches (`-Dbg`, `-Info`, `-Warn`, `-Err`, `-Always`, `-DryRun`).

### Python

log or print through the project’s logger; include actionable context in exceptions.

### Vba

capture `Err.Number` and `Err.Description`, surface clear remediation steps.

## Final Review Checklist
