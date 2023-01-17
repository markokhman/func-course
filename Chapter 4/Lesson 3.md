# Chapter 4. Lesson 3.

In this lesson we are going to practice implementing more different commands, creating functions and even sending messages from inside the contract. We are going to do all of this on top of the existing contract we already have.

What's our plan? Let's break it down:

- our contract will become more strict on commands, we are going to introduce a new op code meant for logic of depositing funds to the contract
- another op code we are going to introduce is withdrawal of funds. During the withdrawal process, the funds are actually sent to the address as a message, so we will learn how to send messages from within the contract
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

(int, slice, slice) get_the_latest_sender() method_id {
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
- if the op code read from the message body did not trigger any of the **if** statements - we are throwing an error with code 777. This number can be choosed by you, but make sure it doesn't intersect with the [official error exit codes](https://ton.org/docs/learn/tvm-instructions/tvm-exit-codes)
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

- to compare the owner's and the sender's addresses we use FunC standar function **equal_slices()**
- we are using **throw_unless()** function to throw an error if the result of comparison was **false**. There is also another way to through an error - **throw_if()**, this one is throwing error if the condition passed into this function is returning true.
- the message body with this op would require to also have an integer stating the amount, requested to withdraw. We compare this amount to the actual balance of contract (standard FunC function **get_balance()**)
- we want our contract to always have some funds, to be able to pay for rent and necessary fess (learn more about fees in Chapter 1), so we would need to set some minimum that would have to stay on the contract and throw an error, if the requested amount doesn't allow that.
- finally, we need to learn how we can send coins from inside the contract

Let's see how this all works together.

First we set the constant for the minimum required storage:

```
const const::min_tons_for_storage = 10000000; ;; 0.01 TON
```

Then, we implement the logic

```
if (op == 3) {
    throw_unless(103, equal_slices(sender_address, owner_address));

    int withdraw_amount = in_msg_body~load_coins();
    var [balance, _] = get_balance();
    throw_unless(104, balance >= withdraw_amount);

    int return_value = min(withdraw_amount, balance - const::min_tons_for_storage);

    cell msg = begin_cell()
      .store_uint (0x18, 6)
      .store_slice(sender_address)
      .store_grams(return_value)
      .store_uint(0, 107)
      .end_cell();
    send_raw_message(msg, 3);
    return();
}
```

Most of the lines should be clear for you, since we've discussed every standard function that is used there.

Let's talk more about this logic that sends actual funds.

**send_raw_message** is a standard function that accepts a cell with message and an integer, that is supposed to define the mode in which the message is supposed to be sent.

The message cell, that we compose prior to passing it into the **send_raw_message()**, consists of 4 values:

- flags
- the 267 bit destination address
- funds amount to send
- body message (in our case we use 106 zeroes + 0 as an indicator that there is no cell with the data)

> TODO: Elaborate on other possible values

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

    cell msg = begin_cell()
      .store_uint (0x18, 6) ;; bounce
      .store_slice(sender_address) ;; 267 bit address
      .store_grams(return_value)
      .store_uint(0, 107) ;; 106 zeroes +  0 as an indicator that there is no cell with the data
      .end_cell();
    send_raw_message(msg, 3); ;; mode, 2 for ignoring errors, 1 for sender pays fees, 64 for returning inbound message value
    return();
  }

  throw(777);
}

(int, slice, slice) get_the_latest_sender() method_id {
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
