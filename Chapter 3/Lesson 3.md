# Chapter 3. Lesson 3.
###### tags: `Chapter 3`

In the previous lesson we've created a compile script and now it's time to start writing FunC code. You know what is cool? We are now equiped to see if our FunC code actually works. Once we write some FunC code - we will simply run **yarn compile** and make sure our code is able to run on TVM.

The smartcontract in this lesson will be very very simple, but this will be enough for use to get familiar with basic FunC sintax and structure.

### Parameters we get with internal message handler recv_internal

Let's get our hands on code. In the previous lesson we've already created a **contracts/main.fc** file. Let's open it and see what we have there:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {

}
```

What we already have is a function that is handling incoming messages. As you already know from the Chapter 1, any transactions on TON are called **messages**.

So, what is the message bringing us? What we can already see are three parameters that are passed into the **recv_internal** function:

- **msg_value** - this parameter is telling us how many TON coin (or grams) are received with this message
- **in_msg** - this is a complete message that we've received, with all the information about who sent it etc. We can see that it has type Cell. What does it mean? The message body is stored as a Cell on the TVM, so there is one whole Cell dedicated to our message with all of it's data.
- **in_msg_body** - this is an actual "readable" part of the message that we received. It has type of slice, because it is part of the Cell, it indicates the "address" from which part of the cell we should start reading if we want to read this slice parameter.

TODO: Elaborate on the contents of the **in_msg** cell.

As you can see, both **msg_value** and **in_msg_body** are derivable from the **in_msg**, but for usability we receive them as parameters into the **receive_internal** function.

### Function specifiers

I'm sure you've noticed a word **impure** right after the params passed into the function. This is one of 3 possible function specifiers:

- impure
- inline/inline_ref
- method_id

One, several, or none of them can be put in a function declaration but currently they must be presented in the right order. For example, it is not allowed to put **impure** after **inline**.

At the moment, we are only interested in the **impure** specifier, but we are going to cover the rest as long as they start appearing in our code.

**impure** specifier means that the function can have some side effects which can't be ignored. For example, we should put impure specifier if the function can modify contract storage, send messages, or throw an exception when some data is invalid and the function is intended to validate this data.

If impure is not specified and the result of the function call is not used, then the FunC compiler may and will delete this function call.

### Importing stdlib.fc

In order to manipulate data and write other logic in our contract, we need to do one more important thing. We need to import FunC standard library. Currently, this library is just a wrapper for the most common assembler of the TVM commands which are not built-in. Each TVM command description used in the library can be found in the [documentation](https://ton.org/docs/develop/func/stdlib).

In order to import stdlib.fc, let's create a folder **imports** inside of our **contracts** folder. Then, create a file **stdlib.fc** and fill it with the contents of the official stdlib.fc library, that you can get [here](https://github.com/ton-blockchain/ton/blob/master/crypto/smartcont/stdlib.fc).

Now, in the very beginning of our main.fc we need to insert the actual import:

```
#include "imports/stdlib.fc";

() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {

}
```

Great, now we are good to go!

### Parsing in_msg

Let's finally learn what we can do with the parameters passed to our **recv_internal** function.

Whenever we want to handle an internal message, before we even start reading the meaningfull **in_msg_body** part, we need to first understand what kind of internal message we've received. There can be different cases. For example, we could receive this message, because our contract previously has sent some message someone and the receiving party wasn't able to accept it, so it "bounced" back. Sometimes we don't want to process this kind of messages. We are going to talk about such scenarious later in this course.

Every message that we receive has flags in it. Flags are basically a 4-bit integer, where each bit ...
(TODO: Elaborate on structure of flags. int_msg_info$0 ihr_disabled:Bool bounce:Bool bounced:Bool).

Those flags will actually tell us valuable information about the message, such as "This message was bounced by the receiver and now it's actually returning back".

So, first thing that our contract is supposed to do when it receives an internal message - parse it. Let's parse the **in_msg**:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
	slice cs = in_msg.begin_parse();
	int flags = cs~load_uint(4);
}
```

Let's break down what exactly happens in this code:

You will need to remember this concept, that **slice** is an "address", a pointer. So when we parse - we parse starting from some place. In this case the **begin_parse()** is telling us from where we should start parsing, it gives us the pointer to a very first bit of the **in_msg** Cell.

