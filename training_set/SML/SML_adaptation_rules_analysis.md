# SML Grammar Adaptation Rules Analysis

## Overview
This document identifies and categorizes the adaptation rules required to transform the generated grammar into the target grammar for the SML (Scenario Modeling Language) domain-specific language.

---

## 1. IMPORT SYSTEM MODIFICATIONS

### Rule 1.1: Import URI Standardization
**Pattern:** Platform-specific URIs → Standard HTTP URIs
- **Generated:** `import "platform:/resource/org.scenariotools.org.sml.model/model/sml.ecore"`
- **Target:** `import "http://www.scenariotools.org/sml"`
- **Generated:** `import "platform:/resource/org.eclipse.emf.ecore/model/Ecore.ecore" as ecore`
- **Target:** `import "http://www.eclipse.org/emf/2002/Ecore" as ecore`

### Rule 1.2: Additional Import Addition
**Pattern:** Add missing expression package import
- **Added in target:** `import "http://www.scenariotools.org/sml/expressions/ScenarioExpressions"`

---

## 2. ROOT RULE SIMPLIFICATION

### Rule 2.1: Specification Rule Restructuring
**Type:** Complete syntax transformation

**Generated structure:**
```
Specification returns Specification:
    {Specification}
    'Specification'
    name=EString
    '{'
        ('domains' '(' domains+=[ecore::EPackage|EString] ( "," ...)* ')' )?
        ('contexts' '(' contexts+=[ecore::EPackage|EString] ( "," ...)* ')' )?
        ('controllableEClasses' '(' controllableEClasses+=[ecore::EClass|EString] ( "," ...)* ')' )?
        ...
    '}';
```

**Target structure:**
```
Specification returns Specification:
    {Specification}
    (imports+=DummyExprClass)*  
    'specification'
    name=ID
    '{'
        ->('domain' domains+=[ecore::EPackage|FQN])*
        ('contexts' contexts+=[ecore::EPackage|FQN])*
        ('controllable' '{' (controllableEClasses+=[ecore::EClass])* '}')?
        channelOptions=ChannelOptions
        ('non-spontaneous events' '{' ... '}')?
        ('parameter ranges' '{' ... '}')?
        (containedCollaborations+=NestedCollaboration |
         ('include collaboration' includedCollaborations+=[Collaboration]))*
    '}';
```

**Key changes:**
- Remove `{Specification}` action (keep implicit creation)
- Change keyword: `'Specification'` → `'specification'`
- Change name type: `name=EString` → `name=ID`
- Add prefix imports: `(imports+=DummyExprClass)*` before keyword
- Transform domain syntax:
  - From: `('domains' '(' domains+=[...] ')' )?`
  - To: `->('domain' domains+=[ecore::EPackage|FQN])*`
  - Note the `->` line break hint and singular keyword
- Transform controllable syntax:
  - From: `('controllableEClasses' '(' ... ')' )?`
  - To: `('controllable' '{' (controllableEClasses+=[ecore::EClass])* '}')?`
- Embed channelOptions directly: `channelOptions=ChannelOptions` (no label, mandatory)
- Add descriptive keywords: `'non-spontaneous events'`, `'parameter ranges'`
- Nest collaboration inclusion: `('include collaboration' includedCollaborations+=[Collaboration])`
- Change collaboration containment to reference `NestedCollaboration` rule

---

## 3. TYPE SYSTEM ENHANCEMENT

### Rule 3.1: Terminal Rule Additions
**Pattern:** Add custom terminal rules

**Added rules:**
```
FQN:
    ID ('.' ID)*;

FQNENUM:
    ID (':' ID)*;

DOUBLE returns ecore::EDouble:
    INT '.' INT;

SIGNEDINT returns ecore::EInt:
    INT;
```

### Rule 3.2: Replace EString with Specific Terminals
**Pattern:** Generic type → Specific terminal
- Replace `EString` with `ID` for names
- Replace `EString` with `FQN` for qualified references
- Replace `EString` with `FQNENUM` for enum literal references

**Examples:**
- `name=EString` → `name=ID`
- `[ecore::EPackage|EString]` → `[ecore::EPackage|FQN]`
- `[ecore::EEnumLiteral|EString]` → `[ecore::EEnumLiteral|FQNENUM]`

