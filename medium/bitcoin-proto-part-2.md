# Bitcoin as a Protocol Buffer (Part 2)
So here’s the thing. The more you dig into this, the more interesting this becomes.

After a first article on this, I set up a GitHub repo to track this, as I think there’s an awful lot more to be said.

After a first pass at a “bitcoin.proto”, I took the obvious next steps of setting up obvious default values and adding in enums to the definition, you know, simple stuff like:
```protobuf
message InventoryVector {
  enum VectorType {
    ERROR = 0;
    TRANSACTION = 1;
    BLOCK = 2;
    FILTERED_BLOCK = 3;
  }
  VectorType type = 1;
  bytes hash_reference = 2;
}
```
…so I added that version of the definition as “bitcoin-1.proto” to the repo. I took it one step further and turned the message wrapper’s command field from a (zero-padded) string into an enum as it seemed the most natural thing to do — a source of discussion for another time, methinks.

We could (and no doubt will) argue all that stuff out line-by-line, field by field… but I suspect there would be little room for dissent after the fact. I mean how many political positions can you take on a version number or a timestamp? Some things likely need a bit more explanation, for instance, I peeled out Coinbase, since it’s not just a transaction — it’s always the first transaction; it can have only one input; and that input has all that hacked-in special sauce in its sig_script.

All in all, those kinds of discussions seem eminently solvable, and often with fairly trivial logic.
A few things started to occur to me in fairly short order. It started with the observation that a lot of the concerns in the protocol are a direct result of it being originally coded in C/C++ (and a few of the bugs too), and frankly, it seemed that this was a case of the choice of implementation language tying people in mental knots. Shrug. Then I started to wonder…

_…what if the Bitcoin protocol was actually implemented as a protocol buffer?_

Well, I guess that was a cool thought, since this is what drops out:

1. Generating clients in multiple languages would become trivial. I would have thought wallet developers would truly appreciate this.
1. A number of items on the languishing wishlist for “hard-fork” rationalizations could be directly addressed.
1. A protobuf-defined wire format could run alongside the current implementation of the wire format, simply by offering it on a different port. No need to hard-fork, no soft-fork hack, just a pleasant alternative choice for other node clients. If people moved across to it, it would gradually become the main protocol. Dream on.
1. Without any other change, a protobuf version would be far more efficient over the wire (right down to the varints used). Neat.
1. Wouldn’t it be awesome to have a block header with a 64-bit nonce and timestamp? I’m sure the miners amongst us could see the value in this for their own scalability. And that kind of change would be much easier to roll out with a protobuf definition

---

Now we get to the elephant in the room... the messaging protocol is one thing, but it’s not the whole story. The current definition surely needs to reflect semantics more closely. Why?
Well, protobuff gives you a certain level of data validation and a smattering of semantics, but in the end an awful lot of validation decisions happen inside the black box. Before we accept the Bitcoin specification game as intractable, as has been asserted by more than one expert, let’s start to take all that apart.

The black box executes rules that transform the data inputs into data outputs. While most transforms are straightforward queries, there are some rules that have more depth, and include state transformations. Usually, these are lumped together as being “the consensus rules”. I’d argue that lumping all the “difficult stuff” together into a big bag is really a means to avoid proper specification and allows a huge amount of unhelpful and uninsightful hand-waving.

What we really need is a formal way to define those rules. Just as protobuf helps clarify and offers interesting options to the messaging protocol, we need a clear language in which to define those rules, so that we can categorize and accurately visualize how the box internals work without recourse to implementation specifics and inherent distractions.

Maybe even, like with protobuf, you could distribute those rules and end up with wide-ranging but easy to implement updates. It’s not a great stretch of the imagination that such rules could be applied as automatic updates? Crazy stuff indeed!

The challenge here is to find an appropriate common language. More specifically, language that is:

1. Easy to learn;
1. Likely be commonly understood by bitcoin folks already;
1. Sufficiently deterministic to serve as a rule specification language;
1. Compact enough to allow us to retain focus on the behavior it expresses and not on the syntax.

Any ideas? Actually, one language leapt out at me as soon as I asked the question to myself in the above way — drumroll, please…

_…the Bitcoin Scripting language_.

I’m not talking about the hobbled version in current nodes, but the full language as orginally defined by Satoshi, OK? So, borrowing from the data structures defined in our proto, we can start to think of what a rule written in Script would look like, e.g. Verify <value> is a valid SHA-256 hash:
```
OP_PUSH Block.merkle_root, OP_SIZE, OP_PUSH 32, OP_EQUALVERIFY
```
…so that’s where I’m going next. Fun stuff, eh? Stay tuned!

---

Comments are welcomed below, or perhaps more usefully and trackably as issues on the repo.
