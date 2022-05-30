# Confondo -- A Bitcoin privacy Idea

## Intro
All bitcoin tracking problems boil down to two things:

1. external observer sees what UTXO you're using as input: can use that info for both forward and backward tracking
2. for that single transaction, assuming multiple inputs and outputs, he can easily track which input funded which output, unless carefully crafted (e.g. [dmix](https://fadibarbara.it/papers/dimx.pdf) or leveraging [subset-sum problem](https://en.wikipedia.org/wiki/Subset_sum_problem))

**Can we flip point 2. on its own head, instead of avoiding it?**

## Proposed Solution
Assume Alice (A) wants to pay Bob (B) 1btc. Main idea is: We get Carol (C) in the mix

Here is the solution:

1. A finds C and knows about her fees for the service (in this case, a 9.5% on the payment)
1. A creates a new address for C, e.g. using [paynym](https://samourai.kayako.com/article/68-what-are-paynyms)
1. A creates a new transaction TX where she basically pays (note that inputs do not balance output; at this stage this transaction would result as invalid if broadcasted):

    |Input              | Output|
    |-----|---|
    |`Addr_A1`: 0.5btc    |`Addr_B`: 1btc|
    |`Addr_A2`: 0.8btc    |`Addr_A_change`: 0.2btc|
    |                  |`Addr_C_fresh`: 1.095btc|

1. A signs TX with `SIGHASH_ALL|SIGHASH_ANYONECANPAY` (inputs and all ouputs signed, but others can add inputs)
1. A sends TX to C (maybe C has a box in her site where you can drop signed tx and a backend to process them)
1. C performs basic checks (is one of the address hers? does she get enough fees?)
1. C adds one or more input in order to pay 1btc and signs them
1. C broadcast a transaction TX similar to this:



    |Input               |Output|
    |--|--|
    |`Addr_A1`: 0.5btc     |`Addr_B`: 1btc|
    |`Addr_A2`: 0.8btc     |`Addr_A_change`: 0.2btc|
    |`Addr_C_old1`: 0.3btc |`Addr_C_fresh`: 1.095btc|
    |`Addr_C_old2`: 0.7btc | |


## (Superficial) Analysis

### Atomicity
One highly undervalued and unappreciated things of Bitcoin tranasctions is that they inherently achieve atomicity by putting inputs and outputs together in the same tranasctions. We use that here: A pays C which pays B. It's an all or nothing transaction.

### External observer point of view
Assume I am Mallory and I see what happens on the blockchain. Of course if I know nothing about any of the addresses, then seeing TX doesn't add any information on what I know.

On the other hand, if I (Mallory) do know *everything* I can possibly know, i.e. :

* `Addr_A` belongs to Alice
* `Addr_B` belongs to Bob
* `Addr_C_old1` and `Addr_C_old2` belong to Carol

(note that I can't have any knowledge of `Addr_A_fresh` and `Addr_C_fresh` by definition, assuming no address reuse) then transaction TX results in something like: **Carol pays Bob, Alice pays a new address and get some change**

Something about the signatures though. By looking closely at the transaction, the signature is strange, since A pays for all outputs, but only for some inputs. Yet the knowledge of an external observer doesn't (*shouldn't*) change 

### Feasibility
The technology to do that already exists: [paynyms](https://samourai.kayako.com/article/68-what-are-paynyms) give A the possibility to have a fresh address from C non interactively. 

One question is: how does A know that C has the inputs to pay for that? A way to bypass that problem is for A to use `SIGHASH_SINGLE`, but then it would be clear that A is interested only in the payment to B which would be detrimental to the privacy-enhancing thing. A way to solve the problem could be some interactive zero-knowledge exchange where C proves to A that she owns a set of UTXOs that can be useful for A. But this is more tricky to do...


## Related Works
### PaySwap
payswap is similar: wehn A pays B, B participates in the transaction and adds some coins. Still, B knows he's being paid by A
## Conclusion
