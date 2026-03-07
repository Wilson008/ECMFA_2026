# Xenia Grammar Adaptation Rules: Generated to Target

## Overview
This document systematically identifies all adaptation rules required to transform the generated Xenia grammar into the target grammar format. The rules are categorized into structural, syntactic, type system, and terminal enhancements.

---

## 1. ROOT RULE SIMPLIFICATION

### Rule 1.1: Remove wrapper keywords and braces from Model rule
**Pattern:**
- FROM: `'Model' '{' (feature '{' elements '}')? '}'`
- TO: Direct feature assignments without wrapper keywords

**Specific Changes:**
```
Generated:
    {Model}
    'Model'
    '{'
        ('headers' '{' headers+=Header ( "," headers+=Header)* '}' )?
        ('entities' '{' entities+=Entity ( "," entities+=Entity)* '}' )?
        ('mapped_entities' '{' mapped_entities+=MappedEntity ( "," mapped_entities+=MappedEntity)* '}' )?
    '}';

Target:
    {Model}
    
    
          headers+=Header  
          (entities+=Entity)*  
          (mapped_entities+=MappedEntity)*  
    ;
```

**Transformation:**
1. Remove `'Model'` keyword
2. Remove outer braces `'{'` and `'}'`
3. Remove feature name literals ('headers', 'entities', 'mapped_entities')
4. Remove inner braces around feature values
5. Change optionality: headers from `?` to required, entities and mapped_entities remain optional with `*`
6. Remove comma separators from multi-valued features (rely on grammar rules)

---

## 2. TYPE SYSTEM ENHANCEMENTS

### Rule 2.1: Eliminate abstract implementation rule (SuperSite_Impl)
**Pattern:**
- Remove intermediate implementation rules that just assign a name
- Consolidate alternatives in the abstract rule

**Specific Changes:**
```
Generated:
    SuperSite returns SuperSite:
        SuperSite_Impl | SiteWithModal | Site;
    
    SuperSite_Impl returns SuperSite:
        {SuperSite}
        'SuperSite'
        name=EString;

Target:
    SuperSite returns SuperSite:
         SiteWithModal | Site;
```

**Transformation:**
1. Remove `SuperSite_Impl` rule entirely
2. Remove `SuperSite_Impl` from SuperSite alternatives
3. Keep only concrete subtypes: `SiteWithModal | Site`

### Rule 2.2: Remove EString type rule and use direct terminals
**Pattern:**
- FROM: Custom type rule `EString returns ecore::EString: STRING | ID;`
- TO: Use `ID` or `STRING` directly in rules

**Specific Changes:**
```
Generated:
    EString returns ecore::EString:
        STRING | ID;

Target:
    [Rule removed]
```

**Transformation:**
1. Delete the EString rule entirely
2. Replace all `EString` references with appropriate terminals:
   - For names/identifiers: use `ID`
   - For string values: use `STRING`

---

## 3. SYNTACTIC ENHANCEMENTS

### Rule 3.1: Add domain-specific concrete syntax to Header
**Pattern:**
- FROM: Generic wrapper with feature names
- TO: Natural language syntax with keywords

**Specific Changes:**
```
Generated:
    Header returns Header:
        {Header}
        'Header'
        '{'
            ('appName' appName=EString)?
            ('sites' '{' sites+=SuperSite ( "," sites+=SuperSite)* '}' )?
        '}';

Target:
    Header returns Header:
        {Header}
        
            'app' appName=ID 'has'  'pages' 
             '[' sites+=SuperSite ( "," sites+=SuperSite)* ']' 
        ;
```

**Transformation:**
1. Remove `'Header'` keyword and outer braces
2. Replace `('appName' appName=EString)?` with `'app' appName=ID 'has' 'pages'`
3. Replace `('sites' '{' ... '}' )?` with `'[' sites+=SuperSite ( "," sites+=SuperSite)* ']'`
4. Change from optional features to required syntax structure
5. Use `ID` instead of `EString`
6. Use square brackets `[]` instead of curly braces `{}`

### Rule 3.2: Transform Entity to choice-based syntax
**Pattern:**
- FROM: Optional features in braces
- TO: Alternative choices with keywords and colons

**Specific Changes:**
```
Generated:
    Entity returns Entity:
        {Entity}
        'Entity'
        '{'
            ('tech' tech=EString)?
            ('path' path=EString)?
            ('mode' mode=Mode)?
        '}';

Target:
    Entity returns Entity:
        {Entity}
        
            'with' ':' tech=STRING |
            'xml' ':' path=STRING |
            'mode' ':' mode=Mode
        ;
```

**Transformation:**
1. Remove `'Entity'` keyword and outer braces
2. Remove feature name literals ('tech', 'path', 'mode')
3. Convert to choice alternatives with `|`
4. Add keyword prefixes: 'with' for tech, 'xml' for path, 'mode' for mode
5. Add `:` separator after each keyword
6. Use `STRING` instead of `EString` for string values
7. Change from optional features `?` to required alternatives

