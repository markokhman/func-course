# Chapter 1. Lesson 3. Breakdown the logic of classic wallet code

### Overview of a wallet

In TON, there is no distinction between "user's account" and "smart contract". Every account is a smart contract. This means, every user’s wallet is actually a piece of code (on a blockchain) that authenticates transfers of coins and other assets.

https://github.com/toncenter/tonweb/blob/master/src/contract/wallet/WalletSources.md

The job of the basic wallet is to:

1. Receive an incoming message containing inner message, destination address and a signature.
2. Check the signature against the built-in public key.
3. Increment a sequence number to prevent replays.
4. Send the inner message to the destination contract.

### Wallet state

From the above description it is clear that the basic state of the wallet is:

1. The owner's public key.
2. Sequence number.
3. Coins on its balance.


### Other assets

In TON, all other assets (other than toncoins) are implemented as distinct contracts that have an _owner_: an address of another contract that is allowed to send specific messages to it.

So to transfer a token, a wallet contract needs to emit a "transfer" message to the token contract that references the wallet’s address as an owner. That token contract will receive the message, check that the message sender (the wallet) matches the `owner` field and then does its thing: either "transfers" some amount as a message to another token (see lesson 5), or switches the owner to a new one (see lesson 4).


### Digging into code

Let's take a look at an excerpt of FunC code from wallet v4. We will ignore all the logic related to plugins and focus on the basic flow.

First, we are interested in _external_ messages: those that come from "outside" the blockchain — these are unauthenticated pieces of data that anyone can send to any contract. They have no "sender" attribute and any contract that receives such data needs to implement its own verification logic.

```
() recv_external(slice in_msg) impure {
```

Then, we read the signature and the expiration time from the message.
Fail immediately without changing anything if the message has expired.

```
  var signature = in_msg~load_bits(512);
  var cs = in_msg;
  var (subwallet_id, valid_until, msg_seqno) = (cs~load_uint(32), cs~load_uint(32), cs~load_uint(32));
  throw_if(36, valid_until <= now());
```

Then, load the entire state of the wallet. In a Tact language that is done automatically, in FunC it must be done explicitly.

```
  var ds = get_data().begin_parse();
  var (stored_seqno, stored_subwallet, public_key, plugins) = (ds~load_uint(32), ds~load_uint(32), ds~load_uint(256), ds~load_dict());
  ds.end_parse();
```

Next, verify that the message is bound to the latest sequence number, matches this specific wallet instance and the signature is valid.

```
  throw_unless(33, msg_seqno == stored_seqno);
  throw_unless(34, subwallet_id == stored_subwallet);
  throw_unless(35, check_signature(slice_hash(in_msg), signature, public_key));
```

Now that we have authenticated the sender, we can start paying transaction fees from this account. All the code before `accept_message` was running on a small budget of "loaned" gas that nodes use to allow external messages to authenticate themselves and reject garbage. But that amount of gas is small enough to avoid denial-of-service issues and not enough to do any useful work except for checking one or two signatures.

Therefore, at this point we need to tell TON that the contract is going to pay for its execution by calling built-in `accept_message` function. 

```
  accept_message();
```

Now, we can update the state of the contract with incremented sequence number.

```
  set_data(begin_cell()
    .store_uint(stored_seqno + 1, 32)
    .store_uint(stored_subwallet, 32)
    .store_uint(public_key, 256)
    .store_dict(plugins)
    .end_cell());
```

To make sure that sequence number is incremented and we won't accept the same message again, we commit the state. This means that even if there is any issue with the inner message (e.g. it is malformed by the user, or there is not enough coins to process it), the execution will fail, but new state with the updated sequence number won't be reverted.

```
  commit();
```

Finally, we read the 8-bit prefix of the message and if it is an all-zero, we iterate through all possible inner messages ("cell references", up to 4 items) and send them out one by one.

Other prefixes are used for other operations, such as setting up or removing plugins. Unknown operations are ignored by the wallets.

```
  cs~touch();
  int op = cs~load_uint(8);

  if (op == 0) { ;; simple send
    while (cs.slice_refs()) {
      var mode = cs~load_uint(8);
      send_raw_message(cs~load_ref(), mode);
    }
    return (); ;; have already saved the storage
  }
  ...
}
```

