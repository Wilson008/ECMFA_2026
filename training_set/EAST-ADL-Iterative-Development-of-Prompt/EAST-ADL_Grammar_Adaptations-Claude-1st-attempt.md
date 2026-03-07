# EAST-ADL Grammar Adaptation Analysis
## From Generated Grammar to Target Grammar

This document identifies all necessary adaptations to transform the generated EAST-ADL grammar into the target grammar.

---

## **1. Root Rule Simplification**

### **Rule: EAXML**

**Generated Grammar (Lines 7-12):**
```
EAXML returns EAXML:
    {EAXML}
    'EAXML'
    '{'
        ('topLevelPackage' '{' topLevelPackage+=EAPackage ( "," topLevelPackage+=EAPackage)* '}' )?
    '}';
```

**Target Grammar (Lines 7-12):**
```
EAXML returns EAXML:
    {EAXML}
    
    
        (  topLevelPackage+=EAPackage (  topLevelPackage+=EAPackage)*  )?
    ;
```

**Adaptation Required:**
- **Delete keywords**: Remove `'EAXML'`, `'{'`, `'}'`
- **Delete nested keywords**: Remove `'topLevelPackage'`, `'{'`, `'}'`
- **Delete separator**: Remove `","`
- **Simplify structure**: Direct reference to `topLevelPackage+=EAPackage` without nesting

**Summary**: Remove all structural keywords and braces, making the rule a simple optional repeating pattern.

---

## **2. Abstract Rule Removals**

The following abstract/grouping rules exist in the generated grammar but are **deleted entirely** in the target grammar:

### **Deleted Rules:**

1. **Line 36-37**: `EAValue` rule - completely removed
2. **Line 39-40**: `FeatureTreeNode` rule - completely removed  
3. **Line 42-43**: `Feature` rule - completely removed
4. **Line 50-51**: `FunctionPort` rule - completely removed
5. **Line 54-55**: `HardwareComponentType` rule - completely removed
6. **Line 57-58**: `HardwarePin` rule - completely removed
7. **Line 62-63**: `PortConnector` rule - completely removed
8. **Line 66-67**: `FunctionPrototype` rule - completely removed
9. **Line 73-74**: `FunctionType` rule - completely removed
10. **Line 83-84**: `RequirementsHierarchy` rule - completely removed
11. **Line 86-87**: `Requirement` rule - completely removed
12. **Line 91-92**: `RequirementsRelationship` rule - completely removed
13. **Line 108-109**: `TimingDescription` rule - completely removed
14. **Line 111-112**: `TimingConstraint` rule - completely removed
15. **Line 125-126**: `EADatatype` rule - completely removed
16. **Line 141**: `Identifiable` rule - completely removed (but reconstructed with modifications)

**Note**: Some of these rules are reconstructed in the target grammar with modifications (see Section 3).

---

## **3. Abstract Rule Reconstructions**

The following rules are removed from their original position but reconstructed with different content:

### **3.1 FunctionPort (Lines 35-36)**

**Target Grammar:**
```
FunctionPort returns FunctionPort:
    FunctionClientServerPort | FunctionFlowPort | FunctionPowerPort;
```

**Change**: Same alternatives, but positioned differently in the file.

### **3.2 HardwareComponentType (Lines 38-39)**

**Target Grammar:**
```
HardwareComponentType returns HardwareComponentType:
    HardwareComponentType_Impl | Actuator | ElectricalComponent | Node | Sensor;
```

**Change**: Same alternatives, repositioned.

### **3.3 HardwarePin (Lines 41-42)**

**Target Grammar:**
```
HardwarePin returns HardwarePin:
    CommunicationHardwarePin | IOHardwarePin | PowerHardwarePin;
```

**Change**: Same alternatives, repositioned.

### **3.4 PortConnector (Lines 44-45)**

**Target Grammar:**
```
PortConnector returns PortConnector:
    HardwarePortConnector | LogicalPortConnector;
```

**Change**: Same alternatives, repositioned.

### **3.5 FunctionPrototype (Lines 47-48)**

**Target Grammar:**
```
FunctionPrototype returns FunctionPrototype:
    AnalysisFunctionPrototype | DesignFunctionPrototype;
```

**Change**: Same alternatives, repositioned.

### **3.6 FunctionType (Lines 50-51)**

