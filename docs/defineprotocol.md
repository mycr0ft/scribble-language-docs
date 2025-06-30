# Defining a Protocol

## A Basic Protocol

The following example shows the basic structure of a Scribble protocol:

```
    module scribble.example.Basic;

    type <xsd> "{http://scribble.org/examples}Greetings" from "HelloWorld.xsd" as Greetings;

    global protocol HelloWorld (role Me, role World) {
            hello(Greetings) from Me to World;
            hello(Greetings) from World to Me;
    }
```

The first line always defines the `module`, which is a *.* separated
name representing the location of the Scribble protocol file within a
hierarchy.

The last part of the module name, `Basic` in this case, represents the
filename with a *.scr* extension. The preceding parts represent a
folder/directory hierarchy. So the protocol above would be contained in
a file *scribble/example/Basic.scr*.

Need to describe root folder, or search path.

The purpose of a protocol is to exchange typed messages between
participants. The third line shows an example of how the protocol
declares these types.

The part between &lt; &gt; identifies the nature of the type, in this
case it is an XSD (XML Schema Definition). The next part, which is in
double quotes, identifies the type in a format appropriate for the
schema. The `from` value (also in double quotes) represents the location
of the schema definition. Finally the `as` value is an alias which the
protocol will use to reference this type.

The final part of this simple protocol is the `global protocol`
definition. The name of this global protocol is `HelloWorld` and it is
associated with two roles, `Me` and `World`. This means that the
protocol will describe the interactions that occur between these roles
from a global (or endpoint neutral) perspective. Later in this guide we
will discuss the topic of projection and the local protocols that
describe the behaviour for each of the roles involved in the global
protocol.

In the body of the global protocol there is two message transfer (or
interaction) statements. This statements identify the message signature
(operation name followed by the types(s) within the braces), and the
`from` and `to` roles. So in this example the role `Me` is saying
*hello* to `World`, and then `World` responds by saying *hello* back.

The role names used in the `from` and `to` clauses must have previously
been defined in the protocol header.

Although described as two separate statements, this simple pattern can
be used to represent a request/response interaction between the two
parties (roles). If this is the intention, then when defining the
protocol it is recommended that the operation name be the same on both
statements.

## Adding Decision Points

The previous section introduced a simple protocol which involved a
sequence of two interactions (or message transfers). Most protocols
involve some decision points which result in the protocol having a
different behaviour.

The following example illustrates how a decision point can be defined.
We’ll change the protocol to represent a more interesting purchasing
example:

```
    module scribble.example.Purchasing;

    type <xsd> "{http://scribble.org/examples}QuoteRequest" from "Purchasing.xsd" as QuoteRequest;
    type <xsd> "{http://scribble.org/examples}Quote" from "Purchasing.xsd" as Quote;
    type <xsd> "{http://scribble.org/examples}Order" from "Purchasing.xsd" as Order;
    type <xsd> "{http://scribble.org/examples}OrderAck" from "Purchasing.xsd" as OrderAck;
    type <xsd> "{http://scribble.org/examples}OutOfStock" from "Purchasing.xsd" as OutOfStock;

    global protocol BuyGoods (role Buyer, role Seller) {
            quote(QuoteRequest) from Buyer to Seller;

            choice at Seller {
                    quote(Quote) from Seller to Buyer;
                    buy(Order) from Buyer to Seller;
                    buy(OrderAck) from Seller to Buyer;
            } or {
                    quote(OutOfStock) from Seller to Buyer;
            }
    }
```
As in the original example, we have a module definition followed by the
type declarations. In this example we have five types defined.

The `global protocol` is similar to the first example, however instead
of simply returning a single response, this example introduces the
`choice` construct.

A `choice` represents a decision point within the protocol. Decisions
must always be made at a nominated role, with that decision then being
communicated to the other roles based on the behaviour defined in the
selected path.

This means that each path within the choice *must* begin within a
message transfer initiated by the role that made the decision.
Therefore, the following choice statement would be invalid:

            choice at Seller {
                    quote(Quote) from Seller to Buyer;
            } or {
                    // Invalid, as initial interaction must come from role 'Seller'
                    cancel(CancelRequest) from Buyer to Seller;
            }

To ensure that the other (non-decision making) roles correctly infer
which path should be taken, the message types of the initial message
transfers from the decision making role must be distinct. So, the
following example would be invalid:

            choice at Seller {
                    quote(Quote) from Seller to Buyer;
                    buy(Order) from Buyer to Seller;
                    buy(OrderAck) from Seller to Buyer;
            } or {
                    // Invalid, as type 'Quote' was used in the other choice path
                    quote(Quote) from Seller to Buyer;
                    reject(Order) from Buyer to Seller;
            }

