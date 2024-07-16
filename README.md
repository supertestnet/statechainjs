# StatechainJS
A vanilla javascript implementation of a statechain client and an operator

# How statechains work in two paragraphs

A user with "normal bitcoin" deposits it into a multisig bitcoin address (2 of 2) where the depositor has one key and an "operator" has the other. When the depositor wants to transfer the bitcoin to someone else, he does not create a bitcoin transaction, instead he sends the statecoin's private key to his recipient. The recipient then talks to the operator and basically says "Don't talk to the previous guy anymore, only talk to me." The operator is trusted not to collude with any previous holder but only sign transactions created by the latest holder. Cryptographic keys are used to prove who the latest holder is. (StatechainJS uses nostr.)

Also, the first thing you do when you create or receive a statecoin is create a "withdrawal transaction" and get the operator to cosign it. You do not consider a utxo transfer complete until you have this withdrawal transaction. This allows you to take your money out even if the operator shuts down. To prevent a prior holder from broadcasting their own withdrawal transaction first, we use decrementing timelocks: the first holder has to wait 2016 blocks to withdraw, the second holder 2013 blocks, etc. This means the latest holder can always withdraw first, but it also means statechain utxos "expire" and turn back into "regular bitcoins" eventually.

Differences between Ruben's Initial [Statechains](https://medium.com/@RubenSomsen/statechains-non-custodial-off-chain-bitcoin-transfer-1ae4845a4a39) Idea, [Mercury Layer](https://docs.mercurylayer.com/), and Supertestnet's [StatechainJS Implementation](https://github.com/supertestnet/statechainjs)

Mercury implementation

- 1 operator with a single musig key
- Absolute timelocks: these limit the amount of time you can hold a statechain utxo before you must submit (and pay for) its backup transaction on the base layer. The user is responsible for doing so at the correct time, and applications can do this automatically.
- Key handling: the operator uses an HSM (Hardware Security Module) to semi-prove key deletion.
- Lightning network interoperability

Initial statechains idea

- Federated model (multiple operators)
- Relative timelocks: users can hold onto a statechain utxo for however long they want, there is no time limit. The current holder is expected to cosign with the operator(s) to withdraw, but, failing that, they may submit (and pay for) a transaction that "starts the clock" on a timelock path, and then the current holder can withdraw without the further assistance of the operators. The latest holder has the "lowest" timelock. Also, any prior holder *may* broadcast the transaction that "starts the clock," but it costs money to do so and they can't withdraw til after the latest holder gets a chance to do so, so prior holders have no incentive to try to do that. Still, because they *can* do it, the latest holder has to watch the chain "just in case," and be ready to broadcast his withdrawal transaction "first."
- Adaptor signatures allow everyone to see each signature created by the operator, so they can see if he colludes with a prior holder to try and cheat the latest holder

StatechainJS implementation

- Regular multisig
- Key handling: the operator temporarily stores his private key in a variable called `recovered_privkey` and overwrites it with `null` after signing the user's transaction. Note that this may leave the private key in RAM until overwritten by new data.
- Absolute timelocks: just like Mercury, statechainjs uses absolute timelocks, though adding support for relative timelocks is on the "to do" list
- No lightning integration
