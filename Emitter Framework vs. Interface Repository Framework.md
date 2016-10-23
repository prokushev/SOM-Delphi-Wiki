# Comparison #

## Availability ##

* Emitter Framework: IBM SOM (2.1 and 3.0) only. somFree contains non-modular SOM compiler written in C++.
* Interface Repository Framework: IBM SOM and somFree.

## Workflow ##

* Emitter Framework: Write IDL, invoke SOM compiler using any chosen emitter for specified IDL. Even if multiple IDLs are specified on the command line, the SOM compiler will process them independently.
* Interface Repository Framework: Write IDL, invoke SOM compiler using IR emitter for specified IDL. IR emitter creates/updates SOM.IR file. Then any tool can read SOM.IR file using Interface Repository Framework.

## Amount of information ##

* Emitter Framework: Limited by what is reachable from processed IDL and included ones. Unresolved forward class declarations are possible. For instance, ::SOMClassMgr::somInterfaceRepository method returns ::Repository object reference, but somcm.idl does not include repostry.idl. Emitters like "imod" collecting information from several IDLs at once have to incrementally edit output file. They put signatures into text file to locate where to append new strings. Incremental updates of text files is hard to implement in more complex cases.
* Interface Repository Framework: Contains information gathered from multiple IDLs in one place. No unresolved forward declarations allowed.

## Information precision ##

* Emitter Framework: Has class for every possible entity in the IDL, including comments and passthru. Type names in arguments and results are fully qualified and has precise names.
* Interface Repository Framework: Comments and passthru are absent. Metaclass name is available from modifier, however, it needs to be resolved. Overriden methods and release order is available. Instance data is available as record TypeCode (tk_struct). There are TypeDef for type definitions, but everywhere else CORBA TypeCode is used to specify types, and precise names are lost. For instance, biter.idl defines class Biter::BINDITER_TWO having method with argument of "CosNaming::BindingList" type. This type becomes in CORBA TypeCode "sequence<Binding>" without any hint to look for Binding into "CosNaming" module. This time CosNaming is an outer module for one of class parents, so it is possible to lookup this name traversing class hierarchy, but in general this is heuristic job.

## Intended environment ##

* Emitter Framework: Static environment. Compiled languages. Documentation generators.
* Interface Repository Framework: Dynamic environment. Scripting languages. Remote procedure calls.

## Metrical data ##

* Emitter Framework: 22 classes: SOMTAttributeEntryC, SOMTBaseClassEntryC, SOMTClassEntryC, SOMTCommonEntryC, SOMTConstEntryC, SOMTDataEntryC, SOMTEmitC, SOMTEntryC, SOMTEnumEntryC, SOMTEnumNameEntryC, SOMTTemplateOutputC, SOMTMetaClassEntryC, SOMTMethodEntryC, SOMTModuleEntryC, SOMTParameterEntryC, SOMTPassthruEntryC, SOMTSequenceEntryC, SOMTStringEntryC, SOMTStructEntryC, SOMTTypedefEntryC, SOMTUnionEntryC, SOMTUserDefinedTypeEntryC.
* Interface Repository Framework: 11 classes + 1 pseudoclass: AttributeDef, ConstantDef, Contained, Container, ExceptionDef, InterfaceDef, ModuleDef, OperationDef, ParameterDef, Repository, TypeDef; and TypeCode pseudoclass.