### Rule 3.3: Transform MappedEntity to choice-based syntax
**Pattern:**
- FROM: Optional features with nested collections
- TO: Alternative choices with keywords and square brackets

**Specific Changes:**
```
Generated:
    MappedEntity returns MappedEntity:
        {MappedEntity}
        'MappedEntity'
        '{'
            ('infoProps' '{' infoProps+=InfoProperty ( "," infoProps+=InfoProperty)* '}' )?
            ('linkedProps' '{' linkedProps+=LinkedProperty ( "," linkedProps+=LinkedProperty)* '}' )?
        '}';

Target:
    MappedEntity returns MappedEntity:
        {MappedEntity}
        
            'info' ':' '[' infoProps+=InfoProperty ( "," infoProps+=InfoProperty)* ']'  |
            'map' ':' '[' linkedProps+=LinkedProperty ( "," linkedProps+=LinkedProperty)* ']' 
        ;
```

**Transformation:**
1. Remove `'MappedEntity'` keyword and outer braces
2. Convert to choice alternatives with `|`
3. Replace 'infoProps' with 'info' keyword, add `:` separator
4. Replace 'linkedProps' with 'map' keyword, add `:` separator
5. Replace curly braces `{}` with square brackets `[]`
6. Change from optional features to required alternatives

### Rule 3.4: Add prefix symbol to SiteWithModal
**Pattern:**
- FROM: Keyword-based declaration
- TO: Symbol prefix with keywords

**Specific Changes:**
```
Generated:
    SiteWithModal returns SiteWithModal:
        {SiteWithModal}
        'SiteWithModal'
        name=EString
        '{'
            ('sites' '{' sites+=SuperSite ( "," sites+=SuperSite)* '}' )?
        '}';

Target:
    SiteWithModal returns SiteWithModal:
        {SiteWithModal}
        '@'
        name=ID
            'with' 'modal' '(' sites+=SuperSite ( "," sites+=SuperSite)* ')' 
        ;
```

**Transformation:**
1. Add `'@'` prefix symbol before name
2. Remove `'SiteWithModal'` keyword
3. Replace outer braces with natural language syntax
4. Replace `('sites' '{' ... '}' )?` with `'with' 'modal' '(' sites+=SuperSite ( "," sites+=SuperSite)* ')'`
5. Use `ID` instead of `EString`
6. Use parentheses `()` instead of curly braces `{}`
7. Change from optional to required sites collection

### Rule 3.5: Add prefix symbol to Site
**Pattern:**
- FROM: Keyword-based declaration
- TO: Symbol prefix only

**Specific Changes:**
```
Generated:
    Site returns Site:
        {Site}
        'Site'
        name=EString;

Target:
    Site returns Site:
        {Site}
        '@'
        name=ID;
```

**Transformation:**
1. Add `'@'` prefix symbol before name
2. Remove `'Site'` keyword
3. Use `ID` instead of `EString`

### Rule 3.6: Simplify InfoProperty with arrow syntax
**Pattern:**
- FROM: Feature names with nested collections in braces
- TO: Direct reference with arrow operator

**Specific Changes:**
```
Generated:
    InfoProperty returns InfoProperty:
        {InfoProperty}
        'InfoProperty'
        '{'
            ('page' page=[Site|EString])?
            ('entities' '{' entities+=InfoEntity ( "," entities+=InfoEntity)* '}' )?
        '}';

Target:
    InfoProperty returns InfoProperty:
        {InfoProperty}
        
        
             page=[Site]
            '->'  entities+=InfoEntity ( "," entities+=InfoEntity)*  
        ;
```

**Transformation:**
1. Remove `'InfoProperty'` keyword and outer braces
2. Remove feature name literals ('page', 'entities')
3. Remove `|EString` from cross-reference (use default ID)
4. Add `'->'` arrow operator between page and entities
5. Remove inner braces around entities collection
6. Change from optional features to required structure
7. Keep comma separators for entities

### Rule 3.7: Simplify LinkedProperty with arrow syntax
**Pattern:**
- FROM: Feature names with optional references
- TO: Direct references with arrow and parentheses

**Specific Changes:**
```
Generated:
    LinkedProperty returns LinkedProperty:
        {LinkedProperty}
        'LinkedProperty'
        '{'
            ('name' name=[Site|EString])?
            ('page' page=RedirectPage)?
        '}';

Target:
    LinkedProperty returns LinkedProperty:
        {LinkedProperty}
        
        
             name=[Site]
            '->' '('page=RedirectPage')'
        ;
```

**Transformation:**
1. Remove `'LinkedProperty'` keyword and outer braces
2. Remove feature name literals ('name', 'page')
3. Remove `|EString` from cross-reference
4. Add `'->'` arrow operator between name and page
5. Wrap page assignment in parentheses `'(' page=RedirectPage ')'`
6. Change from optional features to required structure

