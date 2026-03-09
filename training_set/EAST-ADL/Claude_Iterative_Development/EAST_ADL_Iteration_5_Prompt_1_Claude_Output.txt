# EAST-ADL Grammar Adaptation Analysis
## Transformation from Generated Grammar to Target Grammar

This document identifies the systematic adaptations required to transform the EAST-ADL generated grammar into the target grammar.

---

## 1. IDENTIFIER ATTRIBUTES

### Pattern Identified:
The **`shortName`** attribute is systematically treated as the identifier/name of each grammar rule.

### Transformation:
- **Generated**: `'RuleName' '{' 'shortName' shortName=Identifier ...`
- **Target**: `'RuleName' shortName=Identifier ('{'  ...`

### Key Changes:
1. `shortName` is **moved outside** the curly braces
2. It becomes a **required identifier** immediately after the rule name keyword
3. The opening curly brace `'{'` becomes **optional** (wrapped in `(...)?`)
4. This makes `shortName` the distinguishing identifier for instances

### Examples:
```
Generated:
Actuator returns Actuator:
    'Actuator'
    '{'
        'shortName' shortName=Identifier
        ...
    '}';

Target:
Actuator returns Actuator:
    'Actuator'
         shortName=Identifier
    ('{'
        ...
    '}')?;
```

---

## 2. ATTRIBUTE TYPE CHANGES

### 2.1 String Type Conversions

Multiple string-type attributes have their types changed:

#### From `String0` to `EString`:
- **Attributes**: `path`, `precondition`, `information`, `formalism`
- **Pattern**: Simple type replacement
- **Example**:
  - Generated: `'path' path=String0`
  - Target: `'path' path=EString ';'`

#### From `String0` to `STRING` (terminal):
- **Attributes**: `text`, `uri`
- **Pattern**: Changes to use terminal STRING instead of String0 rule
- **Example**:
  - Generated: `'text' text=String0`
  - Target: `'text' text=STRING ';'`

#### From `String0` to `UUID`:
- **Attributes**: `uuid`
- **Pattern**: Specialized type for UUID values
- **Example**:
  - Generated: `'uuid' uuid=String0`
  - Target: `'uuid' uuid=UUID ';'`

#### From `Identifier` to `EString`:
- **Attributes**: `category`, `name`
- **Pattern**: Broadens from strict identifier to general string
- **Example**:
  - Generated: `'category' category=Identifier`
  - Target: `'category' category=Identifier ';'` (note: some keep Identifier, need to verify pattern)

### 2.2 Complex Reference Type Patterns

#### Single-valued references changing from optional assignment to required:
- **Generated**: `('environmentModel' environmentModel=FunctionPrototype)?`
- **Target**: `('environmentModel' environmentModel=FunctionPrototype ';')?`
- Added semicolon terminator

---

## 3. ATTRIBUTE KEYWORD REMOVAL

### Pattern Identified:
**Multi-valued (list) attributes** have their keyword labels removed when enclosed in braces.

### Rule:
- If attribute uses `+=` (list assignment) AND is enclosed in `'{' ... '}'`
- **Remove**: The attribute keyword
- **Keep**: Only the list pattern

### Examples:

#### Generated:
```
('ownedComment' '{' ownedComment+=Comment ( "," ownedComment+=Comment)* '}' )?
('operation' '{' operation+=Operation ( "," operation+=Operation)* '}' )?
('port' '{' port+=FunctionPort ( "," port+=FunctionPort)* '}' )?
```

#### Target:
```
(  ownedComment+=Comment (  ownedComment+=Comment)*  )?
(  operation+=Operation (  operation+=Operation)*  )?
(  port+=FunctionPort (  port+=FunctionPort)*  )?
```

### Pattern Summary:
- **Remove**: `'attributeName' '{' attributeName+=`
- **Replace with**: `(  attributeName+=`
- **Remove**: Closing `'}' )?` 
- **Replace with**: `)*  )?`

