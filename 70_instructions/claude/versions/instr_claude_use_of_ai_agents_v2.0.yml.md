# CLI and Codex Delegation Guidelines

**Version:** 2.0.0
**Last Updated:** 2025-11-03
**Status:** active
**Maintainer:** PianoMan

---

## Table of Contents


---

## Overview

## Tool Descriptions

### Cli Coordination System



### **Architecture

**

### Desktop Posts

`broadcasts/req_XXXX_taskname.md` or `direct/cli_{PID}/task_XXX.md`

### Cli Executes And Writes

`responses/cli_{PID}/req_XXXX_response.md`

### Results Move To

`completed/req_XXXX_taskname/` when done

### **Cli Capabilities

**

### Parallel Task Execution Across Multiple Instances  Codex Mcp



### **Codex Capabilities

**

## When To Delegate

### Use Cli Coordination When



### Want To Preserve Findings For Later Reference  Use Codex When



### Want Approval Workflow For Safety  Do Not Delegate When



## Delegation Patterns

### Cli Pattern Asynchronous



### 2. Tell User

"Posted req_XXXX. Ask CLI to 'check inbox'"

### ```  Codex Pattern Synchronous



## Examples

### Good Delegation Cli



### User

"Find all Python files"

### You

[Reads directory] [Lists files] [Checks each one]

### ```  Good Delegation Codex



### ```  Bad Pattern Context Waste



## Context Preservation Priority

### Your Context Window Is The Limiting Resource. Preserve It For



### Delegate To Preserve Context For What Matters Most

partnership with the user.

## Quick Decision Tree

### **Key Principle

** Desktop Claude coordinates and synthesizes. CLI and Codex execute and explore. This division of labor prevents context overflow and enables longer, more productive conversations.