### Rule 3.3: Remove Generated Type Rules
**Pattern:** Delete Ecore implementation rules

**Removed rules:**
- `EString returns ecore::EString: STRING | ID;`
- `EInt returns ecore::EInt: '-'? INT;`
- `EDouble returns ecore::EDouble: '-'? INT? '.' INT (('E'|'e') '-'? INT)?;`
- All EPackage, EClass, EAttribute, EReference, EOperation, EParameter rules
- All EAnnotation, ETypeParameter, EGenericType rules
- `ETypedElement`, `EClassifier`, `EStructuralFeature` delegating rules

---

## 4. SYNTACTIC ENHANCEMENT (CONCRETE SYNTAX)

### Rule 4.1: Keyword-Based Syntax Transformation
**Pattern:** Transform verbose attribute-style syntax to compact keyword-based syntax

#### Example 1: ModalMessage
**Generated:**
```
ModalMessage returns ModalMessage:
    {ModalMessage}
    (strict?='strict')?
    (monitored?='monitored')?
    'ModalMessage'
    '{'
        ('collectionModification' collectionModification=DummyDatatype)?
        ('expectationKind' expectationKind=ExpectationKind)?
        ('receiver' receiver=[Role|EString])?
        ('sender' sender=[Role|EString])?
        ('modelElement' modelElement=[ecore::ETypedElement|EString])?
        ('parameters' '{' parameters+=ParameterBinding ( "," ...)* '}' )?
    '}';
```

**Target:**
```
ModalMessage returns ModalMessage:
    (strict?='strict')?
    ((monitored?='monitored')?
    expectationKind=ExpectationKind)?
    sender=[Role]
    '->'
    receiver=[Role] '.'
    modelElement=[ecore::ETypedElement]
    ('.' collectionModification=DummyDatatype)?
    ('(' (parameters+=ParameterBinding ( "," ...)*)? ')')?
    ;
```

**Transformation:**
- Remove class name keyword `'ModalMessage'`
- Remove braces `'{' ... '}'`
- Remove attribute labels: `'expectationKind'`, `'receiver'`, etc.
- Add syntactic operators: `'->'`, `'.'`
- Make parameters use parentheses directly
- Reorder elements for natural flow

#### Example 2: Loop
**Generated:**
```
Loop returns Loop:
    {Loop}
    'Loop'
    '{'
        ('bodyInteraction' bodyInteraction=Interaction)?
        ('loopCondition' loopCondition=Condition)?
    '}';
```

**Target:**
```
Loop returns Loop:
    'while'
    (loopCondition=Condition)?
    bodyInteraction=Interaction
    ;
```

**Transformation:**
- Replace `'Loop'` keyword with `'while'`
- Remove braces
- Remove attribute labels
- Reorder: condition before body
- Make bodyInteraction mandatory

#### Example 3: Alternative
**Generated:**
```
Alternative returns Alternative:
    {Alternative}
    'Alternative'
    '{'
        ('cases' '{' cases+=Case ( "," ...)* '}' )?
    '}';
```

**Target:**
```
Alternative returns Alternative:
    {Alternative}
    'Alternative'
    cases+=Case ('or' cases+=Case)*
    ;
```

**Transformation:**
- Keep keyword
- Remove outer braces
- Replace comma separator with `'or'` keyword
- Change from optional to mandatory (at least one case)

### Rule 4.2: Condition Syntax Transformation
**Pattern:** Transform condition rules to use bracket notation

**Examples:**

**WaitCondition:**
- Generated: Verbose with `'WaitCondition'` keyword and braces
- Target:
```
WaitCondition returns WaitCondition:
    'wait'
    (strict?='strict')?
    (requested?='eventually')?
    '['
    conditionExpression=ConditionExpression
    ']';
```

**InterruptCondition:**
```
InterruptCondition returns InterruptCondition:
    'interrupt'
    '['
    conditionExpression=ConditionExpression
    ']';
```

**ViolationCondition:**
```
ViolationCondition returns ViolationCondition:
    'violation'
    '['
    conditionExpression=ConditionExpression
    ']';
```

**Condition:**
```
Condition returns Condition:
    '['
    conditionExpression=ConditionExpression
    ']';
```

