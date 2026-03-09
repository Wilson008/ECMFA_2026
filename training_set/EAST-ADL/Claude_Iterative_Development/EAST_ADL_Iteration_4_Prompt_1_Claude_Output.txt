# EAST-ADL Grammar Adaptation Analysis

## Overview
This document identifies all adaptations required to transform the generated Xtext grammar into the target grammar for the EAST-ADL language.

---

## 1. IDENTIFIER ATTRIBUTES

### Pattern: `shortName` as Identifier
**Location:** All concrete class rules that have a `shortName` attribute

**Adaptation:**
- **Generated:** `'shortName' shortName=Identifier`
- **Target:** `shortName=Identifier` (keyword removed, positioned before opening brace)

**Affected Rules:** All major entity rules including:
- VehicleLevel, SystemModel, AnalysisLevel, DesignLevel, ImplementationLevel
- Feature_Impl, FeatureGroup, FeatureModel, VehicleFeature
- AnalysisFunctionType_Impl, BasicSoftwareFunctionType, DesignFunctionType_Impl
- FunctionalDevice, FunctionClientServerInterface, HardwareFunctionType, LocalDeviceManager
- All HardwareComponentType subtypes (Actuator, Node, Sensor, etc.)
- All requirement-related rules
- All timing-related rules
- All dependability-related rules
- And many more (applies broadly across the grammar)

**Pattern Count:** ~200+ rules affected

---

## 2. TYPE CHANGES FOR ATTRIBUTES

### 2.1 String0 → EString
**Attributes affected:**
- `uuid` (in most rules)
- `name` (in most rules)
- `body` (in Comment_Impl, Rationale)
- `cardinality` (in Feature_Impl, FeatureGroup, VehicleFeature)
- Various other string-type attributes

**Example:**
- **Generated:** `'uuid' uuid=String0`
- **Target:** `'uuid' uuid=UUID ';'`

### 2.2 Identifier → Identifier (but with different usage patterns)
**Attributes affected:**
- `category` (remains Identifier)

### 2.3 String0 → UUID (special case)
**Attribute:** `uuid`
- **Generated:** `'uuid' uuid=String0`
- **Target:** `'uuid' uuid=UUID ';'`

### 2.4 Specific Type Changes
- **Generated:** `'text' text=String0`
- **Target:** `'text' text=STRING ';'`

- **Generated:** `'uri' uri=String0`
- **Target:** `'uri' uri=STRING ';'`

---

## 3. ATTRIBUTE KEYWORD REMOVAL

### 3.1 Keyword Removal for `shortName`
**Pattern:** The keyword `'shortName'` is removed
- **Generated:** `'shortName' shortName=Identifier`
- **Target:** `shortName=Identifier`

**Positioning:** The identifier moves before the opening brace `'{'`

### 3.2 Keywords Retained
All other attribute keywords are retained, including:
- `'category'`, `'uuid'`, `'name'`, `'cardinality'`, `'body'`, `'text'`, `'uri'`
- All multi-valued attribute keywords
- All reference attribute keywords

---

## 4. SYMBOL ADDITIONS AND REMOVALS

### 4.1 Semicolon Addition Pattern
**Context:** Single-valued attributes that are NOT multi-valued containments or references

**Pattern:**
- **Generated:** `'attributeName' value`
- **Target:** `'attributeName' value ';'`

**Applies to:**
- Simple attributes: `category`, `uuid`, `name`, `cardinality`, `body`, `text`, `uri`
- Boolean attributes: `isElementary`, `isCustomerVisible`, `isRemoved`, etc.
- Enum attributes: `actualBindingTime`, `requiredBindingTime`
- Single-valued references: `featureParameter`, `hardwareComponent`, etc.
- Single-valued containments: `vehicleLevel`, `analysisLevel`, `designLevel`, etc.

**Does NOT apply to:**
- Multi-valued attributes with `'{' ... '}'` or `'(' ... ')'` delimiters
- Multi-valued lists without delimiters

### 4.2 Curly Brace Removal for Multi-Valued Attributes
**Pattern:** Remove `'{'` and `'}'` around multi-valued containments

**Examples:**

**Generated:**
```
('ownedComment' '{' ownedComment+=Comment ( "," ownedComment+=Comment)* '}' )?
```

**Target:**
```
(  ownedComment+=Comment (  ownedComment+=Comment)*  )?
```

**Affected Attributes:**
- `ownedComment`
- `ownedRelationship`
- `technicalFeatureModel`
- `childNode`
- `childFeature`
- `rootFeature`
- `featureConstraint`
- `featureLink`
- `element`
- `subPackage`
- `allocation`
- `portGroup`
- `connector`
- `port`
- `part`
- `operation`
- Many more multi-valued containment attributes

