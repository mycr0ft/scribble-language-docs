
# Scribble Language Guide

These documents are markdown version of those on [the original scribble github repo](https://github.com/scribble/scribble-language-guide). 
Those work well for docbook and adoc format but markdown and mkdocs is more popular these days. 

You will probably want to read the [defining a protocol](defineprotocol) document next.
There are also additional [advanced constructs](advanced) in the scribble language.

# What is Scribble?

Scribble is a language to describe application-level protocols among
communicating systems. A protocol represents an agreement on how
participating systems interact with each other. Without a protocol, it
is hard to do a meaningful interaction: participants simply cannot
communicate effectively, since they do not know when to expect the other
parties to send their data, or whether the other party is ready to
receive a datum it is sending. In fact it is not clear what kinds of
data is to be used for each interaction. It is too costly to carry out
communications based on guess works and with inevitable communication
mismatch (synchronisation bugs). Simply, it is not feasible as an
engineering practice.

Scribble presents a stratified description language:

-   The bottom layer is the type layer, in which we describe the bare
    skeleton of conversations structures as types for interactions
    (known in the literature as session type).

-   The assertion layer allows elaboration of a type-layer description
    using assertions.

-   Finally the third layer, protocol document layer, allows description
    of multiple protocols and constraints over them.

Each layer offers distinct behavioural assurance for validated programs.

# How can it be used?

The development and validation of programs against protocol descriptions
could proceed as follows:

-   A programmer specifies a set of protocols to be used in her
    application.

-   She can verify that those protocols are valid, free from livelocks
    and deadlocks.

-   She develops her application referring to those protocols,
    potentially using communication constructs available in the chosen
    programming language.

-   She validates her programs against protocols using a protocol
    checker, which detects lack of conformance.

-   At the execution time, a local monitor can validate messages with
    respect to given protocols, optionally blocking invalid messages
    from being delivered. 