### Rule 4.3: Timed Condition Syntax
**Pattern:** Add 'timed' prefix and bracket notation

**TimedWaitCondition:**
```
TimedWaitCondition returns TimedWaitCondition:
    'timed' 'wait'
    (strict?='strict')?
    (requested?='eventually')?
    '['
    timedConditionExpression=DummyExprClass
    ']';
```

**TimedInterruptCondition:**
```
TimedInterruptCondition returns TimedInterruptCondition:
    'timed' 'interrupt'
    '['
    timedConditionExpression=DummyExprClass
    ']';
```

**TimedViolationCondition:**
```
TimedViolationCondition returns TimedViolationCondition:
    'timed' 'violation'
    '['
    timedConditionExpression=DummyExprClass
    ']';
```

### Rule 4.4: Role Syntax Transformation
**Generated:**
```
Role returns Role:
    {Role}
    (static?='static')?
    (multiRole?='multiRole')?
    'Role'
    name=EString
    '{'
        ('type' type=[ecore::EClass|EString])?
    '}';
```

**Target:**
```
Role returns Role:
    (static?='static' |
    'dynamic' (multiRole?='multi')?)
    'role'
    type=[ecore::EClass]
    name=ID
    ;
```

**Changes:**
- Add alternative: `static?='static'` OR `'dynamic' (multiRole?='multi')?`
- Change keyword: `'Role'` → `'role'`
- Remove braces
- Make type mandatory and move before name
- Change type reference format

### Rule 4.5: Scenario Syntax Transformation
**Generated:**
```
Scenario returns Scenario:
    (singular?='singular')?
    optimizeCost?='optimizeCost'
    'Scenario'
    name=EString
    '{'
        ('kind' kind=ScenarioKind)?
        'cost' cost=EDouble
        ('contexts' '(' contexts+=[ecore::EClass|EString] ( "," ...)* ')' )?
        ('roleBindings' '{' roleBindings+=RoleBindingConstraint ( "," ...)* '}' )?
        ('ownedInteraction' ownedInteraction=Interaction)?
    '}';
```

**Target:**
```
Scenario returns Scenario:
    (singular?='singular')?
    kind=ScenarioKind
    'scenario'
    name=ID
    ((optimizeCost?='optimize' 'cost') |
    ('cost' '[' cost=DOUBLE ']'))?
    ('context' contexts+=[ecore::EClass] ( "," ...)*)?
    ('bindings' '[' (roleBindings+=RoleBindingConstraint)* ']')?
    ownedInteraction=Interaction
    ;
```

**Changes:**
- Move kind before keyword
- Change keyword: `'Scenario'` → `'scenario'`
- Transform cost: either `'optimize' 'cost'` or `'cost' '[' value ']'`
- Change contexts label to singular `'context'`
- Change bindings from braces to brackets
- Remove attribute labels inside braces
- Make ownedInteraction mandatory

---

## 5. RULE CONSOLIDATION AND GROUPING

### Rule 5.1: Fragment Type Grouping
**Pattern:** Create consolidated grouping rules

**Added rules:**
```
TimedConditionFragment returns TimedConditionFragment:
    TimedWaitCondition | TimedInterruptCondition | TimedViolationCondition;

ConditionFragment returns ConditionFragment:
    WaitCondition | InterruptCondition | ViolationCondition;
```

### Rule 5.2: InteractionFragment Simplification
**Generated:**
```
InteractionFragment returns InteractionFragment:
    Interaction | ModalMessage | Alternative | Loop | Parallel | 
    WaitCondition | InterruptCondition | ViolationCondition | Condition | 
    VariableFragment | TimedViolationCondition | TimedInterruptCondition | 
    TimedWaitCondition | ClockFragment;
```

**Target:**
```
InteractionFragment returns InteractionFragment:
    Interaction | ModalMessage | Alternative | Loop | Parallel | 
    VariableFragment | ConditionFragment | TimedConditionFragment;
```

**Changes:**
- Replace individual condition rules with `ConditionFragment`
- Replace individual timed condition rules with `TimedConditionFragment`
- Remove `Condition` (separate simple bracket notation)
- Remove `ClockFragment`

