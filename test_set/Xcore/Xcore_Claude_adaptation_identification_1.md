# Xcore Grammar Adaptation Rules Analysis

## Overview
This document identifies the systematic adaptation rules required to transform the Xcore generated grammar into the target grammar. The analysis covers structural, syntactic, type system, and terminal enhancements.

---

## Category 1: Root Rule & Import System Modifications

### AR1.1: Grammar Inheritance Change
**Pattern**: Change base grammar inheritance
- **From**: `grammar org.xtext.example.xcore.Xcore with org.eclipse.xtext.common.Terminals`
- **To**: `grammar org.xtext.example.xcore.Xcore with org.eclipse.xtext.xbase.Xbase`
- **Rationale**: Switch from basic Terminals to Xbase for expression support

### AR1.2: Additional Import Statements
**Pattern**: Add four new import statements after grammar declaration
- **Add**:
  ```
  import "http://www.eclipse.org/xtext/xbase/Xbase" as xbase
  import "http://www.eclipse.org/emf/2002/GenModel" as genmodel
  import "http://www.eclipse.org/xtext/common/JavaVMTypes" as javaVMTypes
  import "http://www.eclipse.org/xtext/xbase/Xtype" as xtype
  ```
- **Location**: After grammar declaration, before existing imports
- **Rationale**: Support for Xbase expressions, GenModel references, and Java VM types

---

## Category 2: Type System Enhancements

### AR2.1: Replace EString with QualifiedName
**Pattern**: Replace simple string references with qualified names
- **Locations**:
  - `XPackage.name`: `EString` → `QualifiedName`
  - `XAnnotationDirective` cross-references
  - `XImportDirective.importedObject`: uses `QualifiedName`
- **Rationale**: Support namespace-qualified identifiers

### AR2.2: Replace EString with ID for Simple Names
**Pattern**: Replace EString with ID for entity names
- **Locations**:
  - `XClass.name`: `EString` → `ID`
  - `XDataType.name`: `EString` → `ID`
  - `XEnum.name`: `EString` → `ID`
  - `XTypeParameter.name`: `EString` → `ID`
  - `XAttribute.name`: `EString` → `ID`
  - `XOperation.name`: `EString` → `ID`
  - `XReference.name`: `EString` → `ID`
  - `XParameter.name`: `EString` → `ID`
  - `XEnumLiteral.name`: `EString` → `ID`
- **Rationale**: Use standard identifier type instead of generic string

### AR2.3: Replace EClass References with JvmTypeReference
**Pattern**: Replace empty EClass types with proper Java type references
- **Locations**:
  - `XClass.instanceType`: `EClass` → `JvmTypeReference`
  - `XDataType.instanceType`: `EClass` → `JvmTypeReference`
  - `XEnum.instanceType`: `EClass` → `JvmTypeReference`
- **Rationale**: Enable Java type wrapping functionality

### AR2.4: Replace EClass Body Types with XBlockExpression
**Pattern**: Replace empty EClass body types with Xbase expressions
- **Mapping**:
  - `EClass0` (createBody) → `XBlockExpression`
  - `EClass1` (convertBody) → `XBlockExpression`
  - `EClass3` (getBody) → `XBlockExpression`
  - `EClass4` (setBody) → `XBlockExpression`
  - `EClass5` (isSetBody) → `XBlockExpression`
  - `EClass6` (unsetBody) → `XBlockExpression`
  - `EClass7` (operation body) → `XBlockExpression`
- **Rationale**: Support executable code blocks in grammar

### AR2.5: Replace GenBase References
**Pattern**: Update cross-reference types to use GenModel
- **Locations**:
  - `XGenericType.type`: `[|EString]` → `[genmodel::GenBase|XQualifiedName]`
  - `XReference.opposite`: `[|EString]` → `[genmodel::GenFeature|ValidID]`
  - `XReference.keys`: `[|EString]` → `[genmodel::GenFeature|ValidID]`
