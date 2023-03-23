# Storage Management
## Intro
* In Ethereum Virtual Machine contracts you can access and modify each storage element individually.
* In TON the storage is accessed via `get_data()/set_data()` (c4 register holds the ref to the bag of cells).
* This requires the developer to "manage" the storage manually

## Typical message handler
A typical message handler in TON follows this approach:

```cpp
() handle_something(...) impure {
    (int total_supply, <a lot of vars>) = load_data();

    ... ;; do something, change data

    save_data(total_supply, <a lot of vars>);
}
```

Unfortunately, we are noticing a trend: \<a lot of vars\> is a real enumeration of all contract data fields. For example,

```cpp
(int total_supply, int swap_fee, int min_amount, int is_stopped, int user_count, int max_user_count, 
slice admin_address, slice router_address, slice jettonA_address, slice jettonA_wallet_address, 
int jettonA_balance, int jettonA_pending_balance, slice jettonB_address, slice jettonB_wallet_address, 
int jettonB_balance, int jettonB_pending_balance, int mining_amount, int datetime_amount, int minable_time, 
int half_life, int last_index, int last_mined, cell mining_rate_cell, cell user_info_dict, cell operation_gas, 
cell content, cell lp_wallet_code) = load_data();
```

This approach has a number of disadvantages.

## Problem 1: Hard to update the storage structure
* Adding one more field requires updating the whole contract.

First, if you decide to add another field, say `is_paused`, then you need to update the `load_data()/save_data()` statements throughout the contract. And this is not only labor-intensive, but it also leads to hard-to-catch errors.

In a recent CertiK audit, we noticed the developer mixed up two arguments in places, and wrote:
```cpp
    save_data(total_supply, min_amount, swap_fee, ...
instead of
    save_data(total_supply, swap_fee, min_amount, ...
```

Without an external audit performed by a team of experts, finding such a bug is very difficult. The function with a bug was rarely used, and both confused parameters usually had a value of zero. You really have to know what you’re looking for to pick up on an error like this.

## Problem 2: Namespace pollution
* Reading all the storage fields to the current namespace pollutes it.

Secondly, there is "namespace pollution". Let's explain what the problem is with another example from an audit. In the middle of the function, the input parameter read:

```cpp
(int total_supply, int swap_fee, int min_amount, <a lot of vars>) = load_data();
...
int min_amount = in_msg_body~load_coins();
...
save_data(total_supply, swap_fee, min_amount, <a lot of vars>);
```

That is, there is a shadowing of the storage field by a local variable, and at the end of the function, this replaced value is stored in storage. The attacker had the opportunity to overwrite the state of the contract. 

* FunC allows redeclaring the variables.

## Problem 3: Gas cost increased
And finally, 
* Parsing the entire storage and packing it back on every call to every function increases the gas cost.

## Solution 1: Use global variables
At the prototyping stage, where it is not entirely obvious what will be stored in the contract, global variables can be used. 
```cpp
global int var1;
global cell var2;
global slice var3;

() load_data() impure {
    var cs = get_data().begin_parse();
    var1 = cs~load_coins();
    var2 = cs~load_ref();
    var3 = cs~load_bits(512);
}

() save_data() impure {
    set_data(
        begin_cell()
            .store_coins(var1)
            .store_ref(var2)
            .store_bits(var3)
            .end_cell()
        );
}
```

That way, if you find out that you need another variable, you just add a new global variable and modify load_data() and save_data(). No changes throughout the contract are needed. However,

* No more than 31 global variables possible.
* Globals are more expensive than storing on the stack.

## Solution 2: Use "nested" storage
After prototyping we recommend following this storage organization approach:
```cpp
() handle_something(...) impure {
    (slice swap_data, cell liquidity_data, cell mining_data, cell discovery_data) = load_data();
    (int total_supply, int swap_fee, int min_amount, int is_stopped) = swap_data.parse_swap_data();
    …
    swap_data = pack_swap_data(total_supply + lp_amount, swap_fee, min_amount, is_stopped);
    save_data(swap_data, liquidity_data, mining_data, discovery_data);
}
```
* If variable used often (like `is_paused`), it is provided immediately by `load_data()`.
* If paramater group is needed only in one scenario, don't unpack it.

Storage consists of blocks of related data. If a parameter is used in each function, for example, `is_paused`, then it is provided immediately by `load_data()`. 
If a parameter group is needed only in one scenario, then it does not need to be unpacked, it will not have to be packed, and it will not clog the namespace.

* If new variable added, less code pieces have to be updated.

If the storage structure requires changes (usually adding a new field), then much fewer edits will have to be made.

* Nested variables can be sub-nested.

Moreover, the approach can be repeated. If there are 30 storage fields in our contract, then initially you can get four groups, and then get a couple of variables and another subgroup from the first group. The main thing is not to overdo it.

* Cell can store up to 1023 bits and up to 4 refs. You will split anyway.

Note that since a cell can store up to 1023 bits of data and up to 4 references, you will have to split the data into different cells anyway.

Hierarchical data is one of the main features of TON, let's use it for its intended purpose.

## Use `end_parse()`
* Use `end_parse()` wherever possible when reading data from storage and from the message payload. 

Since TON uses bit streams with variable data format, it’s helpful to ensure that you read as much as you write. This can save you an hour of debugging.

## Use helper functions and avoid magic numbers
This code is from real project. It can scary even experienced developer due to a lot of magic numbers

```cpp
var msg = begin_cell()
    .store_uint(0xc4ff, 17)         ;; 0 11000100 0xff
    .store_uint(config_addr, 256)
    .store_grams(1 << 30)           ;; ~1 gram of value
    .store_uint(0, 107)
    .store_uint(0x4e565354, 32)
    .store_uint(query_id, 64)
    .store_ref(vset);
    
send_raw_message(msg.end_cell(), 1);
```

Introduce as many constants and wrappers as needed to have an expressive code

```cpp
    var msg = begin_cell()
        .store_msg_flags(BOUNCEABLE)
        .store_slice(to_wallet_address)
        .store_coins(amount)
        .store_msgbody_prefix_stateinit()
        .store_ref(state_init)
        .store_ref(master_msg);

    send_raw_message(msg.end_cell(), SEND_MODE_PAY_FEES_SEPARETELY);
```

## Bug example
Don't forget about all the traditional pitfalls and potential bugs not related to TON specifically. Here is an example from a real project.

```cpp
() handle_transfer(...) impure {
  ...
  (slice from_user_info, int from_flag) = user_info_dict.udict_get?(256, from_addr_hash);
  int from_balance = from_user_info~load_coins();
  ...
  (slice to_user_info, int to_flag) = user_info_dict.udict_get?(256, to_addr_hash);
  int to_balance = to_user_info~load_coins();
  ...
  ;; save decreased from_balance to user_info_dict
  ;; save increased to_balance to user_info_dict
}
```
Transferring money to the same address practically doubles the balance, since `to_balance` overwrites zeroed `from_balance`.

## Resume

* Use "nested" storage organization and global variables to keep storage operation robust.
* Write wrappers and declare constants to keep the code expressive.
* Test your code.
* Perform a thorough audit to avoid losses.