---

## 4. SYMBOL ADDITIONS AND REMOVALS

### 4.1 Semicolon Addition

**Rule**: Add semicolon (`;`) after **single-valued** and **required** attributes.

#### Single-valued attributes with semicolons:
```
Generated: 'category' category=Identifier
Target:    'category' category=Identifier ';'

Generated: 'uuid' uuid=String0
Target:    'uuid' uuid=UUID ';'

Generated: 'path' path=String0
Target:    'path' path=EString ';'

Generated: 'representation' representation=FunctionBehaviorKind
Target:    'representation' representation=FunctionBehaviorKind ';'

Generated: 'kind' kind=QualityRequirementKind
Target:    'kind' kind=QualityRequirementKind ';'
```

#### Reference attributes with semicolons:
```
Generated: 'identifiable_target' identifiable_target=[Identifiable|EString]
Target:    'identifiable_target' identifiable_target=[Identifiable|EString] ';'

Generated: 'anomaly' anomaly=[Anomaly|EString]
Target:    'anomaly' anomaly=[Anomaly|EString] ';'

Generated: 'function' function=[FunctionType|EString]
Target:    'function' function=[FunctionType|EString] ';'
```

### 4.2 Curly Brace Removals

**For multi-valued attributes without keywords:**

#### Comma separator removal in lists:
```
Generated: '{' topLevelPackage+=EAPackage ( "," topLevelPackage+=EAPackage)* '}' 
Target:      topLevelPackage+=EAPackage (  topLevelPackage+=EAPackage)*

Generated: '{' mode+=Mode ( "," mode+=Mode)* '}'
Target:      mode+=Mode (  mode+=Mode)*
```

**Pattern**:
1. Remove `'{' ... '}'` wrapper
2. Remove comma `","` separators
3. Change from `( "," item+=Type)*` to `(  item+=Type)*` (two spaces instead of comma)

### 4.3 Curly Brace Conversions for Top-Level Rules

**EAXML rule transformation:**
```
Generated:
EAXML returns EAXML:
    {EAXML}
    'EAXML'
    '{'
        ('topLevelPackage' '{' topLevelPackage+=EAPackage ( "," topLevelPackage+=EAPackage)* '}' )?
    '}';

Target:
EAXML returns EAXML:
    {EAXML}
    
    
        (  topLevelPackage+=EAPackage (  topLevelPackage+=EAPackage)*  )?
    ;
```

Key changes:
1. Removed `'EAXML'` keyword
2. Removed outer `'{' ... '}'` braces
3. Applied multi-valued attribute pattern (removed keyword, braces, commas)

### 4.4 Optional Curly Braces Pattern

Most rules have their curly braces made optional:
```
Generated: '{'  ... '}';
Target:    ('{'  ... '}')?;
```

**Exception**: Rules with **required** attributes keep mandatory braces:
- ModeGroup (has required `precondition` and `mode`)
- FunctionBehavior (has required `path` and `representation`)
- ReuseMetaInformation (has required `information` and `isReusable`)
- QualityRequirement (has required `kind`)
- RequirementsRelationshipGroup (has required `requirementsRelationship`)

---

## 5. GRAMMAR RULE ADDITIONS

### 5.1 New Terminal Rules Added

The target grammar adds custom terminal rules at the end:

```xtext
EString returns ecore::EString:
    STRING | ID;

terminal EABINARY:
    ('0b'('0'..'1')*);

terminal EAOCTAL:
    ('0'('1'..'7')('0'..'7')*);

terminal EAHEX:
    ('0x'('0'..'9'|'a'..'f')*);

terminal EAEXPONENT:
    ('0'..'9')+('e'|'E')('+'|'-')?('0'..'9')+;

UUID returns ecore::EString:
    STRING;
```

### Purpose:
- **EString**: Union of STRING and ID terminals
- **EABINARY/EAOCTAL/EAHEX/EAEXPONENT**: Number format support
- **UUID**: String-based UUID representation

