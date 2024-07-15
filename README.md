# Statechainjs
A vanilla javascript implementation of a statechain client and an operator

Differences between Ruben's Initial [Statechains](https://medium.com/@RubenSomsen/statechains-non-custodial-off-chain-bitcoin-transfer-1ae4845a4a39) Idea, [Mercury Layer](https://docs.mercurylayer.com/), and Supertestnet's [Statechains JS Implementation](https://github.com/supertestnet/statechainjs)

Initial Statechains Idea:

- Federated Model
- Adaptor Signatures

Mercury Layer:

- MuSig
- Absolute Timelocks: The decrementing timelock backup mechanism limits the number of transfers that can be made within the lock-out time. The user is responsible for submitting backup transactions to the Bitcoin network at the correct time, and applications can do this automatically.
- HSM for Key Deletion: Utilizes a Hardware Security Module (HSM) to semi-prove key deletion.
- Lightning Network Interoperability

Statechains JS Implementation:

- Regular Multisig
- Key Handling: The operator temporarily stores the private key in a variable called `recovered_privkey` and overwrites it with `null` after signing the user's transaction. Note that this may leave the private key in RAM until overwritten by new data.
- No Lightning Integration
