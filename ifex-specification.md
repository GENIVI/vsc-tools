# IFEX CORE SPECIFICATION

(C) 2022, 2023 - MBition GmbH  
(C) 2022 - COVESA  
(C) 2021 - Magnus Feuer  

This document contains an introduction to the Interface Exchange (IFEX)
framework and specification of the core Interface Description Language/Model
(also known as ifex-idl and ifex-core).  IFEX is the name for the _technology_
(language, tools, etc.) behind the Vehicle Service Catalog (VSC) project.

License: Creative Commons Attribution 4.0 International
License (CC-BY-4.0), described [here](https://creativecommons.org/licenses/by/4.0/)

<!-- Heading and TOC -->
Documentation generated from: 8b82cf9e114b9d5c94b4ea783da640b2c062039e

- [FEATURES](#features)  
    - [Features that are not included _in the core IDL_, but worth describing](#features-that-are-not-included-in-the-core-idl-but-worth-describing)  
    - [Feature concept details](#feature-concept-details)  
        - [Target Environment](#target-environment)  
        - [Namespace](#namespace)  
    - [Interface](#interface)  
        - [Method](#method)  
        - [Errors](#errors)  
        - [Event](#event)  
        - [Property](#property)  
        - [Return values](#return-values)  
        - [Return/status communication in asynchronous vs. synchronous method calls](#returnstatus-communication-in-asynchronous-vs-synchronous-method-calls)  
    - [Data Streams](#data-streams)  
        - [Service (not a core IDL feature)](#service-not-a-core-idl-feature)  
        - [Signals (not a core IDL feature)](#signals-not-a-core-idl-feature)  
- [NAMESPACE VERSIONING](#namespace-versioning)  
- [INTERFACE VERSIONING](#interface-versioning)  
- [FUNDAMENTAL TYPES](#fundamental-types)  
- [LAYERS CONCEPT](#layers-concept)  
    - [Extending the interface model by mimicking the structure](#extending-the-interface-model-by-mimicking-the-structure)  
- [DEPLOYMENT LAYER](#deployment-layer)  
- [More Layer Information](#more-layer-information)  
- [IFEX FILE SYNTAX, SEMANTICS AND STRUCTURE](#ifex-file-syntax-semantics-and-structure)  
    - [NODE TYPES](#node-types)  
    - [Namespace](#namespace)  
            - [Mandatory fields for Namespace](#mandatory-fields-for-namespace)  
            - [Optional fields for Namespace](#optional-fields-for-namespace)  
    - [Event](#event)  
            - [Mandatory fields for Event](#mandatory-fields-for-event)  
            - [Optional fields for Event](#optional-fields-for-event)  
    - [Argument](#argument)  
            - [Mandatory fields for Argument](#mandatory-fields-for-argument)  
            - [Optional fields for Argument](#optional-fields-for-argument)  
    - [Method](#method)  
            - [Mandatory fields for Method](#mandatory-fields-for-method)  
            - [Optional fields for Method](#optional-fields-for-method)  
    - [Error](#error)  
            - [Mandatory fields for Error](#mandatory-fields-for-error)  
            - [Optional fields for Error](#optional-fields-for-error)  
    - [Typedef](#typedef)  
            - [Mandatory fields for Typedef](#mandatory-fields-for-typedef)  
            - [Optional fields for Typedef](#optional-fields-for-typedef)  
    - [Include](#include)  
            - [Mandatory fields for Include](#mandatory-fields-for-include)  
            - [Optional fields for Include](#optional-fields-for-include)  
    - [Struct](#struct)  
            - [Mandatory fields for Struct](#mandatory-fields-for-struct)  
            - [Optional fields for Struct](#optional-fields-for-struct)  
    - [Member](#member)  
            - [Mandatory fields for Member](#mandatory-fields-for-member)  
            - [Optional fields for Member](#optional-fields-for-member)  
    - [Enumeration](#enumeration)  
            - [Mandatory fields for Enumeration](#mandatory-fields-for-enumeration)  
            - [Optional fields for Enumeration](#optional-fields-for-enumeration)  
    - [Option](#option)  
            - [Mandatory fields for Option](#mandatory-fields-for-option)  
            - [Optional fields for Option](#optional-fields-for-option)  
    - [Property](#property)  
            - [Mandatory fields for Property](#mandatory-fields-for-property)  
            - [Optional fields for Property](#optional-fields-for-property)  
    - [Interface](#interface)  
            - [Mandatory fields for Interface](#mandatory-fields-for-interface)  
            - [Optional fields for Interface](#optional-fields-for-interface)  

<!-- Features and introduction -->
<!-- Features and introduction -->
--------------------
# FEATURES
The format supports the following features

* **Namespaces**  
  Logical grouping of methods, events, properties, and defined data types that can be nested.

* **Interfaces**  
  Logical grouping of methods, events, properties, and defined data types that are intended to be exposed as a 'public' (externally usable) interface from a software component.

* **Methods**  
  A function or remote-procedure-call, sent to and executed by a particular targeted component (server). Methods can have input and output parameters, as well as separate return-parameters, and errors.

* **Rich type system**
  Named data types that can be enumerations, structs or typedefs, and collections (arrays) of types, and each of those types can be nested.  Type aliases are supported, and constraints such as a limited value-range (work in progress).

* **Events**  
  A fire-and-forget call/message, sent to or invoked on zero or more subscribing instances. The Event has no return a value and does not guarantee delivery (refer to target-dependent details).

* **Data Properties**  
  An observable shared state object that can be read and set, and which is available to all\* subscribing entities.
  (\*individual access control rules are of course possible to apply in the target environment) .

* **Layered architecture and Deployment files**  
  IFEX Core IDL is designed to be a fundamental format for describing all sorts of software interfaces.  Interface descriptions should be as generic as possible, meaning the same interface description can be reused in a completely different environment.  The IFEX concept adds on a layered architecture that adds deployment-specific data to an interface definition so that it can avoid cluttering the core IDL.  Other composable features are likely to be created using the generic layering concept.  This separation of interface and deployment was pioneered by **Franca IDL** and IFEX continues and extends the principle.

## Features that are not included _in the core IDL_, but worth describing:

* **Service**

This current version of the IFEX Core IDL has removed the "service:" keyword but the concept of a service should still be defined in terms of the interfaces the service provides.  (See next chapter).

* **Signal**

IFEX Core IDL does not include the name Signal but supports both **Events** and **Properties** that do cover two different interpretations of the word signal.  See next chapter for a more detailed analysis.

----

## Feature concept details

### Target Environment

- Target Environment is a catch-all term to indicate the context and environment of those artifacts that are generated from source IDL.
- For each type of output result from a generator or conversion tool, there will be environment-specific details to consider, and in the documentation we often refer to all of those as simply the "target environment".
- The term simply signifies any and all things that are unique about that environment.

For example:
- The chosen programming language in the case of code-generation tools.
- The chosen communication protocol or binding technology.
- Any details pertaining to other levels of software technology that is being targeted.  (For example if HTTP is an applicable protocol to use then details of transport layer security (TLS) would apply to that environment, whereas for other protocols it might not).

Please note that in the text, "target-environment dependent", may sometimes be shortened to simply target-dependent and similar simplifications may occur.


### Namespace

- A Namespace is a way to separate definitions inside named groups.
- Namespaces are used to ensure that local names will not clash with identically named items in other namespace.
- Namespaces affect visibility, in other words what objects can be _seen_ or _reached_ from within a part of a program.
- Namespaces can contain other Namespaces, creating a hierarchy.
- How visibility/reachability is handled _might_ be specific to the target language or environment, but IFEX expects the following behavior to be the normal one unless there are very good reasons for exceptions:
  - Items within different (non-parent) namespaces are isolated from each other unless otherwise specified, and these frequently used hierarchy rules apply:  Everything within a child namespace can see and reach items in any of its parent namespaces.  For non-parent relationship, it is required to either specify an item using a fully-qualified "path" through the namespace hierarchy, or to make a statement of reference or inclusion of another namespaces into the current namespace.

## Interface

- Generally, Interface is a broad term.  In IFEX core IDL we are concerned with functional software interfaces, nothing related to hardware, electrical or mechanical.
- An IFEX Interface is like a specialization of the Namespace concept, but used specifically to group things that are intended to be the externally visible (a.k.a. public) interface of a software component that defines its interface using the IFEX file.
- Design note: It would have been possible to generalize even further and to say that "an Interface is just a Namespace".  In that case, a layer would have been used to indicate which Namespace object(s) shall be considered one that holds the primary objects of our 'exported' interface.  But since the purpose of the IDL is primarily to define an interface it seems appropriate to give it special status by giving it its own keyword in the IDL itself.  Also, the nature of "visibility of objects" and "avoiding name clashes" (main reason for Namespace) is subtly different from the nature of "which objects do we consider being part of our outwardly displayed interface".
- In this definition, the  Interface is not itself a Namespace (in terms of object visibility) but it must be defined within a Namespace.  All the Interface's objects belong to the parent Namespace of the Interface.
- An Interface can contain all things a Namespace can contain, except another Interface (no nesting).
- An IFEX **Interface** is often defined by a collection of **Methods**, but could also expose functionality through **Event**s and data **Properties**.  The interface might also include variables, constants, method-arguments, types, observable data-items and events that are required by the client using the interface.  That can be done for clarity, but from visibility standpoint it is identical to including them in the immediate parent Namespace.
- The IFEX core IDL strives to be usable in all areas of a computing system.  The interface definition therefore does not define details around communication hardware, implementation language, data-communication protocols, bit-encoding and in-memory layouts etc.  Such information is however added through supplementary information in the mappings and tools that consume the interface description.
- All Methods/Properties/Events defined or referred to by an Interface are required to be within the same Namespace, i.e. it is not possible to build an Interface by referring to Methods that have been defined in different Namespaces.

### Method

- A method is a function with a (mandatory) name, plus a set of (optional) **input parameters**, **output parameters**, **return parameters** and **error conditions**.
- Like most other objects, a Method is always defined in the context of a Namespace, but that is often as a result of being within the context of an Interface within a Namespace.

### Errors

- An IFEX Error definition is a condition that can appear while (attempting to) execute a given Method.
- An error is defined by referencing a **datatype** that define the type of information that shall be communicated when an error occurs.  Enumeration types are common, but it could be any defined type.
- There can be _more than one_ error, each with its own datatype, defined for a method.  
- NOTE: Don't mistake this for what most programming environments describe, which is to define a single enumeration, or structured/variant type, where each returned _value_ may indicate a different "error".  What IFEX provides is the possibility of defining multiple different error returns, each with their own name and type (similar to multiple input or output-variables in a function).  Through overlays, that enables efficiently separating business-logic errors (that are known when the interface is defined), from transport errors (that are only known later, when a particular target environment has been selected).  The difference is subtle and of course the generated code might be required to combine the information into a single struct/variant/union type because of limitations, but the difference is that it can now be done at the deployment stage, and not at the interface definition stage.

- It is not specified by IFEX core IDL how errors are propagated to the caller of a function - that translation is target language/environment specific.  It may be done using a return or output parameter in one programming language and using Exception-handling in another.  Network communication protocols may implement their own error-condition side-channels or other methods.

### Event

- An Event is a time-sensitive communication with _fire-and-forget_ behavior, sent between parts in the distributed system.
- An IFEX Event can carry multiple pieces of data with it, each of which can use an arbitrary data type.
- An IFEX Event definition looks similar to a Method but with some differences in definition and behavior:
  - An Event has a name, and optionally a set of parameters that are distributed with the event, each having a name and a datatype.  This mimics a Method signature.  Just like a method, parameters are defined as **input parameters**. It is viewed from the perspective of the _receiver_ of the event.
  - An Event can **not** have a Return type, nor any out-parameters.
  - An Event may have **Error**s defined but with limited effect.  Since the expected behavior is "fire and forget", the usage of Error is likely to only be to report very fundamental errors, such as the network protocol returning an error because the connection was closed, etc.  (Note that such low-level Errors would normally not appear in a generic and reusable interface description, but in that case be added when specific deployment-details are layered on to the definition).
- _Fire-and-forget_ expected behavior means that on the IFEX semantics level it is _not expected_ to be a guaranteed delivery.  It is in reality decided by the target environment mapping whether an Event is sent over a reliable or unreliable transport, but in either case, there is **no reply** defined for an Event.  When an interface is designed it shall use a **Method** if reliability is expected.  In the case of a Method, a return value would indicate that the information was received.

**Avoiding event confusion:**

- An IFEX **Event** is not a persistent data item, and thus has no built-in concepts of active-low/high, level or edge-trigger, rising/falling edge, etc. (other than what its name and documentation indicates).  It is there to expressly indicate a particular thing "has happened".  It happened "at this point in time", and is communicated as nearly as possible to the time of the happening.

- An IFEX **Event** is a separate and _expressly defined_ object in the interface description. It should be used for something central to the "business-logic" of the interface.  It is not to be mixed up with lower level "happenings" in a system that do not need to be expressly defined.  Events shall not be confused with messages that are _automatically_ sent by some target environment directly, either that they are always done (hidden as part of that implementation) or  _as a result of_ the already encoded behavior of other objects already in the IDL.  

    - The most obvious example is this:  In many systems, subscribing to changes of an observable data **Property** may trigger something that the target environment might call "update events", but those are a consequence of the behavior of the observable Property.   Assuming that the target-mapping is generating both sides of the communication, then these do not need explicit Event objects in the IDL itself.  It can sometimes be difficult to draw this distinction, but remember that if these low-level behaviors can be automatically generated during code-generation as a consequence of some object that is already in the interface (plus sometimes, some additional layered information), then it is not necessary to provide more information to the code-generators for them to succeed.  Those low-level mechanisms are normally decided by the particular data protocol or target environment and would therefore not be reusable in another context.  Since they are automatically generated from other objects (Property for example), there is no need to expressly define them in the fundamental Interface description.  The strategy should be: Avoid overlapping definitions in the core IDL, and let the target environment mappings decide what the auto-generated result of that input information is (as long as it follows the _generally expected behavior_ described in this specification.   A Property is described as "observable" in the core IFEX IDL.  Most of the time, let the target mapping decide how that observability is implemented, and avoid modeling explicit events for that purpose.  That said, in certain cases, when auto-translating an interface description from another IDL that already have explicit events encoded, then it might not be possible to follow this guideline, but just try where possible to lift the interface level to objects with more built-in semantic meaning, such as Method and Property.

- Design tip:  If there is a need to model On/Off events, to avoid duplicating each one into two different Events -> give the event a single name and transfer a boolean as argument: FooState(bool) where the true/false value indicates the on/off state.  Another option might be to define an observable boolean data **Property** and use the target environment's Pub/Sub capability to get updates on its state.  (Depending on the details of the target environment, the latter might also give reliability guarantees that Events do not have).

- Conceptually, an Event can be sent from client to server or from server to client.
(\*) TO CLARIFY: Shall the direction of an event be defined, and how?

- If some events are to be transferred as a "broadcast" (from server to all clients) while other events would be addressed uniquely to particular client(s), then define this difference as extra information in a deployment model / target mapping.  It is not specified in the IFEX core IDL interface description.

### Property

- A **Property** is an observable data item.  It belongs in an **Interface**, has a canonical name in addition to possible **aliases** (alternative names), and a data type.
- A Property is defined as a single item but the data type could be an arbitrary data type, including large and composite data types.
- The singular nature of a property suggests that it is generally expected behavior is that its value shall updated and communicated as an atomic operation, (it is either fully updated or not at all).
- The available features for update and communication are target-environment specific, but the most common expectation is that a Property can be updated and _observed_, meaning that a client can call a method to get or set the current value of the property (with appropriate access control rules) and often also _subscribe_ to be notified of when the value of a property was changed for any reason.
- N.B. In some other language definitions this type of item is called an **Attribute** or **Field**.
- A property can be seen as analogous (and conceptually equal) to a Signal in the Vehicle Signal Specification (VSS).

### Return values

- A Return Value specifies what is returned from a Method execution.  Although "out parameters" could serve a similar purpose, the Return Value has been given special status.  This is for convenience in some cases, but the following chapters on asynchronous methods and data streaming interfaces, the need for special meaning is clarified.
- In target mappings it is of course possible that return values might be implemented as out-parameters in some programming language, or vice versa that out-parameters are embedded into a composite return value.   On the receiving side, the items might be converted back to their original separated meaning.  Such decision depend on the capability of the target environment.
- While return-value is often traditionally used for indicating success/error, please note that the abstract Errors concept is more powerful and should often be preferred in new designs.
- Only one single return value is supported, but it can of course use any data type including composite types.
- The term Status may be used in other technologies and the meaning of return value can be seen as equivalent of status.

### Return/status communication in asynchronous vs. synchronous method calls

- In computing systems there are two main paradigms regarding invocation of a procedure/function: Blocking/synchronous vs. Non-blocking/asynchronous.

  - In a blocking/synchronous environment, the client invokes an operation ("calls a function") and will halt until the operation is completed ("the function returns") and then continue its own execution flow. When completed, a return-value (result) may be communicated back to the client.

  - In an asynchronous setup, a client can trigger the start of an operation and then continue doing other work while the method executes in parallel.  Programming languages and environments have different mechanisms for either notifying the client that the operation is finished, or for the client to ask if it is finished or, at some later point in time decide to go into a "blocking" mode and wait for the operation to finish.  In any case, the executing server may will report back the result in some manner when the operation is finished, but it could also give continuous reports as to the progress of the function ("streaming updates").

- In the IFEX core IDL, methods can only be defined _without_ defining their (synchronous/asynchronous) invocation strategy.  It is more flexible to define this separately in a deployment-model/target-mapping because it leads to interface descriptions that can be reused in a wider set of circumstances.  Perhaps it is true that some operations only are suited for one or the other, but many operations can in general be done in either a synchronous or asynchronous manner.  The desire for sync vs. async might change in new circumstances, and the availability of sync/async might also be constrained by the target environment capability. To encourage the design of widely reusable and generic interfaces, the sync/async aspect is therefore defined separate from the main IDL.  

- In all cases, an IFEX Method may define a Return Value type.  The return value can be defined for the interface description, without knowing the manner that the method will be invoked (e.g. the chosen communication protocol). IFEX-IDL-language has the unique feature of using the Return Value type to serve two different purposes in sync vs async situations:

- As discussed, the details _may_ be target-environment dependent, but we can here define the _generally expected_ behavior of the return type in a system built from an IFEX interface:  
  1. In the case of a blocking/synchronous call, the Return Value type is the value communicated _once_ after the operation completes.  
  2. In the case of a non-blocking/asynchronous call, the same defined Return Value type is also used with the difference that it _may_ be returned _multiple times_.  Using that behavior is optional and target-dependent, but if it is used then a target environment may choose to provide a regular stream of status updates while the method executes.  In each such update, a value is transferred that matches the defined Return Value type\*\*.  
  - It follows that only valid values of the Return Value type shall be returned. 

  - \*\* Design note:  This is already in itself a very powerful mechanism and should suffice in a majority of cases, but if any additional status/streaming-data functionality is required, that may of course be done by defining a data Property in the interface, whose value is used to indicate the status of the system for the executing method.  This would enable to listen (subscribe) to update to the Property, or fetch it on-demand, depending on what features the target mapping provides for Properties.  Note that there is no _formal_ way to describe the relationship between Property and Method in this design approach - it would rely on documenting the relationship in descriptions.

  - \*\*\*Design note 2: While the interface can now be defined without deciding up front about synchronous/asynchronous operation, if an async operation _may_ be expected it is still a good idea to design the return value with this in mind.  It is likely to include something to indicate that processing is under way.  For example the streaming returns during an operation might be: Started, Processing..., Processing..., Processing..., Done!"`  (Note that this, like most details, can be extended by overlays.)

- While a return value can also indicate that the operation did NOT complete successfully, the more capable **Errors** concept should not be overlooked.

- After an operation has been completed and its equivalent of "DONE" has been communicated, the function is expected to seize any further communication of the return/status type.

## Data Streams

It might be noticed that the IFEX core IDL does not seem to have an explicit type for "streaming data" type interfaces.  However, when reading the previous chapter about asynchronous methods it should be clear that there is a natural way to do it:

1. Define a method on the server, which the client would call to initiate a stream.
2. Define the method **return value** as the datatype that describes the items that are streamed. (each byte, each packet, or to whatever level of detail the stream can be subdivided).
3. When executing, the server streams continuous "updates" of this return value type for as long as the stream is open.

Note here, that the explicit separation of **Error** s from return value, which is most other programming and IDL environments fail to do, enables modeling Error information that would be returned to the client when a problem occurs, and that is completely independent of the return type that describes what the streaming data looks like.

Here we have a situation where it is likely to be known at interface-definition time how the method is intended to be used.  It is therefore recommended that the interface designer makes note of this in a comment, that the particular method indicates a streaming interface.

As described in the previous chapter, to make a method asynchronous will require a target-environment deployment layer that annotates the method to be "asynchronous".  If it is necessary, the design of that target mapping is of course free to use more explicit metadata to make it even clearer and since Layer Types are completely open for design, it could also add more information such as the data rate and exact method of streaming, and so on.

Example (not normative):
```
methods:
  - name: mymethod
    asynchronous: true
    streaming: true    # example, the layer design (i.e. not part of this IDL core specification) decides what makes sense
```
 
### Service (not a core IDL feature)

The current version of the IFEX Core IDL removed the "service:" keyword that was
in the earliest specification.  In the future a new concept may be appropriate to
add but at the moment it is expected that any definition of "service" will be
defined externally from the core IDL, and make reference to IFEX Core IDL
file(s) that describe Interfaces.

It is still worthwhile to prepare our understanding of what a service is:

- A service is a software functionality or set of functionalities that can be activated by a client. A key characteristic of a service in support of SOA is that it has low coupling to other services and strives to provide a useful function also if used independently from other services.
- In IFEX philosophy, a **Service** provides one or more **Interface**s and it is anticipated that interfaces are described using the IFEX Core IDL.
- Most likely, a service definition will only make sense together with deployment information such as the chosen target environment and technology (a specific RPC protocol, a REST-style web-service, etc.).  Therefore, it will require additional files ("layers") independent of the Interface descriptions anyhow, so leaving it out of the core IDL makes sense.. 

### Signals (not a core IDL feature)

The word Signal is interpreted by some as the _transfer_ of a _value_ associated 
with a name/id for what that value represents.  This suggests that a signal
is something happening at a point in time.  Such value transfer ought to
be semantically equivalent to single-argument Event, and is therefore supported
using Event in IFEX.  Another interpretation is that the word Signal rather represents
the underlying data item itself, independent of what time update-events are sent.
In this second case, value-transfers are defined more as a consequence of the data
changing, or a predefined frequency of update -  for example "subscribing to
changes of a Signal".  In this second interpretation the Signal is represented
by a Property in IFEX.  The Vehicle Signal Specification (VSS) defines "signals"
that have a name, a datatype, a unit and meaning but the transfer of data is
defined by separate protocols built on top of VSS.  In other words, VSS
typically uses the the second interpretation, and VSS Signals can then be
represented by Properties in IFEX.  (Refer to the [separate
analysis](static-vss_integration_proposal) of the IFEX/VSS relationship).

--------------------

# NAMESPACE VERSIONING

IFEX namespaces can have a major and minor version, specified by
`major_version` and `minor_version` keys, complemented by an additional
free format string named `version_label`.  These are defined as optional
in the core IDL, but most development processes ought to treat them as mandatory
at some point in the working process, to support management of API
compatibility between components.

There is also an optional `patch_version` number, to simplify compatibility
with other interface descriptions that use the semantic versioning principle.
(Please see the recommendation _against_ using `patch_version` in new
designs, described in the Interface Versioning chapter).

Namespace version management can be used to understand the impact of changes
on various namespace levels and how this may affect compatibility.
For general API-compatibility evaluation, the version on the **Interface**
node might be used more often, however.

Namespace versioning shall follow the Semantic Versioning principle.  Follow
the more detailed description under Interface Versioning.

Namespace versioning can be used build-time to ensure that the correct
version of all needed namespace implementations are deployed.


# INTERFACE VERSIONING

An Interface is essentially a specialization of the Namespace concept.
Interfaces shall be versioned in the same manner, and are even more
important since they are the likely place to determine if components
will communicate correctly (i.e. use the compatible versions of Interface
description).

There could be rules implemented in validation tools that ensure interface
versions match the versioning of namespaces.  (E.g. Don't claim an interface is
compatible if its parent namespace has changed in an incompatible way).

Versioning shall follow the Semantic Versioning principle:

Bumped minor numbers identifies backward-compatible additions to the
previous version.  This means that if a client requires version 1.3 of
a server namespace and its methods, it knows that it can safely
interface a server implementation of version 1.4 of that same
namespace.

Bumped major versions identifies non-compatible changes to the
previous version, such as a changed method signature. This means that
a client requiring version 1.3 of a server knows that it cannot invoke
a version 2.0 server implementation of that interface since that
implementation is not backward compatible.

For details that are not mentioned here, refer to https://semver.org, but if
there is a contradiction, this specification shall take precedence.

An optional field for patch version exists but it is recommended to avoid it
in most development processes since minor (or major) version should be used
indicate an actual change to the interface.  Unlike code, in this context we
are mostly interested in if there is a change in the interface and if it is
backwards compatible.

If `patch_version` is used anyway, it ought to be for documentation related
changes but small changes such as formatting or fixing a spelling mistake in
comments might also be recognized by using the git commit hash, which is a
guaranteed unique identifier of the IDL file contents.  A processes that uses
that commit hash won't rely on a human remembering to update it, and could be
an efficient replacement compared to constantly having to update the
patch-level version number.  In the end, it is however up to each adopter to
decide on which process they want to use.

Note also the additional field `version_label` that is a free-form field in
which implementations can place their own type of identifying information.
Companies might have their own compatibility information, their own way
of describing interface versions, or other information that goes into this
free-form text.  Some kind of unique hash of the interface content could be useful,
but it then requires hashing the content without the version_label line itself of course.

If the file is going to be transferred from one git to another system, the
process might fill in the version_label using the git commit hash _after_ the
file is taken out of git and transferred to that other system.  But it is of
course not possible write the commit hash into the file beforehand and commit
that content into git (because the actual commit hash is not decided until the
file is committed - and it would effectively depend on itself).  In these
cases, perhaps the development process may choose to perform another hash
calculation (SHA/MD5) of the interface content (again leaving out of course the
version_label lines themselves since there would otherwise be a circular
dependency again).

Ultimately, versioning can be used build-time to ensure that the correct
version of all needed interface/namespace implementations are deployed, while
also detecting if multiple, non-compatible versions of an interface are
required to be supported at the same time.


<!-- Types, constraints/ranges, type resolution in namespaces. -->
----

# FUNDAMENTAL TYPES

These are the supported fundamental (primitive) types, as generated from the source code model.  These primitive types are identical to the types used in the VSS (Vehicle Signal Specification) model, and of course they should easily match typical datatypes in other interface description systems.

|Name|Description|Min value|Max value|
|----|-----------|---------|---------|
|uint8 | unsigned 8-bit integer | 0 | 255|
|int8 | signed 8-bit integer | -128 | 127|
|uint16 | unsigned 16-bit integer | 0 | 65535|
|int16 | signed 16-bit integer | -32768 | 32767|
|uint32 | unsigned 32-bit integer | 0 | 4294967295|
|int32 | signed 32-bit integer | -2147483648 | 2147483647|
|uint64 | unsigned 64-bit integer | 0 | 2^64 - 1|
|int64 | signed 64-bit integer | -2^63 | 2^63 - 1|
|boolean | boolean value | False | True|
|float | floating point number | -3.4e -38 | 3.4e 38|
|double | double precision floating point number | -1.7e -300 | 1.7e 300|
|string | character string | N/A | N/A|

<!-- Layers concept, IFEX File Syntax, semantics and structure -->
-----------------------

# LAYERS CONCEPT

The IFEX approach implements a layered approach to the definition of interfaces,
and potentially other aspects of a system.  The core interface file (Interface
Description Language, or Interface Description Model) shall contain only a
_generic_ interface description that can be as widely applicable as possible.

As such, it avoids including specific information that only applies in certain
interface contexts, such as anything specific to the chosen transport protocol,
the programming language, and so on.

Each new **Layer Type** defines what new metadata it provides to the overall
model.  A Layer Type Specification may be written as a human-readable document,
but is often provided as a "YAML schema" type of file, that can be used to
programatically validate layer input against formal rules.

Layers do not always need to add new _types_ of information.  It is possible to
'overlay' files that use the same schema as an original file (such as the
ifex-idl), for the purpose of adding some details that were not yet defined,
removing nodes, or even redefining/changing the definition of some things
defined in the original file.

In other words, tools are expected to be able to process the layer types that
are relevant for the task at hand, but furthermore, some tools are expected to
process multiple files of the same type and to merge their contents according
to predefined rules.  Conflicting information could for example be handled by
writing a warning, or the tool may stipulate that the last provided layer file
to takes precedence over previous definitions.  (Refer to detailed documentation
for each tool).

An example of this:

**File: `comfort-service.yml`**  
```YAML
name: comfort
  typedefs:
    - name: movement_t
      datatype: int16
      min: -1000
      max: 1000
      description: The movement of a seat component
```

**File: `redefine-movement-type.yml`**  
```YAML
name: comfort
  typedefs:
    - name: movement_t
      datatype: int8 # Replaces int16 of the original type
```

Tools that combine this information will end up processing a combined YAML
structure that looks like this, in this case following the principle that the
last-definition takes precedence.

```YAML
name: comfort
  typedefs:
    - name: movement_t
      datatype: int8 # (Replaced datatype)
      min: -1000
      max: 1000
      description: The movement of a seat component
```

## Extending the interface model by mimicking the structure

Most layers\*\*\* will follow the same hierarchical structure as the original
interface definiton written in the IFEX Core IDL.  If a layer object list
element (e.g. `events`) is also defined in the IFEX file, the IFEX's list will
traversed recursively and extended by the deployment file's corresponding list.

\*\* For more information on layer types and different strategies, refer to the developers manual.

In this example, an overlay is used to add a new parameter to an existing
interface.  This is a bit of a peculiar case, but some development strategies
may use this to more clearly local modifications on top of a standard interface
description - such as an industry standard API.  Separating it into an overlay
makes it more explicit, but this should be understood as an example rather than
a design guideline.


**File: `comfort-service.yml`**  
```YAML
name: comfort
events:
  - name: seat_moving
    description:  The event of a seat starting or stopping movement
    in:
      - name: status
        datatype: uint8
      - name: row
        datatype: uint8

```

**File: `add_seat_moving_in_parameter.yml`**  
```YAML
name: comfort
events:
- name: seat_moving: 
    in:
      - name: extended_status_text
        datatype: string
```

The combined structure to be processed will look like the following:  (We show it here
using YAML syntax, but this combined representation might only exist only
inside the tool memory, after reading more than one input file.)

```YAML
name: comfort
events:
  - name: seat_moving
    description:  The event of a seat starting or stopping movement
    in:
      - name: status
        datatype: uint8
      - name: row
        datatype: uint8
      - name: extended_status_text
        datatype: string
```


Note: There is not a fixed list of layer types -- while some might be agreed
upon in a community and documented within the IFEX project, the capability is
also there to allow extensions that have not yet been envisioned.  Some aspects
of API and system design might be unique to one company's way of working, and
therefore defined and used only there.


# DEPLOYMENT LAYER

Deployment layer, a.k.a. Deployment Model files, is a specialization of the
general layers concept.  This terminology is used to indicate a type of layer
that in adds additional metadata that is directly related to the interface 
described in the IDL.  It is information needed to process, or interpret, IFEX
interface files in a particular target environment.

An example of deployment file data is a D-Bus interface specification to be used
for a namespace, or a SOME/IP method ID to be used for a method call.  

By separating the extension data into their own deployment files the
core IFEX specification can be kept independent of deployment details
such as network protocols and topology.

An example of a IFEX file sample and a deployment file extension to
that sample is given below:


**File: `comfort-service.yml`**  
```YAML
name: comfort
namespaces:
  - name: seats
    description: Seat interface and datatypes.

    structs: ...
    methods: ...
   ...
```

**File: `comfort-dbus-deployment.yml`**  

```YAML
name: comfort
namespaces: 
  - name: seats
    dbus_interface: com.genivi.cabin.seat.v1
```

The combined YAML structure to be processed will look like this:

```YAML
name: comfort
namespaces: 
  - name: seats
    description: Seat interface and datatypes.
    dbus_interface: com.genivi.cabin.seat.v1

    structs: ...
    methods: ...
```

The semantic difference between a regular IFEX file included by an
`includes` list object and a deployment file is that the deployment
file can follow a different specification/schema and add keys that
are not allowed in the plain IDL layer.  In the example above, the
`dbus_interface` key-value pair can only be added in a deployment file since
`dbus_interface` is not a part of the regular IFEX IDL file syntax.


# More Layer Information

For more information about Layer Types and Layer design, refer to the corresponding chapter in the [developers-manual.md](https://covesa.github.io/ifex/developers-manual)

----------

# IFEX FILE SYNTAX, SEMANTICS AND STRUCTURE

A Vehicle Service Catalog is stored in one or more YAML files.  The
root of each YAML file is assumed to be a `namespace` object and needs
to contain at least a `name` key, and, optionally, a `description`. In
addition to this other namespaces, `includes`, `datatypes`, `methods`,
`events`, and `properties` can be specified.

A complete IFEX file example is given below:

**NOTE: This example might be outdated**

```YAML
---
name: comfort
major_version: 2
minor_version: 1
description: A collection of interfaces pertaining to cabin comfort.

# Include generic error enumeration to reside directly
# under comfort namespace
includes:
  - file: vsc-error.yml
    description: Include standard VSC error codes used by this namespace

namespaces:
  - name: seats
    description: Seat interface and datatypes.

    typedefs:
      - name: movement_t
        datatype: uint16
  
    structs:
      - name: position_t
        description: The position of the entire seat
        members:
          - name: base
            datatype: movement_t
            description: The position of the base 0 front, 1000 back
    
          - name: recline
            datatype: movement_t
            description: The position of the backrest 0 upright, 1000 flat
    
    enumerations:
      - name: seat_component_t
        datatype: uint8
        options:
          - name: base
            value: 0
          - name: recline
            value: 1
    
    methods:
      - name: move
        description: Set the desired seat position
        in: 
          - name: seat
            description: The desired seat position
            datatype: movement.seat_t
    
    
    events:
      - name: seat_moving
        description: The event of a seat beginning movement
        in:
          - name: status
            description: The movement status, moving (1), not moving (0)
            datatype: uint8
    
    properties:
      - name: seat_moving
        description: Specifies if a seat is moving or not
        type: sensor
        datatype: uint8
```


The following chapters specifies all YAML objects and their keys
supported by IFEX.  The "Lark grammar" specification refers to the Lark
grammar that can be found [here](https://github.com/lark-parser/lark).
The terminals used in the grammar (`LETTER`, `DIGIT`, etc) are
imported from
[common.lark](https://github.com/lark-parser/lark/blob/master/lark/grammars/common.lark)

----

## NODE TYPES

The chapters that follow this one specify the node types for the core interface language/model.  They are generated from a "source of truth" which is the actual python source code of `ifex_ast.py`.  This means that while the examples are free-text and may need manual updating, the list of fields and optionality should always match the behavior of the tool(s).  => Always trust the tables over the examples, and report any discrepancies.

<!-- Syntax specification (generated from source) -->
----
## Namespace


Dataclass used to represent IFEX Namespace.

A namespace is a logical grouping of other objects, allowing for separation of datatypes, methods,
events, and properties into their own spaces that do not interfere with identically named objects
in other namespaces. Namespaces can be nested inside other namespaces to an arbitrary depth, building
up a scalable namespace tree. Namespaces can be reused either locally in a single file via YAML anchors,
or across YAML files using the includes object. The root of a YAML file is assumed to be a namespaces object,
and can contain the keys listed below.

A namespace example is given below.

```yaml
namespaces:
  - name: seats
    major_version: 1
    minor_version: 3
    description: Seat interface and datatypes.
```

#### Mandatory fields for Namespace:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |

#### Optional fields for Namespace:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |
| major_version | A single **int** |
| minor_version | A single **int** |
| version_label | A single **str** |
| events | A list of **Event**_s_ |
| methods | A list of **Method**_s_ |
| typedefs | A list of **Typedef**_s_ |
| includes | A list of **Include**_s_ |
| structs | A list of **Struct**_s_ |
| enumerations | A list of **Enumeration**_s_ |
| properties | A list of **Property**_s_ |
| namespaces | A list of **Namespace**_s_ |
| interface | A single **Interface** |


----
## Event


Dataclass used to represent IFEX Event.

Each events list object specifies a fire-and-forget call, executed by zero or more subscribing instances,
that does not return a value. Execution is best effort to UDP level with server failures not being reported.

A events sample list object is given below:

```yaml
events:
  - name: seat_moving
    description: Signal that the seat has started or stopped moving

    in:
      - name: status
        description: The movement status, moving (1), not moving (0)
        datatype: boolean

      - name: row
        description: The row of the seat
        datatype: uint8
```

#### Mandatory fields for Event:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |

#### Optional fields for Event:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |
| input | A list of **Argument**_s_ |


----
## Argument


Dataclass used to represent IFEX method Argument.

```yaml
methods:
  - name: current_position
    input:
      - name: row
        description: The desired seat to row query
        datatype: uint8
        range: $ < 10 and $ > 2

```

#### Mandatory fields for Argument:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |
| datatype | A single **str** |

#### Optional fields for Argument:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |
| arraysize | A single **int** |
| range | A single **str** |


----
## Method


Dataclass used to represent IFEX Event.

Each methods list object specifies a method call, executed by a single server instance,
that optionally returns a value. Execution is guaranteed to TCP level with server failure being reported.

A methods sample list object is given below:

```yaml
methods:
  - name: current_position
    description: Get the current position of a seat

    input:
      - name: row
        description: The desired seat to row query
        datatype: uint8
        range: $ < 10 and $ > 2

      - name: index
        description: The desired seat index to query
        datatype: uint8
        range: $ in_interval(1,4)

    output:
      - name: seat
        description: The state of the requested seat
        datatype: seat_t

    errors:
      - datatype: .stdvsc.error_t
        range: $ in_set("ok", "in_progress", "permission_denied")
```

#### Mandatory fields for Method:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |

#### Optional fields for Method:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |
| errors | A list of **Error**_s_ |
| input | A list of **Argument**_s_ |
| output | A list of **Argument**_s_ |
| returns | A list of **Argument**_s_ |


----
## Error


Dataclass used to represent a IFEX method error.

The optional error element defines an error value to return.  Note that the
concept allows for _multiple_ Errors.  This is easy to misunderstand: It is
not only multiple different error values as you are used to from most programming
environments.  Multiple value choices can be handled by a single Enumeration error type.
The concept also allows multiple independent return _parameters_, each having
their own data type (and name).

The purpose of this is to be able to separate different error categories
and to define them independently using layers.  For example, a method is likely
to have a collection of business-logic errors defined in its interface
description and represented by one enum type.  At a later time
transport-protocol specific errors can be added when the interface is
deployed over a certain protocol, and that error parameter has a different
name and a different type (enumeration or otherwise).

Error elements are returned in addition to any out elements specified for the method call.

Please see the methods sample code above for an example of how error parameter lists are used If no error element is
specified, no specific error code is returned. Results may still be returned as an out parameter`

```yaml
methods:
  - name: current_position
    description: Get the current position of a seat

    errors:
      - name: "progress"
        datatype: .stdvsc.error_t
        range: $ in_set("ok", "in_progress", "permission_denied")
      - <possibly additional error definition>

```

#### Mandatory fields for Error:

|Field Name|Contents|
|-----|-----------|
| datatype | A single **str** |

#### Optional fields for Error:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |
| description | A single **str** |
| arraysize | A single **int** |
| range | A single **str** |


----
## Typedef


Dataclass used to represent IFEX Typedef.

A Typedef is an alias to an existing fundamental type or defined type, including structs, enumerators, etc.  It can also be used to name and define a variant type.  The new data type name can be used in the definition of other datatypes, method- and event-parameters, and properties.

A typedefs list object example is given below:

```yaml
typedefs:
  - name: movement_t
    datatype: int16
    min: -1000
    max: 1000
    description: The movement of a seat component
```

**Variant types**

The fields `datatype` and `datatypes` are mutually exclusive - only one of them may have a value.  The field `datatypes` is used to specify multiple types associated with this type name.  Doing this makes it a `variant` type.  It is also possible to define a variant type using the `variant<a,b,c>`-style syntax in any location requiring a datatype, but the ability to specify it as a list can be more useful if the number of types is large.  Handling this as a list can also allow Layers to extend a variant type more conveniently.

```yaml
typedefs:
  - name: StringOrStruct
    datatypes:
      - string
      - MyStruct
      - OtherStruct
    description: The Thing, represented in one of 3 ways.
```

**NOTE!** The fields `datatype` and `datatypes` are defined as optional in the language model, but one of them must be defined.


#### Mandatory fields for Typedef:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |

#### Optional fields for Typedef:

|Field Name|Contents|
|-----|-----------|
| datatype | A single **str** |
| datatypes | A list of **str**_s_ |
| description | A single **str** |
| arraysize | A single **int** |
| min | A single **int** |
| max | A single **int** |


----
## Include


Dataclass used to represent IFEX Include.

Each includes list object specifies a IFEX YAML file to be included into the namespace hosting the includes list.
The included file's structs, typedefs, enumerators, methods, events,
and properties lists will be appended to the corresponding lists in the hosting namespace.

A includes sample list object is given below:

```yaml
namespaces:
  - name: top_level_namespace
    includes
    - file: vsc-error.yml;
      description: Global error used by methods in this file
```


#### Mandatory fields for Include:

|Field Name|Contents|
|-----|-----------|
| file | A single **str** |

#### Optional fields for Include:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |


----
## Struct


Dataclass used to represent IFEX Struct.

Each structs list object specifies an aggregated data type.
The new data type can be used by other datatypes, method & event parameters, and properties.

A structs list object example is shown below:

```yaml
structs:
  - name: position_t
      description: The complete position of a seat
      members:
      - name: base
          datatype: movement_t
          description: The position of the base 0 front, 1000 back

      - name: recline
          datatype: movement_t
          description: The position of the backrest 0 upright, 1000 flat
```


#### Mandatory fields for Struct:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |

#### Optional fields for Struct:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |
| members | A list of **Member**_s_ |


----
## Member


Dataclass used to represent IFEX Struct Member.

Each members list object defines an additional member of the struct.
Each member can be of a fundamental or defined/complex datatype.

Please see the struct sample code above for an example of how members list objects are used.

```yaml
structs:
  - name: position_t
    description: The complete position of a seat
    members:
      - name: base
        datatype: movement_t
        description: The position of the base 0 front, 1000 back
```

#### Mandatory fields for Member:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |
| datatype | A single **str** |

#### Optional fields for Member:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |
| arraysize | A single **int** |


----
## Enumeration


Dataclass used to represent IFEX Enumeration.

Each enumerations list object specifies an enumerated list (enum) of options, where each option can have its own
integer value.  The new data type defined by the enum can be used by other datatypes, method & event parameters, and
properties.

A enumerations example list object is given below:

```yaml
enumerations:
  - name: seat_component_t
    datatype: uint8
    options:
      - name: base
        value: 0

      - name: cushion
        value: 1
```

#### Mandatory fields for Enumeration:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |
| datatype | A single **str** |
| options | A list of **Option**_s_ |

#### Optional fields for Enumeration:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |


----
## Option


Dataclass used to represent IFEX Enumeration Option.

Each options list object adds an option to the enumerator.

Please see the enumerations sample code above for an example of how options list objects are used.

```yaml
options:
  - name: base
    value: 0
    description: description of the option value
```

#### Mandatory fields for Option:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |
| value | A single **Any** |

#### Optional fields for Option:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |


----
## Property


Dataclass used to represent IFEX Property.

Each properties list object specifies a shared state object that can be
read and set, and which is available to all subscribing entities.  A
properties sample list object is given below, together with a struct
definition:

```yaml
properties:
  - name: dome_light_status
    description: The dome light status
    datatype: dome_light_status_t

```

#### Mandatory fields for Property:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |
| datatype | A single **str** |

#### Optional fields for Property:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |
| arraysize | A single **int** |


----
## Interface


Dataclass used to represent IFEX Interface

An Interface is a container of other items in a similar way as a Namespace, but it does not introduce a new
namespace level. Its purpose is to explicitly define what shall be considered part of the exposed as a "public
API" in the output.

Note that as with all IFEX concepts, the way it is translated into target code could be slightly varying, and
controlled by the target mapping and deployment information, but all mappings shall _strive_ to stay close to the
IFEX expected behavior.  Thus, the Interface section would omit for example internal helper-methods and type
definitions that are only used internally, while those definitions might still need to be in the parent Namespace
for the code generation to work.

An Interface can contain all the same child node types as Namespace can, except for additional (nested) Interfaces.
As mentioned, it does NOT introduce an additional namespace level regarding the visibility/reacahbility of the
items.  Its contents is not hidden from other Namespace or Interface definitions within the same Namespace.  To put
it another way, from visibility/reachability point of view all the content inside an interface container is part of
the "parent" Namespace that the Interface object is placed in.  Of course, if additional nested namespaces are
placed _below_ the Interface node, those Namespaces introduce new namespace levels, as usual.

Only one Interface can be defined per Namespace, and it cannot include other Interfaces

An Interface example is given below.

```yaml
namespaces:
  - name: seats
    major_version: 1
    minor_version: 3
    description: Seat interface and datatypes.
    interface:
      typedefs:
        - ...
      methods:
        - ...
      properties:
        - ...
```

#### Mandatory fields for Interface:

|Field Name|Contents|
|-----|-----------|
| name | A single **str** |

#### Optional fields for Interface:

|Field Name|Contents|
|-----|-----------|
| description | A single **str** |
| major_version | A single **int** |
| minor_version | A single **int** |
| version_label | A single **str** |
| events | A list of **Event**_s_ |
| methods | A list of **Method**_s_ |
| typedefs | A list of **Typedef**_s_ |
| includes | A list of **Include**_s_ |
| structs | A list of **Struct**_s_ |
| enumerations | A list of **Enumeration**_s_ |
| properties | A list of **Property**_s_ |
| namespaces | A list of **Namespace**_s_ |



<!-- End of document -->