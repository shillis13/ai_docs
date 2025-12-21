# VBA Framework Chain Analysis - Refined
# Date: 2025-12-17
# Focus: Breaking transitive dependency chains

## The Root Cause

The Table cluster (Cell, Row, Table, mod_TableRowCell, FilterCriteria, Filter, CompareOptions) 
depends on **15 external modules**. Everything that touches the cluster inherits ALL 15.

```
   Your module
        │
        ▼
   [Table Cluster] ──────┬── Stack (66 lines) ✅ OK
        │                ├── iFormatter (17 lines) ✅ OK  
        │                ├── mod_ErrorHandling (485) ⚠️ WHY?
        │                ├── mod_Misc (575) ⚠️ VAGUE
        │                ├── mod_IsValid (382) ⚠️ INLINE?
        │                ├── mod_RangeMethods (2793) ❌ WRONG LAYER
        │                ├── mod_Collections (975) ⚠️ HEAVY
        │                ├── mod_ContainerUtils (2555) ❌ TOO HEAVY
        │                ├── mod_Comparisons (739) 
        │                ├── mod_EnumFlaggedTypes (1071)
        │                ├── mod_CallStack (197)
        │                ├── mod_DeferredMsgBox (350)
        │                ├── mod_AppStateManager (247)
        │                ├── ExcelRangeMethods (1165) ❌ WRONG LAYER
        │                └── mod_TableFiltering (423)
        │
        ▼
   ALL 15 modules dragged in (~11K lines!)
```

## True Leaf Modules (Zero chains)

These can be used without ANY transitive deps:
- vbaWindowsMethods (50 lines)
- Excel_ApplicationFunctions (126 lines)  
- iFormatter (17 lines)

## Strategy: Slim the Cluster's External Dependencies

### Priority 1: Remove Excel/Range deps from Cell

**Cell.cls** depends on:
- mod_RangeMethods ← WHY? Cell is DATA, not Excel
- mod_ErrorHandling ← Move to App layer

**Action**: Audit Cell.cls for:
- What from mod_RangeMethods does it actually use?
- Can that be inlined or moved to a helper?

### Priority 2: Remove ErrorHandling from data classes

**mod_ErrorHandling** is called by Cell, Row, Table
- Data classes shouldn't handle errors - they should RAISE them
- Caller (App layer) should handle

**Action**: Replace error handling in data classes with simple `Err.Raise`

### Priority 3: Inline mod_IsValid checks

**mod_IsValid** (382 lines) provides validation
- Used by: Cell, FilterCriteria, Row, Table
- Simple checks like `IsEmpty`, `IsNull` should be inline

**Action**: Identify which IsValid functions are trivial, inline them

### Priority 4: Review mod_Misc usage

**mod_Misc** (575 lines) is a grab-bag
- Used by: Cell, Row, mod_TableRowCell
- Symptom of "I don't know where to put this"

**Action**: 
- Split mod_Misc into focused modules
- Or inline the simple functions used by cluster

---

## Cluster Dependencies by Module

### Cell.cls (403 lines) - 6 external deps
```
✅ iFormatter        - interface, OK
⚠️ mod_EnumFlaggedTypes - needed?
⚠️ mod_ErrorHandling - move to caller
⚠️ mod_IsValid       - inline simple checks
⚠️ mod_Misc          - review usage
❌ mod_RangeMethods   - shouldn't be here
```

### Row.cls (1302 lines) - 10 external deps
```
✅ Stack             - OK
✅ iFormatter        - OK
⚠️ mod_Collections   - review usage
⚠️ mod_Comparisons   - OK for sorting?
⚠️ mod_ContainerUtils - TOO HEAVY
⚠️ mod_EnumFlaggedTypes
⚠️ mod_ErrorHandling
⚠️ mod_IsValid
⚠️ mod_Misc
⚠️ mod_RangeMethods  - shouldn't be here
```

### Table.cls (3435 lines) - 13 external deps
Largest, most deps - splitting would help

---

## Immediate Actions

### Action 1: Audit Cell → mod_RangeMethods
Find what Cell actually uses from mod_RangeMethods:
```bash
grep -n "mod_RangeMethods\|RangeMethods\." Cell_v09.cls
# Or search for specific function names from mod_RangeMethods
```

### Action 2: Count ErrorHandling usage in cluster
```bash
grep -c "mod_ErrorHandling\|LogError\|RaiseError" Cell*.cls Row*.cls Table*.cls
```

### Action 3: List mod_Misc functions used by cluster
```bash
grep -oh "\b[A-Za-z_]*\b" Cell*.cls Row*.cls Table*.cls | sort | uniq > cluster_words.txt
grep "^Public " mod_Misc*.bas | sed 's/.*Public \(Sub\|Function\) //' | cut -d'(' -f1 > misc_funcs.txt
comm -12 <(sort cluster_words.txt) <(sort misc_funcs.txt)
```
