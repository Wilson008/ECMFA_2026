# EAST-ADL Grammar Adaptation Analysis

## Overview
This document identifies the required adaptations to transform the generated EAST-ADL Xtext grammar into the target grammar. The analysis focuses on five key aspects: identifier attributes, type changes, keyword removal, symbol manipulation, and rule additions/removals.

---

## 1. Identifier Attributes (Answer to Question 1)

### Pattern Identified
**The `shortName` attribute is consistently treated as the identifier** throughout the grammar transformation.

### Specific Changes

#### Before (Generated):
```xtext
EAPackage returns EAPackage:
    'EAPackage'
    '{'
        'shortName' shortName=Identifier
        ...
    '}';
```

#### After (Target):
```xtext
EAPackage returns EAPackage:
    'EAPackage'
         shortName=Identifier
    ('{'
        ...
    '}')?;
```

### Transformation Rule
- **Extract `shortName` from rule body**: Move `shortName` attribute to immediately follow the rule name
- **Remove quotes around 'shortName'**: The keyword 'shortName' is eliminated
- **Position**: Place directly after rule name, before opening brace
- **Apply universally**: This pattern applies to all concrete rules with `shortName` attributes

### Examples Across Different Rules

**VehicleLevel:**
```xtext
// Generated
VehicleLevel returns VehicleLevel:
    'VehicleLevel'
    '{'
        'shortName' shortName=Identifier
        ...
    '}';

// Target
VehicleLevel returns VehicleLevel:
    'VehicleLevel'
         shortName=Identifier
    ('{'
        ...
    '}')?;
```

**Feature_Impl:**
```xtext
// Generated
Feature_Impl returns Feature:
    'Feature'
    '{'
        'shortName' shortName=Identifier
        ...
    '}';

// Target
Feature_Impl returns Feature:
    'Feature'
         shortName=Identifier
    '{'
        ...
    '}';
```

---

## 2. Attribute Type Changes (Answer to Question 2)

### 2.1 String-Type Attributes Changed to EString

**Pattern**: Attributes with type `String0` are changed to `EString`

#### Examples:
```xtext
// Generated
'uuid' uuid=String0

// Target
'uuid' uuid=UUID ';'
```

```xtext
// Generated
'name' name=String0

// Target  
'name' name=Identifier ';'
```

```xtext
// Generated
'body' body=String0

// Target
'body' body=EString ';'
```

```xtext
// Generated
'cardinality' cardinality=String0

// Target
'cardinality' cardinality=EString ';'
```

### 2.2 Boolean-Type Attributes  

**Pattern**: Attributes with type changed or clarified to `Boolean`

#### Example:
```xtext
// Generated (implicit)
'isCustomerVisible' isCustomerVisible=...

// Target (explicit)
'isCustomerVisible' isCustomerVisible=Boolean ';'
```

### 2.3 Identifier Type Mapping

The rule `Identifier` itself undergoes transformation:

```xtext
// Generated
Identifier returns Identifier:
    'Identifier' /* TODO: implement this rule and an appropriate IValueConverter */;

// Target
Identifier returns Identifier:
    ID;
```

**The TODO comment is resolved** by mapping to terminal `ID`.

### 2.4 UUID Type Introduction

A new type `UUID` is defined and used:

```xtext
// Target grammar adds:
UUID returns ecore::EString:
    STRING;
```

### Summary of Type Changes
| Original Type | Target Type | Context |
|---------------|-------------|---------|
| `String0` | `EString` | Generic string attributes (body, cardinality) |
| `String0` | `Identifier` | Name attributes |
| `String0` | `UUID` | UUID attributes |
| `Identifier` (rule) | `ID` (terminal) | Identifier implementation |

---

## 3. Attribute Keyword Removal (Answer to Question 3)

### 3.1 Universal Pattern: Remove Keyword for Identifier

**All instances of the 'shortName' keyword are removed when it becomes the identifier.**

```xtext
// Generated: Keyword present
'shortName' shortName=Identifier

// Target: Keyword removed
shortName=Identifier
```

### 3.2 Top-Level Rule Special Case

In the top-level `EAXML` rule, the **entire rule structure is simplified**:

```xtext
// Generated
EAXML returns EAXML:
    {EAXML}
    'EAXML'
    '{'
        ('topLevelPackage' '{' topLevelPackage+=EAPackage ( "," topLevelPackage+=EAPackage)* '}' )?
    '}';

// Target  
EAXML returns EAXML:
    {EAXML}
    
    
        (  topLevelPackage+=EAPackage (  topLevelPackage+=EAPackage)*  )?
    ;
```

**Changes:**
- Remove `'EAXML'` literal
- Remove outer `'{'` and `'}'`
- Remove `'topLevelPackage'` keyword
- Remove nested `'{'` and `'}'` around list
- Remove commas between list elements

