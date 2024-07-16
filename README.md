# StatechainJS
A vanilla javascript implementation of a statechain client and an operator

Differences between Ruben's Initial [Statechains](https://medium.com/@RubenSomsen/statechains-non-custodial-off-chain-bitcoin-transfer-1ae4845a4a39) Idea, [Mercury Layer](https://docs.mercurylayer.com/), and Supertestnet's [Statechains JS Implementation](https://github.com/supertestnet/statechainjs)

Initial statechains idea

- Federated model
- Adaptor signatures

Mercury implementation

- MuSig
- Absolute Timelocks: these limit the amount of time you can hold a statechain utxo before you must submit (and pay for) its backup transaction on the base layer. The user is responsible for doing so at the correct time, and applications can do this automatically.
- Key handling: the operator uses an HSM (Hardware Security Module) to semi-prove key deletion.
- Lightning network interoperability

StatechainJS implementation

- Regular multisig
- Key handling: the operator temporarily stores his private key in a variable called `recovered_privkey` and overwrites it with `null` after signing the user's transaction. Note that this may leave the private key in RAM until overwritten by new data.
- No lightning integration