### 4.3 Comma Removal in Lists
**Pattern:** Remove `","` separator between list elements

**Generated:**
```
ownedComment+=Comment ( "," ownedComment+=Comment)*
```

**Target:**
```
ownedComment+=Comment (  ownedComment+=Comment)*
```

**Note:** In some multi-valued references within parentheses, commas are retained:
```
('traceableSpecification' '(' traceableSpecification+=[Reference|EString] ( "," traceableSpecification+=[Reference|EString])* ')' )?
```

But in others, commas are removed:
```
('errorModelPrototype' '(' errorModelPrototype+=[ErrorModelPrototype|EString] (  errorModelPrototype+=[ErrorModelPrototype|EString])* ')' )?
```

### 4.4 Keyword Removal for Multi-Valued Containments
**Pattern:** For certain multi-valued containments, the keyword is removed

**Example:**
**Generated:**
```
'childFeature' '{' childFeature+=Feature ( "," childFeature+=Feature)* '}' 
```

**Target:**
```
  childFeature+=Feature (  childFeature+=Feature)*  
```

**Affected:** `childFeature` in FeatureGroup (required, not optional)

### 4.5 Parentheses Retained
**Pattern:** Parentheses `'('` and `')'` are retained for:
- Cross-references with context
- Multi-valued references

**Example:**
```
('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] (  traceableSpecification+=[TraceableSpecification|EString])* ')' )?
```

---

## 5. GRAMMAR RULE STRUCTURE CHANGES

### 5.1 Root Rule Changes (EAXML)
**Generated:**
```xtext
EAXML returns EAXML:
    {EAXML}
    'EAXML'
    '{'
        ('topLevelPackage' '{' topLevelPackage+=EAPackage ( "," topLevelPackage+=EAPackage)* '}' )?
    '}';
```

**Target:**
```xtext
EAXML returns EAXML:
    {EAXML}
    
    
        (  topLevelPackage+=EAPackage (  topLevelPackage+=EAPackage)*  )?
    ;
```

**Changes:**
- Removed: `'EAXML'` keyword
- Removed: Outer `'{'` and `'}'`
- Removed: `'topLevelPackage'` keyword
- Removed: Inner `'{'` and `'}'`
- Removed: Commas between list elements

### 5.2 Standard Rule Pattern Changes
**Pattern for most concrete class rules:**

**Generated:**
```xtext
RuleName returns Type:
    'RuleName'
    '{'
        'shortName' shortName=Identifier
        ('attribute' attribute=Type)?
        ...
    '}';
```

**Target:**
```xtext
RuleName returns Type:
    'RuleName'
         shortName=Identifier
    ('{'
        ('attribute' attribute=Type ';')?
        ...
    '}')?;
```

**Key Changes:**
1. `shortName` moved before opening brace with keyword removed
2. Opening brace and its content made optional with `('{'...'}' )?`
3. Semicolons added after single-valued attributes
4. Curly braces and commas removed from multi-valued containments

### 5.3 Exception: Feature_Impl Structure
**Generated:**
```xtext
Feature_Impl returns Feature:
    'Feature'
    '{'
        'shortName' shortName=Identifier
        ('category' category=Identifier)?
        ...
    '}';
```

**Target:**
```xtext
Feature_Impl returns Feature:
    'Feature'
         shortName=Identifier
    '{'
        ('category' category=Identifier ';')?
        ...
    '}';
```

**Note:** The curly braces and their content are NOT optional (no `?` after the closing brace)

### 5.4 Exception: FeatureGroup childFeature
**Generated:**
```xtext
'childFeature' '{' childFeature+=Feature ( "," childFeature+=Feature)* '}' 
```

**Target:**
```xtext
  childFeature+=Feature (  childFeature+=Feature)*  
```

**Note:** This is a required attribute (no `?`), and both keyword and braces are removed

---

## 6. DATA TYPE RULES ADDED

The target grammar adds several data type rules that are not present in the generated grammar:

### 6.1 EString Data Type Rule
```xtext
EString returns ecore::EString:
    STRING | ID;
```

**Usage:** Replaces most `String0` references throughout the grammar

### 6.2 UUID Data Type Rule
```xtext
UUID returns ecore::EString:
    STRING;
```

**Usage:** Specifically for `uuid` attributes

### 6.3 Terminal Rules Added
```xtext
terminal EABINARY:
    ('0b'('0'..'1')*);

terminal EAOCTAL:
    ('0'('1'..'7')('0'..'7')*);

terminal EAHEX:
    ('0x'('0'..'9'|'a'..'f')*);

terminal EAEXPONENT:
    ('0'..'9')+('e'|'E')('+'|'-')?('0'..'9')+;
```

