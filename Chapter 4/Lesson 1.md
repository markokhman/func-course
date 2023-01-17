# Chapter 4. Lesson 1.

In this chapter we are going to solely focus on writing FunC code, understanding it and writing local tests with help of TypeScript.

Let's get back to the code of contract we've programmed in the previous chapter. We are going to continue building on top of this contract.

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
	slice cs = in_msg.begin_parse();
	int flags = cs~load_uint(4);
	slice sender_address = cs~load_msg_addr();

	set_data(begin_cell().store_slice(sender_address).end_cell());
}

slice get_the_latest_sender() method_id {
   slice ds = get_data().begin_parse();
   return ds~load_msg_addr();
}
```

Let me elaborate a bit more on our plan during this lesson. We are going to write a contract, that will have following logic:

1. It will receive a message with a specific command (commands are usd to identify which part of the functionality of our contract should be executed against the data received in message)
2. Based on this specific command, it increases the number that is stored in our c4 storage.
3. It provides a getter method that is returning current counter value and the address that has sent the message with increment op code.

---

We are already familiar with concepts of writing to data storage and getter methods, so this will not be of a bit problem, rather a good practice for our skills.

However, the concept of commands is new. Commands are also called operation codes, or shortly - op codes. Op codes are usually stored in the very beginning of the in_msg_body slice that is passed into the recv_internal function. Op code is simply an integer, that we use to indicate certain logic block. This is a standard, but op codes don't appear there by themseves - you have to pass them once you are composing a message before sending it.

Important to notice, that when we read the data, we have to do it in the same order as we've written it there. To make sure the **get_data()** works properly on the first run - we are going to get back to the concept of initial data, so we are storing a certain value in the c4 storage from the very moment of contract deploy.

### Op code

As I've mentioned before, we are going to put the op code in the very beginning of the **in_msg_body**, so let's write code that will read it from there:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
	slice cs = in_msg.begin_parse();
	int flags = cs~load_uint(4);
	slice sender_address = cs~load_msg_addr();

	int op = in_msg_body~load_uint(32);

	set_data(begin_cell().store_slice(sender_address).end_cell());
}

slice get_the_latest_sender() method_id {
   slice ds = get_data().begin_parse();
   return ds~load_msg_addr();
}
```

In FunC, for some variables we have to specify the amount of bits that are allocated in the memory to store variables. This is why we have to set 32 in **~load_uint(32)** method, it means that this integer will be maximum of 32 bit size.

The op code we are going to use for incrementing the counter value is going to be 1. As simple as that. Let's wrap our logic in to an if statement, that checks the op code is equal to 1:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
	slice cs = in_msg.begin_parse();
	int flags = cs~load_uint(4);
	slice sender_address = cs~load_msg_addr();

	int op = in_msg_body~load_uint(32);

	if (op == 1) {
	    ;; counter logic is coming here
	    set_data(begin_cell().store_slice(sender_address).end_cell());
	}
}

slice get_the_latest_sener() method_id {
   slice ds = get_data().begin_parse();
   return ds~load_msg_addr();
}
```

> In FunC we use double semicolon **;;** to start a comment line

### Counter logic

Our local storage structure is going to be as following:

- counter_value - integer
- sender_address - slice, the address of the wallet that sent increment command most recently

In our **recv_internal** function we suppose that the local storage already has this data stored, so we just read it. We don't need to read the previous most recent sender, because when we will save values back to storage - we are already going to use the new value for this - the sender_address of current message.

Once we get the counter value, we want to increment it. The last step would be simply saving the data in the same order that we read it:

```
slice ds = get_data().begin_parse();
int counter_value = ds~load_uint(32);
set_data(
  begin_cell().store_uint(counter_value + 1, 32).store_slice(sender_address).end_cell()
);
```

But now we also need to update our getter method, so it also reads the storage data in a proper order:

```
(int, slice) get_the_latest_sender() method_id {
  slice ds = get_data().begin_parse();
  return (
    ds~load_uint(32), ;; counter_value
    ds~load_msg_addr() ;; the most recent sender
  );
}
```

You can notice, that now, as we return more then one value, we are also wrapping the expected result in function definition into round braces, as well as return result.

Let's see how our final code looks like:

```
#include "imports/stdlib.fc";

() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
  slice cs = in_msg.begin_parse();
  int flags = cs~load_uint(4);
  slice sender_address = cs~load_msg_addr();

  int op = in_msg_body~load_uint(32);

  if (op == 1) {
    slice ds = get_data().begin_parse();
    int counter_value = ds~load_uint(32);
    set_data(
      begin_cell().store_uint(counter_value + 1, 32).store_slice(sender_address).end_cell()
    );
    return ();
  }

  return ();
}

(int, slice) get_the_latest_sender() method_id {
  slice ds = get_data().begin_parse();
  return (
    ds~load_uint(32),
    ds~load_msg_addr()
  );
}
```

You can simply check that our contract compiles properly by running `yarn compile`.

In the next lesson we are going to update our tests for the smartcontract, so it would be helping us in validating contract's behaviour.
