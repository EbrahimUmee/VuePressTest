# Peggo

Peggo is a Go implementation of the Gravity Bridge Orchestrator originally implemented by [Injective Labs](https://github.com/InjectiveLabs/) . Peggo itself is a fork of the original Gravity Bridge Orchestrator implemented by [Althea](https://github.com/althea-net)

### **Installation**

To install the `peggo` binary:

`$ make install`

### **How to run**

#### **Setup**

First we must register the validator's Ethereum key. This key will be used to sign claims going from Ethereum to Umee and to sign any transactions sent to Ethereum (batches or validator set updates).

`$ umeed tx gravity set-orchestrator-address \\ {validatorAddress} \\ {orchestrator-address} \\ {ethAddress} \\ --eth-priv-key="..." \\ --chain-id="..." \\ --fees="..." \\ --keyring-backend=... \\ --keyring-dir=... \\ --from=...`

#### **Run the orchestrator**

`export PEGGO_ETH_PK={ethereum private key} $ peggo orchestrator {gravityAddress} \\ --eth-rpc=$ETH_RPC \\ --relay-batches=true \\ --valset-relay-mode=minimum \\ --cosmos-chain-id=... \\ --cosmos-grpc="tcp://..." \\ --tendermint-rpc="http://..." \\ --cosmos-keyring=... \\ --cosmos-keyring-dir=... \\ --cosmos-from=...`

#### **Send a transfer from Umee to Ethereum**

This is done using the command `umeed tx gravity send-to-eth`, use the `--help` flag for more information.

If the coin doesn't have a corresponding ERC20 equivalent on the Ethereum network, the transaction will fail. This is only required for Cosmos originated coins and anyone can call the `deployERC20` function on the Gravity Bridge contract to fix this (Peggo has a helper command for this, see `peggo bridge deploy-erc20 --help` for more details).

This process takes longer than transfers the other way around because they get relayed in batches rather than individually. It primarily depends on the amount of transfers of the same token and the fees the senders are paying.

Important notice: if an "unlisted" (with no monetary value) ERC20 token gets sent into Umee it won't be possible to transfer it back to Ethereum, unless a validator is configured to batch and relay transactions of this token.

#### **Send a transfer from Ethereum to Umee**

Any ERC20 token can be sent to Umee and it's done using the command `peggo bridge send-to-cosmos`, use the `--help` flag for more information. It can also be done by calling the `sendToCosmos` method on the Gravity Bridge contract.

The ERC20 tokens will be locked in the Gravity Bridge contract and new coins will be minted on Umee with the denomination `gravity{token_address}`. This process takes around 3 minutes or 12 Ethereum blocks.

### **How it works**

Peggo allows transfers of assets back and forth between Ethereum and Umee. It supports both assets originating on Umee and assets originating on Ethereum (any ERC20 token).

It works by scanning the events of the contract deployed on Ethereum (Gravity) and relaying them as messages to the Umee chain; and relaying transaction batches and validator sets from Umee to Ethereum.

#### **Events and messages observed/relayed**

#### **Ethereum**

**Deposits** (`SendToCosmosEvent`): emitted when sending tokens from Ethereum to Umee using the `sendToCosmos` function on Gravity.

**Withdraw** (`TransactionBatchExecutedEvent`): emitted when a batch of transactions is sent from Umee to Ethereum using the `submitBatch` function on the Gravity Bridge contract by a validator. This serves as a confirmation to Umee that the batch was sent successfully.

**Valset update** (`ValsetUpdatedEvent`): emitted on init of the Gravity Bridge contract and on every execution of the `updateValset` function.

**Deployed ERC 20** (`ERC20DeployedEvent`): emitted when executing the function `deployERC20`. This event signals Umee that there's a new ERC20 deployed from Gravity, so Umee can map the token contract address to the corresponding native coin. This enables transfers from Umee to Ethereum.

#### **Umee**

**Validator sets**: Umee informs the Gravity Bridge contract who are the current validators and their power. This results in an execution of the `updateValset` function.

**Request batch**: Peggo will check for new transactions in the Outgoing TX Pool and if the transactions' fees are greater than the set minimum batch fee, it will send a message to Umee requesting a new batch.

**Batches**: Peggo queries Umee for any batches ready to be relayed and relays them over to Ethereum using the `submitBatch` function on the Gravity Bridge contract.