Protocols where the decision is conveyed within the message contents is
not currently handled by this version of Scribble.

## Concurrency

This section will discuss how protocols can be defined where behaviour
can be performed concurrently (i.e. in parallel). We will extend the
previous example by adding some new roles, and a parallel construct to
check concurrently whether the buyer’s credit is acceptible and if there
is enough stock to satisfy the quote.

    module scribble.example.Purchasing;

    type <xsd> "{http://scribble.org/examples}QuoteRequest" from "Purchasing.xsd" as QuoteRequest;
    type <xsd> "{http://scribble.org/examples}Quote" from "Purchasing.xsd" as Quote;
    type <xsd> "{http://scribble.org/examples}CreditReport" from "Purchasing.xsd" as CreditReport;
    type <xsd> "{http://scribble.org/examples}StockAvailability" from "Purchasing.xsd" as StockAvailability;
    type <xsd> "{http://scribble.org/examples}Order" from "Purchasing.xsd" as Order;
    type <xsd> "{http://scribble.org/examples}OrderAck" from "Purchasing.xsd" as OrderAck;
    type <xsd> "{http://scribble.org/examples}InsufficientCredit" from "Purchasing.xsd" as InsufficientCredit;
    type <xsd> "{http://scribble.org/examples}OutOfStock" from "Purchasing.xsd" as OutOfStock;

    global protocol BuyGoods (role Buyer, role Seller, role CreditAgency, role Store) {
            quote(QuoteRequest) from Buyer to Seller;

            par {
                    creditCheck(Quote) from Seller to CreditAgency;
                    creditCheck(CreditReport) from CreditAgency to Seller;
            } and {
                    stockCheck(Quote) from Seller to Store;
                    stockCheck(StockAvailability) from Store to Seller;
            }

            choice at Seller {
                    quote(Quote) from Seller to Buyer;
                    buy(Order) from Buyer to Seller;
                    buy(OrderAck) from Seller to Buyer;
            } or {
                    quote(InsufficientCredit) from Seller to Buyer;
            } or {
                    quote(OutOfStock) from Seller to Buyer;
            }
    }

Each of the paths defined within the par construct are expected to occur
concurrently. The protocol will not progress past the par construct
until each of the paths within the par have completed.

To avoid potential conflicts between message exchanges in the concurrent
paths, it is currently necessary to use distinct operator names between
role pairs in different paths. So (for example), in the above example,
the second path could not use *creditCheck* as an operator.

## Recursion

The recursion construct can be used to define protocols involving
repetition.

    module scribble.example.Purchasing;

    type <xsd> "{http://scribble.org/examples}QuoteRequest" from "Purchasing.xsd" as QuoteRequest;
    type <xsd> "{http://scribble.org/examples}Quote" from "Purchasing.xsd" as Quote;
    type <xsd> "{http://scribble.org/examples}Order" from "Purchasing.xsd" as Order;
    type <xsd> "{http://scribble.org/examples}OrderAck" from "Purchasing.xsd" as OrderAck;
    type <xsd> "{http://scribble.org/examples}OutOfStock" from "Purchasing.xsd" as OutOfStock;

    global protocol BuyGoods (role Buyer, role Seller) {

            quote(QuoteRequest) from Buyer to Seller;

            rec SubmitQuote {
                    choice at Seller {
                            quote(Quote) from Seller to Buyer;
                            buy(Order) from Buyer to Seller;
                            buy(OrderAck) from Seller to Buyer;
                    } or {
                            quote(OutOfStock) from Seller to Buyer;
                    }

                    choice at Buyer {
                            quote(QuoteRequest) from Buyer to Seller;

                            continue SubmitQuote;
                    } or {
                            quit() from Buyer to Seller;
                    }
            }
    }

The `rec` block defines the scope of the recursion, with a label to
identify the block (i.e. SubmitQuote in this example).

The `continue` statement defines where the recursion should actually
occur. In this example, after the initial quote request has been
handled, the `Buyer` then has the option to either submit a further
quote, or quit the session. If they submit another quote request, then
it will continue back to the start of the recursion block where it will
await the response from the `Seller`.

The recursion block will only complete when the `Buyer` sends a `quit`
message to the `Seller`.