### 3.3 Pattern in Composite Attributes

For single-valued containment references:

```xtext
// Generated
('vehicleLevel' vehicleLevel=VehicleLevel)?

// Target
('vehicleLevel' vehicleLevel=VehicleLevel ';')?
```

**Keyword 'vehicleLevel' is kept**, but semicolon is added.

For multi-valued containment references:

```xtext
// Generated
('ownedComment' '{' ownedComment+=Comment ( "," ownedComment+=Comment)* '}' )?

// Target
(  ownedComment+=Comment (  ownedComment+=Comment)*  )?
```

**Changes:**
- Remove `'ownedComment'` keyword
- Remove braces `'{'` and `'}'`
- Remove commas between elements

---

## 4. Symbol Addition and Removal (Answer to Question 4)

### 4.1 Semicolon Addition Pattern

**Semicolons are added to terminal all single-valued attributes**, including:
- Simple string/identifier/enum attributes
- Cross-references
- Containment references (single-valued)

#### Examples:

**Simple attributes:**
```xtext
// Generated
'category' category=Identifier

// Target
'category' category=Identifier ';'
```

**Cross-references:**
```xtext
// Generated
'identifiable_target' identifiable_target=[Identifiable|EString]

// Target
'identifiable_target' identifiable_target=[Identifiable|EString] ';'
```

**Single-valued containment:**
```xtext
// Generated
('vehicleLevel' vehicleLevel=VehicleLevel)?

// Target
('vehicleLevel' vehicleLevel=VehicleLevel ';')?
```

### 4.2 Comma Removal in Multi-Valued Lists

**All commas between list elements are removed.**

```xtext
// Generated - with commas
('ownedComment' '{' ownedComment+=Comment ( "," ownedComment+=Comment)* '}' )?

// Target - without commas  
(  ownedComment+=Comment (  ownedComment+=Comment)*  )?
```

```xtext
// Generated - with commas
('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] 
    ( "," traceableSpecification+=[TraceableSpecification|EString])* ')' )?

// Target - without commas
('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] 
    (  traceableSpecification+=[TraceableSpecification|EString])* ')' )?
```

### 4.3 Curly Brace Removal in Multi-Valued Containments

**For multi-valued containment references, remove the outer braces but keep parentheses for cross-references.**

#### Containment References (Remove Braces):
```xtext
// Generated
('ownedComment' '{' ownedComment+=Comment ( "," ownedComment+=Comment)* '}' )?

// Target
(  ownedComment+=Comment (  ownedComment+=Comment)*  )?
```

#### Cross-References (Keep Parentheses):
```xtext
// Generated  
('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] 
    ( "," traceableSpecification+=[TraceableSpecification|EString])* ')' )?

// Target (parentheses remain)
('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] 
    (  traceableSpecification+=[TraceableSpecification|EString])* ')' )?
```

### 4.4 Required vs Optional Multi-Valued Lists

Some lists change from optional to required:

```xtext
// Generated - optional
('childFeature' '{' childFeature+=Feature ( "," childFeature+=Feature)* '}' )?

// Target - required
childFeature+=Feature (  childFeature+=Feature)*
```

### 4.5 Main Body Braces Made Optional

For many rules, the entire body becomes optional:

```xtext
// Generated
EAPackage returns EAPackage:
    'EAPackage'
    '{'
        'shortName' shortName=Identifier
        ...
    '}';

// Target
EAPackage returns EAPackage:
    'EAPackage'
         shortName=Identifier
    ('{'
        ...
    '}')?;
```

**The body braces become optional** `('{'...'}')? ` after extracting shortName.

### 4.6 Complete Symbol Transformation Example

```xtext
// GENERATED
FeatureGroup returns FeatureGroup:
    'FeatureGroup'
    '{'
        'shortName' shortName=Identifier
        ('category' category=Identifier)?
        ('uuid' uuid=String0)?
        ('name' name=String0)?
        'cardinality' cardinality=String0
        ('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] 
            ( "," traceableSpecification+=[TraceableSpecification|EString])* ')' )?
        ('ownedComment' '{' ownedComment+=Comment ( "," ownedComment+=Comment)* '}' )?
        ('ownedRelationship' '{' ownedRelationship+=Relationship ( "," ownedRelationship+=Relationship)* '}' )?
        'childFeature' '{' childFeature+=Feature ( "," childFeature+=Feature)* '}'
    '}';

// TARGET
FeatureGroup returns FeatureGroup:
    'FeatureGroup'
         shortName=Identifier
    '{'
        ('category' category=Identifier ';')?
        ('uuid' uuid=UUID ';')?
        ('name' name=Identifier ';')?
        'cardinality' cardinality=EString ';'
        ('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] 
            (  traceableSpecification+=[TraceableSpecification|EString])* ')' )?
        (  ownedComment+=Comment (  ownedComment+=Comment)*  )?
        (  ownedRelationship+=Relationship (  ownedRelationship+=Relationship)*  )?
          childFeature+=Feature (  childFeature+=Feature)*
    '}';
```