**Target Grammar:**
```
FunctionType returns FunctionType:
    AnalysisFunctionType_Impl | BasicSoftwareFunctionType | DesignFunctionType_Impl | FunctionalDevice | HardwareFunctionType | LocalDeviceManager;
```

**Change**: Same alternatives, repositioned.

### **3.7 RequirementsHierarchy (Lines 53-54)**

**Target Grammar:**
```
RequirementsHierarchy returns RequirementsHierarchy:
    RequirementsHierarchy_Impl | FunctionalSafetyConcept | TechnicalSafetyConcept;
```

**Change**: Same alternatives, repositioned.

### **3.8 Requirement (Lines 56-57)**

**Target Grammar:**
```
Requirement returns Requirement:
    Requirement_Impl | QualityRequirement;
```

**Change**: Same alternatives, repositioned.

### **3.9 RequirementsRelationship (Lines 59-60)**

**Target Grammar:**
```
RequirementsRelationship returns RequirementsRelationship:
    DeriveRequirement | Refine | Satisfy | RequirementsLink | Verify;
```

**Change**: Same alternatives, repositioned.

### **3.10 TimingDescription (Lines 62-63)**

**Target Grammar:**
```
TimingDescription returns TimingDescription:
    EventChain | AUTOSAREvent | EventFaultFailure | EventFeatureFlaw | EventFunction | EventFunctionClientServerPort | EventFunctionFlowPort | ExternalEvent | ModeEvent | StateEvent;
```

**Change**: Same alternatives, repositioned.

### **3.11 TimingConstraint (Lines 65-66)**

**Target Grammar:**
```
TimingConstraint returns TimingConstraint:
    NonConcurrenceConstraint | NonPreemptiveConstraint | PrecedenceConstraint | AgeConstraint | ArbitraryConstraint | BurstConstraint | DelayConstraint | ExecutionTimeConstraint | InputSynchronizationConstraint | OrderConstraint | OutputSynchronizationConstraint | PatternConstraint | PeriodicConstraint | ReactionConstraint | RepetitionConstraint | SporadicConstraint | StrongDelayConstraint | StrongSynchronizationConstraint | SynchronizationConstraint;
```

**Change**: Same alternatives, repositioned.

### **3.12 EADatatype (Lines 68-69)**

**Target Grammar:**
```
EADatatype returns EADatatype:
    ArrayDatatype | CompositeDatatype | EABoolean | EANumerical | EAString | Enumeration | RangeableValueType;
```

**Change**: Same alternatives, repositioned.

### **3.13 Identifiable (Line 71-72)**

**Generated Grammar (Line 141):**
```
Identifiable returns Identifiable:
    [Very long list of alternatives...]
```

**Target Grammar (Lines 71-72):**
```
Identifiable returns Identifiable:
    [Very long list - appears to be the same or similar alternatives, repositioned]
```

**Change**: Repositioned from line 141 to line 71-72, content appears preserved.

---

## **4. New Abstract Rules Added**

The target grammar introduces several **new abstract rules** that don't exist in the generated grammar:

### **4.1 GenericConstraint (Lines 74-75)**

**Target Grammar:**
```
GenericConstraint returns GenericConstraint:
    GenericConstraint_Impl | TakeRateConstraint;
```

**Adaptation**: Add new rule grouping GenericConstraint types.

### **4.2 BehaviorConstraintInternalBinding (Lines 77-78)**

**Target Grammar:**
```
BehaviorConstraintInternalBinding returns BehaviorConstraintInternalBinding:
    BehaviorConstraintBindingAttribute | BehaviorConstraintBindingEvent;
```

**Adaptation**: Add new rule grouping behavior constraint bindings.

### **4.3 BehaviorConstraintParameter (Lines 80-81)**

**Target Grammar:**
```
BehaviorConstraintParameter returns BehaviorConstraintParameter:
    BehaviorConstraintBindingAttribute | BehaviorConstraintBindingEvent | Attribute_Impl | TransitionEvent_Impl;
```

**Adaptation**: Add new rule grouping behavior constraint parameters.

### **4.4 Attribute (Lines 83-84)**

**Target Grammar:**
```
Attribute returns Attribute:
    Attribute_Impl | BehaviorConstraintBindingAttribute;
```

**Adaptation**: Add new rule grouping Attribute types.

