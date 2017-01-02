# The State of Bitcoin... as a Protocol Buffer

Only fair to TL;DR that what I’m addressing here is the internal computational state of a Bitcoin node. If you are looking for analysis of the economics or politics of the game this may not be for you.

If you have read the first three parts, then you are likely wondering how far this protocol buffer madness is going to be dragged out. The good news is that it all ends here and here’s why.

The goal of this exploration was to find a way to write a full and unambiguous specification for Bitcoin, and we are getting close. Such a specification should be implementation-independent, and any implementation offered up should be judged on its conformity to that specification. In other words, we have to treat an actual Bitcoin node as a black box whose output behavior is deterministic for all inputs. 

---

If you only like to watch the first minute and last minute of a movie, you may want to skip this section  -  but I suggest you don’t!

To get a feel for the black box approach let’s look at the features revealed by the simplest non-degenerate case. Let’s say I have a box which accepts an input and gives an output. Let’s poke it with inputs…
```
“zeptochain” -> error
1 -> error
-1 -> error
'z' -> error
[1,2,3] -> error
```

...well, that stinks, right? I probably should have told you, or specified, that it only accepts numbers between 1,000 and 10,000 as valid input messages. For those input messages it will output a number, for anything else it will output an error message. OK, let’s try again...
```
1000 -> error
```

Darn it! You are likely getting a bit annoyed with my box by now — why does that situation sound familiar? Yep, I actually did fully specify the constraint, but I made the (bad) assumption that you’d understand that when I said “between”, I meant exclusive of the limits, so the valid inputs are only 1001, 1002, 1003... and so on up to 9999. Hmph, OK. Back to the box...
```
1001 -> 15, 2500 -> 21, 8476 -> 0, 8477 -> 9, 8477 -> 18,
8477 -> 27, 8477 -> 7, 8477 -> 16, 1001 -> 2
```

It “works”! But what on earth is it doing? If you enjoy puzzles, then it’s possible and fun to try to figure it out. What’s happening here is actually pretty simple:
1. Take the input
1. Add it to the previous output
1. Find the remainder of that number divided by 29
1. Send that as the output

So when I sent it 8477 for the second time it calculated (8477 + 9) / 29 and sent the remainder back. Now, any programmer could choose to implement this box in any language knowing the above, and you couldn’t tell the difference between my implementation and theirs. Well, not quite... here’s the missing piece — the wire protocol (as a protobuff):
```protobuf
message InputMessage {
  int32 value = 1;
}
enum Error {
  ERROR = 0;
}
message OutputMessage {
  oneof output {
    int32 value = 1;
    Error error = 2;
  }
}
```
Yay, congrats! You should now be able to code your very own version of the box in whatever language or platform you choose. It would of course help enormously to have the rules laid out in an unambiguous and compact form, rather than just a natural language description. So first, there’s the validation rules...
```
[OP_PUSH, 1000, OP_PUSH, InputMessage.value, OP_GREATERTHAN]
[OP_PUSH, 10000, OP_PUSH, InputMessage.value, OP_LESSTHAN]
```

Note that we already have a way to communicate these rules over the wire, since we showed how to specify Bitcoin Script as a protobuff in the last article. Finally, let’s get to the rule for the behavior...
```
[OP_PUSH, 29, OP_PUSH, InputMessage.value, OP_PUSH, the_last_output_value, OP_ADD, OP_MOD]
```
O...M...G... what’s with this push of `the_last_output_value`? This is really getting very tedious. Well, newsflash! Specification is a tedious process because it needs to be complete. The issue here is that the box is maintaining __state__, and we haven’t yet defined that. 

Let’s do that now  -  and yes, guess what? We’re going to use a protocol buffer, since we can both define our state and specify a means to communicate it over the wire in one fell swoop...
```protobuf
message State {
  int32 last_output_value = 1;
}
```

Now there’s a decent specification of what the box does, you can fully predict its behavior, and validate that it’s obeying the rules! You could extend the box to be able to declare its specification in an entirely platform independent format and to tell you its current state. When you prod it with input, you can know what to expect  -  completely.

Although our journey has indeed become a little tedious, I’d argue that being fobbed off with statements like the following are far more tedious:

> “Officially Bitcoin script is defined by its reference implementation”

Go ahead, take a look at this goop… then ask yourself, how much code do you have to read and analyze before the statement above reaches actual definition of the behavior? The fundamental issue here is that Bitcoin Script itself needs a proper specification… so it rather looks like we will have to face that issue in full in the near future.

---

Fortunately, there is some light at the end of this tunnel, and we can get a better perspective on Bitcoin itself.
You should be convinced by now that a complete specification of Bitcoin requires definition of messaging, rules and state. All three are required, any two “just won’t do”. We hadn’t considered the state of Bitcoin nodes up to this point, but it turns out that the amount of state required to understand “the black box that is Bitcoin” is relatively trivial. Let’s take a first swipe:

```protobuf
message NodeState {
  bytes genesis_block_hash = 1;
  bytes current_block_hash = 2;
  int32 current_block_height = 3;
}
```

And that’s it. The state of Bitcoin as a protocol buffer. It’s my proposition that everything else can be derived from the validation and execution rules — but please feel free to point out what’s missing in the above specification if you notice something.

---

OK, what about the big bag of rules? Sure there’s bound to be quite a few, but to get a start let’s pick up on what might be thought of as the top three...

1. The “main chain” is defined by the chain with the greatest amount of work behind it.
1. Every 210,000 blocks, the valid coinbase reward is halved.
1. Every 2016 blocks, the difficulty is adjusted so that the next 2016 blocks will take two weeks.

…such statements are very far from a specification of the Bitcoin contract, as we need such terms and conditions to be far more accurately stated or even true. And that’s where we’ll be going next time. 

Let’s face it, RTSL (a.k.a. the source code is the specification) is not a valid answer for the majority, and __the majority deserve to be able to fully understand the contract they are signing up for__. No wonder the community is in such disarray.
