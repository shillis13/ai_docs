# JavaScript Analysis Playbook

**Version:** 1.0.0
**Last Updated:** 2025-01-31
**Maintainer:** PianoMan
**Status:** auxiliary

## Overview

The JavaScript REPL can lint, pattern-match, and summarize files across languages by scripting quick analyses. Favor small, composable helpers so results stay explainable and can be chained per project.

## Core Checks by Language

### Python
- Imports before code, no imports after function defs
- Detect unused imports, missing docstrings, naming drift from snake_case
- Map dependencies and flag missing definitions

### PowerShell
- Verify `param` blocks, `[CmdletBinding()]`, and unified logging usage
- Confirm naming conventions and presence of synopsis headers

### JavaScript/TypeScript
- Inspect import/export symmetry and unused variables
- Validate JSDoc presence and function declaration style

### Markdown/YAML/Shell
- Enforce heading hierarchy, fenced code syntax, duplicate keys, and unquoted shell variables

## Reusable Helpers

### validatePythonStructure
```javascript
export function validatePythonStructure(content) {
  const lines = content.split('\n');
  let seenFunction = false;
  return lines.reduce((issues, raw, idx) => {
    const line = raw.trim();
    if (/^(def|class)\s+/.test(line)) seenFunction = true;
    if (/^(import|from)\s+/.test(line) && seenFunction) {
      issues.push(`Import after definition at line ${idx + 1}`);
    }
    return issues;
  }, []);
}
```

### analyzeFileStructure
```javascript
export function analyzeFileStructure(content, language, patterns) {
  return Object.entries(patterns[language] || {}).reduce((hits, [name, regex]) => {
    hits[name] = (content.match(regex) || []).length;
    return hits;
  }, {});
}
```

## Project Patterns

- Batch process a file map: `Object.entries(files).map(([name, content]) => ({ name, metrics: calculateMetrics(content) }))`
- Build dependency graphs with extracted call sites, then surface unused or circular references
- Generate dashboards by aggregating metrics (LOC, functions, comments, complexity)

## Future Enhancements

- Add pluggable rule sets per language
- Layer visual outputs (graphs, diagrams) on top of JSON results
- Integrate with CI hooks or pre-commit scripts for automated gating