- **Rationale**: Use proper GenModel metamodel references

### AR2.6: Replace EInt with SignedInt
**Pattern**: Define custom signed integer type
- **Remove**: `EInt returns ecore::EInt: '-'? INT;`
- **Add**: `SignedInt returns ecore::EInt: '-'? INT;`
- **Update**: `XEnumLiteral.value`: use `SignedInt` instead of `EInt`
- **Rationale**: More explicit naming for signed integers

### AR2.7: Remove EBoolean, Use Native Booleans
**Pattern**: Remove explicit boolean type definition
- **Remove**: `EBoolean returns ecore::EBoolean: 'true' | 'false';`
- **Locations**: `XDataType.serializable`, `XEnum.serializable` directly use boolean flags
- **Rationale**: Integrate with Xbase boolean handling

---

## Category 3: Syntactic Structure Transformations

### AR3.1: XPackage - Transform to Natural Syntax
**Pattern**: Replace verbose labeled blocks with keyword-based syntax
- **From**:
  ```
  'XPackage' name=EString '{'
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('importDirectives' '{' importDirectives+=XImportDirective ( "," importDirectives+=XImportDirective)* '}' )?
      ...
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  'package' name=QualifiedName
  (importDirectives+=XImportDirective)*
  (annotationDirectives+=XAnnotationDirective)*
  (classifiers+=XClassifier)*
  ```
- **Changes**:
  - Remove artificial 'XPackage' keyword
  - Add natural 'package' keyword
  - Remove braces and label keywords
  - Change optional grouped lists to simple repetitions
  - Remove commas between elements

### AR3.2: XAnnotation - Transform to @ Syntax
**Pattern**: Replace verbose syntax with annotation-style syntax
- **From**:
  ```
  'XAnnotation' '{'
      ('source' source=[XAnnotationDirective|EString])?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('details' '{' details+=XStringToStringMapEntry ( "," details+=XStringToStringMapEntry)* '}' )?
  '}'
  ```
- **To**:
  ```
  '@' source=[XAnnotationDirective|XQualifiedName]
  ( '(' details+=XStringToStringMapEntry ( "," details+=XStringToStringMapEntry)* ')' )?
  ```
- **Changes**:
  - Remove nested annotations support
  - Use @ prefix instead of keyword
  - Use parentheses for details instead of braces
  - Remove 'source' and 'details' label keywords

### AR3.3: XImportDirective - Transform to Import Syntax
**Pattern**: Replace verbose syntax with standard import syntax
- **From**:
  ```
  'XImportDirective' '{'
      ('importedNamespace' importedNamespace=EString)?
      ('importedObject' importedObject=[ecore::EObject|EString])?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
  '}'
  ```
- **To**:
  ```
  'import'
  (importedNamespace=QualifiedNameWithWildcard |
   importedObject=[ecore::EObject|QualifiedName])
  ```
- **Changes**:
  - Add 'import' keyword
  - Remove braces and all label keywords
  - Remove annotations support
  - Make namespace/object choice explicit (not both optional)

### AR3.4: XAnnotationDirective - Transform to Annotation Declaration
**Pattern**: Replace verbose syntax with annotation definition syntax
- **From**:
  ```
  'XAnnotationDirective' name=EString '{'
      ('sourceURI' sourceURI=EString)?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
  '}'
  ```
- **To**:
  ```
  =>('annotation' sourceURI=STRING) 'as' name=ValidID
  ```
- **Changes**:
  - Add 'annotation' keyword with predicate `=>`
  - Add 'as' keyword separator
  - Remove braces and label keywords
  - Remove annotations support
  - Make sourceURI required (not optional)

### AR3.5: XStringToStringMapEntry - Transform to Key-Value Syntax
**Pattern**: Replace verbose syntax with assignment syntax
- **From**:
  ```
  'XStringToStringMapEntry' '{'
      ('key' key=EString)?
      ('value' value=EString)?
  '}'
  ```
- **To**:
  ```
  key=QualifiedName '=' value=STRING
  ```
