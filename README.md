# GUIDE: Connecting to Cronos with IBC

***Overview:*** 

The Inter-Blockchain Communication (IBC) protocol enables secure and efficient data exchange between blockchains. Cronos Chain, was built using the Cosmos SDK, which natively supports IBC. This guide outlines the steps to connect Cronos Chain with another IBC-enabled chain, using Osmosis as an example. The same process applies to any IBC-enabled chain.

### Prerequisites:

Requirements for IBC Integration with Cronos:

- ***Compatibility:***
  - Ensure your chain supports IBC and complies with the latest IBC specifications.
  - Light Client Implementation: Both chains must implement light clients to verify states.
      
- ***Infrastructure:***
  - Cronos Full Node: An operational Cronos full node with IBC enabled.
  - Your desired Chain Full Node : An operational full node with IBC enabled.
  - Relayer Software: Such as  Hermes(Rust) or rly(Go) installed and configured.


### Relayer Setup:

Most of the IBC complexity is abstracted away by the relayer software. It handles the transport and application layers. For a detailed overview on how IBC works under the hood, please refer to [IBC offcial site](https://ibc.cosmos.network/v8/ibc/light-clients/overview/) or on the [Cosmos SDK site](https://tutorials.cosmos.network/academy/3-ibc/1-what-is-ibc.html#high-level-overview-of-ibc).

The two main options for relayer software are [Hermes](https://hermes.informal.systems) a rust based implementation and [rly](https://github.com/cosmos/relayer/tree/main/docs) a go based implementation. 

This guide focuses on Hermes. You can explore the alternative rly implementation [here](https://github.com/cosmos/relayer/tree/main/docs).

***1. Install Hermes:***

It is best to follow the [Hermes Quickstart Guide](https://hermes.informal.systems/quick-start/) for installation instructions.

***2. Configure the chains:***

To connect the chains, you'll need to configure Hermes. The configuration file, config.toml, contains the necessary details for each chain involved in the IBC connection. Here's how to set it up:

  - Locate the Config File: The Hermes configuration file is typically located at $HOME/.hermes/config.toml.

  - Simplify with Auto-Config: Hermes offers an auto-config feature that uses the [chain-registry]() to automatically populate the configuration for chains like Osmosis and Cronos, which are already included in the registry. Execute the following command in your terminal:
  
  ```
  hermes config auto --output $HOME/.hermes/config.toml --chain  osmosis:keyosmosis cronos:keycronos --chain 
  ```
This command retrieves the necessary settings for both chains and writes them to config.toml.

  ***3. Verify the Output:*** If the command runs successfully, you should see the following message:
  ```
  SUCCESS "Config file written successfully : $HOME/.hermes/config.toml."
  ```

Review the generated Config File: Open the config.toml file to ensure it contains the required details. It should resemble the following structure:

<details>
<summary>***Click to expand the config.toml***</summary>

```toml
[global]
log_level = 'info'
[mode.clients]
enabled = true
refresh = true
misbehaviour = true

[mode.connections]
enabled = false

[mode.channels]
enabled = false

[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = false

[rest]
enabled = false
host = '127.0.0.1'
port = 3000

[telemetry]
enabled = false
host = '127.0.0.1'
port = 3001
[[chains]]
id = 'osmosis-1'
type = 'CosmosSdk'
rpc_addr = 'https://rpc.osmosis.interbloc.org/'
event_source = { mode = 'push', url = 'wss://rpc.osmosis.interbloc.org/websocket', batch_delay = '500ms' }
grpc_addr = 'https://grpc-osmosis-ia.notional.ventures/'
rpc_timeout = '10s'
account_prefix = 'osmo'
key_name = 'keyosmosis'
key_store_type = 'Test'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 400000
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '5s'
max_block_time = '30s'
memo_prefix = ''
sequential_batch_tx = false

[chains.trust_threshold]
numerator = '1'
denominator = '3'

[chains.gas_price]
price = 0.1
denom = 'uosmo'

[chains.packet_filter]
policy = 'allow'
list = [[
    'transfer',
    'channel-0',
]]

[chains.address_type]
derivation = 'cosmos'
[[chains]]
id = 'cronosmainnet_25-1'
type = 'CosmosSdk'
rpc_addr = 'https://rpc.cronos.org/'
event_source = { mode = 'push', url = 'wss://rpc.cronos.org/websocket', batch_delay = '500ms' }
grpc_addr = 'https://cronos-grpc.publicnode.com:443'
rpc_timeout = '10s'
account_prefix = 'crc'
key_name = 'keycronos'
key_store_type = 'Test'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 400000
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '5s'
max_block_time = '30s'
memo_prefix = ''
sequential_batch_tx = false

[chains.trust_threshold]
numerator = '1'
denominator = '3'

[chains.gas_price]
price = 0.1
denom = 'basecro'

[chains.packet_filter]
policy = 'allow'
list = [[
    'transfer',
    'channel-25',
]]

[chains.address_type]
derivation = 'cosmos'
```
</details>

***5. Manual Adjustments (if needed):*** If you're using custom nodes or non-default settings, update fields like rpc_addr, grpc_addr and gas parameters manually. One way to fine-tune gas settings based on transaction requirements is to reference recent IBC transactions on mintscan.io to estimate appropriate gas values.


### Health Check

Run these commands to validate your setup:

  1. Health Check:
  ```
  hermes health-check
  ```

  Expected Output:
  
  ```
  SUCCESS performed health check for all chains in the config
  ```

  2. Config validation:
  ```
  hermes config validate
  ```

  Expected output:

  ```
  SUCCESS "configuration is valid"
  ```

### Create the connection

***Note:*** It is crucial to confirm whether a connection already exists between the chains. Creating a duplicate connection can lead to unnecessary complications. If a connection exists, skip this section and proceed to the last step.

One way to check for a pre-existing connection is to run this command:

```
hermes query clients --host-chain cronosmainnet_25-1 --reference-chain osmosis-1
```

  1. Create New Clients (Only If No Existing Connection)

  - From Osmosis to Cronos:

  ```
  hermes create client --host-chain cronosmainnet_25-1 --reference-chain osmosis-1
  ```

  - From Cronos to Osmosis:

  ```
  hermes create client --host-chain osmosis-1 --reference-chain cronosmainnet_25-1
  ```

Expected output:
<details>
<summary>***Click to expand the SUCCESS response***</summary>
  
```rust
SUCCESS Connection {
    delay_period: 0ns,
    a_side: ConnectionSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "cronosmainnet_25-1",
                version: 1,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-255",
        ),
        connection_id: Some(
            ConnectionId(
                "connection-2525",
            ),
        ),
    },
    b_side: ConnectionSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "osmosis-1",
                version: 1,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-0",
        ),
        connection_id: Some(
            ConnectionId(
                "connection-0",
            ),
        ),
    },
}
```
  
</details>

***Important:*** Take note of the connection_id fields in the output. These identifiers will be required for the next steps.

### Create a channel

Once the connection is established, create a channel:

```
hermes create channel --a-chain cronosmainnet_25-1 --a-connection connection-2525 --a-port transfer --b-port transfer
```

A succesful output would look like this:

Expected output:
<details>
<summary>***Click to expand the SUCCESS response***</summary>

```rust
SUCCESS Channel {
    ordering: Unordered,
    a_side: ChannelSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "cronosmainnet_25-1",
                version: 1,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-255",
        ),
        connection_id: ConnectionId(
            "connection-2525",
        ),
        port_id: PortId(
            "transfer",
        ),
        channel_id: Some(
            ChannelId(
                "channel-25",
            ),
        ),
        version: None,
    },
    b_side: ChannelSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "osmosis-1",
                version: 1,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-0",
        ),
        connection_id: ConnectionId(
            "connection-0",
        ),
        port_id: PortId(
            "transfer",
        ),
        channel_id: Some(
            ChannelId(
                "channel-0",
            ),
        ),
        version: None,
    },
    connection_delay: 0ns,
}
}
```
</details>

***Note:*** Write down the channel_id fields. If a connection already existed, reuse the existing channels in your `config.toml` file.


### Start the relayer

  1. Create a log file to get the output of the relayer with this command

  ```
  sudo touch /var/log/hermes.log 
  ```

  2. Start Hermes

  ```
  hermes start &> hermes.log
  ```

### Congratulations!

You have successfully established an IBC connection between two chains. 🎉
