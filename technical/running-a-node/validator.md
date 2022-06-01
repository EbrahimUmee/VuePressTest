---
description: A detailed step-by-step guide on how to run an Umee validator
---

# Validator

This guide contains instructions on how to setup and run an Umee validator. First, be sure to check out the full node [instructions](full-node.md) on how to install and configure the `umeed` binary as this guide assumes you already have it installed and configured.

In order to become an **active** validator, you must have more stake than the [bottom validator](https://www.mintscan.io/umee/validators). You may still execute the following steps, but you will not be active and therefore won't receive staking rewards.

## Keyring

Prior to creating your validator, you must first create your "operator" key. Note, this is not your consensus key and will not be used for signing. Instead, it is used to identify you validator in the Umee network.

```bash
$ umeed keys add <key-name> [flags]
```

By default, `umeed` will store keys in your OS-backed keyring. You can change this behavior by specifying the `--keyring-backend` flag.

If you already have a key that you'd like to import via a mnemonic, you can provide a `--recover` flag and the `keys add` command will prompt you for the BIP39 mnemonic.

Visit the Cosmos SDK's keyring [documentation](https://docs.cosmos.network/v0.43/run-node/keyring.html) for more information.

## Ethereum Node

The Gravity Bridge requires that validators also run a `peggo` orchestrator in addition to the `umeed` process. The orchestrator requires access to a `geth` node's RPC instance. A `geth` light client can be used, **but a full node is preferable**.

You may choose to operate your own `geth` node or use a publicly available one. However, in production environments, it is recommended that you run your own. Depending on what network you're running your Umee validator on, you'll want to connect to or setup your `geth` node to the appropriate Ethereum network. See the `geth` CLI [documentation](https://geth.ethereum.org/docs/interface/command-line-options) for more information on how to connect to different Ethereum networks.

To setup a `geth` node, first install the binary from go-ethereum's [download](https://geth.ethereum.org/downloads/) page. Then, create a [systemd](https://systemd.io/) service:

{% code title="/etc/systemd/system/geth.service" %}
```bash
[Unit]
Description=Geth node
After=online.target

[Service]
Type=root
User=root
ExecStart=/usr/bin/geth --syncmode "light" --http ...
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```
{% endcode %}

{% hint style="info" %}
If you need to access the `geth` instance externally, be sure to set `--http.addr=0.0.0.0`.
{% endhint %}

Reload [systemd](https://systemd.io/) and start the `geth` service:

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start geth
```

Finally, verify the `geth` service is running and healthy and enable it:

```bash
$ sudo systemctl status geth
$ sudo systemctl enable geth
```

## Create Validator

Once you have `umeed` and `geth` running, you can finally create your validator on the Umee network via a `MsgCreateValidator` transaction:

```bash
# Validator's consensus public key
# e.g. {"@type":"/cosmos.crypto.ed25519.PubKey","key":"..."}
$ UMEE_VAL_CONS_KEY=$(umeed tendermint show-validator)
$ umeed tx staking create-validator \
  --amount=<amount> \
  --pubkey=$UMEE_VAL_CONS_KEY \
  --moniker="<moniker>" \
  --chain-id="<chain-id>" \
  --commission-rate="<commission>" \
  --commission-max-rate="<max-commission>" \
  --commission-max-change-rate="<max-commission-rate-change>" \
  --min-self-delegation="<min-self-delegation>" \
  --fees=<fees> \
  --from=<key-name>
```

Note, you must use the `chain-id` that corresponds to the network you're joining (`umee-1` for mainnet). The key`key-name` corresponds to the validator operator key you created earlier.

## Gravity Bridge

Validators are also required to run a critical component of the Gravity Bridge known as an orchestrator (`peggo`). The orchestrator serves multiple purposes, but it mainly acts as an off-chain relayer and oracle between the Umee network and Ethereum.

The orchestrator requires a few components in order to run successfully:

* Umee gRPC instance
* Ethereum RPC instance
* Signing key on the Umee chain with funds to relay transactions to Umee
* Signing key on the Ethereum chain with funds to relay transactions to Ethereum

The Umee network uses `peggo` implementation of the Gravity Bridge Orchestrator originally implemented by [Injective Labs](https://github.com/InjectiveLabs/). Peggo itself is a fork of the original Gravity Bridge Orchestrator implemented by [Althea](https://github.com/althea-net). Visit the [releases](https://github.com/umee-network/peggo/releases) page to download the corresponding version of `peggo`.&#x20;

```bash
$ PEGGO_VERSION=v0.2.4
$ wget https://github.com/umee-network/peggo/releases/download/$PEGGO_VERSION/peggo-$PEGGO_VERSION-linux-amd64.tar.gz
$ tar xvf peggo-$PEGGO_VERSION-linux-amd64.tar.gz && cd peggo-$PEGGO_VERSION-linux-amd64 && rm LICENSE README.md
$ chmod +x ./peggo
$ mv ./peggo /usr/local/bin
$ peggo --help
```

Now we must configure our `peggo` keys:

{% hint style="warning" %}
Do not use the same address for the validator and orchestrator.
{% endhint %}

{% hint style="info" %}
To send transaction below, your node must be synced with the `umee-1` network
{% endhint %}

```bash
# umeed tx gravity set-orchestrator-address [validator-address] [orchestrator-address] [ethereum-address] [flags]
umeed tx gravity set-orchestrator-address VALIDATOR_OPERATOR_ADDRESS ORCHESTRATOR_ADDRESS ETHEREUM_ADDRESS --chain-id="umee-1" --from=<key-name>
```

Assuming you already have `umeed` and `geth` instances running, next we should set variables for set up `peggo`:

```bash
$ PEGGO_ETH_PK="ETHEREUM_ADDRESS_PRIVATE_KEY"
$ ETH_RPC="YOUR_ETH_NODE_ADDRESS"
$ BRIDGE_ADDR="0xb564ac229e9d6040a9f1298b7211b9e79ee05a2c"
$ START_HEIGHT="14211970"
$ ORCHESTRATOR_WALLET_NAME="ORCHESTRATOR_ADDRESS_WALLET_NAME"
$ ORCHESTRATOR_WALLET_PASSWORD="ORCHESTRATOR_ADDRESS_WALLET_PASSWORD"
$ ALCHEMY_ENDPOINT="YOUR_WSS_ALCHEMY_ENDPOINT" # https://www.alchemy.com/
```

{% hint style="info" %}
The Gravity Bridge contract must be deployed prior to configuring and starting the orchestrator process. Please see the Umee [repository](https://github.com/umee-network/umee) for more information.
{% endhint %}

Next, create a `peggod` [systemd](https://systemd.io/) service:

```bash
$ echo "[Unit]
Description=Peggo Service
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/peggo orchestrator $BRIDGE_ADDR \
  --bridge-start-height=\"$START_HEIGHT\" \
  --eth-rpc=\"$ETH_RPC\" \
  --relay-batches=true \
  --relay-valsets=false \
  --cosmos-chain-id=umee-1 \
  --cosmos-keyring-dir=\"$HOME/.umee\" \
  --cosmos-from=\"$ORCHESTRATOR_WALLET_NAME\" \
  --cosmos-from-passphrase=\"$ORCHESTRATOR_WALLET_PASSWORD\" \
  --eth-alchemy-ws=\"$ALCHEMY_ENDPOINT\" \
  --cosmos-keyring=\"os\" \
  --log-level debug \
  --log-format text
Environment=\"PEGGO_ETH_PK=$PEGGO_ETH_PK\"
Restart=always
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/peggod.service
$ sudo mv $HOME/peggod.service /etc/systemd/system
```

Finally enable `peggod` at startup and run:

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable peggod
$ sudo systemctl restart peggod
```

You can check `peggod` logs via the following command:

```
$ journalctl -u peggod -f -o cat
```
