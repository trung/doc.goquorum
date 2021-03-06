---
description: Transaction and Contract Privacy
---

# Transaction and Contract Privacy

GoQuorum achieves Transaction Privacy by:

1. Enabling transaction Senders to create a private transaction by marking who is privy to that transaction via the `privateFor` parameter
1. Replacing the payload of a private transaction with a hash of the encrypted payload, such that the original payload is not visible to participants who are not privy to the transaction
1. Storing encrypted private data off-chain in a separate component called the [Privacy Manager](PrivateTransactionManager.md). The Privacy Manager encrypts private data, distributes the encrypted data to other parties that are privy to the transaction, and returns the decrypted payload to those parties

GoQuorum introduces the notion of 'Public Transactions' and 'Private Transactions'. Note that this is a notional concept only and GoQuorum does not introduce new Transaction Types, but rather, the Ethereum Transaction Model has been extended to include an optional `privateFor` parameter (the population of which results in a Transaction being treated as private by GoQuorum) and the Transaction Type has a new `IsPrivate` method to identify such Transactions.

## Public Transactions

Public Transactions are those Transactions whose payload is visible to all participants of the same GoQuorum network. These are [created as standard Ethereum Transactions in the usual way](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethsendtransaction).

Examples of Public Transactions may include Market Data updates from some service provider, or some reference data update such as a correction to a Bond Security definition.

!!! Note
    GoQuorum Public Transactions are not Transactions from the public Ethereum network. Perhaps a more appropriate term would be 'common' or 'global' Transactions, but 'Public' is used to contrast with 'Private' Transactions.

## Private Transactions

Private Transactions are those Transactions whose payload is only visible to the network participants whose public keys are specified in the `privateFor` parameter of the Transaction .  `privateFor` can take multiple addresses in a comma separated list. (See Creating Private Transactions under the [Developing Smart Contracts](../../HowTo/Use/DevelopingSmartContracts.md#creating-private-transactions/contracts) section).

When the GoQuorum Node encounters a Transaction with a non-null `privateFor` value, it sets the `V` value of the Transaction Signature to be either `37` or `38` (as opposed to `27` or `28` which are the values used to indicate a Transaction is 'public' as per standard Ethereum as specified in the Ethereum yellow paper).

## Public vs Private Transaction Handling

Public Transactions are executed in the standard Ethereum way, and so if a Public Transaction is sent to an Account that holds Contract code, each participant will execute the same code and their underlying StateDBs will be updated accordingly.

Private Transactions, however, are not executed per standard Ethereum: prior to the sender's GoQuorum Node propagating the Transaction to the rest of the network, it replaces the original Transaction Payload with a hash of the encrypted Payload that it receives from Constellation/Tessera. Participants that are party to the Transaction will be able to replace the hash with the actual payload via their Constellation/Tessera instance, whilst those Participants that are not party will only see the hash.

The result is that if a Private Transaction is sent to an Account that holds Contract code, those participants
who are not party to the Transaction will simply end up skipping the Transaction, and therefore not execute the Contract code.
However those participants that are party to the Transaction will replace the hash with the original Payload
before calling the EVM for execution, and their StateDB will be updated accordingly. In absence of making corresponding
changes to the geth client, these two sets of participants would therefore end up with different StateDBs and
not be able to reach consensus. In order to support this bifurcation of contract state, Quorum stores the state of
Public contracts in a Public State Trie that is globally synchronised, and it stores the state of Private contracts
in a Private State Trie that is not synchronised globally.