### **4.5 Anomaly (Lines 86-87)**

**Target Grammar:**
```
Anomaly returns Anomaly:
    FailureOutPort | FaultInPort | InternalFaultPrototype | ProcessFaultPrototype;
```

**Adaptation**: Add new rule grouping Anomaly types.

### **4.6 AnalysisFunctionType (Lines 89-90)**

**Target Grammar:**
```
AnalysisFunctionType returns AnalysisFunctionType:
    AnalysisFunctionType_Impl | FunctionalDevice;
```

**Adaptation**: Add new rule grouping AnalysisFunctionType.

### **4.7 DesignFunctionType (Lines 92-93)**

**Target Grammar:**
```
DesignFunctionType returns DesignFunctionType:
    DesignFunctionType_Impl | BasicSoftwareFunctionType | HardwareFunctionType | LocalDeviceManager;
```

**Adaptation**: Add new rule grouping DesignFunctionType.

### **4.8 ConfigurationDecisionModelEntry (Lines 95-96)**

**Target Grammar:**
```
ConfigurationDecisionModelEntry returns ConfigurationDecisionModelEntry:
    ConfigurationDecision | ConfigurationDecisionFolder;
```

**Adaptation**: Add new rule grouping configuration decision entries.

### **4.9 Event (Lines 98-99)**

**Target Grammar:**
```
Event returns Event:
    AUTOSAREvent | EventFaultFailure | EventFeatureFlaw | EventFunction | EventFunctionClientServerPort | EventFunctionFlowPort | ExternalEvent | ModeEvent | StateEvent;
```

**Adaptation**: Add new rule grouping Event types.

### **4.10 Quantification (Lines 101-102)**

**Target Grammar:**
```
Quantification returns Quantification:
    Quantification_Impl | AttributeQuantificationConstraint;
```

**Adaptation**: Add new rule grouping Quantification types.

---

## **5. Terminal Statement Separator Changes**

Throughout many data structure rules, there's a systematic pattern change:

### **Pattern in Generated Grammar:**
```
'attributeName' attributeValue
```
(No semicolon after single-valued attributes)

### **Pattern in Target Grammar:**
```
'attributeName' attributeValue ';'
```
(Semicolon added after single-valued attributes)

### **Examples:**

**Generated Grammar (Line 3190):**
```
'identifiable_target' identifiable_target=[EAElement|EString]
```

**Target Grammar (Line 3055):**
```
'identifiable_target' identifiable_target=[EAElement|EString] ';'
```

**Affected Rules (Sample):**
- `Realization_realized` (Line 3055)
- `Realization_realizedBy` (Line 3062)
- `VVCase_vvSubject` (Line 3075)
- `VVTarget_element` (Line 3083)
- `FaultFailure_anomaly` (Line 3102)
- `BehaviorConstraintPrototype_errorModelTarget` (Line 3115)
- `BehaviorConstraintPrototype_functionTarget` (Line 3121)
- `BehaviorConstraintPrototype_hardwareComponentTarget` (Line 3128)

**Adaptation Pattern:**
- Add semicolon `;` after single-valued attribute assignments in data structure rules
- This applies systematically across many rules where single attributes appear

---

## **6. Multi-Valued Attribute Separator Changes**

### **Pattern in Generated Grammar:**
```
( "," element+=Type)*
```
(Comma separator between repeated elements)

### **Pattern in Target Grammar:**
```
(  element+=Type)*
```
(No separator, just whitespace)

### **Examples:**

**Generated Grammar (Line 3101):**
```
('errorModelPrototype' '(' errorModelPrototype+=[ErrorModelPrototype|EString] ( "," errorModelPrototype+=[ErrorModelPrototype|EString])* ')' )?
```

**Target Grammar (Line 3101):**
```
('errorModelPrototype' '(' errorModelPrototype+=[ErrorModelPrototype|EString] (  errorModelPrototype+=[ErrorModelPrototype|EString])* ')' )?
```

**Affected Rules (Sample):**
- `FaultFailure_anomaly` (Line 3101)
- `BehaviorConstraintPrototype_errorModelTarget` (Line 3114)
- `BehaviorConstraintPrototype_functionTarget` (Line 3122)
- `BehaviorConstraintPrototype_hardwareComponentTarget` (Line 3129)