- **Changes**:
  - Remove keyword and braces
  - Remove 'key' and 'value' labels
  - Add '=' operator
  - Make both required (not optional)
  - key uses QualifiedName, value uses STRING

### AR3.6: XTypeParameter - Transform to Generic Type Parameter Syntax
**Pattern**: Replace verbose syntax with Java-style generic syntax
- **From**:
  ```
  'XTypeParameter' name=EString '{'
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('bounds' '{' bounds+=XGenericType ( "," bounds+=XGenericType)* '}' )?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  name=ID
  ('extends' bounds+=XGenericType ( "&" bounds+=XGenericType)* )?
  ```
- **Changes**:
  - Remove 'XTypeParameter' keyword and braces
  - Move annotations outside
  - Replace 'bounds' label with 'extends' keyword
  - Use '&' separator instead of commas for multiple bounds

### AR3.7: XClass - Comprehensive Transformation
**Pattern**: Transform to class declaration syntax
- **From**:
  ```
  (abstract?='abstract')? (interface?='interface')?
  'XClass' name=EString '{'
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('instanceType' instanceType=EClass)?
      ('typeParameters' '{' typeParameters+=XTypeParameter ( "," typeParameters+=XTypeParameter)* '}' )?
      ('members' '{' members+=XMember ( "," members+=XMember)* '}' )?
      ('superTypes' '{' superTypes+=XGenericType ( "," superTypes+=XGenericType)* '}' )?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  ((abstract?='abstract'? 'class') | interface?='interface')
  name=ID
  ( '<' typeParameters+=XTypeParameter ( "," typeParameters+=XTypeParameter)* '>' )?
  ('extends' superTypes+=XGenericType ( "," superTypes+=XGenericType)* )?
  ('wraps' instanceType=JvmTypeReference)?
  '{'
      (members+=XMember)*
  '}'
  ```
- **Changes**:
  - Move annotations outside
  - Rewrite modifiers: make 'class' keyword explicit, use choice for class vs interface
  - Remove 'XClass' keyword
  - Add angle brackets for type parameters
  - Replace 'superTypes' label with 'extends' keyword
  - Replace 'instanceType' label with 'wraps' keyword
  - Remove all internal braces and label keywords
  - Remove commas from member list

### AR3.8: XDataType - Transform to Type Declaration
**Pattern**: Transform to type alias syntax with optional bodies
- **From**:
  ```
  'XDataType' name=EString '{'
      ('serializable' serializable=EBoolean)?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('instanceType' instanceType=EClass)?
      ('typeParameters' '{' typeParameters+=XTypeParameter ( "," typeParameters+=XTypeParameter)* '}' )?
      ('createBody' createBody=EClass0)?
      ('convertBody' convertBody=EClass1)?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  'type' name=ID
  ( '<' typeParameters+=XTypeParameter ( "," typeParameters+=XTypeParameter)* '>' )?
  'wraps' instanceType=JvmTypeReference
  ((serializable?='create' createBody=XBlockExpression)? &
   ('convert' convertBody=XBlockExpression)?)
  ```
- **Changes**:
  - Replace 'XDataType' with 'type' keyword
  - Move annotations outside
  - Add angle brackets for type parameters
  - Make 'wraps' required (not optional)
  - Remove braces around bodies
  - Use 'create' and 'convert' keywords instead of labels
  - Use unordered group `&` for optional bodies
  - Repurpose serializable flag as 'create' keyword presence

### AR3.9: XEnum - Transform to Enum Declaration
**Pattern**: Transform to enum syntax
- **From**:
  ```
  'XEnum' name=EString '{'
      ('serializable' serializable=EBoolean)?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('instanceType' instanceType=EClass)?
      ('typeParameters' '{' typeParameters+=XTypeParameter ( "," typeParameters+=XTypeParameter)* '}' )?
      ('createBody' createBody=EClass0)?
      ('convertBody' convertBody=EClass1)?
      ('literals' '{' literals+=XEnumLiteral ( "," literals+=XEnumLiteral)* '}' )?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  'enum' name=ID '{'
      (literals+=XEnumLiteral ( (",")? literals+=XEnumLiteral)* )?
  '}'
  ```
