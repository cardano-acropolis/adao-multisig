# ADAO Multisig

## Objectives

 1. Easy-to-use user interface
 2. Generate multi-sig wallets (wallet addresses and keys for multisig
    transactions)
 3. Make it easy to sign transactions etc
 4. Flesh out and expand ADAO user interface
 5. Make it a catalyst alternative.

## Tasks

 - [ ] Discover how to lock funds in script address. (convert script
       to script address)
 - [ ] Choose UI
 - [ ] Write script proof-of-concept
 - [ ] Tom shows it off -- create an awesome demo
 - [ ] Figure out how to manage staking rights after locking funds.
 - [ ] [Build electron app](https://www.youtube.com/watch?v=kN1Czs0m1SU)
 
## Progress Notes

We have chosen to use the cardano-wallet commandline tool to set up a
server to create mnemonic phrases which can then be used to generate
addresses. We have then chosen to use the cardano-address CLI to
generate multi-sig scripts. We generate a multi-sig address. We can
then put those multi-sig scripts into the cardano-wallet API to submit
the transaction to lock the funds into the multi-sig address. 

Below, we use the cardano-wallet CLI to generate private and public
address keys. [Here](https://input-output-hk.github.io/cardano-wallet/api/edge/) is a link to the cardano-wallet API.

```bash
cardano-wallet recovery-phrase generate >> new_phrase.txt
cat new_phrase.txt | cardano-wallet key from-recovery-phrase Shared >> root.skey
cat root.skey | cardano-wallet key child 1852H/1815H/0H >> acct.skey
cat acct.skey | cardano-wallet key child 1852H/1815H >> addr.skey
cat addr.skey | cardano-wallet key public --with-chain-code >> addr.vkey
```

Below is an example of using the cardano-address CLI to generate keys and a script hash and preimage.

```
cardano-address recovery-phrase generate --size 15 \
    | cardano-address key from-recovery-phrase Shared > root_shared.xsk
cat root_shared.xsk \
    | cardano-address key child 1854H/1815H/0H/0/0 > signingKey1.xsk
cat signingKey1.xsk \
    | cardano-address key public --without-chain-code \
    | cardano-address key hash > verKey1.vkh
cat root_shared.xsk \
    | cardano-address key child 1854H/1815H/0H/0/1 > signingKey2.xsk
cat signingKey2.xsk \
    | cardano-address key public --without-chain-code \
    | cardano-address key hash > verKey2.vkh
cardano-address script hash "all [$(cat verKey1.vkh),$(cat verKey2.vkh)]"
cardano-address script preimage "all [addr_shared_vkh1zxt0uvrza94h3hv4jpv0ttddgnwkvdgeyq8jf9w30mcs6y8w3nq, active_from 100, active_until 150]"
```

We might be able to serve the wallet remotely and the address
locally. The trick will be to convert between the cardano-wallet
format and the cardano-address format. Fortunately, I have already
published a stack exchange answer about this: [my post](https://cardano.stackexchange.com/questions/2505/create-transaction-by-cardano-node-using-payment-address-generated-by-cardano-ad/2514#2514)

```
cardano-cli key convert-cardano-address-key --shelley-payment-key --signing-key-file signingKey2.xsk --out-file signingKey2.skey
```

More information may be found
[here](https://github.com/input-output-hk/cardano-addresses#overview). It
also tells you how exactly the cborhex is constructed.

This last bit can be done as an NPM package in javascript in the
browser. You can find the API
[here](https://input-output-hk.github.io/cardano-addresses/typescript/) and [here](https://www.npmjs.com/package/cardano-addresses).

[Here](https://input-output-hk.github.io/cardano-addresses/haddock/cardano-addresses-3.6.1/Cardano-Address-Style-Shared.html)
is the Haddock documentation for the shared addresses.

### Important

[cardano-cli --tx-in-scrpt-file info](https://github.com/input-output-hk/iohk-monitoring-framework/wiki/Transaction-Generator:-Usage-Guide). [Here is documentation for simple scripts.](https://github.com/input-output-hk/cardano-node/blob/7a056fd4b0c810906f66b3acc3031b4a02472d45/doc/reference/simple-scripts.md)
This is actually completely sufficient.

Tom's idea: allocate a portion of governance tokens to Holger's pool.

```
{
  "type": "all",
  "scripts":
  [
    {
      "type": "sig",
      "keyHash": "e09d36c79dec9bd1b3d9e152247701cd0bb860b5ebfd1de8abb6735a"
    },
    {
      "type": "sig",
      "keyHash": "a687dcc24e00dd3caafbeb5e68f97ca8ef269cb6fe971345eb951756"
    },
    {
      "type": "sig",
      "keyHash": "0bd1d702b2e6188fe0857a6dc7ffb0675229bab58c86638ffa87ed6d"
    }
  ]
}
```

### Electron app

[Setup](https://www.youtube.com/watch?v=kN1Czs0m1SU)

The electron app needs to contact the Cardano Wallet API (or another
API) to submit the transaction. It needs to store one's voting
information. It must be able to sign transactions for the official
vote. It might be bale forward them to a node that will submit the
transaction. Otherwise, the file can merely be sent via other
means. It needs to be able to send data to other members using the app
to decide upon the official vote.

[Support Trezor](https://wiki.trezor.io/Trezor_Connect_API) by
incorporating their JavaScript.

[Helpful for managing dependencies](https://webpack.electron.build/dependency-management) yes.

[For the API requests](https://www.geeksforgeeks.org/http-rest-api-calls-in-electronjs/)

So now we just need to wrap up by adding the API requests and finishing the UI.

#### Incorporating the cardano-js-sdk

20211113133724NJ This is a work in progress. I finally have my system
configured correctly.

Documentation is located [here](https://input-output-hk.github.io/cardano-js-sdk/). 

[This video](https://www.youtube.com/watch?v=GU-2T7k9NfI) was quite useful in learning about webpack. 

### Educational Microfinancing

Let's look into this.

### ADAO Voting Rules State Machine

The State Machine can be located by finding the token that it
contains. The state machine can be revised by voting to create a new
state machine with different parameters.

### Roadmap

(This is tentative)

 1. Multisig wallet electron app.
    1. Preleminary vote
    2. Official vote to move forward with the winning preleminary vote
       (must meet a certain quota of signatures).
 2. Smart contract with endpoints callable from the electron app.
    1. This phase begins when everyone votes to transfer the from the
       multisig wallet to a new address and issue tokens.
