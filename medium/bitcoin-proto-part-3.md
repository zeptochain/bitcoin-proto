# Bitcoin Script as a Protocol Buffer

OK. I just gave the game away by shoving it into the title. A bit late to say “spoiler alert”, I guess.

Last time I left the idea hanging that Bitcoin’s consensus rules could perhaps be expressed most effectively using Bitcoin’s own scripting language. What should give us additional confidence in this choice is that Bitcoin Script is a language that was explicitly designed for expressing distributed contracts. Surely, it would be impossible to argue that the consensus rules for Bitcoin, when taken together, are anything other than a contract?

Unfortunately, the present situation is that the contract terms that we expected to agree to is delivered to us as an unordered, highly cross-referenced, constantly changing, “book of codes” written in a language that only a minority actually understand, and which includes a massive dead weight of rules and concerns that have nothing at all to do with the contract. You shouldn’t have to suck that up; you wouldn’t just suck that up if your home insurance policy or credit card conditions were delivered to you in that form. Surely we deserve better.

So I guess the right thing to do is:

1. Find all the rules
1. Write a Script for each that encapsulates that constraint
1. Write a natural language explanation for each script
1. Put it all in one place and in a logical order
1. See if we actually do agree with the resulting contract

Time to get to work, right? All that should be enough for anyone to have to swallow! But wait a moment… we’re not going to let ourselves off the hook that easy. Instead, let’s go all the way and express Script itself as a protocol buffer, to see where that path takes us. As it turns out, it too leads somewhere kinda interesting.

Let’s take quick look at how you could do something this ludicrous, since you’re likely wondering about that, and the “how” bit is actually pretty easy:
```protobuf
message Script {
  repeated Operation operations = 1;
}
message Operation {
  enum Code {
    OP_FALSE = 0;
    // and so on
    OP_NOP10 = 185;
  }
  Code opcode = 1;
  bytes data = 2;
}
```

OK, now we know it’s trivially possible (the full expression of the above is in the code repo on GitHub), we can talk about why you might want to — and even entertain the idea that you actually should want to.

1. We now have a lingua franca to launch higher level discussions, since any data structures and constraint functions now have a standard and platform-neutral format.
1. The proto file can be used to generate the required reference client code in under a minute for pretty much any language and any platform.
1. Each node has the capability to be exact and specific about which rules it is willing to observe, and is able to tell its peers.
1. These consensus rules could be transmitted, accepted, loaded and executed as scripts at runtime. You know, a bit like Ethereum, but limited to deterministic, fully analyzable rules.
1. The serialized form of a protocol buffer offers an alternative but more robust byte-accurate representation of data structures that is eminently suitable for the calculation and verification of hash digests and digital signatures.

Pretty worthwhile stuff, hunh? But before we step off the cliff, there’s a real burning issue that we need to address. What could that be? Well, weirdly, it’s about a specific systems programming language.

---

OK, this point has been building for a while. It’s time to me to state that there’s an issue on which I disagree with Satoshi’s view.

The crowd gasps... _what are you saying?!?_

Actually, it’s somewhat less controversial than you’re thinking. The problem is that the language used to code for a Bitcoin “reference” node is C++, and the decisions made for, and the properties of, that expression in software are highly influenced by this choice. This causes all sorts of issues, and Satoshi was unsure whether it would be possible to have more than one implementation of a “reference” client. I don’t share that view. Why?

It’s a multi-platform, multi-language, and _multi-core_ world, folks. And Bitcoin is distributed, concurrent, and _message-oriented_ system.

Back in the 90’s C++ was the hair-shirt language by which all others should be measured. The incumbent C++ crowd acts more than a lil’ bit like the ASM crowd a decade before that, and the machine code crowd the decade or so before that. If you hold the belief that C++ is the “one true language”, then it’s easy to simply ignore what’s been discovered since. C++ basically locks you into a 20th Century systems programming mindset. You can gradually become a servant of your own codebase, and you are certainly destined to run persistently into the worst kinds of issues, i.e. those related to security and concurrency.

My big thing point here is that C++ is not the greatest choice for a reference implementation of something like Bitcoin, since it eventually traps you into thinking that the only way ahead is to declare that “the code is the documentation”. For many, that outcome has already happened. But the enforced mindset and distractions you inherit from the use of that tool can negatively affect your ability to think clearly about the real issues. So it really isn’t progress at all. Worse than that, it’s a very damaging development — a total mis-step.

---

Finally, let’s take a moment to see where we’ve landed, as it has gone far beyond the original motivation of merely specifying how Bitcoin works now:
1. We have the opportunity to build an unambiguous specification with little room for misunderstandings.
1. We have a language and platform neutral serialization format for all of the data structures and constraint functions that could be contained in this specification.
1. Proposed new rules could be distributed and evaluated in unambiguous form, and could be accepted or rejected to form a new consensus.

So is this still Bitcoin or is this some alt-coin? I strongly suspect the former. In fact, I actually feel quite strongly that it would be more Bitcoin than Bitcoin currently is. You may or may not buy into this proposition, but there’s really only one way to find out if all this will pan out.

Yes, it’s time to get to work.
