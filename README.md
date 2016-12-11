# A Specification for the Bitcoin Protocol

As started by [this article](https://medium.com/@zeptochain/reimagining-bitcoin-as-a-protocol-buffer-2d0https://medium.com/@zeptochain/reimagining-bitcoin-as-a-protocol-buffer-2d0d5d79a3c#.h7k50yzyp0d5d79a3c#.h7k5yzyp0) on medium.com.

## Reimagining Bitcoin as a Protocol Buffer
A discussion about how it wouldn’t be possible to establish a standard for Bitcoin, much less an RFC, got the wheels going. I wondered just how hard could it be to specify Bitcoin using [Google’s protocol buffer language](https://developers.google.com/protocol-buffers/) as a starting point?

After working through the Bitcoin messaging documentation, for maybe _an hour_, I got a pretty good start on it.

What this exercise showed me is a few things:

1. Thanks largely to __Bitcoin’s Creator, Satoshi Nakamoto__, Bitcoin is not a particularly difficult or complex messaging protocol
1. There are _not that many fields that need deep constraints_, but these additional validity rules will need to be specified in some formal way
1. When stated as protobufs, _there’s obvious room for improvement_ that would be consistent with the intent of the protocol

I think this may just be the first pass, since I see value in taking it further.

For now, let’s just suspend getting into detail on the differences in wire format between the protocol buffer codegen standard and those of Bitcoin, as there’s a lot to be said there.
Let’s focus on the message structures.

The basic stuff of Bitcoin is of course __blocks__ and __transactions__. These fall out quite naturally as:
```protobuf
message Block {
  uint32 version = 1;
  bytes previous_block = 2;
  bytes merkle_root = 3;
  uint32 timestamp = 4;
  uint32 bits = 5;
  uint32 nonce = 6;
  repeated Transaction transactions = 7;
}

message Transaction {
  uint32 version = 1;
  repeated TransactionInput inputs = 2;
  repeated TransactionOutput outputs = 3;
  uint32 lock_time = 4;
}

message TransactionInput {
  bytes outpoint_ref = 1;
  uint32 outpoint_index = 2;
  bytes sig_script = 3;
  uint32 sequence = 4;
}

message TransactionOutput {
  uint32 value = 1;
  bytes pk_script = 2;
}
```

What about the rest of the communications protocol? Well, first you have the generic message wrapper:

```protobuf
message Message {
  fixed32 magic = 1;
  string command = 2;
  uint32 payload_length = 3;
  uint32 checksum = 4;
  bytes payload = 5;
}
```

There’s a couple of other frequently used data structures:

```protobuf
message NetAddress {
  uint32 timestamp = 1;
  uint64 services = 2;
  bytes ip_address = 3;
  uint32 port = 4;
}

message InventoryVector {
  uint32 type = 1;
  bytes hash_reference = 2;
}
```

Once you have the above, then the rest of the structures and messages just fall out in fairly simple order:
```protobuf
message Version {
  uint32 version = 1;
  uint64 services = 2;
  uint64 timestamp = 3;
  NetAddress addr_to = 4;
  NetAddress addr_from = 5;
  uint64 nonce = 6;
  string user_agent = 7;
  uint32 start_height = 8;
  bool relay = 9;
}

message Verack { }

message Addr {
  repeated NetAddress address_list = 1;
}

message Inv {
  repeated InventoryVector vectors = 1;
}

message GetData {
  repeated InventoryVector vectors = 1;
}

message NotFound {
  repeated InventoryVector vectors = 1;
}

message GetBlocks {
  uint32 version = 1;
  repeated bytes block_locator_hashes = 2;
  bytes hash_stop = 3;
}

message GetHeaders {
  uint32 version = 1;
  repeated bytes block_locator_hashes = 2;
  bytes hash_stop = 3;
}

message GetAddr { }

message Mempool { }

message Ping {
  uint64 nonce = 1;
}

message Pong {
  uint64 nonce = 1;
}

message Alert {
  bytes payload = 1;
  bytes signature = 2;
}

message Reject {
  string message = 1;
  uint32 ccode = 2;
}

message FilterLoad {
  bytes filter = 1;
  uint32 number_of_hash_functions = 2;
  uint32 tweak_nonce = 3;
  uint32 flags = 4;
}

message FilterAdd {
  bytes data = 1;
}

message FilterClear { }

message MerkleBlock {
  uint32 version = 1;
  bytes previous_block = 2;
  bytes merkle_root = 3;
  uint32 timestamp = 4;
  uint32 bits = 5;
  uint32 nonce = 6;
  uint32 total_transactions = 7;
  repeated bytes hashes = 8;
  bytes flags = 9;
}
```
NOTE: What’s included above does not include the latest protocol change, i.e. __SegWit__, as that topic deserves a whole discussion for itself. The reason for that is that it touches Coinbase owing to the soft-fork and we need to break out on Coinbase transactions first.

I’ll leave you now with the thought that… maybe this specification thing isn’t so impossible after all?