- **Changes**:
  - Replace 'XEnum' with 'enum' keyword
  - Move annotations outside
  - Remove all features except literals (serializable, instanceType, typeParameters, bodies)
  - Keep braces for enum body
  - Make comma optional between literals
  - Remove braces around literal list

### AR3.10: XGenericType - Transform to Type Reference Syntax
**Pattern**: Transform to generic type syntax with type arguments
- **From**:
  ```
  'XGenericType' '{'
      ('type' type=[|EString])?
      ('upperBound' upperBound=XGenericType)?
      ('typeArguments' '{' typeArguments+=XGenericType ( "," typeArguments+=XGenericType)* '}' )?
      ('lowerBound' lowerBound=XGenericType)?
  '}'
  ```
- **To**:
  ```
  type=[genmodel::GenBase|XQualifiedName]
  (=>'<'typeArguments+=XGenericTypeArgument ( "," typeArguments+=XGenericTypeArgument)* '>')?
  ```
- **Changes**:
  - Remove 'XGenericType' keyword and braces
  - Make type required
  - Remove 'type' label
  - Remove bounds (handled in wildcard rule)
  - Use angle brackets with predicate for type arguments
  - Replace 'typeArguments' label
  - Use `XGenericTypeArgument` instead of recursive `XGenericType`

### AR3.11: XMultiplicity - Implement Proper Syntax
**Pattern**: Replace TODO with actual multiplicity syntax
- **From**: `'XMultiplicity' /* TODO: implement this rule and an appropriate IValueConverter */`
- **To**: `'[' ('?' | '*' | '+' | (INT ('..' (INT | '?' | '*'))?))? ']'`
- **Rationale**: Standard UML/Ecore multiplicity notation

### AR3.12: XAttribute - Comprehensive Transformation
**Pattern**: Transform to attribute declaration syntax
- **From**:
  ```
  (unordered?='unordered')? (unique?='unique')? (readonly?='readonly')? 
  (volatile?='volatile')? (transient?='transient')? (unsettable?='unsettable')? 
  (derived?='derived')? (iD?='iD')?
  'XAttribute' name=EString '{'
      ('multiplicity' multiplicity=XMultiplicity)?
      ('defaultValueLiteral' defaultValueLiteral=EString)?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('type' type=XGenericType)?
      ('getBody' getBody=EClass3)?
      ('setBody' setBody=EClass4)?
      ('isSetBody' isSetBody=EClass5)?
      ('unsetBody' unsetBody=EClass6)?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  ((unordered?='unordered')? & (unique?='unique')? & (readonly?='readonly')? &
   (transient?='transient')? & (volatile?='volatile')? & (unsettable?='unsettable')? &
   (derived?='derived')? & (iD?='iD')?)
  type=XGenericType
  ( multiplicity=XMultiplicity)?
  name=ID
  ('=' defaultValueLiteral=STRING)?
  (('get' getBody=XBlockExpression)? &
   ('set' setBody=XBlockExpression)? &
   ('isSet' isSetBody=XBlockExpression)? &
   ('unset' unsetBody=XBlockExpression)?)
  ```
- **Changes**:
  - Move annotations outside
  - Use unordered group `&` for modifiers
  - Remove 'XAttribute' keyword and braces
  - Move type before name
  - Add '=' for default value
  - Replace label keywords with 'get', 'set', 'isSet', 'unset'
  - Use unordered group for optional bodies
  - Remove all internal braces

