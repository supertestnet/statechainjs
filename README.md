# StatechainJS
A vanilla javascript implementation of a statechain client and an operator

# How statechains work in two paragraphs

A user with "normal bitcoin" deposits it into a multisig bitcoin address (2 of 2) where the depositor has one key and an "operator" has the other. When the depositor wants to transfer the bitcoin to someone else, he does not create a bitcoin transaction, instead he sends the statecoin's private key to his recipient. The recipient then talks to the operator and basically says "Don't talk to the previous guy anymore, only talk to me." The operator is trusted not to collude with any previous holder -- he must only sign transactions created by the latest holder. Cryptographic keys are used to prove who the latest holder is. (StatechainJS uses nostr.) Note that an operator can rugpull someone by depositing coins to their own statechain (that way they have both keys), then sending those statechain utxos to someone in exchange for goods/services, then using both keys to sweep the funds from the multisig, so that they end up with the goods/services *and* the money. Operators must be trusted not to do this.

Also, the first thing you do when you create or receive a statecoin is create a "withdrawal transaction" and get the operator to cosign it. You do not consider a utxo transfer complete until you have this withdrawal transaction. This allows you to keep your money (minus a transaction fee) even if the operator shuts down, which is an advantage over something like ecash, where, if the mint shuts down, he takes every user's money with him. To prevent a prior holder from broadcasting their own withdrawal transaction first, we use decrementing timelocks: the first holder has to wait 2016 blocks to withdraw, the second holder 2013 blocks, the third holder 2010 blocks, etc. This means the latest holder can always withdraw first, but it also means statechain utxos "expire" and turn back into "regular bitcoins" eventually -- and it also means the latest holder must be online (or use a watchtower) when his timelock expires, otherwise, a few blocks later, a prior holder can sweep the bitcoins from the multisig.

# Implementations & specs

[Mercury implementation](https://docs.mercurylayer.com/)

- There is 1 operator and the bitcoins are held in a single musig key with two "keyshards" -- the operator's and the depositor's
- Absolute timelocks: these limit the amount of time you can hold a statechain utxo before you must withdraw on the base layer. The user is responsible for doing so at the correct time, and applications can do this automatically.
- Key handling: the operator uses an HSM (Hardware Security Module) to semi-prove key deletion after cosigning each new holder's withdrawal transaction. He only keeps partial recovery data and gives the latest holder the rest of the recovery data. That user he is expected to "pass it back" to the operator in the future if he wants him to "recover" his private key and sign a new transaction, or pass it on to the next holder so he can do this. The recovery data is one-time-use (it changes every time) and new holders "refresh it" whenever they receive coins, so by consequence, a prior holder cannot get the operator to sign a new transaction -- only the latest holder can do this.
- Lightning network interoperability

[Initial statechains idea](https://medium.com/@RubenSomsen/statechains-non-custodial-off-chain-bitcoin-transfer-1ae4845a4a39)

- Statechains were originally designed for a federated model with multiple operators. Whether to use musig or regular multisig was left unspecified
- Relative timelocks: users can hold onto a statechain utxo for however long they want, there is no time limit. The current holder is expected to cosign with the operator(s) to withdraw, but, failing that, they may submit (and pay for) a transaction that "starts the clock" on a timelock path, and then the current holder can withdraw without the further assistance of the operators. The latest holder has the "lowest" timelock, so they can always withdraw first. Also, any prior holder *may* broadcast the transaction that "starts the clock," but it costs money to do so and they can't withdraw til after the latest holder gets a chance to do so, so prior holders have no incentive to try to do that. Still, because they *can* do it, the latest holder has to watch the chain "just in case," and be ready to broadcast his withdrawal transaction "first."
- Adaptor signatures allow everyone to see each signature created by the operator, so they can see if he colludes with a prior holder to try and cheat the latest holder

StatechainJS implementation

- There is 1 operator and the bitcoins are held in a regular 2 of 2 multisig address (musig is not used)
- Key handling: like Mercury, the operator deletes his private key after every transfer and only stores partial recovery data, giving the rest to the latest holder. Unlike mercury, an HSM is not used. Instead, the operator temporarily stores his private key in a variable called `recovered_privkey` and overwrites it with `null` after signing the user's transaction. Note that this may leave the private key in RAM until it gets overwritten by new data, so a motivated operator could probably recover it in order to collude with a prior holder to pull off a rugpull.
- Absolute timelocks: just like Mercury, statechainjs uses absolute timelocks, though adding support for relative timelocks is on the "to do" list
- No lightning integration