**Adaptation Pattern:**
- Remove comma separator `","` from multi-valued attribute repetitions
- Keep only whitespace between repeated elements

---

## **7. Enum Indentation Changes**

### **Pattern in Generated Grammar:**
```
enum EnumName returns EnumName:
                    VALUE1 = 'VALUE1' | VALUE2 = 'VALUE2' | ...;
```
(Tab indentation with multiple tabs)

### **Pattern in Target Grammar:**
```
enum EnumName returns EnumName:
                VALUE1 = 'VALUE1' | VALUE2 = 'VALUE2' | ...;
```
(Space indentation, reduced indentation level)

**Affected Enums:**
- `ASILKind` (Line 3163/3067)
- `FunctionBehaviorKind` (Line 3201/3066)
- `QualityRequirementKind` (Line 3204/3069)
- `ControllabilityClassKind` (Line 3221/3086)
- `ExposureClassKind` (Line 3224/3089)
- `SeverityClassKind` (Line 3227/3092)
- `DevelopmentCategoryKind` (Line 3230/3095)
- `LifecycleStageKind` (Line 3240/3105)
- `GenericConstraintKind` (Line 3243/3108)

**Adaptation Pattern:**
- Change indentation from tabs to spaces
- Reduce indentation depth (appears to use fewer leading characters)

---

## **8. New Terminal and Type Rules Added**

The target grammar adds several new rules at the end:

### **8.1 EString Type Rule (Lines 3132-3133)**

**Target Grammar:**
```
EString returns ecore::EString:
    STRING | ID;
```

**Adaptation**: Add new type rule defining EString as either STRING or ID terminal.

### **8.2 Terminal EABINARY (Lines 3135-3136)**

**Target Grammar:**
```
terminal EABINARY:
    ('0b'('0'..'1')*);
```

**Adaptation**: Add new terminal rule for binary literals.

### **8.3 Terminal EAOCTAL (Lines 3138-3139)**

**Target Grammar:**
```
terminal EAOCTAL:
    ('0'('1'..'7')('0'..'7')*);
```

**Adaptation**: Add new terminal rule for octal literals.

### **8.4 Terminal EAHEX (Lines 3140-3142)**

**Target Grammar:**
```
terminal EAHEX:
    ('0x'('0'..'9'|'a'..'f')*);
```

**Adaptation**: Add new terminal rule for hexadecimal literals.

### **8.5 Terminal EAEXPONENT (Lines 3144-3145)**

**Target Grammar:**
```
terminal EAEXPONENT:
    ('0'..'9')+('e'|'E')('+'|'-')?('0'..'9')+;
```

**Adaptation**: Add new terminal rule for scientific notation numbers.

### **8.6 UUID Type Rule (Lines 3147-3148)**

**Target Grammar:**
```
UUID returns ecore::EString:
    STRING;
```

**Adaptation**: Add new type rule defining UUID as STRING terminal.

---

## **Summary of Adaptation Categories**

### **A. Structural Simplifications**
1. Root rule keyword removal (EAXML)
2. Removal of structural keywords and braces

### **B. Rule Reorganization**
1. Deletion of abstract rules from original positions
2. Repositioning of abstract rules to earlier in grammar
3. Preservation of rule alternatives during reorganization

### **C. New Rule Additions**
1. 10 new abstract grouping rules added
2. 6 new terminal and type rules added

### **D. Syntax Pattern Changes**
1. Addition of semicolons after single-valued attributes
2. Removal of comma separators in multi-valued attributes
3. Indentation style changes in enum declarations

### **E. Type System Enhancements**
1. New numeric literal terminals (binary, octal, hex, exponent)
2. New type rules (EString, UUID)

---

## **Adaptation Complexity Assessment**

**Total Identified Adaptations: ~60+ distinct changes**

**Categories by Frequency:**
- Semicolon additions: ~8-10 rules
- Comma removals: ~4-6 rules
- Enum reformatting: ~9 enums
- Rule deletions: ~15 rules
- Rule repositions: ~13 rules
- New rule additions: ~16 rules
- Root rule simplification: 1 major change
- Terminal additions: 6 new terminals/types

**Complexity Level: High**
- Requires deep understanding of grammar structure
- Involves both deletion and creation operations
- Includes systematic pattern changes across many rules
- Requires precise handling of whitespace and punctuation
