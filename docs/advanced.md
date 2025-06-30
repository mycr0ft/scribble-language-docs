# Advanced Constructs

## Interruptible

The *interruptible* construct is used to describe a communication where
one party, that is pending a message from another party, decides to
interrupt the conversation (i.e. abort, timeout, cancel). This situation
needs special consideration, as the interrupt message may cross the
response from the other party, leading to non-deterministic behaviour in
both roles.

    module scribble.example.Interruptible;

    type <xsd> "{http://scribble.org/examples}Order" from "Examples.xsd" as Order;
    type <xsd> "{http://scribble.org/examples}Receipt" from "Examples.xsd" as Receipt;

    global protocol GInterruptibleTest(role Buyer,role Seller) {
            interruptible MyLabel: {
                    buy(Order) from Buyer to Seller;
                    buy(Receipt) from Seller to Buyer;
            } with {
                    cancel(Order) by Buyer;
            }
    }

The *interruptible* construct enables the normal flow of behaviour to be
described within the main block, with the possible interrupt messages to
be defined in the **with** clause.

## Multicast

The multicast pattern enables a message to be sent from one role to more
than one other role.

    module scribble.example.Multicast;

    type <xsd> "{http://scribble.org/examples}Notification" from "Examples.xsd" as Notification;

    global protocol Notify (role Client, role ServerA, role ServerB) {
            send(Notification) from Client to ServerA, ServerB;
    }

In this example, the message is sent from the *Client* role to roles
*ServerA* and *ServerB* at the same time.

## Sub-Protocols

As protocols become more complicated, as with any type of program, we
need a way to decompose the description into more manageable components.
The other significant advantage of this decomposition is to create
reuseable components that can be used in building many other higher
level protocols (especially in combination with the geneics support
described in the next section).

Sub-protocols can either be:

-   invoked within the same module, if the *calling* and *called*
    protocols are defined in the same module file

-   defined in separate modules with,

    -   the **do** statement in the *calling* protocol providing the
        fully qualified module and protocol name

    -   the *calling* prototocol’s module importing the *called*
        protocol’s module

Each of these approaches will be described in the following
sub-sections.

### Invoking a protocol in the same module

The following example shows how to call a protocol that has been defined
within the same module.

    module scribble.example.SubProtocols;

    type <xsd> "{http://scribble.org/examples}Notification" from "Examples.xsd" as Notification;

    global protocol Main (role Client, role Server) {
            do Sub(Client as R1, Server as R2);
    }

    global protocol Sub (role R1, role R2) {
            send(Notification) from R1 to R2;
    }

The global protocol *Main* is calling the global protocol *Sub*. In this
example, the roles defined within the *Main* protocol are mapped to the
ones declared in the *Sub* protocol using the **as** keyword. Roles can
also be associated based on their position within the list, e.g.

            do Sub(Client, Server);

### Expliciting referencing a protocol in another module

The following example shows how to call a protocol that has been defined
within another module based on a fully qualified name. The first module
is the *main* module with the *calling* protocol,

    module scribble.example.MainModule;

    global protocol Main (role Client, role Server) {
            do scribble.example.SubModule.Sub(Client as R1, Server as R2);
    }

and the second module is the *sub* module with the *called* protocol,

    module scribble.example.SubModule;

    type <xsd> "{http://scribble.org/examples}Notification" from "Examples.xsd" as Notification;

    global protocol Sub (role R1, role R2) {
            send(Notification) from R1 to R2;
    }

### Importing a protocol from another module

The following example shows how to call a protocol that has been
imported from another module. The first module is the *main* module with
the *calling* protocol,

    module scribble.example.MainModule;

    import scribble.examples.SubModule;

    global protocol Main (role Client, role Server) {
            do Sub(Client as R1, Server as R2);
    }

and the second module is the *sub* module with the *called* protocol,

    module scribble.example.SubModule;

    type <xsd> "{http://scribble.org/examples}Notification" from "Examples.xsd" as Notification;

    global protocol Sub (role R1, role R2) {
            send(Notification) from R1 to R2;
    }

In the main module, the fully *sub* module is imported, allowing any of
the types or protocols defined in that module to be referenced based on
the local names (e.g. *Sub* for the protocol name).

It is also possible to just import a particular protocol or type from
another module, using the following variation of the import statement:

    from scribble.examples.SubModule import Sub;

Finally, if there is a name conflict with the protocol or type being
imported, then it can be locally renamed using an alias. For example,

    from scribble.examples.SubModule import Sub as OtherProtocol;

    ...

            do OtherProtocol(Client as R1, Server as R2);
    ...

## Generics and parameters

TO BE DEFINED