### Rule 5.3: ParameterExpression Simplification
**Generated:**
```
ParameterExpression returns ParameterExpression:
    WildcardParameterExpression | ValueParameterExpression | 
    VariableBindingParameterExpression
    ;
```

**Target:** (Same structure, keep as is)

---

## 6. SPECIFIC RULE TRANSFORMATIONS

### Rule 6.1: EventParameterRanges Transformation
**Generated:**
```
EventParameterRanges returns EventParameterRanges:
    {EventParameterRanges}
    'EventParameterRanges'
    '{'
        ('event' event=[ecore::ETypedElement|EString])?
        ('rangesForParameter' '{' rangesForParameter+=RangesForParameter ( "," ...)* '}' )?
    '}';
```

**Target:**
```
EventParameterRanges returns EventParameterRanges:
    event=[ecore::ETypedElement|FQN]
    '(' rangesForParameter+=RangesForParameter ( "," ...)* ')'
    ;
```

### Rule 6.2: ChannelOptions Transformation
**Generated:**
```
ChannelOptions returns ChannelOptions:
    {ChannelOptions}
    (allMessagesRequireLinks?='allMessagesRequireLinks')?
    'ChannelOptions'
    '{'
        ('messageChannels' '{' messageChannels+=MessageChannel ( "," ...)* '}' )?
    '}';
```

**Target:**
```
ChannelOptions returns ChannelOptions:
    {ChannelOptions}
    ('channels'
    '{'
    (allMessagesRequireLinks?='allMessagesRequireLinks')?
    (messageChannels+=MessageChannel)*
    '}')?;
```

**Changes:**
- Remove `'ChannelOptions'` keyword
- Add `'channels'` wrapper keyword
- Remove inner attribute label
- Remove comma separators
- Make entire structure optional

### Rule 6.3: Interaction Transformation
**Generated:**
```
Interaction returns Interaction:
    {Interaction}
    'Interaction'
    '{'
        ('fragments' '{' fragments+=InteractionFragment ( "," ...)* '}' )?
        ('constraints' constraints=ConstraintBlock)?
    '}';
```

**Target:**
```
Interaction returns Interaction:
    {Interaction}
    '{' (fragments+=InteractionFragment)* '}'
    (constraints=ConstraintBlock)?
    ;
```

**Changes:**
- Remove `'Interaction'` keyword
- Remove `'fragments'` label
- Remove comma separators for fragments
- Make fragments directly contained in braces

### Rule 6.4: ConstraintBlock Transformation
**Generated:**
```
ConstraintBlock returns ConstraintBlock:
    {ConstraintBlock}
    'ConstraintBlock'
    '{'
        ('consider' '{' consider+=Message ( "," ...)* '}' )?
        ('ignore' '{' ignore+=Message ( "," ...)* '}' )?
        ('interrupt' '{' interrupt+=Message ( "," ...)* '}' )?
        ('forbidden' '{' forbidden+=Message ( "," ...)* '}' )?
    '}';
```

**Target:**
```
ConstraintBlock returns ConstraintBlock:
    {ConstraintBlock}
    'constraints'
    '['
        ('consider' consider+=ConstraintMessage |
        'ignore' ignore+=ConstraintMessage |
        'forbidden' forbidden+=ConstraintMessage |
        'interrupt' interrupt+=ConstraintMessage)*
    ']';
```

**Changes:**
- Change keyword: `'ConstraintBlock'` → `'constraints'`
- Change delimiters: `'{' ... '}'` → `'[' ... ']'`
- Remove inner braces for each category
- Use alternatives with `|` instead of sequence
- Make repeating with `*`
- Reference `ConstraintMessage` instead of `Message`

### Rule 6.5: Collaboration Transformation
**Generated:**
```
Collaboration returns Collaboration:
    {Collaboration}
    'Collaboration'
    name=EString
    '{'
        ('domains' '(' domains+=[ecore::EPackage|EString] ( "," ...)* ')' )?
        ('contexts' '(' contexts+=[ecore::EPackage|EString] ( "," ...)* ')' )?
        ('imports' '{' imports+=DummyExprClass ( "," ...)* '}' )?
        ('roles' '{' roles+=Role ( "," ...)* '}' )?
        ('scenarios' '{' scenarios+=Scenario ( "," ...)* '}' )?
    '}';
```

