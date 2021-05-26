# Generate payment keys and addresses

We need to create two sets of keys and addresses. One set to control our funds \(make and receive payments\) and one set to control our stake \(to participate in the protocol delegating our stake\)

Let's produce our cryptographic keys first, as we will need them to later create our addresses:

{% hint style="danger" %}
**WARNING**

**For ease of use, and for this course only, we will keep our Cardano Testnet key files in the server. But this is NOT SECURE.**

**In a real life scenario \(MAINNET\), you need to have your keys under cold storage.**
{% endhint %}

## Payment key pair

Generate a _payment key pair_:

```text
 cardano-cli address key-gen \
 --verification-key-file payment.vkey \
 --signing-key-file payment.skey
```

This will create two files \(here named `payment.vkey` and `payment.skey`\), one containing the _public verification key_, one the _private signing key_.

The files are in plain-text format and human readable:

```text
 cat payment.vkey

 > type: VerificationKeyShelley
 > title: Free form text
 > cbor-hex:
 >  18af58...
```

The first line describes the file type and should not be changed. The second line is a free form text that we could change if we so wished. The key itself is the cbor-encoded byte-string in the fourth line.

## Payment address

We then use `payment.vkey` to create our `payment address`:

```text
 cardano-cli address build \
 --payment-verification-key-file payment.vkey \
 --out-file payment.addr \
 --testnet-magic 1097911063
```

This created the file payment.addr that is already associated with our stake keys:

```text
cat payment.addr
> addr_test1vrxfh9larxm7uhwfks4danzgffhsjkwwtxam6ad8sy7566sp0r7fj
```

To query your address \(see the utxo's at that address\), you first need to set environment variable `CARDANO_NODE_SOCKET_PATH` to the socket-path specified in your node configuration. In this example we will use the relay node created in the previous steps:

```text
export CARDANO_NODE_SOCKET_PATH=~/relay/db/node.socket
```

Make sure that your node is running. Then, use `cardano-cli query utxo` to find out if any transactions left a balance at this address:

```text
cardano-cli query utxo \
--address $(cat payment.addr) \
--testnet-magic 1097911063
```

You should see something like this:

```text
                       TxHash                                 TxIx        Lovelace
----------------------------------------------------------------------------------
```



{% hint style="info" %}
QUESTIONS AND FEEDBACK

If you have any questions and suggestions while taking the lessons please feel free to ask in the [forum](https://forum.cardano.org/c/english/operators-talk/119) and we will respond as soon as possible.
{% endhint %}
