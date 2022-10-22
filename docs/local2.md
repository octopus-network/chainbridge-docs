# Evm chain ↔ Substrate chain 配置使用指南

## 前置条件

- chainbridge `v1.2.4` binary (see [README](https://github.com/chainsafe/chainbridge/#building))
- cb-sol-cli (see [README](https://github.com/ChainSafe/chainbridge-deploy/tree/master/cb-sol-cli#cb-sol-cli-documentation))

## 1. 启动本地链

## 启动substrate chain

```bash
git clone git@github.com:octopus-network/barnacle.git
git checkout feature/add-chainbridge-v1
cargo build --release
rm -rf .tao
./target/release/appchain-barnacle   --dev  -d .tao --alice --ws-external --rpc-external --enable-offchain-indexing true
```

### 连接polkadotjs

1. Access the PolkadotJS Portal for Centrifuge, as an example Substrate chain, [here](https://portal.chain.centrifuge.io/)
2. Connect to your local Substrate chain:
    - Click the network in the top-left corner
    - Select the Development dropdown
    - Set `ws://localhost:9944` as the custom endpoint
    - Click `Switch` to connect
    

****On-Chain Setup (Ethereum) Evm侧配置****

### **Deploy Contracts 部署合约**

To deploy the contracts on to the Ethereum chain, run the following:

要将合同部署到以太坊链上，请运行以下内容：

### Generate Resource Id

使用chainbridge-cli生成chainid：0，代币为BAR的resoureid

```bash
cargo run generate-resource-id 0 BAR
🎈🎈chain id: 0, token_name: BAR, generate resource id: 0x00000000000000000000000000000072721892618b7ea5114b9c71cdbbe3af00
```

### Set Env

```bash
export RESOURCE_ID="0x00000000000000000000000000000072721892618b7ea5114b9c71cdbbe3af00"
export GN_RESOURCE_ID="0x000000000000000000000000000000c76ebe4a02bbc34786d860b355f5a5ce00"
export SRC_GATEWAY="http://127.0.0.1:7545/"
export SRC_ADDR="0xAbB3F12B6282B680084f76d57992b343Ba384842"
export SRC_PK="0x2dcb2186b235b3a57e23de018e6c88758b0f52008e37a716fc8be74fbb85aac0"
```

- Set `SRC_GATEWAY` to an endpoint of testnet/mainnet to test accordingly.
- `SRC_ADDR`/ `SRC_PK` must have sufficient balance for testing.

### deploy bridge contract

使用默认的relayers ["0xff93B45308FD417dF303D6515aB04D9e89a750Ca","0x8e0a907331554AF72563Bd8D43051C2E64Be5d35","0x24962717f8fA5BA3b931bACaF9ac03924EB475a0","0x148FfB2074A9e59eD58142822b3eB3fcBffb0cd7","0x4CEEf6139f00F9F4535Ad19640Ff7A0137708485"]

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 deploy \
    --bridge --erc20Handler \
    --relayerThreshold 1 \
    --chainId 0 \
    --erc20 \
    --erc20Name "Bar Coin" \
    --erc20Symbol Bar \
    --erc20Decimals 18
```

```bash
Contract Addresses
================================================================
Bridge:             0xF1C9B5a7D223D811f5D0125bD83e7c6034A40EA5
----------------------------------------------------------------
Erc20 Handler:      0x609A88c8f71306BfEBaE0CA424eBb587c85DA069
----------------------------------------------------------------
Erc721 Handler:     Not Deployed
----------------------------------------------------------------
Generic Handler:    Not Deployed
----------------------------------------------------------------
Erc20:              0xcc5d33C6d43D22bb5F232c31aDdF144B8DE7dde0
```

- Set the env accordingly

```bash
export SRC_BRIDGE=0xF1C9B5a7D223D811f5D0125bD83e7c6034A40EA5
export SRC_HANDLER=0x609A88c8f71306BfEBaE0CA424eBb587c85DA069
export SRC_TOKEN=0xcc5d33C6d43D22bb5F232c31aDdF144B8DE7dde0
```

### Deploy Generic Handler

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 deploy \
		--bridgeAddress $SRC_BRIDGE
    --genericHandler \
    --relayerThreshold 1 \
    --chainId 0

# Generic Handler:    0xD82EB76a09b8af49AfB936BA9F760b1a2606D028
export SRC_GN_HANDLER=0xD82EB76a09b8af49AfB936BA9F760b1a2606D028
```

### Dploy Centriguge Asset for Generic Data Transfer

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 deploy \
    --bridgeAddress $SRC_BRIDGE \
    --centAsset \
    --relayerThreshold 1 \
    --chainId 0

# Generic Handler:    0x672a4B60BF28Fe34ED26c5466de8782AAA9983b5
export CENT_ASSET=0x672a4B60BF28Fe34ED26c5466de8782AAA9983b5
```

### **Register Resources Ethereum 注册资源在以太坊**

- **NOTE:** The below registrations will **not** notify you upon successful completion.

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 bridge register-resource \
    --bridge $SRC_BRIDGE \
    --handler $SRC_HANDLER \
    --resourceId $RESOURCE_ID \
    --targetContract $SRC_TOKEN

cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 bridge register-generic-resource \
    --bridge $SRC_BRIDGE \
    --handler $SRC_GN_HANDLER \
    --resourceId $GN_RESOURCE_ID
```

### **Specify Token Semantics 指定令牌语义**

To allow for a variety of use cases, the Ethereum contracts support both the `transfer` and the `mint/burn` ERC methods.

为了允许各种用例，以太坊合同支持转移和薄荷/燃烧ERC方法。

*Register the erc20 contract as mintable/burnable:*

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 bridge set-burn \
    --bridge $SRC_BRIDGE \
    --handler $SRC_HANDLER \
    --tokenContract $SRC_TOKEN
```

*Register the associated handler as a minter:*

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 erc20 add-minter \
    --minter $SRC_HANDLER \
    --erc20Address $SRC_TOKEN
```

****On-Chain Setup (Substrate) 在substrate侧的配置****

### **Register Relayers**

First, we need to register the account of the relayer on Substrate (cb-sol-cli deploys contracts with the 5 test keys preloaded).

Steps to register the relayers:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Choose the `addRelayer` method of `chainBridge`
3. Select **Alice** as the relayer `AccountId`

### **Register Resources Substrate**

Steps to register resources: ERC20 

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `chainBridge.setResource`, passing both the `Id` and `Method` listed below for each of the transfer types you wish to use

**Fungible (Native asset):**

Id: 0x00000000000000000000000000000072721892618b7ea5114b9c71cdbbe3af00

Method: 0x436861696e4272696467655472616e736665722e7472616e73666572 (utf-8 encoding of "ChainBridgeTransfer.transfer")

Steps to register resources: Generic data 

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `chainBridge.setResource`, passing both the `Id` and `Method` listed below for each of the transfer types you wish to use

**Fungible (Generic Data):**

Id: 0x000000000000000000000000000000c76ebe4a02bbc34786d860b355f5a5ce00

Method: 0x436861696e4272696467655472616e736665722e72656d61726b (utf-8 encoding of "ChainBridgeTransfer.remark")

### 

### **Whitelist Chains**

Steps to whitelist chains:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `chainBridge.whitelistChain`, specifying `0` for the Ethereum chain ID

## **Run Relayer**

Steps to run a relayer:

1. Clone the [https://github.com/octopus-network/ChainBridge/tree/oct-dev](https://github.com/octopus-network/ChainBridge/tree/oct-dev)
2. Install the ChainBridge binary
3. Create `config.json` using the sample provided below as a starting point
4. Start relayer as a binary using the default "Alice" key

*Clone repo:*

```bash
git clone git@github.com:octopus-network/ChainBridge.git --branch oct-dev --single-branch chainbridge-oct
```

*Build ChainBridge and move it to your GOBIN path:*

```bash
cd ChainBridge && make build
```

*Run relayer*:

```bash
./build/chainbridge --config config.json --testkey alice --verbosity trace --latest
```

Sample `config.json`:

```bash
{
  "chains": [
    {
      "name": "Goerli",
      "type": "ethereum",
      "id": "0",
      "endpoint": "ws://127.0.0.1:7545",
      "from": "0xAbB3F12B6282B680084f76d57992b343Ba384842",
      "opts": {
        "bridge": "0xF1C9B5a7D223D811f5D0125bD83e7c6034A40EA5",
        "erc20Handler": "0x609A88c8f71306BfEBaE0CA424eBb587c85DA069",
        "genericHandler": "0x609A88c8f71306BfEBaE0CA424eBb587c85DA069",
        "gasLimit": "1000000",
        "maxGasPrice": "10000000000",
        "blockConfirmations": "51"
      }
    },
    {
      "name": "sub",
      "type": "substrate",
      "id": "1",
      "endpoint": "ws://localhost:9944",
      "from": "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY",
      "opts": {
          "useExtendedCall":"true"
      }
    }
  ]
}
```

- This is an example config file for a single relayer ("Alice") using the contracts we've deployed. "Alice" must have sufficient balance for testing.
- Customize according to your environment.

## **Fungible Transfers**

### **Substrate Native Token ⇒ ERC 20**

Steps to transfer an ERC-20 token:

1. Select the `Extrinsics` tab in the PolkadotJS Portal
2. Call ChainBridgeTransfer.transferNative with parameters such as these:
    - Amount: `500000000000000000000`
    - Recipient: `0xff93B45308FD417dF303D6515aB04D9e89a750Ca`
    - Dest Id: `0`

You can query the recipient's balance on Ethereum with this:

*Query token balance of account: Oxff..750Ca*:

```bash
cb-sol-cli --url $SRC_GATEWAY erc20 balance --address "0xff93B45308FD417dF303D6515aB04D9e89a750Ca" --erc20Address "you native token erc20 address"
```

### **ERC20 ⇒ Substrate Native Token**

If necessary, you can mint some tokens:

*Mint 1000 ERC20 tokens*:

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 erc20 mint \
    --amount 999 \
    --erc20Address $SRC_TOKEN
```

Before initiating the transfer we have to approve the bridge to take ownership of the tokens:

*Approve bridge to assume custody of tokens:*

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 erc20 approve \
    --amount 9000000000000000000 \
    --erc20Address $SRC_TOKEN \
    --recipient $SRC_HANDLER
```

To initiate a transfer on the Ethereum chain use this command (Note: there will be a few block delay before the relayer will process the transfer):

*Transfer 100 token to account: 0xd4..da27d*:

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 erc20 deposit \
    --amount 100 \
    --dest 1 \
    --bridge $SRC_BRIDGE \
    --recipient "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d" \
    --resourceId $RESOURCE_ID
```

## Evm Chain erc20 token  → substrate chian

### 部署一个新的合约代币为usdc

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 deploy \
    --erc20 \
    --erc20Name "USDC Stable Coin" \
    --erc20Symbol USDC\
    --erc20Decimals 18
```

```bash
# Erc20:              0x4d57a26f6D552F6eBC6cFB201db550bF821Fe615
# Set Env
export SRC2_TOKEN=0x4d57a26f6D552F6eBC6cFB201db550bF821Fe615
```

### **Register Resources Ethereum 注册资源在以太坊**

使用chainbridge-cli生成chainid：0，代币为BAR的resoureid

```bash
cargo run generate-resource-id 0 USDC
🎈🎈chain id: 0, token_name: USDC, generate resource id: 0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800

# Set Env
export RESOURCE2_ID=0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800
```

*Register fungile resource ID with erc20 contract:*

注册fungible resource ID 和erc20 contract 关联：

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 bridge register-resource \
    --bridge $SRC_BRIDGE \
    --handler $SRC_HANDLER \
    --resourceId $RESOURCE2_ID \
    --targetContract $SRC2_TOKEN
```

****On-Chain Setup (Substrate) 在substrate侧的配置****

### 创建跨链资产USDC

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Choose the forceCreate method of chainBridgeAssets
    1. id: 0, multiaddress: id, id: Alice, issufficient: true, minbalance: 1
3. choose the forceSetMetadata metod of chainBridgeAssets
    1. id: 0, name: 0x55534443, symbol: 0x55534443, decimals: 18, isFrozen: false
    

### **Register Resources Substrate**

Steps to register resources:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `chainBridge.setResource`, passing both the `Id` and `Method` listed below for each of the transfer types you wish to use

**Fungible :**

Id: 0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800

Method: 0x436861696e4272696467655472616e736665722e7472616e73666572 (utf-8 encoding of "ChainBridgeTransfer.transfer")

## Set token id

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Choose the setTokenId method of chainBridgeTransfer
    - resourceId: 0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800
    - tokenId: 0,
    - tokenName (USDC): 0x55534443
- This is an example config file for a single relayer ("Alice") using the contracts we've deployed.

### **ERC20(USDC) ⇒ Substrate chain(usdc)**

If necessary, you can mint some tokens:

*Mint 1000 ERC20 tokens*:

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 erc20 mint \
    --amount 999 \
    --erc20Address $SRC2_TOKEN
```

查看账户的资产

```bash
cb-sol-cli --url $SRC_GATEWAY erc20 balance --address $SRC_ADDR --erc20Address $SRC2_TOKEN
```

Before initiating the transfer we have to approve the bridge to take ownership of the tokens:

*Approve bridge to assume custody of tokens:*

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 erc20 approve \
    --amount 9000000000000000000 \
    --erc20Address $SRC2_TOKEN \
    --recipient $SRC_HANDLER
```

To initiate a transfer on the Ethereum chain use this command (Note: there will be a few block delay before the relayer will process the transfer):

*Transfer 1 token to account: 0xd4..da27d*:

```bash
cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 erc20 deposit \
    --amount 100 \
    --dest 1 \
    --bridge $SRC_BRIDGE \
    --recipient "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d" \
    --resourceId $RESOURCE2_ID

cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 generic deposit \
    --dest 1 \
    --bridge $SRC_BRIDGE \
    --recipient "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d" \
    --resourceId $RESOURCE_GN_ID
```

转移之后查看资产

```bash
cb-sol-cli --url $SRC_GATEWAY erc20 balance --address $SRC_ADDR --erc20Address $SRC2_TOKEN
```

### substrate侧查看mint出来的跨链资产

Steps to transfer an ERC-20 token:

1. Select the “Developer” → ”Chain State” tab in the PolkadotJS Portal
2. 选择chainbridgeAssets的account方法
    1. id： 0
    2. accountid： alice
    

可以查看跨链过来的0号资产

## substrate chain （usdc) → Erc20 (usdc)

Steps to transfer an ERC-20 token:

1. Select the `Extrinsics` tab in the PolkadotJS Portal
2. Call ChainBridgeTransfer.genericTokenTransfer with parameters such as these:
    - Amount: 100000000000000000000
    - resource_id: 0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800
    - Recipient: `0xff93B45308FD417dF303D6515aB04D9e89a750Ca`
    - Dest Id: `0`

可以通过ChainbridgeAssets中的account方法查看所有的资产已经全部销毁转移到了evm 侧

## generic data transfer  EVM → substrate

```bash
node index.js --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 generic deposit     \
    --dest 1     --bridge $SRC_BRIDGE     \
    --recipient "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"     \
    --resourceId $GN_RESOURCE_ID --data 0xc0ffee
```

## generic data transfer substrate → EVM

register resourceId on EVM

```bash
export GN_RESOURCE2_ID=0x000000000000000000000000000000f44be64d2de895454c3467021928e55e00
# cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 bridge register-generic-resource \
#     --bridge $SRC_BRIDGE \
#     --handler $SRC_GN_HANDLER \
#     --resourceId $GN_RESOURCE2_ID

cb-sol-cli --url $SRC_GATEWAY --privateKey $SRC_PK --gasPrice 10000000000 bridge register-generic-resource \
    --bridge $SRC_BRIDGE \
    --handler $SRC_GN_HANDLER \
    --resourceId $GN_RESOURCE2_ID \
    --targetContract $CENT_ASSET \
    --hash \
    --deposit "" \
    --execute "store(bytes32)"
```

For this example we will transfer a 32 byte hash to a registry on Ethereum.

Steps to transfer data to Ethereum:

1. Select the `Extrinsics` tab in the PolkadotJS Portal
2. Call `ChainBridgeTransfer.transferHash` with parameters such as these:
    - Hash: `0x699c776c7e6ce8e6d96d979b60e41135a13a2303ae1610c8d546f31f0c6dc730`
    - Dest ID: `0`

*Verify transfer of hash*

```bash
cb-sol-cli --url $SRC_GATEWAY cent getHash --address $CENT_ASSET  --hash 0x699c776c7e6ce8e6d96d979b60e41135a13a2303ae1610c8d546f31f0c6dc730
```