**Target:**
```
Collaboration returns Collaboration:
    (imports+=DummyExprClass)*  
    ('domain' domains+=[ecore::EPackage|FQN])*
    ('contexts' contexts+=[ecore::EPackage|FQN])*
    'collaboration'
    name=ID
    '{'
        (roles+=Role)*
        (scenarios+=Scenario)*
    '}';
```

**Changes:**
- Move imports before keyword
- Change keyword case: `'Collaboration'` → `'collaboration'`
- Make domains/contexts singular labels
- Remove parentheses for domains/contexts
- Remove labels for roles/scenarios
- Remove comma separators

### Rule 6.6: NestedCollaboration Addition
**Pattern:** Add specialized nested collaboration rule

```
NestedCollaboration returns Collaboration:
    'collaboration'
    name=ID
    '{'
        roles+=Role*
        scenarios+=Scenario*
    '}';
```

**Note:** Simplified version without domains/contexts/imports

### Rule 6.7: AbstractRanges Transformation
**Generated:**
```
AbstractRanges returns AbstractRanges:
    IntegerRanges | StringRanges | EnumRanges;
```

**Target:**
```
AbstractRanges returns AbstractRanges:
    '[' (IntegerRanges | StringRanges | EnumRanges) ']';
```

**Change:** Wrap alternatives in brackets

### Rule 6.8: IntegerRanges Transformation
**Generated:**
```
IntegerRanges returns IntegerRanges:
    {IntegerRanges}
    'IntegerRanges'
    '{'
        ('min' min=EInt)?
        ('max' max=EInt)?
        ('values' '{' values+=EInt ( "," ...)* '}' )?
    '}';
```

**Target:**
```
IntegerRanges returns IntegerRanges:
    (min=(INT | SIGNEDINT) '..'
    max=(INT | SIGNEDINT))
    (','  values+=(INT | SIGNEDINT) ( "," ...)*  )? | 
    values+=(INT | SIGNEDINT) ("," values+=(INT | SIGNEDINT))*
    ;
```

**Changes:**
- Remove keyword and braces
- Add range syntax: `min '..' max`
- Add alternative for list-only syntax
- Use `INT | SIGNEDINT` instead of `EInt`

### Rule 6.9: StringRanges Transformation
**Generated:**
```
StringRanges returns StringRanges:
    {StringRanges}
    'StringRanges'
    '{'
        ('values' '{' values+=EString ( "," ...)* '}' )?
    '}';
```

**Target:**
```
StringRanges returns StringRanges:
    (values+=STRING ( "," ...)*  )
    ;
```

**Changes:**
- Remove keyword and braces
- Remove attribute label
- Use `STRING` instead of `EString`
- Make mandatory

### Rule 6.10: EnumRanges Transformation
**Generated:**
```
EnumRanges returns EnumRanges:
    {EnumRanges}
    'EnumRanges'
    '{'
        ('values' '(' values+=[ecore::EEnumLiteral|EString] ( "," ...)* ')' )?
    '}';
```

**Target:**
```
EnumRanges returns EnumRanges:
    (values+=[ecore::EEnumLiteral|FQNENUM] ( "," ...)*  )
    ;
```

**Changes:**
- Remove keyword and braces
- Remove attribute label and parentheses
- Use `FQNENUM` instead of `EString`
- Make mandatory

### Rule 6.11: MessageChannel Transformation
**Generated:**
```
MessageChannel returns MessageChannel:
    {MessageChannel}
    'MessageChannel'
    '{'
        ('event' event=[ecore::ETypedElement|EString])?
        ('channelFeature' channelFeature=[ecore::EStructuralFeature|EString])?
    '}';
```

**Target:**
```
MessageChannel returns MessageChannel:
    event=[ecore::ETypedElement|FQN]
    'over' channelFeature=[ecore::EStructuralFeature|FQN]
    ;
```

**Changes:**
- Remove keyword and braces
- Add `'over'` connecting keyword
- Make both attributes mandatory

### Rule 6.12: ParameterBinding Transformation
**Generated:**
```
ParameterBinding returns ParameterBinding:
    {ParameterBinding}
    'ParameterBinding'
    '{'
        ('bindingExpression' bindingExpression=ParameterExpression)?
    '}';
```

