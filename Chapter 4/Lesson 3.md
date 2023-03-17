# Chapter 4. Lesson 3.
###### tags: `Chapter 4`

In this lesson we are going to practice implementing more different commands, creating functions and even sending messages from inside the contract. We are going to do all of this on top of the existing contract we already have.

What's our plan? Let's break it down:
- our contract will become more strict on commands, we are going to introduce a new op code meant for logic of depositing funds to the contract
- another op code we are going to introduce is withdrawal of funds. During the withdrawal process, the funds are actually sent to the address as a message, so we will learn how to send messages from within the contract
- any funds that will arrive with unkonwn operation codes will be returned to the sender
- we will introduce a new value in our storage - owner of the contract. Only owner of the contract is able to withdraw funds. We will also separate the storage load and write logic into functions, to keep our main code clean


---
Ready? Set. Go!

### Separating storage management logic

First of all, let's deal with the contract owner's data in storage (we need to keep in mind, that we would have to place this address into storage on initiating the contract). We are going to also create two new functions - **load_data** and **save_data**:

```
(int, slice, slice) load_data() inline {
  var ds = get_data().begin_parse();
  return (
  	ds~load_uint(32), ;; counter_value
   	ds~load_msg_addr(), ;; the most recent sender
    	ds~load_msg_addr() ;; owner_address
  );
}

() save_data(int counter_value, slice recent_sender, slice owner_address) impure inline {
  set_data(begin_cell()
    .store_uint(counter_value, 32) ;; counter_value
    .store_slice(recent_sender) ;; the most recent sender
    .store_slice(owner_address) ;; owner_address
    .end_cell());
}

```

There is a few things to note here:
- **inline** specifier. You already know a bit about specifiers. If a function has inline specifier, its code is actually substituted in every place where the function is called. It is forbidden to use recursive calls in inlined functions.
- why **load_data()** has no **impure** function specifier? We've already covered that in Chapter 3, but let's recap. The answer is, because this function doesn't affect the state of the contract. It is read only.

This is how we are going to use those functions further in our code:
```
var (counter_value, recent_sender, owner_address) = load_data();

save_data(counter_value,recent_sender,owner_address);
```

> We don't always have to read all the parameters, but we must ensure that we are writing all of them back, because the cell is full renewed once we write data.


### New op codes

Let's see how our storage management functions are placed now in code and also introduce more op codes and also throw an error if the op code is unknown to our contract:
```
#include "imports/stdlib.fc";

(int, slice, slice) load_data() inline {
  var ds = get_data().begin_parse();
  return (
    ds~load_uint(32), ;; counter_value
    ds~load_msg_addr(), ;; the most recent sender
    ds~load_msg_addr() ;; owner_address
  );
}

() save_data(int counter_value, slice recent_sender, slice owner_address) impure inline {
  set_data(begin_cell()
    .store_uint(counter_value, 32) ;; counter_value
    .store_slice(recent_sender) ;; the most recent sender
    .store_slice(owner_address) ;; owner_address
    .end_cell());
}

() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
  slice cs = in_msg.begin_parse();
  int flags = cs~load_uint(4);
  slice sender_address = cs~load_msg_addr();

  int op = in_msg_body~load_uint(32);

  var (counter_value, recent_sender, owner_address) = load_data();
	
  if (op == 1) {
    save_data(counter_value + 1, sender_address, owner_address);
	return();
  }

  if (op == 2) {
    ;; deposit
  }

  if (op == 3) {
    ;; withdrawal
  }

  throw(777);
}

(int, slice, slice) get_contract_storage_data() method_id {
  var (counter_value, recent_sender, owner_address) = load_data();
  return (
    counter_value,
    recent_sender,
    owner_address
  );
}
```
A few things to note:
- we also updated the **get_the_latest_sender** function to return the owner address
- if the op code read from the message body did not trigger any of the **if** statements - we are throwing an error with code 777. This number can be choosed by you, but make sure it doesn't intersect with the [official error exit codes](https://ton.org/docs/learn/tvm-instructions/tvm-exit-codes). `exit_code` higher than 1, it is considered to be an error code, therefore an exit with such a code may cause the transaction to revert/bounce.
- we use function **return()** to successfully exit the contract execution

Overall, the code looks clean, doesn't it?

### Deposit and withdrawal

It's very easy for us on deposit (op == 2), as in this case we are simply finishing the execution with success, so the funds are accepted. Otherwise - funds will be return to sender. 
```
if (op == 2) {
    return();
}
```

However, it's getting a bit more complicated with the withdrawal. We have to compare the sender address to the owner address of smartcontract. Let's see what are some things that we need to know in order to implement this logc: 
- to compare the owner's and the sender's addresses we use FunC standar function **equal_slice_bits()**
- we are using **throw_unless()** function to throw an error if the result of comparison was **false**. There is also another way to through an error - **throw_if()**, this one is throwing error if the condition passed into this function is returning true.
- the message body with this op would require to also have an integer stating the amount, requested to withdraw. We compare this amount to the actual balance of contract (standard FunC function **get_balance()**) 
- we want our contract to always have some funds, to be able to pay for rent and necessary fess (learn more about fees in Chapter 1), so we would need to set some minimum that would have to stay on the contract and throw an error, if the requested amount doesn't allow that.
- finally, we need to learn how we can send coins from inside the contract via an internal message