### AR3.13: XOperation - Comprehensive Transformation
**Pattern**: Transform to operation declaration syntax
- **From**:
  ```
  (unordered?='unordered')? (unique?='unique')?
  'XOperation' name=EString '{'
      ('multiplicity' multiplicity=XMultiplicity)?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('type' type=XGenericType)?
      ('typeParameters' '{' typeParameters+=XTypeParameter ( "," typeParameters+=XTypeParameter)* '}' )?
      ('parameters' '{' parameters+=XParameter ( "," parameters+=XParameter)* '}' )?
      ('exceptions' '{' exceptions+=XGenericType ( "," exceptions+=XGenericType)* '}' )?
      ('body' body=EClass7)?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)* 'op'
  (unordered?='unordered' unique?='unique'? |
   unique?='unique' unordered?='unordered'?)?
  ( '<' typeParameters+=XTypeParameter ( "," typeParameters+=XTypeParameter)* '>' )?
  (type=XGenericType | 'void')
  ( multiplicity=XMultiplicity)?
  name=ID
  '('(parameters+=XParameter ( "," parameters+=XParameter)* )?')'
  ('throws' exceptions+=XGenericType ( "," exceptions+=XGenericType)* )?
  ( body=XBlockExpression)?
  ```
- **Changes**:
  - Add 'op' keyword after annotations
  - Replace 'XOperation' keyword
  - Use choice for modifier order
  - Add angle brackets for type parameters
  - Add 'void' option for return type
  - Move type before name
  - Add parentheses for parameters
  - Replace 'exceptions' label with 'throws' keyword
  - Remove 'body' label
  - Remove all braces except parameter list parentheses

### AR3.14: XReference - Comprehensive Transformation  
**Pattern**: Transform to reference declaration syntax with containment modifiers
- **From**:
  ```
  (unordered?='unordered')? (unique?='unique')? (readonly?='readonly')? 
  (volatile?='volatile')? (transient?='transient')? (unsettable?='unsettable')? 
  (derived?='derived')? (container?='container')? (containment?='containment')? 
  (resolveProxies?='resolveProxies')? (local?='local')?
  'XReference' name=EString '{'
      ('multiplicity' multiplicity=XMultiplicity)?
      ('opposite' opposite=[|EString])?
      ('keys' '(' keys+=[|EString] ( "," keys+=[|EString])* ')' )?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('type' type=XGenericType)?
      ('getBody' getBody=EClass3)?
      ('setBody' setBody=EClass4)?
      ('isSetBody' isSetBody=EClass5)?
      ('unsetBody' unsetBody=EClass6)?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  ((containment?='contains' resolveProxies?='resolving'?) |
   (resolveProxies?='resolving' containment?='contains') |
   (container?='container' resolveProxies?='resolving'?) | 
   (resolveProxies?='resolving' container?='container') |
   ('refers' local?='local'?) | (local?='local' 'refers'))
  ((unordered?='unordered')? & (unique?='unique')? & (readonly?='readonly')? &
   (transient?='transient')? & (volatile?='volatile')? & (unsettable?='unsettable')? &
   (derived?='derived')?)
  type=XGenericType
  ( multiplicity=XMultiplicity)?
  name=ID
  ('opposite' opposite=[genmodel::GenFeature|ValidID])?
  ('keys' keys+=[genmodel::GenFeature|ValidID] ( "," keys+=[genmodel::GenFeature|ValidID)* )?
  (('get' getBody=XBlockExpression)? &
   ('set' setBody=XBlockExpression)? &
   ('isSet' isSetBody=XBlockExpression)? &
   ('unset' unsetBody=XBlockExpression)?)
  ```
- **Changes**:
  - Move annotations outside
  - Add reference kind choice (contains/resolving/container/refers/local combinations)
  - Use unordered group for remaining modifiers
  - Remove 'XReference' keyword
  - Move type before name
  - Keep 'opposite' and 'keys' keywords but remove extra parentheses from keys
  - Replace body label keywords with 'get', 'set', 'isSet', 'unset'
  - Use unordered group for bodies