**Target:**
```
ParameterBinding returns ParameterBinding:
    bindingExpression=ParameterExpression
    ;
```

**Changes:**
- Complete simplification to single reference
- Remove all keywords and wrappers

### Rule 6.13: FeatureAccessBindingExpression Transformation
**Generated:**
```
FeatureAccessBindingExpression returns FeatureAccessBindingExpression:
    {FeatureAccessBindingExpression}
    'FeatureAccessBindingExpression'
    '{'
        ('featureaccess' featureaccess=DummyExprClass)?
    '}';
```

**Target:**
```
FeatureAccessBindingExpression returns FeatureAccessBindingExpression:
    featureaccess=DummyExprClass
    ;
```

### Rule 6.14: WildcardParameterExpression Transformation
**Generated:**
```
WildcardParameterExpression returns WildcardParameterExpression:
    {WildcardParameterExpression}
    'WildcardParameterExpression'
    ;
```

**Target:**
```
WildcardParameterExpression returns WildcardParameterExpression:
    '*'
    ;
```

**Change:** Replace keyword with wildcard symbol

### Rule 6.15: ValueParameterExpression Transformation
**Generated:**
```
ValueParameterExpression returns ValueParameterExpression:
    {ValueParameterExpression}
    'ValueParameterExpression'
    '{'
        ('value' value=DummyExprClass)?
    '}';
```

**Target:**
```
ValueParameterExpression returns ValueParameterExpression:
    value=DummyExprClass
    ;
```

### Rule 6.16: VariableBindingParameterExpression Transformation
**Generated:**
```
VariableBindingParameterExpression returns VariableBindingParameterExpression:
    {VariableBindingParameterExpression}
    'VariableBindingParameterExpression'
    '{'
        ('variable' variable=DummyExprClass)?
    '}';
```

**Target:**
```
VariableBindingParameterExpression returns VariableBindingParameterExpression:
    'bind'
    variable=DummyExprClass
    ;
```

**Change:** Replace keyword with `'bind'`, remove braces

### Rule 6.17: Case Transformation
**Generated:**
```
Case returns Case:
    {Case}
    'Case'
    '{'
        ('caseInteraction' caseInteraction=Interaction)?
        ('caseCondition' caseCondition=Condition)?
    '}';
```

**Target:**
```
Case returns Case:
    {Case}
    (caseCondition=Condition)?
    caseInteraction=Interaction
    ;
```

**Changes:**
- Remove keyword and braces
- Remove attribute labels
- Reorder: condition before interaction
- Make interaction mandatory

### Rule 6.18: ConditionExpression Transformation
**Generated:**
```
ConditionExpression returns ConditionExpression:
    {ConditionExpression}
    'ConditionExpression'
    '{'
        ('expression' expression=DummyExprClass)?
    '}';
```

**Target:**
```
ConditionExpression returns ConditionExpression:
    expression=DummyExprClass
    ;
```

### Rule 6.19: ConstraintMessage Creation
**Pattern:** Create specialized message rule for constraints

**Target:**
```
ConstraintMessage returns Message:
    sender=[Role] '->'
    receiver=[Role] '.'
    modelElement=[ecore::ETypedElement]
    ('.' collectionModification=DummyDatatype)?
    '(' (parameters+=ParameterBinding ( "," ...)*)? ')'
    ;
```

**Note:** Simplified message without strict/monitored/expectationKind flags

### Rule 6.20: RoleBindingConstraint Transformation
**Generated:**
```
RoleBindingConstraint returns RoleBindingConstraint:
    {RoleBindingConstraint}
    'RoleBindingConstraint'
    '{'
        ('role' role=[Role|EString])?
        ('bindingExpression' bindingExpression=BindingExpression)?
    '}';
```

**Target:**
```
RoleBindingConstraint returns RoleBindingConstraint:
    role=[Role] '=' 
    bindingExpression=BindingExpression
    ;
```

**Changes:**
- Remove keyword and braces
- Add `'='` operator
- Make both attributes mandatory

### Rule 6.21: RangesForParameter Transformation
**Generated:**
```
RangesForParameter returns RangesForParameter:
    {RangesForParameter}
    'RangesForParameter'
    '{'
        ('parameter' parameter=[ecore::ETypedElement|EString])?
        ('ranges' ranges=AbstractRanges)?
    '}';
```