Let's see how this all works together.

First we set the constant for the minimum required storage:
```
const const::min_tons_for_storage = 10000000; ;; 0.01 TON
```

Then, we implement the logic of a withdrawal. It will look like this:
```
if (op == 3) {
    throw_unless(103, equal_slices(sender_address, owner_address));

    int withdraw_amount = in_msg_body~load_coins();
    var [balance, _] = get_balance();
    throw_unless(104, balance >= withdraw_amount);

    int return_value = min(withdraw_amount, balance - const::min_tons_for_storage);

    
    ;; TODO: Sending internal message with funds
    
    return();
}
```

As you can see we are reading the amount of coins that are requested or withdrawal (it will be stored in our `in_msg_body` right after the op code, we will do this in next lesson) checking the balance is bigger or equal then the requested withdrawal amount.

We also use a good technique of making sure that the minimum amount for storage is kept on the contract.

Let's talk more about this logic that sends actual funds.

### Sending internal message

**send_raw_message** is a standard function that accepts a **cell with message** and an **integer** that is containing sum of `mode` and `flag`. There are currently 3 Modes and 3 Flags for messages. You can combine a single mode with several (maybe none) flags to get a required mode. Combination simply means getting sum of their values. A table with descriptions of Modes and Flags is given in [this documentation part](https://ton.org/docs/develop/func/stdlib/#send_raw_message). 

In our example we want to send a regular message and pay transfer fees separately, so we use the Mode 0 and Flag +1 to get `mode = 1`.

The message cell, that we compose prior to passing it into the **send_raw_message()** is probably the most advanced thing you will need to understand so far. 

We are going to spend a bit time to make sure this part becomes clear for you. But let me give you two advices before we start:
1. First of all - get used to how data is stored in cells. This is called **serialization**. At this moment you will need to get familiar and used to this, as you will be serializing data into a cell quite often as everything is stored in cells.
2. Second - get comfortable with [TON documentation](https://ton.org/docs). That's pretty much the only way to get around with all those serialization structures, as it is extremely hard to remember them. 
> One thing you might find very usefull is [old documentation portal](https://ton-blockchain.github.io/docs), as it has some topics elaborated in a very basic and nice way.

Let's break down the structure of the message cell that we are going to pass into the **send_raw_message**. As you already understood, we need to put a number of bits into a cell and there is certain logic on which bit is responsible for what.

The message cell starts with 1bit prefix 0, then there are three 1-bit flags, namely whether Instant Hypercube Routing disabled (currently always true), whether message should be bounced if there are errors during it's processing, whether message itself is result of bounce. Then source and destination addresss are serialized, followed by the value of the message and four integers related to message forwarding fees and time.

If a message is sent from the smart contract, some of those fields will be rewritten to the correct values. In particular, validator will rewrite `bounced`, `src`, `ihr_fee`, `fwd_fee`, `created_lt` and `created_at`. That means two things: first, another smart-contract during handling message may trust those fields (sender may not forge source address, bounced flag, etc); and second, that during serialization we may put to those fields any valid values (anyway those values will be overwritten).

Straight-forward serialization of the message would be as follows (taken from [documentation portal](https://ton.org/docs/develop/smart-contracts/messages#message-layout)):
```
  var msg = begin_cell()
    .store_uint(0, 1) ;; tag
    .store_uint(1, 1) ;; ihr_disabled
    .store_uint(1, 1) ;; allow bounces
    .store_uint(0, 1) ;; not bounced itself
    .store_slice(source) ;; ex. addr_none
    .store_slice(destination)
    ;; serialize CurrencyCollection (see below)
    .store_coins(amount)
    .store_dict(extra_currencies)
    .store_coins(0) ;; ihr_fee
    .store_coins(fwd_value) ;; fwd_fee 
    .store_uint(cur_lt(), 64) ;; lt of transaction
    .store_uint(now(), 32) ;; unixtime of transaction
    .store_uint(0,  1) ;; no init-field flag (Maybe)
    .store_uint(0,  1) ;; inplace message body flag (Either)
    .store_slice(msg_body)
  .end_cell();
```

However, instead of step-by-step serialization of all fields, usually developers use shortcuts. Thus, let's consider how messages can be sent from the smart contract using an example:
```
var msg = begin_cell()
    .store_uint(0x18, 6)
    .store_slice(addr)
    .store_coins(grams)
    .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)
    .store_uint(op_code, 32)
    .store_uint(query_id, 64);
    
send_raw_message(msg.end_cell(), mode);
```

Let's have a closer look on what is going on here. 

1. `.store_uint(0x18, 6)`

    We start composing a cell by putting `0x18` value into 6 bits that is `0b011000` if we translate it from hexadecimal. What is it? 

    `0b011000`

    - First bit is 0—1bit prefix which indicates that it is `int_msg_info` (internal message info).
    - Then there are 3 bits 1, 1 and 0, meaning:
        - Instant Hypercube Routing is disabled (we will not go into details of what it is, too low level stuff that you will not need while writing contracts at the moment)
        - messages can be bounced
        - message is not the result of bouncing itself.
    - Then there should be sender address, however since it anyway will be rewritten (when validator will put the actual address of sender here as we discussed above) with the same effect any valid address may be stored there. The shortest valid address serialization is that of addr_none and it serializes as a two-bit string 00.

    Thus, **.store_uint(0x18, 6)** is the optimized way of serializing the tag and the first 4 fields.

2. `.store_slice(addr)` - this line serializes the destination address.

3. `.store_coins(grams)` - this line simply serializes the coins amount (`grams` is just a veriable with an number of coins)

4. `.store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)` - this one is interesting. Technically we just write big a mount of zeros into the cell (namely the amount of zeros equals to 1 + 4 + 4 + 64 + 32 + 1 + 1). Why so? We know that there is a clear structure of how those values are consumed and that means every 0 we put there is for some reason.

    Interesting is the part that those zeros are working in two ways - some of them are put as 0 because the validator will revrite the value anyways, some of them are put as 0 because this feature is not supported yet (ex. extra currencies).
    
    Just to be sure we understand why there is so many zeros, let's break down it's intended structure:
    - First bit stands for empty extra-currencies dictionary.
    - Then we have two 4-bit long fields. Since `ihr_fee` and `fwd_fee` will be overwritten, we may as well put there zeroes.
    - Then we put zero to `created_lt` and `created_at` fields. Those fields will be overwritten as well; however, in contrast to fees, these fields have a fixed length and are thus encoded as 64- and 32-bit long strings.
    - Next zero-bit means that there is no init field.
    - The last zero-bit means that msg_body will be serialized in-place. **This basically indicates if there is `msg_body` coming with custom layout**.

5. `.store_uint(op_code, 32).store_uint(query_id, 64);` this part is the easiest one - we are passing a message body with a custom layout, meaning we can put any data there as long as the receiver knows how to handle it.


Let's see how this applies to our funds withdrawal code:
```
if (op == 3) {
    throw_unless(103, equal_slices(sender_address, owner_address));

    int withdraw_amount = in_msg_body~load_coins();
    var [balance, _] = get_balance();
    throw_unless(104, balance >= withdraw_amount);

    int return_value = min(withdraw_amount, balance - const::min_tons_for_storage);
    
    int msg_mode = 1; ;; 0 (Ordinary message) + 1 (Pay transfer fees separately from the message value)
    
    var msg = begin_cell()
        .store_uint(0x18, 6)
        .store_slice(sender_address)
        .store_coins(return_value)
        .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1);
    
    send_raw_message(msg.end_cell(), msg_mode);
    
    return();
}
```

### Finalizing our contract code

One last thing will be to add a new getter method that returns the balance of our contract:

```
int balance() method_id {
  var [balance, _] = get_balance();
  return balance;
}
```

Let's have a look one more time how our final code looks:
```
#include "imports/stdlib.fc";

const const::min_tons_for_storage = 10000000; ;; 0.01 TON

(int, slice, slice) load_data() inline {
  var ds = get_data().begin_parse();
  return (
    ds~load_uint(32), ;; counter_value
    ds~load_msg_addr(), ;; the most recent sender
    ds~load_msg_addr() ;; owner_address
  );
}

() save_data(int counter_value, slice recent_sender, slice owner_address) impure inline {
  set_data(begin_cell()
    .store_uint(counter_value, 32) ;; counter_value
    .store_slice(recent_sender) ;; the most recent sender
    .store_slice(owner_address) ;; owner_address
    .end_cell());
}

() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
  slice cs = in_msg.begin_parse();
  int flags = cs~load_uint(4);
  slice sender_address = cs~load_msg_addr();

  int op = in_msg_body~load_uint(32);

  var (counter_value, recent_sender, owner_address) = load_data();
	
  if (op == 1) {
    save_data(counter_value + 1, sender_address, owner_address);
    return();
  }

  if (op == 2) {
    return();
  }

  if (op == 3) {
        throw_unless(103, equal_slices(sender_address, owner_address));

        int withdraw_amount = in_msg_body~load_coins();
        var [balance, _] = get_balance();
        throw_unless(104, balance >= withdraw_amount);

        int return_value = min(withdraw_amount, balance - const::min_tons_for_storage);

        int msg_mode = 1; ;; 0 (Ordinary message) + 1 (Pay transfer fees separately from the message value)

        var msg = begin_cell()
            .store_uint(0x18, 6)
            .store_slice(addr)
            .store_coins(grams)
            .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1);

        send_raw_message(msg.end_cell(), msg_mode);

        return();
    }

  throw(777);
}

(int, slice, slice) get_contract_storage_data() method_id {
  var (counter_value, recent_sender, owner_address) = load_data();
  return (
    counter_value,
    recent_sender,
    owner_address
  );
}

int balance() method_id {
  var [balance, _] = get_balance();
  return balance;
}
```

Let's check that our code compiles with `yarn compile` and let's proceed to writing tests for our updated contract.