**Purpose:** Support for various numeric literal formats

---

## 7. GRAMMAR RULES REMOVED

### 7.1 Placeholder Rules Removed
The following placeholder rules from the generated grammar are removed:

```xtext
Identifier returns Identifier:
    'Identifier' /* TODO: implement this rule and an appropriate IValueConverter */;

String0 returns String:
    'String' /* TODO: implement this rule and an appropriate IValueConverter */;
```

**Reason:** Replaced by proper data type rules and terminal references

---

## 8. ENUM RULE FORMATTING CHANGES

### 8.1 Indentation Changes
**Generated:**
```xtext
enum EnumName returns EnumName:
                    VALUE1 = 'VALUE1' | VALUE2 = 'VALUE2';
```

**Target:**
```xtext
enum EnumName returns EnumName:
                VALUE1 = 'VALUE1' | VALUE2 = 'VALUE2';
```

**Change:** Reduced indentation (fewer spaces before enum values)

---

## 9. CROSS-REFERENCE PATTERNS

### 9.1 Pattern Consistency
**Cross-references maintain the pattern:**
```xtext
attributeName=[Type|EString]
```

**No changes** to the cross-reference syntax itself, but:
- Type references now use `EString` (the data type rule) instead of assuming string matching
- Context references follow the same multi-valued pattern changes

### 9.2 Multi-Valued References with Context
**Generated:**
```xtext
('identifiable_context' '(' identifiable_context+=[Identifiable|EString] ( "," identifiable_context+=[Identifiable|EString])* ')' )?
```

**Target (in some rules):**
```xtext
('identifiable_context' '(' identifiable_context+=[Identifiable|EString] (  identifiable_context+=[Identifiable|EString])* ')' )?
```

**Change:** Comma removal (but parentheses retained)

---

## 10. SPECIFIC RULE ADAPTATIONS

### 10.1 Comment Rules
**Generated:**
```xtext
Comment_Impl returns Comment:
    'Comment'
    '{'
        'body' body=String0
    '}';
```

**Target:**
```xtext
Comment_Impl returns Comment:
    'Comment'
    '{'
        'body' body=EString ';' 
    '}';
```

**Changes:**
- Type: `String0` → `EString`
- Added semicolon after `body`
- No `shortName` identifier (special case)

### 10.2 EAPackage Rule
**Generated:**
```xtext
EAPackage returns EAPackage:
    'EAPackage'
    '{'
        'shortName' shortName=Identifier
        ('category' category=Identifier)?
        ('uuid' uuid=String0)?
        ('name' name=String0)?
        ('ownedComment' '{' ownedComment+=Comment ( "," ownedComment+=Comment)* '}' )?
        ('subPackage' '{' subPackage+=EAPackage ( "," subPackage+=EAPackage)* '}' )?
        ('element' '{' element+=EAPackageableElement ( "," element+=EAPackageableElement)* '}' )?
    '}';
```

**Target:**
```xtext
EAPackage returns EAPackage:
    'EAPackage'
         shortName=Identifier
    ('{'
        ('category' category=Identifier ';')?
        ('uuid' uuid=UUID ';')?
        ('name' name=Identifier ';')?
        (  ownedComment+=Comment (  ownedComment+=Comment)*  )?
        (  subPackage+=EAPackage (  subPackage+=EAPackage)*  )?
        (  element+=EAPackageableElement (  element+=EAPackageableElement)*  )?
    '}')?;
```

**Changes:**
- `shortName` identifier moved before brace, keyword removed
- Body made optional
- Type changes: `uuid=String0` → `uuid=UUID`, `name=String0` → `name=Identifier`
- Semicolons added after single attributes
- Curly braces and commas removed from multi-valued containments

---

## 11. SYSTEMATIC ADAPTATION PATTERNS

### Pattern 1: Identifier Attribute Promotion
**Applies to:** All rules with `shortName` attribute
**Steps:**
1. Remove `'shortName'` keyword
2. Move `shortName=Identifier` before opening brace
3. Add space/indentation for formatting

### Pattern 2: Single Attribute Termination
**Applies to:** All single-valued attributes except multi-valued with delimiters
**Steps:**
1. Add `;` after the attribute value
2. Keep the attribute keyword

### Pattern 3: Multi-Valued Containment Simplification
**Applies to:** Multi-valued containment attributes
**Steps:**
1. Remove outer `'{'` and `'}'`
2. Remove comma separators `","`
3. Keep attribute keyword if present
4. Keep optionality marker `?` if present

