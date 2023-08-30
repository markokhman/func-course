# Understanding transaction cost

###### tags: `Chapter 2`

*TON is a complicated system, and nothing could be even more complicated in TON than its model of transaction costs and fees. So let's take a deep dive :ocean: into what kind of fees exist in TON. <u>First of all, let's talk about why this is important.</u>*

## Importance of fees.

> :globe_with_meridians: *TON is a public network, and like any public network, it is subject to attacks by anyone who wishes to do it, so extra care should be taken of protecting this shared resource from denial of service attacks.*

:exclamation: This means that if there is any non-trivial cost that some participants in the network bear that could be amplified by the external actors, this creates a huge risk for the liveliness of the whole ecosystem. :warning:  That's why anything that has any noticeable cost and can be amplified must be accounted for explicitly in terms of fees.


###### *In TON there are generally three categories of fees:*
- *Gas cost.*
- *Rent.*
- *Message fees.*

## Gas fees.

> :computer: TON is a computation platform. Your contract may contain arbitrary code and anyone can upload into the network any code they want. Once they do that, then the whole network will process :hammer: messages :envelope: running this code. 

> <u> :red_car: The term gas comes from the gasoline, like a fuel for the execution, and it was originally invented in Ethereum. The idea is that for each operation in your code, there is a nominal gas cost that allows you to specify how some operations are more or less expensive than the others. And then there is a global parameter that specifies the gas price, that specifies how much all of these gas units cost at the present prices.</u>

> :exclamation: *TON is designed, unlike Ethereum or Bitcoin, in a way that doesn't produce the market for the scarce amount of gas or the block size. Instead, it shards infinitely so that if you pay one gas price today, and tomorrow you have a much higher load :small_red_triangle:, then the validators of the network will earn higher fees, but the users will not have to compete on fees and will pay the same price, just the network will scale out horizontally.*


## Consideration regarding gas.

<u>*You as a designer have to take care of a couple of considerations regarding gas.*</u>

 > ***You should take care of deciding who is going to pay for the gas costs, whether it's your contract or it's the sender of a message that sends the message to this contract.*** :weary:
 
 *The best strategy is to put all the costs on the sender, and design your contract in a way that the costs are more or less predictable to the sender, so they could attach enough coins to their message to cover the gas costs*


## Rent.

> <u>***Rent*** - cost of a single bit of data that the contract stores per unit of time. </u>
<u>
> :question: *You may notice if you use a wallet that if you haven't used a wallet for several days, then the first transaction that you send has a slightly higher fee than the next one.*

> :astonished: This is because your wallet was idle for a week or two and some noticeable amount of rent accumulated that was charged when you did this transaction.
</u>

## Message fees.

> :thought_balloon: *This comes to play in the action phase when your contract creates the outgoing messages and specifies new state for itself. And these are typically quite low fees because there's not much data transmitted between the contracts.*

## Considerations in the design of smart contracts

1. <u>Contracts **should not :runner: run out of gas and rent**.
2. **If you put the cost of the gas and execution on the sender, then you may not always be able to guarantee some specific cost of execution for various reasons**. But what you could guarantee is some kind of upper bound. :umbrella:
3. Strive to design your contracts to be of a **constant cost in terms of storage and in terms of computation**. Because this way your rent and your gas costs could be more predictable.</u>