### Rule 3.8: Simplify InfoEntity with colon separator
**Pattern:**
- FROM: Feature names with nested collection in braces
- TO: Direct assignment with colon separator

**Specific Changes:**
```
Generated:
    InfoEntity returns InfoEntity:
        {InfoEntity}
        'InfoEntity'
        '{'
            ('entries' '{' entries+=InfoEntry ( "," entries+=InfoEntry)* '}' )?
            ('infoValue' infoValue=EString)?
        '}';

Target:
    InfoEntity returns InfoEntity:
        {InfoEntity}
        
        
              entries+=InfoEntry  
            ':' infoValue=STRING
        ;
```

**Transformation:**
1. Remove `'InfoEntity'` keyword and outer braces
2. Remove feature name literals ('entries', 'infoValue')
3. Remove inner braces around entries collection
4. Add `:` colon separator between entries and infoValue
5. Use `STRING` instead of `EString`
6. Change from optional features to required structure
7. Change entries from multi-valued `*` to single value (remove comma separator)

### Rule 3.9: Simplify RedirectPage
**Pattern:**
- FROM: Feature name with parentheses and cross-reference qualifier
- TO: Direct cross-reference collection

**Specific Changes:**
```
Generated:
    RedirectPage returns RedirectPage:
        {RedirectPage}
        'RedirectPage'
        '{'
            ('site' '(' site+=[Site|EString] ( "," site+=[Site|EString])* ')' )?
        '}';

Target:
    RedirectPage returns RedirectPage:
        {RedirectPage}
        
        
              site+=[Site] ( "," site+=[Site])*  
        ;
```

**Transformation:**
1. Remove `'RedirectPage'` keyword and outer braces
2. Remove feature name literal 'site'
3. Remove inner parentheses around cross-reference collection
4. Remove `|EString` from cross-references
5. Change from optional to required structure
6. Keep comma separators for multi-valued references

---

## 4. TERMINAL ENHANCEMENTS

### Rule 4.1: Update enum Mode literals
**Pattern:**
- FROM: Uppercase literal values
- TO: Lowercase descriptive values

**Specific Changes:**
```
Generated:
    enum Mode returns Mode:
                    DEV = 'DEV' | PROD = 'PROD';

Target:
    enum Mode returns Mode:
                    DEV = 'development' | PROD = 'production';
```

**Transformation:**
1. Replace `'DEV'` with `'development'`
2. Replace `'PROD'` with `'production'`

### Rule 4.2: Update enum InfoEntry literals
**Pattern:**
- FROM: Uppercase literal values
- TO: Lowercase literal values

**Specific Changes:**
```
Generated:
    enum InfoEntry returns InfoEntry:
                    MOD = 'MOD' | FREQ = 'FREQ' | PRIO = 'PRIO';

Target:
    enum InfoEntry returns InfoEntry:
                    MOD = 'mod' | FREQ = 'freq' | PRIO = 'prio';
```

**Transformation:**
1. Replace `'MOD'` with `'mod'`
2. Replace `'FREQ'` with `'freq'`
3. Replace `'PRIO'` with `'prio'`

---

## 5. CROSS-CUTTING PATTERNS

### Pattern 5.1: Generic wrapper to domain-specific syntax
- Remove generic wrapper keywords (rule type names used as literals)
- Remove outer structural braces
- Replace feature name literals with domain keywords
- Add symbolic operators (`:`, `->`, `@`)

### Pattern 5.2: Collection syntax transformation
- Replace `'{' collection '}'` with `'[' collection ']'` for list-like features
- Use `'(' collection ')'` for nested/grouped elements
- Remove unnecessary nesting levels

### Pattern 5.3: Type system simplification
- Remove custom type rules when standard terminals suffice
- Use `ID` for identifiers/names
- Use `STRING` for string values
- Remove cross-reference qualifiers (`|EString`) when default ID matching applies

### Pattern 5.4: Optionality changes
- Root rule: Make first feature required, others optional with `*`
- Complex rules: Convert optional features to required alternatives or structures
- Maintain comma separators only where syntactically meaningful

---

## Summary Statistics

- **Rules Modified:** 11 (Model, SuperSite, Header, Entity, MappedEntity, SiteWithModal, Site, InfoProperty, LinkedProperty, InfoEntity, RedirectPage)
- **Rules Removed:** 2 (SuperSite_Impl, EString)
- **Enums Modified:** 2 (Mode, InfoEntry)
- **Structural Changes:** 9 major transformations
- **Type System Changes:** 2 removals
- **Terminal Changes:** 5 literal value updates

## Adaptation Categories Summary

1. **Root Rule Simplification:** 1 rule (Model)
2. **Type System Enhancements:** 2 rules (SuperSite hierarchy, EString removal)
3. **Syntactic Enhancements:** 9 rules (Header, Entity, MappedEntity, SiteWithModal, Site, InfoProperty, LinkedProperty, InfoEntity, RedirectPage)
4. **Terminal Enhancements:** 2 rules (Mode, InfoEntry enums)
