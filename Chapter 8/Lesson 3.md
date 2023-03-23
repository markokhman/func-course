# Gas Management
## Intro
* In Ethereum Virtual Machine if the user provides too little gas, everything will be reverted.
* If they provide enough, the actual costs will automatically be calculated and deducted from their balance.

In Solidity, gas is not much of a concern for contract developers. If the user provides too little gas, everything will be reverted as if nothing had happened (but the gas will not be returned). If they provide enough, the actual costs will automatically be calculated and deducted from their balance.

In TON, the situation is different:

* If there is not enough gas, the transaction will be partially executed.
* If there is too much gas, the excess must be returned. This is the developer’s responsibility.
* TON can't do everything itself because of asynchronous nature.

If a “group of contracts” exchanges messages, then control and calculation must be carried out in each message.
TON cannot automatically calculate the gas. The complete execution of the transaction with all its consequences can take a long time, and by the end, the user may not have enough toncoins in their wallet. The carry-value principle is used here.

## Calculate Gas and Check msg_value
We can use our message flow diagram to estimate the cost of each handler in each of the scenarios and insert a check for the sufficiency of msg_value.

* Find "entry points" on message flow.
* Estimate the cost of each handler.
* Check in "entry points" that the `msg_value` is enough.

* You can't demand enough with a margin everywhere (say 1 TON). The gas is divided among the "consequences".

Let's say your contract sends three messages, then you can only send 0.33 TON to each. This means that they should “demand” less. It’s important to calculate the gas requirements of your whole contract carefully.

Things get more complicated if, during development, your code starts sending more messages. Gas requirements need to be rechecked and updated.

## Return Gas Excesses Carefully
* If excess gas is not returned to the sender, the funds will accumulate in your contracts over time. 
* Nothing terrible, but suboptimal. You can add a function for raking out excesses.
* Popular contracts like TON Jetton return to the sender with the message `op::excesses`.

If excess gas is not returned to the sender, the funds will accumulate in your contracts over time. In principle, nothing terrible, this is just suboptimal practice. You can add a function for raking out excesses, but popular contracts like TON Jetton still return to the sender with the message op::excesses.

* `send_raw_message(msg, SEND_MODE_CARRY_ALL_REMAINING_MESSAGE_VALUE)` pass the rest of gas.
* Useful if message flow is linear: each handler sends only one message.

TON has a useful mechanism: SEND_MODE_CARRY_ALL_REMAINING_MESSAGE_VALUE = 64. When using this mode in send_raw_message(), the rest of the gas will be forwarded further with the message (or back) to the new recipient. It is convenient if the message flow is linear: each message handler sends only one message. But there are cases when it is not recommended to use this mechanism:

## When not recommended to carry all the remaining gas

* `storage_fee` is deducted from the balance of the contract, not from the incoming gas.

`storage_fee` is deducted from the balance of the contract, and not from the incoming gas. So if there are no other non-linear handlers in your contract, then over time, `storage_fee` can eat up the entire balance because everything that comes in has to go out.

* Emitting event eats contract balance, not gas.

If your contract emits events, i.e. sends a message to an external address. The cost of this action is deducted from the balance of the contract, and not from msg_value.

* Attaching value to the message or using `SEND_MODE_PAY_FEES_SEPARETELY` eats contract balance.
```cpp
    var msg = begin_cell().store_uint(0x18, 6).store_slice(destination)
      .store_coins(10000000)                ;; This will be deducted from contract balance
      .store_uint(1, 1 + 4 + 4 + 64 + 32 + 1 + 1).store_ref(msg_body);
    send_raw_message(msg.end_cell(), 0);

    emit_log_cell_ref(query_id, ...         ;; This also spends contract balance
```

If your contract attaches value when sending messages or uses SEND_MODE_PAY_FEES_SEPARETELY = 1. These actions deduct from the balance of the contract, which means that returning unused is "working at a loss."

## Common way to calculate gas cost
Here is the code snippet from TON Wallet. This is a common way to keep the contract balance stable.

```cpp
int ton_balance_before_msg = my_ton_balance - msg_value;
int storage_fee = const::min_tons_for_storage - min(ton_balance_before_msg, const::min_tons_for_storage);
msg_value -= storage_fee + const::gas_consumption;

if(forward_ton_amount) {
    msg_value -= (forward_ton_amount + fwd_fee);
    ...
}

if (msg_value > 0) {    ;; there is still something to return
    var msg = begin_cell()
        .store_uint(0x10, 6)
        .store_slice(response_address)
        .store_coins(msg_value)
    ...
```

Remember, if the value of the contract balance runs out, **the transaction will be partially executed**, and this cannot be allowed.

## Resume

* TON requires deliberate efforts from developers to manage the gas: calculate the spending and checking if enough provided.
* Gas cost can rise over time if "unbounded data structure" used.
* TON recommends returning the excesses to the sender - it is also on developer.
* If run out of gas, the transaction will be partially executed. That can be a critical issue.