### AR3.15: XParameter - Transform to Parameter Syntax
**Pattern**: Transform to method parameter syntax
- **From**:
  ```
  (unordered?='unordered')? (unique?='unique')?
  'XParameter' name=EString '{'
      ('multiplicity' multiplicity=XMultiplicity)?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
      ('type' type=XGenericType)?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  (unordered?='unordered' unique?='unique'? |
   unique?='unique' unordered?='unordered'?)?
  type=XGenericType
  ( multiplicity=XMultiplicity)?
  name=ID
  ```
- **Changes**:
  - Move annotations outside
  - Use choice for modifier order
  - Remove 'XParameter' keyword and braces
  - Move type before name
  - Make type required
  - Remove all label keywords

### AR3.16: XEnumLiteral - Transform to Enum Literal Syntax
**Pattern**: Transform to enum constant syntax
- **From**:
  ```
  'XEnumLiteral' name=EString '{'
      ('value' value=EInt)?
      ('literal' literal=EString)?
      ('annotations' '{' annotations+=XAnnotation ( "," annotations+=XAnnotation)* '}' )?
  '}'
  ```
- **To**:
  ```
  (annotations+=XAnnotation)*
  name=ID
  ('as' literal=STRING)?
  ('=' value=SignedInt)?
  ```
- **Changes**:
  - Move annotations outside
  - Remove 'XEnumLiteral' keyword and braces
  - Remove label keywords
  - Add 'as' keyword for literal
  - Add '=' for value
  - Reverse order: name, then literal, then value

---

## Category 4: Terminal Rule Additions

### AR4.1: Add XBlockExpression Override
**Pattern**: Define Xbase block expression
- **Add**:
  ```
  @Override
  XBlockExpression returns xbase::XBlockExpression:
      {xbase::XBlockExpression}
      '{'
          (expressions+=XExpressionOrVarDeclaration ';'?)*
      '}'
  ;
  ```
- **Rationale**: Override Xbase expression blocks for code bodies

### AR4.2: Add XGenericTypeArgument Rule
**Pattern**: Support wildcard type arguments
- **Add**:
  ```
  XGenericTypeArgument returns XGenericType:
      XGenericType |
      XGenericWildcardTypeArgument
  ;
  ```
- **Rationale**: Allow both concrete types and wildcards

### AR4.3: Add XGenericWildcardTypeArgument Rule
**Pattern**: Define wildcard type arguments
- **Add**:
  ```
  XGenericWildcardTypeArgument returns XGenericType:
      {XGenericType}
      '?' ('extends' upperBound=XGenericType | 'super' lowerBound=XGenericType)?
  ;
  ```
- **Rationale**: Support Java-style wildcard generics

### AR4.4: Add XQualifiedName Rule
**Pattern**: Define qualified name syntax
- **Add**:
  ```
  XQualifiedName:
      XID ('.' XID)*
  ;
  ```
- **Rationale**: Support dot-separated namespaces

### AR4.5: Add XID Rule
**Pattern**: Extend ID to include keywords
- **Add**:
  ```
  XID:
      ID | 'get' | 'isSet' | 'set' | 'unset'
  ;
  ```
- **Rationale**: Allow method name keywords as identifiers

### AR4.6: Add ValidID Override
**Pattern**: Extend ValidID for Xbase
- **Add**:
  ```
  @Override
  ValidID:
      XID | 'void'
  ;
  ```
- **Rationale**: Include void type keyword

### AR4.7: Add FeatureCallID Override
**Pattern**: Define comprehensive feature call identifiers
- **Add**:
  ```
  @Override
  FeatureCallID:
      ValidID | 'abstract' | 'annotation' | 'as' | 'class' | 'container' | 
      'contains' | 'convert' | 'create' | 'derived' | 'enum' | 'extends' | 
      'extension' | 'id' | 'import' | 'keys' | 'interface'| 'local' | 'op' | 
      'opposite' | 'package' | 'readonly' | 'refers' | 'resolving' | 'static' | 
      'throws' | 'transient' | 'unique' | 'unordered' | 'unsettable'| 'volatile' | 'wraps'
  ;
  ```