---

## 6. GRAMMAR RULE REMOVALS

### 6.1 Removed Type Union Rules

The generated grammar contains many empty/blank lines between type union declarations (lines 27-140). The target grammar consolidates these, removing excessive blank lines.

### Pattern:
```
Generated: (many blank lines between union rules)
Target: (consecutive union rules without blank lines)
```

### 6.2 TODO Comment Rules Removed

The generated grammar has placeholder rules that are removed:
```
Generated:
Numerical returns Numerical:
    'Numerical' /* TODO: implement this rule and an appropriate IValueConverter */;

Float returns Float:
    'Float' /* TODO: implement this rule and an appropriate IValueConverter */;

Integer returns Integer:
    'Integer' /* TODO: implement this rule and an appropriate IValueConverter */;
```

These TODO placeholder rules are **kept in target** but may need implementation.

---

## 7. SPECIAL PATTERNS

### 7.1 Context-Sensitive Parentheses for Multi-valued References

**Reference lists use parentheses instead of braces:**

```
Generated: ('mode' '(' mode+=[Mode|EString] ( "," mode+=[Mode|EString])* ')' )?
Target:    ('mode' '(' mode+=[Mode|EString] (  mode+=[Mode|EString])* ')' )?
```

**Pattern**: 
- Keep parentheses `'(' ... ')'`
- Remove comma separators
- Change `( "," item+=[Type|EString])*` to `(  item+=[Type|EString])*`

### 7.2 Containment vs Reference Distinction

**Containment (owned objects)**: Use pattern without braces/commas
```
(  ownedComment+=Comment (  ownedComment+=Comment)*  )?
```

**References (cross-references)**: Keep parentheses, remove commas
```
('traceableSpecification' '(' traceableSpecification+=[TraceableSpecification|EString] (  traceableSpecification+=[TraceableSpecification|EString])* ')' )?
```

---

## 8. INDENTATION AND FORMATTING

### Pattern Identified:
- **Generated grammar**: Uses tabs or consistent indentation
- **Target grammar**: Uses 4-space indentation consistently
- **Multi-valued patterns**: Use 2-space separation `(  item+=Type)*`

### Whitespace in list patterns:
```
Generated: ( "," item+=Type)*
Target:    (  item+=Type)*
```
Two spaces replace the comma-space pattern.

---

## SUMMARY OF ADAPTATION RULES

### Priority 1: Structural Changes
1. **Move `shortName` outside curly braces** as identifier
2. **Make curly braces optional** `('{'  ... '}')?` for most rules
3. **Remove keywords from multi-valued containment attributes**
4. **Remove curly braces from multi-valued containment lists**
5. **Remove commas from all list patterns**, replace with two spaces

### Priority 2: Type System Changes
6. **Change string types**: String0 → {EString, STRING, UUID}
7. **Add semicolons** to single-valued and required attributes
8. **Keep parentheses** for reference lists (cross-references)

### Priority 3: Grammar Additions
9. **Add terminal rules**: EString, EABINARY, EAOCTAL, EAHEX, EAEXPONENT, UUID

### Priority 4: Cleanup
10. **Remove excessive blank lines** between union type rules
11. **Standardize indentation** to 4 spaces
12. **Maintain TODO rules** for Numerical, Float, Integer

---

## ADAPTATION VERIFICATION CHECKLIST

For each grammar rule, verify:
- [ ] `shortName` moved outside braces as identifier
- [ ] Curly braces optional (unless rule has required attributes)
- [ ] Multi-valued containment: keyword removed, braces removed, commas removed
- [ ] Multi-valued references: keyword kept, parentheses kept, commas removed
- [ ] Single-valued attributes: semicolon added
- [ ] String types: String0 → EString/STRING/UUID as appropriate
- [ ] Indentation: 4 spaces standard, 2 spaces in list patterns
- [ ] Enum formatting: proper spacing around `|` operators