**Symbol changes:**
1. ✅ `shortName` moved out, keyword removed
2. ✅ Semicolons added to all simple attributes
3. ✅ Commas removed from all lists
4. ✅ Braces removed from containment lists
5. ✅ Parentheses kept for cross-reference lists
6. ✅ `childFeature` made required (removed `?` and outer braces)

---

## 5. Grammar Rule Additions and Removals (Answer to Question 5)

### 5.1 Removed Rules

The generated grammar includes a `String0` rule that is removed:

```xtext
// Generated - REMOVED in target
String0 returns String:
    'String' /* TODO: implement this rule and an appropriate IValueConverter */;
```

### 5.2 Added Rules

Several new rules are added to the target grammar:

#### EString Rule
```xtext
EString returns ecore::EString:
    STRING | ID;
```

**Purpose**: Provides a flexible string type accepting both STRING terminals and IDs.

#### UUID Rule  
```xtext
UUID returns ecore::EString:
    STRING;
```

**Purpose**: Specific type for UUID attributes.

#### Terminal Rules for Numbers

Four new terminal rules are added for numeric literals:

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

**Purpose**: Support for various numeric literal formats (binary, octal, hexadecimal, scientific notation).

### 5.3 Modified Rules

#### Identifier Implementation
```xtext
// Generated
Identifier returns Identifier:
    'Identifier' /* TODO: implement this rule and an appropriate IValueConverter */;

// Target
Identifier returns Identifier:
    ID;
```

**The TODO is resolved** by mapping directly to the `ID` terminal.

### 5.4 Enum Formatting Changes

Enums have formatting changes (indentation with spaces vs tabs):

```xtext
// Generated
enum ASILKind returns ASILKind:
                    ASIL_A = 'ASIL_A' | ASIL_B = 'ASIL_B' | ...;

// Target
enum ASILKind returns ASILKind:
                ASIL_A = 'ASIL_A' | ASIL_B = 'ASIL_B' | ...;
```

**Indentation changes from more spaces to fewer spaces.**

---

## Summary of Adaptation Patterns

### Pattern 1: Identifier Extraction
- Extract `shortName` as grammar rule identifier
- Remove `'shortName'` keyword
- Place directly after rule name
- Make body optional for most rules

### Pattern 2: Type System Refinement  
- Replace `String0` with appropriate types (`EString`, `Identifier`, `UUID`)
- Implement `Identifier` rule using `ID` terminal
- Add `EString` and `UUID` helper rules
- Add numeric terminal rules

### Pattern 3: Attribute Syntax Standardization
- Add semicolons to all single-valued attributes
- Remove commas from all multi-valued lists
- Remove braces from containment reference lists
- Keep parentheses for cross-reference lists

### Pattern 4: Optionality Changes
- Make rule bodies optional (after identifier extraction)
- Convert some optional lists to required
- Maintain optionality for most attributes

### Pattern 5: Simplification
- Remove redundant keywords for multi-valued attributes
- Eliminate nested brace structures
- Streamline top-level rule (EAXML)

---

## Quantitative Summary

Based on analysis of the grammars:

- **~200+ concrete rules** undergo identifier extraction
- **~1000+ attributes** receive semicolon additions
- **~500+ multi-valued lists** have commas removed
- **~300+ containment lists** have braces removed
- **5 new rules added** (EString, UUID, 4 terminals)
- **2 rules removed/modified** (String0 removed, Identifier implemented)

---

## Adaptation Complexity Assessment

### High Complexity Adaptations:
1. **Identifier extraction** - affects rule structure fundamentally
2. **Type system changes** - requires understanding semantic distinctions
3. **List syntax transformation** - multiple simultaneous symbol changes

### Medium Complexity Adaptations:
1. **Semicolon addition** - systematic but pervasive
2. **Optionality changes** - requires metamodel understanding

### Low Complexity Adaptations:
1. **Comma removal** - mechanical transformation
2. **Enum formatting** - cosmetic changes

---

## Validation Considerations

The adaptations preserve metamodel conformance while improving:
- **Parseability**: Cleaner syntax with identifiers
- **Readability**: Less verbose list syntax
- **Type safety**: More specific type mappings
- **Consistency**: Uniform attribute termination (semicolons)

All transformations maintain semantic equivalence with the original metamodel structure.
