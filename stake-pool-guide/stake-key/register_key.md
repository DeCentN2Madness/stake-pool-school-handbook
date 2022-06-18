# Register stake address in the blockchain

Before, we created our payment keys and address, which allow us to control our funds; we also created our stake keys and stake address, which allow us to participate in the protocol by delegating our stake or by creating a stake pool.

We need to _register_ our **stake key** in the blockchain. To achieve this, we:

* Create a registration certificate
* Determine the Time-to-Live \(TTL\)
* Calculate the fees and deposit   
* Submit the certificate to the blockchain with a transaction

## Create a _registration certificate_:

```text
cardano-cli stake-address registration-certificate \
--stake-verification-key-file stake.vkey \
--out-file stake.cert
```

Once the certificate has been created, we must include it in a transaction to post it to the blockchain.

## Determine the TTL

As before, to create the transaction you need to determine the TTL querying the tip and adding some slots to give you sufficient time to build the transaction.

## Draft transaction

```text
cardano-cli transaction build-raw \
--tx-in b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee#1 \
--tx-out $(cat paymentwithstake.addr)+0 \
--ttl 0 \
--fee 0 \
--out-file tx.raw \
--certificate-file stake.cert
```

## Calculate fees and deposit

This transaction has only 1 input \(the UTXO used to pay the transaction fee\) and 1 output \(our `paymentwithstake.addr` to which we are sending the change\). This transaction has to be signed by both the payment signing key `payment.skey` and the stake signing key `stake.skey`; and includes the certificate `stake.cert`:

```text
cardano-cli transaction calculate-min-fee \
--tx-body-file tx.raw \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 1 \
--byron-witness-count 0 \
--testnet-magic 1097911063 \
--protocol-params-file protocol.json

> 171485
```

In this transaction we have to not only pay transaction fees, but also include a _deposit_ \(which we will get back when we deregister the key\) as stated in the protocol parameters:

The deposit amount can be found in the `protocol.json` under `stakeAddressDeposit`:

```text
    ...
    "stakeAddressDeposit": 2000000,
    ...
```

Query the UTXO:

```text
    cardano-cli query utxo \
        --address $(cat payment.addr) \
        --testnet-magic 1097911063

    >                            TxHash                                 TxIx        Lovelace
    > ----------------------------------------------------------------------------------------
    > b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee     1      1000000000
```

So if we had 1000 ada, to calculate the change to send back to `payment.addr` we run:

```text
expr 1000000000 - 171485 - 2000000

> 997828515
```

## Submit the certificate with a transaction:

Build the transaction:

```text
cardano-cli transaction build-raw \
--tx-in b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee#1 \
--tx-out $(cat paymentwithstake.addr)+997828515 \
--ttl 987654 \
--fee 171485 \
--out-file tx.raw \
--certificate-file stake.cert
```

Sign it:

```text
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--testnet-magic 1097911063 \
--out-file tx.signed
```

And submit it:

```text
cardano-cli transaction submit \
--tx-file tx.signed \
--testnet-magic 1097911063
```

Your stake key is now registered on the blockchain.

{% hint style="info" %}
QUESTIONS AND FEEDBACK

If you have any questions and suggestions while taking the lessons please feel free to ask in the [forum](https://forum.cardano.org/c/english/operators-talk/119) and we will respond as soon as possible.
{% endhint %}