**Target:**
```
RangesForParameter returns RangesForParameter:
    parameter=[ecore::ETypedElement]
    '=' ranges=AbstractRanges
    ;
```

### Rule 6.22: VariableFragment Transformation
**Generated:**
```
VariableFragment returns VariableFragment:
    {VariableFragment}
    'VariableFragment'
    '{'
        ('expression' expression=DummyExprClass)?
    '}';
```

**Target:**
```
VariableFragment returns VariableFragment:
    expression=DummyExprClass
    ;
```

### Rule 6.23: BindingExpression Simplification
**Generated:**
```
BindingExpression returns BindingExpression:
    FeatureAccessBindingExpression | ObjectQueryBindingExpression | 
    ParameterExpression_Impl | WildcardParameterExpression | 
    ValueParameterExpression | VariableBindingParameterExpression;
```

**Target:**
```
BindingExpression returns BindingExpression:
    FeatureAccessBindingExpression;
```

**Change:** Reduce to single alternative (others removed or moved)

---

## 7. ATTRIBUTE AND FLAG MODIFICATIONS

### Rule 7.1: Boolean Flag Value Changes
**Pattern:** Change flag literal values

**Examples:**
- `requested?='requested'` → `requested?='eventually'` (in WaitCondition)
- `optimizeCost?='optimizeCost'` → part of `'optimize' 'cost'` phrase (in Scenario)

### Rule 7.2: Mandatory vs Optional Changes
**Pattern:** Change cardinality of elements

**Examples:**
- Make bodyInteraction mandatory in Loop
- Make ownedInteraction mandatory in Scenario
- Make event mandatory in EventParameterRanges
- Make both attributes mandatory in MessageChannel

---

## 8. REMOVED ELEMENTS

### Rule 8.1: Remove Complete Rules
**Pattern:** Delete entire rule definitions

**Removed rules:**
- `ClockFragment`
- `ObjectQueryBindingExpression`
- `ObjectQueryValue`
- `ParameterExpression_Impl`
- `Message_Impl`
- All Ecore metamodel rules (EPackage, EClass, EAttribute, etc.)
- All Ecore type infrastructure rules

### Rule 8.2: Remove from Alternative Lists
**Pattern:** Remove specific alternatives from union types

**Examples:**
- Remove `ClockFragment` from InteractionFragment
- Remove `ObjectQueryBindingExpression` from BindingExpression

---

## 9. FORMATTING AND WHITESPACE

### Rule 9.1: Line Break Hints
**Pattern:** Add `->` for formatting hints

**Example:**
```
->('domain' domains+=[ecore::EPackage|FQN])*
```

### Rule 9.2: Consistent Spacing
**Pattern:** Standardize whitespace in rule bodies
- Single space after keywords
- Consistent spacing around operators
- Aligned multi-line structures

---

## Summary Statistics

### Transformation Categories:
1. **Import modifications:** 3 rules
2. **Root rule restructuring:** 1 major rule (Specification)
3. **Type system enhancements:** 20+ reference type changes, 4 new terminal rules
4. **Syntactic enhancements:** 23 rule transformations
5. **Rule consolidations:** 3 grouping rules
6. **Terminal additions:** 4 new rules
7. **Rule removals:** 15+ complete rules removed

### Complexity Metrics:
- **High complexity transformations:** Specification, Scenario, ModalMessage, Loop
- **Medium complexity:** Role, EventParameterRanges, Collaboration, ConstraintBlock
- **Low complexity:** Parameter expressions, ranges, simple wrappers
- **Total rules transformed:** ~35 rules
- **Total rules removed:** ~15 rules
- **Total rules added:** ~6 rules

### Key Adaptation Patterns:
1. **Verbose to concise:** Remove keywords and attribute labels
2. **Bracketed structures:** Transform braces to brackets for constraints/conditions
3. **Operator-based syntax:** Add '->', '.', '=', 'over' operators
4. **Natural language keywords:** 'while', 'wait', 'bind', 'over'
5. **Type specialization:** EString → ID/FQN/FQNENUM
6. **Mandatory refinement:** Make previously optional elements required
7. **Structural flattening:** Remove nested braces, inline elements
