---
description: A detailed step-by-step guide on how to run an Umee full node
---

# Full Node

## Install

Before proceeding, be sure to reference the Umee [repository](https://github.com/umee-network/umee) to determine what version of `umeed` is required for the network you want to run your node on as well as the chain ID.

Operators can choose to either install the `umeed` binary from the source or download a release for your specific OS and architecture from the [releases](https://github.com/umee-network/umee/releases) page.

To install the `umeed` binary from source, you will need to have Golang [1.17+](https://golang.org/dl/) installed on your OS. Reference the [install](https://golang.org/doc/install) page for instructions on how to install and configure your environment correctly.

Once you have your environment setup correctly, clone the Umee [repository](https://github.com/umee-network/umee) and install the binary:

```bash
# Replace vX.Y.Z with the relevant release version, e.g. v1.0.1
$ git clone --depth 1 --branch vX.Y.Z https://github.com/umee-network/umee.git
$ cd umee && make install
```

Finally, verify your `umeed` version:

{% hint style="warning" %}
Verify you are using v1.0.2 until your node will be synced on block 24615, then you should upgrade to v1.0.3.
{% endhint %}

```bash
$ umeed version
```

## Hardware Requirements

The recommended hardware to run an Umee node will vary depending on the use case and desired functionalities of the node. For example, a significant amount of disk space can be required if the node will act as an archive node, i.e. `pruning=nothing` or if the node is a state-sync snapshot provider. In general, we recommend at a minimum the following specifications:

* 2+ vCPU
* 4+ GB RAM
* 120+ GB SSD

## Initialize

Before we can start the `umeed` process, we must first initialize our node:

```bash
# Replace moniker with your desired node's moniker and the chain ID
# of the network you are joining.
$ umeed init <moniker> --chain-id <chain-id>
```

The above command creates and initializes an `.umee` directory in your `$HOME` path by default. This directory contains all the configuration files needed for you to run your node along with a default `genesis.json` file. You can overwrite the location of the `.umee` directory by specifying a `--home` flag.

{% hint style="info" %}
Note, when joining an existing network, the provided `--chain-id` value does not matter as you'll be replacing the auto-generated `genesis.json` anyway.
{% endhint %}

Once initialized, overwrite the default `genesis.json` file with genesis state file for the particular network that you are joining. You may retrieve the genesis state file from the Umee [repository](https://github.com/umee-network/umee) or another trusted source:

```bash
$ cd ~/.umee/config
$ wget https://raw.githubusercontent.com/umee-network/umee/main/.../genesis.json
```

{% hint style="info" %}
Note, be sure to verify the genesis file SHA256 sum with what is referenced in the Umee repository.
{% endhint %}

## Configure

Before starting your node, it is important to verify and update any relevant configuration. There are three primary configuration files located in `~/.umee/config/`:

* `config.toml`: Used to configure Tendermint. Learn more about Tendermint's [configuration](https://docs.tendermint.com/master/nodes/configuration.html).
* `app.toml`: Used to configure the Umee application. This includes configurations such as application pruning, state sync, minimum fees, and API/gRPC settings.
* `client.toml`: Used to configure client-side inputs when using the `umeed` CLI to interact with the network. This configuration is entirely optional and is primarily used as a convenience to avoid having to input the same arguments into `umeed` CLI commands.

Both `config.toml` and `app.toml` are heavily commented and should be used as a reference on how to tweak your settings.

We recommend at least verifying and potentially modifying the following configurations:

* `app.toml`
  * `minimum-gas-prices`: Set this to a non-empty value to have the node require minimum fees when processing transactions, e.g. `minimum-gas-prices = "0.001uumee"`. Recall, a transaction's fee is calculated by `fee = ⌈tx.gasLimit * tx.gasPrice⌉`, so the transaction's fee must be at least `⌈minimumGasPrices * tx.gasPrice⌉`.
  * `pruning`: By default, the application will keep an unbonding period's, in blocks, worth of application data (362880) and prune the remaining application state. You should change this value if you want to prune all application state, `pruning = "everything"`, or if you'd like to keep all application state (archive), `pruning = "nothing"`.
  * `min-retain-blocks`: This value pertains to Tendermint block pruning. This is different than `pruning` as that relates to application state. When non-zero, the application will inform Tendermint to prune blocks beyond the `min-retain-blocks` threshold. Note, internally the application enforces a minimum as at least an unbonding period's worth of blocks must be kept for safety.
  * `api.enable`: Enable the application's API (gRPC HTTP gateway) service.
  * `grpc.enable`: Enable the application's gRPC service.
* `config.toml`&#x20;
  * `rpc.laddr`: By default, Tendermint's RPC listens on the localhost interface. If you wish to reach the Tendermint RPC externally, set this to `0.0.0.0` to listen on all interfaces/IPs.
  * `p2p.external_address`: Set this value to the node's public and static IP address. This will help ensure other nodes in Tendermint's p2p gossip layer will be able to successfully reach and connect to your node.
  * `p2p.seeds`: Set to a list of available seed nodes (if any). Seed nodes are nodes that purely exchange peer information with your node and then disconnect. Set this value if you do not have any available or known peers to connect to.
  * `p2p.persistent_peers`: Set to a list of available and trusted peers (if any).

## Cosmovisor and Service Management

We encourage the use of [systemd](https://systemd.io/) in addition to [cosmovisor](https://docs.cosmos.network/master/run-node/cosmovisor.html), a cosmos binary supervisor, to manage your `umeed` process.

For a detailed guide on setting up [cosmovisor](https://docs.cosmos.network/master/run-node/cosmovisor.html), please see the [setup instructions](https://docs.cosmos.network/master/run-node/cosmovisor.html#setup).

{% hint style="warning" %}
Only set `DAEMON_ALLOW_DOWNLOAD_BINARIES=true` if you're running a full-node. Validators are encouraged to download or build the upgraded binaries ahead of time and verify they are correct.
{% endhint %}