Then we parse a 4 bit integer by calling **load_uint(4)** and assign the result to an **int** variable **flags**.

Once we are going to call some more **~load\_{\*}** on the **cs** variable, we will actually continue parsing from the place the previous **~load\_{\*}** finished.

In case we will try to parse something that doesn't actually exist in the cell - our contract will exit with code 9. You can read more about standard code errors [here](https://ton.org/docs/learn/tvm-instructions/tvm-exit-codes)

There is some more valuable information in the **in_msg** Cell, namely - the sender address, so let's continue to parse:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
	slice cs = in_msg.begin_parse();
	int flags = cs~load_uint(4);
	slice sender_address = cs~load_msg_addr();
}
```

When we want to create a variable that stores address, we always use **slice** type, so we only store the **pointer** from where the memory should read the address, once it's needed.

What are we going to do with variables that we already have? Let me first introduce you to another 2 capabilities of smartcontract:

1. Our smartcontract has a persistent storage (called c4 storage)
2. Our smartcontract is able to have a getter method, that would allow anybody from outside the world get some data from our contract.

Using those two new capabilities, we can do a following thing. We can store the sender address in our storage and create a getter method, that would return when it's called.

In other words, our getter would always return the address of a contract that sent a message to our contract the latest.

### Using persistent storage

In order to store same data in our c4 persistent storage, we are going to use FunC standard function **set_data**. This function is accepting and storing a Cell.

> If we want to store more data then fits into a Cell, we can easily write a "link" to other Cell inside of the first one. Such link is called **ref**. We can write up to 4 **refs** into a Cell.

Let's updated our code with a **set_data** function, and learn how we pass a Cell into it.

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
	slice cs = in_msg.begin_parse();
	int flags = cs~load_uint(4);
	slice sender_address = cs~load_msg_addr();

	set_data(begin_cell().end_cell());
}
```

To pass a Cell into a **set_data** or we need to first construct it. This is done easily with two functions **begin_cell()** and **end_cell()**.

At the moment we are passing an empty Cell, that is techinically fine, but we want to write the address of message sender into storage, so we should update our cell with it:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {
	slice cs = in_msg.begin_parse();
	int flags = cs~load_uint(4);
	slice sender_address = cs~load_msg_addr();

	set_data(begin_cell().store_slice(sender_address).end_cell());
}
```

We use method **.store_slice()** for this purpose.

Voila, we have a smartcontract that is able to write a sender's address into the persistant storage! Every time our contract will receive an internal message, it will replace the Cell that is stored in c4 with a new cell that will have a new sender's address. As simple as that.

### Using getter methods

As you remember, we didn't want to just store the sender's address. We wanted anybody to be able to read the latest sender's address. To access such data from ouside of TVM, our contract need to have a special function.

We've been talking about function specifiers recently. To make our data accessible ouside the TVM, we are going to create a function and use specifier **method_id**. If function has this specifier set - then it can be called in lite-client or ton-explorer as a get-method by its name.

Let's create one:

```
slice get_the_latest_sender() method_id {

}
```

> Getter functions are placed ouside of the **recv_internal** function

As you can see, we define what time is supposed to be returned by that function, function's name and the **method_id** specifier.

Now let's write the logic of reading data from the persistent storage and returning it's value. For thise we are using FunC standard **get_data** function:

```
slice get_the_latest_sender() method_id {
   slice ds = get_data().begin_parse();
   return ds~load_msg_addr();
}
```

As you can see, we are using the **begin_parse()** again to get a pointer, from which we are going to parse the Cell stored in c4 storage.

To load the stored address we are using **~load_msg_addr** to load the address.

### Compiling our contract

Our final code looks like that:

```
\#include "imports/stdlib.fc";

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

Quite a simple contract, but we've learned a lot while code it, didn't we?

Since we have written all the planned code - let's run the lovely `yarn compile` in our terminal.

If you've done everything step-by-step with me, you're supposed to see a following outcome in the terminal:

```
=================================================================
Compile script is running, let's find some FunC code to compile...
 - Compilation successful!
 - Compiled code saved to build/main.compiled.json
✨  Done in 1.40s.
```

Let's check the **build/main.compiled.json**, and we will see that it's contents changed - the hex value is much longer this time :) This is because you've written your first FunC code! My congratulations!