### Pattern 4: Body Optionalization
**Applies to:** Most concrete class rules (with exceptions)
**Steps:**
1. Wrap opening brace and entire body in `('{'...'}' )?`
2. Exceptions: Rules where body is required (e.g., Feature_Impl, VehicleFeature)

### Pattern 5: Type Substitution
**Applies to:** Attributes with specific types
**Steps:**
1. Replace `String0` with `EString` or `UUID` (for uuid) or `STRING` (for text/uri) or `Identifier` (for name in some cases)
2. Replace `Identifier` placeholder with actual terminal references

### Pattern 6: Enum Formatting
**Applies to:** All enum rules
**Steps:**
1. Adjust indentation (reduce spaces)
2. Keep structure otherwise identical

---

## 12. SUMMARY OF ADAPTATION TYPES

### By Category:

1. **Structural Adaptations** (~200 rules)
   - Identifier attribute promotion
   - Body optionalization

2. **Type Adaptations** (~400+ attribute occurrences)
   - String0 → EString
   - String0 → UUID
   - String0 → STRING
   - String0 → Identifier (for some name attributes)

3. **Symbol Adaptations** (~600+ occurrences)
   - Semicolon additions
   - Curly brace removals
   - Comma removals

4. **Keyword Adaptations** (~200 rules)
   - shortName keyword removal

5. **Rule Additions** (6 new rules)
   - EString data type rule
   - UUID data type rule
   - 4 terminal rules (EABINARY, EAOCTAL, EAHEX, EAEXPONENT)

6. **Rule Removals** (2 placeholder rules)
   - Identifier placeholder
   - String0 placeholder

7. **Formatting Adaptations** (~40 enum rules)
   - Indentation changes in enum definitions

---

## 13. SPECIAL CASES AND EXCEPTIONS

### 13.1 Rules Without shortName Identifier
**Rules that don't follow the identifier promotion pattern:**
- Comment_Impl
- Rationale
- Various helper/relationship rules without identifiers

### 13.2 Rules with Required Body
**Rules where body is NOT optional:**
- Feature_Impl
- VehicleFeature
- FeatureGroup
- Some relationship/binding rules

### 13.3 Attributes with Special Handling
**childFeature in FeatureGroup:**
- Required (not optional)
- Keyword removed
- Braces removed
- Commas removed

### 13.4 Mixed Comma Removal Patterns
**Some parenthesized references keep commas:**
```xtext
('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] ( "," traceableSpecification+=[TraceableSpecification|EString])* ')' )?
```

**Others remove commas:**
```xtext
('errorModelPrototype' '(' errorModelPrototype+=[ErrorModelPrototype|EString] (  errorModelPrototype+=[ErrorModelPrototype|EString])* ')' )?
```

**Determining factor:** Appears to be rule-specific; requires case-by-case analysis

---

## 14. ADAPTATION STATISTICS

### Estimated Counts:
- **Total grammar rules:** ~350 concrete rules + ~40 abstract rules + ~40 enum rules
- **Rules with identifier adaptation:** ~200 rules
- **Attributes with type changes:** ~400+ occurrences
- **Attributes requiring semicolons:** ~600+ occurrences
- **Multi-valued attributes requiring brace/comma removal:** ~300+ occurrences
- **New data type rules:** 6 rules
- **Removed placeholder rules:** 2 rules

### Adaptation Complexity:
- **High complexity:** Multi-valued attribute pattern determination
- **Medium complexity:** Type substitution rules, body optionalization
- **Low complexity:** Identifier promotion, semicolon addition

---

## 15. VALIDATION AND TESTING CONSIDERATIONS

### Critical Validation Points:
1. **Identifier resolution:** Ensure cross-references work with new EString rule
2. **Optional body handling:** Verify parser handles both empty and populated bodies
3. **List parsing:** Ensure comma-free lists parse correctly
4. **Type conversion:** Verify UUID, EString, STRING, Identifier distinctions
5. **Terminal lexing:** Ensure new terminals don't conflict with existing rules

### Testing Strategy:
1. Test identifier-only instances (no body)
2. Test full instances with all attributes
3. Test multi-valued attributes with 0, 1, and multiple elements
4. Test cross-references with new type rules
5. Test numeric literals with new terminal rules

---

## CONCLUSION

The adaptation from generated to target grammar involves systematic patterns applied across the entire grammar, with some rule-specific exceptions. The main categories of adaptations are:

1. **Identifier attribute promotion** (structural)
2. **Type substitutions** (semantic)
3. **Symbol management** (syntactic)
4. **Body optionalization** (structural)
5. **Data type rule additions** (foundational)

These adaptations transform a verbose, fully-explicit generated grammar into a more concise, user-friendly target grammar while maintaining semantic equivalence.
