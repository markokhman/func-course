# Handling External Messages
## Intro
TON is an innovative blockchain that incorporates novel concepts in smart contract design.
However, the development of smart contracts still requires accuracy. In addition to the traditional aspects of security and attacks, new ones are added that are unique to TON. Let's discuss them.

In this shapter we will talk about the external messages.

## Smart Contract entry points
As you probably already know, TON smart contracts have three possible entry points:
* Every contract must have a function `recv_internal` function.
* Optional `recv_external` is for inbound external messages.
* `run_ticktock` is called in ticktock transactions of special smart contracts. Not used by regular contracts.

Here we will talk about `recv_external`.
The concept is familiar: the user makes a transaction off-chain, usually, signs it with their private key and sends it to the contract.

## Naive wallet implementation
A wallet in TON is just a smart contract. 
Let’s write a naive wallet, it only forwards the messages signed with our private key.
The basic implementation looks like this:
```
;; the only function that can get messages not from other smart contracts
() recv_external(slice in_msg) impure {     
    var signature = in_msg~load_bits(512);  ;; read the signature from the incoming message
    var cs = in_msg;                        ;; make a copy of the incoming message
    var ds = get_data().begin_parse();      ;; reading the contract storage
    var public_key = ds~load_uint(256);     ;; read the authorized key (contract owner)
    throw_unless(34, check_signature(slice_hash(in_msg), signature, public_key));
    accept_message();                       ;; we are ready to pay for gas to process this
    var mode = cs~load_uint(8);             ;; read the mode from incoming message
    send_raw_message(cs~load_ref(), mode);  ;; send the provided message
}
```

Indeed this contract will process the messages signed by the contract owner. But nothing prevents malicious parties from reading the payload and retransmitting. That is known as Replay attack.

I repeat. The attacker gets the message signed by the contract owner and sends it again to this wallet. It will be executed again.

## Replay Attack
Wallet owner now needs to supply a **timestamp** until which message is valid.
It still can be replayed during that time though.

```
() recv_external(slice in_msg) impure {
    var signature = in_msg~load_bits(512);
    var cs = in_msg; 
    var valid_until = cs~load_uint(32);         ;; added
    throw_if(35, valid_until <= now());         ;; added
    var ds = get_data().begin_parse();
    var public_key = ds~load_uint(256);
    throw_unless(34, check_signature(slice_hash(in_msg), signature, public_key));
    accept_message();
    var mode = cs~load_uint(8);
    send_raw_message(cs~load_ref(), mode);
}
```

## Replay attack
Wallet owner now needs to supply a **sequence number** that’s incremented over the stored one. This ensures any message can be processed only once:
```
() recv_external(slice in_msg) impure {
    var signature = in_msg~load_bits(512);
    var cs = in_msg; 
    var (msg_seqno, valid_until) = (cs~load_uint(32), cs~load_uint(32));            ;; changed
    throw_if(35, valid_until <= now());
    var ds = get_data().begin_parse(); 
    var (stored_seqno, public_key) = (ds~load_uint(32), ds~load_uint(256));         ;; changed
    throw_unless(33, msg_seqno == stored_seqno);                                    ;; added
    throw_unless(34, check_signature(slice_hash(in_msg), signature, public_key));
    accept_message();
    var mode = cs~load_uint(8);
    send_raw_message(cs~load_ref(), mode);

    set_data(begin_cell()                                   ;; added
        .store_uint(stored_seqno + 1, 32)
        .store_uint(public_key, 256)
        .end_cell());                                   
}
```

## Denial of Service Attack
`recv_external` should be used carefully and `accept_message()` only after verification.
Otherwise, gas could deplete the contract funds with gas-free malicious executions

```
() recv_external(slice in_msg) impure {
    var signature = in_msg~load_bits(512);
    var cs = in_msg; 
    var (msg_seqno, valid_until) = (cs~load_uint(32), cs~load_uint(32));    
    throw_if(35, valid_until <= now());
    var ds = get_data().begin_parse(); 
    var (stored_seqno, public_key) = (ds~load_uint(32), ds~load_uint(256)); 
    throw_unless(33, msg_seqno == stored_seqno);                                    
    throw_unless(34, check_signature(slice_hash(in_msg), signature, public_key));

    accept_message();                           ;; accept after checking signature only

    var mode = cs~load_uint(8);
    send_raw_message(cs~load_ref(), mode);
    set_data(begin_cell()
        .store_uint(stored_seqno + 1, 32)
        .store_uint(public_key, 256)
        .end_cell());                                   
}
```
Also notice that the same signed message can be replayed on another wallet of the same owner. Or message in testnet can be replayed at mainnet.

## Failed Replay Attack

Note that if after `accept_message()` some error is thrown, transaction will be written to the blockchain and fees will be deducted from the contract balance, but storage will not be updated and actions will not be applied as in any transaction with an error exit code. 
As a result, if the contract accepts an external message and then throws an exception due to an error in the message data or sending an incorrectly serialized message, it will pay for processing but have **no way of preventing message replay**. The same message will be accepted by the contract over and over until it consumes the entire balance.

That's why we will save the updated state before the message parsing and execution:
```
() recv_external(slice in_msg) impure {
    var signature = in_msg~load_bits(512);
    var cs = in_msg; 
    var (msg_seqno, valid_until) = (cs~load_uint(32), cs~load_uint(32));    
    throw_if(35, valid_until <= now());
    var ds = get_data().begin_parse(); 
    var (stored_seqno, public_key) = (ds~load_uint(32), ds~load_uint(256)); 
    throw_unless(33, msg_seqno == stored_seqno);                                    
    throw_unless(34, check_signature(slice_hash(in_msg), signature, public_key));
    accept_message();                           

    set_data(begin_cell()                       ;; moved
        .store_uint(stored_seqno + 1, 32)
        .store_uint(public_key, 256)
        .end_cell());
    commit();                                   ;; added

    var mode = cs~load_uint(8);                 ;; can fail
    send_raw_message(cs~load_ref(), mode);      ;; can fail
}
```

## EVM vs TON

In Ethereum Virtual Machine (EVM) replay protection goes out-of-the-box:
* The private key has only one associated `address`
* `nonce` is automatically increased after each processed transaction
* Wallet owner pays for gas and failed transaction can't be replayed

TON is more flexible:
* One private key can control many wallets
* Wallet can have any logic coded
* External transaction is **credited** with 10k gas. If accepted, it will [spend](https://ton.org/docs/develop/smart-contracts/guidelines/accept#external-messages) contract balance.

## Recommendation
Luckily there is a simple recommendation for most of the contract developers: **avoid processing external messages**.

* Let the "wallet" contract do this job, have only `recv_internal` in your contract.
* Even if you are writing a "multisig" or special wallet, you can still process only internal messages from regular wallets.
* You will get EVM-style transactions in this case with all the flexibility under the hood.