- **Rationale**: Allow all keywords as feature call identifiers in expressions

### AR4.8: Add QualifiedNameWithWildcard (Implicit)
**Pattern**: Support wildcard imports
- **Used in**: `XImportDirective.importedNamespace=QualifiedNameWithWildcard`
- **Expected**: `QualifiedName '.*'?` or similar
- **Rationale**: Enable star imports

---

## Category 5: Rule Removals

### AR5.1: Remove EString Definition
**Pattern**: Remove custom EString terminal
- **Remove**: `EString returns ecore::EString: STRING | ID;`
- **Rationale**: Use QualifiedName and ID directly

### AR5.2: Remove EObject Definition
**Pattern**: Remove empty EObject rule
- **Remove**: `EObject returns ecore::EObject: {ecore::EObject} 'EObject' ;`
- **Rationale**: Use cross-references directly

### AR5.3: Remove EClass Definition
**Pattern**: Remove empty EClass rule
- **Remove**: `EClass returns : {} '' ;`
- **Rationale**: Replaced by JvmTypeReference

### AR5.4: Remove All EClassN Definitions
**Pattern**: Remove all placeholder body types
- **Remove**: 
  - `EClass0 returns : {} '' ;`
  - `EClass1 returns : {} '' ;`
  - `EClass2 returns : {} '' ;`
  - `EClass3 returns : {} '' ;`
  - `EClass4 returns : {} '' ;`
  - `EClass5 returns : {} '' ;`
  - `EClass6 returns : {} '' ;`
  - `EClass7 returns : {} '' ;`
  - `EClass8 returns : {} '' ;`
  - `EClass9 returns : {} '' ;`
- **Rationale**: Replaced by XBlockExpression

### AR5.5: Remove EBoolean Definition
**Pattern**: Remove boolean terminal
- **Remove**: `EBoolean returns ecore::EBoolean: 'true' | 'false';`
- **Rationale**: Use Xbase boolean handling

### AR5.6: Remove XDataType_Impl Reference
**Pattern**: Update XClassifier alternatives
- **From**: `XClass | XDataType_Impl | XEnum`
- **To**: `XClass | XDataType | XEnum`
- **Rationale**: Remove implementation suffix, use abstract type

---

## Summary Statistics

### Total Adaptation Rules: 65+

#### By Category:
1. **Root Rule & Import System**: 2 rules
2. **Type System Enhancements**: 7 rules
3. **Syntactic Structure Transformations**: 16 major rules (covering all main grammar rules)
4. **Terminal Rule Additions**: 8 rules
5. **Rule Removals**: 6 rules

#### Complexity Indicators:
- **High-complexity transformations**: XClass, XDataType, XAttribute, XOperation, XReference (complete restructuring)
- **Medium-complexity**: XPackage, XAnnotation, XEnum, XParameter
- **Low-complexity**: Type replacements, terminal additions

#### Key Patterns:
1. **Keyword replacement**: Artificial keywords → natural keywords
2. **Label removal**: Remove all feature labels, use keywords or operators
3. **Brace elimination**: Remove structural braces, use natural syntax
4. **Type system upgrade**: EString/EClass → QualifiedName/ID/JvmTypeReference/XBlockExpression
5. **Modifier organization**: Sequential → unordered groups or choices
6. **Annotation placement**: Inside → outside rule definitions

---

## Implementation Notes

### High-Impact Changes:
1. Grammar inheritance switch enables Xbase expression support
2. Type system changes enable proper Java integration
3. Syntactic transformations make grammar user-friendly and IDE-compatible

### Dependencies:
- Many changes are interdependent (e.g., type changes require syntax changes)
- Terminal additions must come before their usage in main rules
- Import additions are prerequisites for type references

### Validation Requirements:
- Ensure all cross-references resolve correctly after type changes
- Verify unordered groups don't create ambiguities
- Test wildcard and generic type parsing
- Validate expression blocks compile correctly
