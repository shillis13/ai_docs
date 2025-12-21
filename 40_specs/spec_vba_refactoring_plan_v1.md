# VBA Framework Refactoring Plan
# Version: 1.0
# Date: 2025-12-17
# Priority: Working Code > Clear Arch > Reduce Deps > Minimal Changes

## Framework Scope (After Exclusions)

| Metric | Before | After Exclusions |
|--------|--------|------------------|
| Modules | 69 | 58 |
| Lines | 48,192 | 36,488 |
| Public Procs | 675 | 570 |

**Excluded**: *_Tmp (duplicates), wb_* (application), modArraySupport (replaced by mod_Arrays)

---

## Test Coverage Assessment

**Good News**: mod_Tests.bas has 27 tests covering core classes:
- Test_Stack, Test_Cell_*, Test_Row_*, Test_Table_*, Test_Filter*

**Modules with embedded tests**:
| Module | Tests | Notes |
|--------|-------|-------|
| mod_Tests | 27 | Central test suite |
| mod_TableRowCell | 12 | |
| mod_ContainerUtils | 11 | |
| mod_DeferredMsgBox | 5 | |
| mod_Sorting | 4 | |
| mod_RangeMethods | 3 | |
| mod_Misc | 1 | |
| mod_TableFiltering | 1 | |

**Coverage Gap**: Foundation modules (Stack, CompareOptions, iFormatter) have no direct tests.

---

## Dependency Layers (Build Order)

The analyzer computed 55 layers. Here's the simplified view:

### Layer 0-3: TRUE FOUNDATIONS (No/minimal deps)
```
Layer 0: Excel_ApplicationFunctions, iFormatter, vbaWindowsMethods
Layer 1: CallStackLevel
Layer 2: ExcelColorClass  
Layer 3: Stack (66 lines, 1 dep, 38 dependents) ← MODEL PRIMITIVE
```

### Layer 4-10: CORE VALUE TYPES
```
Layer 4: CompareOptions (328 lines)
Layer 5: FormatterExcel
...
```

### Layer 40-55: HIGH-LEVEL (Many deps)
```
Layer 46: Cell (403 lines, 11 deps, 49 dependents)
Layer 50: Row (1302 lines, 15 deps, 39 dependents)
Layer 55: Table (3435 lines, 18 deps, 35 dependents)
```

**Key Insight**: Cell is at Layer 46 but has 49 dependents. It should be lower (fewer deps).

---

## Circular Dependency Clusters

### Cluster 1: Core Data Types (THE BIG ONE)
```
    Cell ←→ Row ←→ Table
      ↑       ↑       ↑
      └─FilterCriteria─┘
            ↑
      CompareOptions
```

**All depend on each other**. This is the root cause of high coupling.

### Cluster 2: Error Handling
```
mod_ErrorHandling ←→ mod_CallStack ←→ mod_IsValid
        ↑                  ↑
    (Cell, Row, Table)  CallStackLevel
```

### Cluster 3: Container Utilities
```
mod_ContainerUtils ←→ mod_Collections ←→ mod_Sorting
```

### Cluster 4: Range Methods
```
ExcelRangeMethods ←→ mod_RangeMethods ←→ mod_ApplicationMethods
```

---

## Refactoring Strategy

### Principle: "Peel the Onion" from Outside In

Start with leaf modules (no dependents), work toward core.
Each change is tested before proceeding.

### Phase 1: CLEANUP (Low Risk)
**Goal**: Remove noise, consolidate duplicates

| Task | Risk | Effort |
|------|------|--------|
| Delete *_Tmp files | None | 5 min |
| Move wb_* to separate folder | None | 10 min |
| Replace modArraySupport with mod_Arrays | Low | 1 hr |

### Phase 2: BREAK UTILITY CYCLES (Medium Risk)
**Goal**: Make utility modules acyclic

**2.1: Fix mod_ErrorHandling** (18 dependents)
- Current deps: Cell, Row, Stack, Table, mod_AppStateManager, mod_CallStack, mod_IsValid, mod_Misc
- Target deps: Stack, mod_CallStack ONLY
- Action: Inline simple checks, remove data type dependencies

**2.2: Fix mod_ContainerUtils ←→ mod_Collections**
- These do similar things - should one depend on the other, not mutual
- Review which procedures cause the cycle
- Either merge or establish clear layering

**2.3: Fix VbaFileMethods ←→ VbaModuleMethods**
- Review what's shared
- Extract common code or merge

### Phase 3: REDUCE CELL DEPENDENCIES (Higher Risk)
**Goal**: Move Cell from Layer 46 toward Layer 10

Cell currently depends on:
1. CompareOptions ← Keep (value type)
2. FilterCriteria ← Remove (Cell shouldn't filter)
3. Row ← Keep (parent reference)
4. Table ← REMOVE (Cell shouldn't need Table)
5. iFormatter ← Keep (interface)
6. mod_EnumFlaggedTypes ← Review
7. mod_ErrorHandling ← Keep (but simplified)
8. mod_IsValid ← Inline simple checks
9. mod_Misc ← Inline simple checks
10. mod_RangeMethods ← Remove (Cell is data, not Range)
11. mod_TableRowCell ← REMOVE (wrong direction)

**Target**: Cell depends only on CompareOptions, iFormatter, Row

### Phase 4: REDUCE ROW DEPENDENCIES
Similar analysis for Row (1302 lines, 15 deps)

### Phase 5: SPLIT TABLE.CLS (3435 lines!)
**Goal**: Make Table manageable

Proposed split:
```
Table.cls (core operations)
├── TableData.cls (data storage, rows/columns)
├── TableFilter.cls (filter operations)  
├── TableFormat.cls (formatting, print)
└── TableIO.cls (range read/write)
```

---

## Immediate Next Steps

1. **Run existing tests** - Ensure mod_Tests passes before any changes
2. **Create backup** - Git commit current state
3. **Execute Phase 1** - Cleanup (delete _Tmp, move wb_*)
4. **Run tests again** - Verify nothing broke
5. **Start Phase 2.1** - Simplify mod_ErrorHandling

---

## Questions Resolved

| Question | Answer |
|----------|--------|
| *_Tmp files | Duplicates - delete |
| wb_* modules | Application - exclude from framework |
| modArraySupport | Replace with mod_Arrays |
| Test coverage | mod_Tests covers core classes |
| Priority | Working > Architecture > Deps > Changes |
