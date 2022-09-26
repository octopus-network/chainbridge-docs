# Running Locally
## Prerequisites

- Docker, docker-compose
- chainbridge binary (see [Install](installation.md))
- cb-sol-cli (see [README](https://github.com/octopus-network/chainbridge-deploy/blob/oct-dev/cb-sol-cli/README.md))

## Steps To Get Started
- [Running Locally](#running-locally)
  - [Prerequisites](#prerequisites)
  - [Steps To Get Started](#steps-to-get-started)
  - [Start Geth Chain](#start-geth-chain)
  - [Start subsrate chain](#start-subsrate-chain)
  - [Connect to PolkadotJS Portal](#connect-to-polkadotjs-portal)
  - [On-Chain Setup (Ethereum)](#on-chain-setup-ethereum)
    - [Deploy Contracts](#deploy-contracts)
    - [Register Resources Ethereum](#register-resources-ethereum)
      - [Generate You substrate native token resource id](#generate-you-substrate-native-token-resource-id)
    - [Specify Token Semantics](#specify-token-semantics)
  - [On-Chain Setup (Substrate)](#on-chain-setup-substrate)
    - [Register Relayers](#register-relayers)
    - [Register Resources Substrate](#register-resources-substrate)
    - [Whitelist Chains](#whitelist-chains)
  - [Run Relayer](#run-relayer)
  - [Fungible Transfers](#fungible-transfers)
    - [Substrate Native Token ⇒ ERC 20](#substrate-native-token--erc-20)
    - [ERC20 ⇒ Substrate Native Token](#erc20--substrate-native-token)
    - [New Erc20 Token =\> substrate](#new-erc20-token--substrate)
      - [部署一个新的合约代币为usdc](#部署一个新的合约代币为usdc)
      - [部署一个新的erc20handler合约](#部署一个新的erc20handler合约)
        - [Register Resources Ethereum 注册资源在以太坊](#register-resources-ethereum-注册资源在以太坊)
      - [Specify Token Semantics 指定令牌语义](#specify-token-semantics-指定令牌语义)
      - [On-Chain Setup (Substrate) 在substrate侧的配置](#on-chain-setup-substrate-在substrate侧的配置)
        - [创建跨链资产USDC](#创建跨链资产usdc)
          - [**Register Resources Substrate**](#register-resources-substrate-1)
        - [Set token id](#set-token-id)
        - [启动relayer为新的erc20handle](#启动relayer为新的erc20handle)
      - [**ERC20(USDC) ⇒ Substrate chain(usdc)**](#erc20usdc--substrate-chainusdc)
      - [substrate侧查看mint出来的跨链资产](#substrate侧查看mint出来的跨链资产)
      - [substrate chain （usdc) → Erc20 (usdc)](#substrate-chain-usdc--erc20-usdc)
  - [Non-Fungible Transfers](#non-fungible-transfers)
    - [Substrate NFT ⇒ ERC721](#substrate-nft--erc721)
    - [ERC721 ⇒ Substrate NFT](#erc721--substrate-nft)
  - [Generic Data Transfer](#generic-data-transfer)
    - [Generic Data Substrate ⇒ Eth](#generic-data-substrate--eth)

## Start Geth Chain

The easiest way to get started is to use the below docker-compose snippet. 

This will start one or two geth instance:
```yaml

version: '3'
services:
  geth1:
    image: "chainsafe/chainbridge-geth:20200505131100-5586a65"
    container_name: geth1
    ports:
      - "8545:8545"

  geth2:
    image: "chainsafe/chainbridge-geth:20200505131100-5586a65"
    container_name: geth2
    ports:
      - "8546:8545"
```

_Start Chains:_
```bash
docker-compose -f docker-compose-2-geth.yml up -V
```

(Use `-V` to always start with new chains. These instructions depend on deterministic Ethereum addresses, which are used as defaults implicitly by some of these commands. Avoid re-deploying the contracts without restarting both chains, or ensure to specify all the required parameters.)

## Start subsrate chain
```
git clone git@github.com:octopus-network/barnacle.git
git checkout feature/add-chainbridge-v1
cargo build --release
./target/release/appchain-barnacle   --dev  --tmp --alice --ws-external --rpc-external --enable-offchain-indexing true
```
## Connect to PolkadotJS Portal

1. Access the PolkadotJS Portal for Centrifuge, as an example Substrate chain, [here](https://portal.chain.centrifuge.io/)
2. Connect to your local Substrate chain:
    - Click the network in the top-left corner
    - Select the Development dropdown
    - Set `ws://localhost:9944` as the custom endpoint
    - Click `Switch` to connect

## On-Chain Setup (Ethereum)
### Deploy Contracts

To deploy the contracts on to the Ethereum chain, run the following:

_Deploy Contracts:_
```bash
cb-sol-cli deploy --all --relayerThreshold 1
```

After running, the expected output looks like this:
```bash
================================================================
Url:        http://localhost:8545
Deployer:   0xff93B45308FD417dF303D6515aB04D9e89a750Ca
Gas Limit:   8000000
Gas Price:   20000000
Deploy Cost: 0.0

Options
=======
Chain Id:    0
Threshold:   1
Relayers:    0xff93B45308FD417dF303D6515aB04D9e89a750Ca,0x8e0a907331554AF72563Bd8D43051C2E64Be5d35,0x24962717f8fA5BA3b931bACaF9ac03924EB475a0,0x148FfB2074A9e59eD58142822b3eB3fcBffb0cd7,0x4CEEf6139f00F9F4535Ad19640Ff7A0137708485
Bridge Fee:  0
Expiry:      100

Contract Addresses
================================================================
Bridge:             0x62877dDCd49aD22f5eDfc6ac108e9a4b5D2bD88B
----------------------------------------------------------------
Erc20 Handler:      0x3167776db165D8eA0f51790CA2bbf44Db5105ADF
----------------------------------------------------------------
Erc721 Handler:     0x3f709398808af36ADBA86ACC617FeB7F5B7B193E
----------------------------------------------------------------
Generic Handler:    0x2B6Ab4b880A45a07d83Cf4d664Df4Ab85705Bc07
----------------------------------------------------------------
Erc20:              0x21605f71845f372A9ed84253d2D024B7B10999f4
----------------------------------------------------------------
Erc721:             0xd7E33e1bbf65dC001A0Eb1552613106CD7e40C31
----------------------------------------------------------------
Centrifuge Asset:   Not Deployed
----------------------------------------------------------------
WETC:               Not Deployed
================================================================
```

### Register Resources Ethereum

#### Generate You substrate native token resource id

use [chainbridge-cli](https://github.com/octopus-network/chainbridge-cli) to generate:
```bash 
cargo run generate-resource-id 0 BAR
    Finished dev [unoptimized + debuginfo] target(s) in 0.56s
     Running `target/debug/chainbridge-cli generate-resource-id 0 BAR`
👉🏼👉🏼generate resource id: Start!
🎈🎈chain id: 0, token_name: BAR, generate resource id: 0x00000000000000000000000000000072721892618b7ea5114b9c71cdbbe3af00
🌈🌈generate resource id: Successfull!🌈🌈
```

* **NOTE:** The below registrations will **not** notify you upon successful completion.

_Register fungile resource ID with erc20 contract:_
```bash
cb-sol-cli bridge register-resource --resourceId "0x00000000000000000000000000000072721892618b7ea5114b9c71cdbbe3af00" --targetContract "0x21605f71845f372A9ed84253d2D024B7B10999f4"
```
_Register non-fungible resource ID with erc721 contract:_
```bash
cb-sol-cli bridge register-resource --resourceId "0x00000000000000000000000000000072721892618b7ea5114b9c71cdbbe3af00" --targetContract "0xd7E33e1bbf65dC001A0Eb1552613106CD7e40C31" --handler "0x3f709398808af36ADBA86ACC617FeB7F5B7B193E"
```
_Register generic resource ID:_
```bash
cb-sol-cli bridge register-generic-resource --resourceId "0x00000000000000000000000000000072721892618b7ea5114b9c71cdbbe3af00" --targetContract "0xc279648CE5cAa25B9bA753dAb0Dfef44A069BaF4" --handler "0x2B6Ab4b880A45a07d83Cf4d664Df4Ab85705Bc07" --hash --deposit "" --execute "store(bytes32)"
```

### Specify Token Semantics

To allow for a variety of use cases, the Ethereum contracts support both the `transfer` and the `mint/burn` ERC methods.

For simplicity's sake the following examples only make use of the  `mint/burn` method:

_Register the erc20 contract as mintable/burnable:_
```bash
cb-sol-cli bridge set-burn --tokenContract "0x21605f71845f372A9ed84253d2D024B7B10999f4"
```
_Register the associated handler as a minter:_
```bash
cb-sol-cli erc20 add-minter --minter "0x3167776db165D8eA0f51790CA2bbf44Db5105ADF"
```
_Register the erc721 contract as mintable/burnable:_
```bash
cb-sol-cli bridge set-burn --tokenContract "0xd7E33e1bbf65dC001A0Eb1552613106CD7e40C31" --handler "0x3f709398808af36ADBA86ACC617FeB7F5B7B193E"
```
_Add the handler as a minter:_
```bash
cb-sol-cli erc721 add-minter --minter "0x3f709398808af36ADBA86ACC617FeB7F5B7B193E"
```

## On-Chain Setup (Substrate)

### Register Relayers

First, we need to register the account of the relayer on Substrate (cb-sol-cli deploys contracts with the 5 test keys preloaded). 

Steps to register the relayers:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Choose the `addRelayer` method of `chainBridge`
3. Select **Alice** as the relayer `AccountId`

### Register Resources Substrate

Steps to register resources:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `chainBridge.setResource`, passing both the `Id` and `Method` listed below for each of the transfer types you wish to use

**Fungible (Native asset):**

Id: `0x000000000000000000000000000000c76ebe4a02bbc34786d860b355f5a5ce00`

Method: `0x436861696e4272696467655472616e736665722e7472616e73666572` (utf-8 encoding of "ChainBridgeTransfer.transfer")

**NonFungible(ERC721):**

Id: `0x000000000000000000000000000000e389d61c11e5fe32ec1735b3cd38c69501`

Method: `0x4572633732312e6d696e745f657263373231` (utf-8 encoding of "Erc721.mint_erc721")

**Generic (Hash Transfer):**

Id: `0x00000000000000000000000000000072721892618b7ea5114b9c71cdbbe3af00`

Method:  `0x436861696e4272696467655472616e736665722e72656d61726b` (utf-8 encoding of "ChainBridgeTransfer.remark")

### Whitelist Chains

Steps to whitelist chains:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `chainBridge.whitelistChain`, specifying `0` for the Ethereum chain ID

## Run Relayer

Steps to run a relayer:

1. Clone the [ChainBridge repository](https://github.com/octopus-network/ChainBridge)
2. Install the ChainBridge binary
3. Create `config.json` using the sample provided below as a starting point
4. Start relayer as a binary using the default "Alice" key

_Clone repo:_
```bash
git clone git@github.com:octopus-network/ChainBridge.git
```
_Build ChainBridge and move it to your GOBIN path:_
```bash
cd ChainBridge
git switch oct-dev
make install
```
_Run relayer_:
```bash
chainbridge --config config.json --testkey alice --verbosity trace --latest
```


Sample `config.json`:
```json
{
  "chains": [
    {
      "name": "eth",
      "type": "ethereum",
      "id": "0",
      "endpoint": "ws://localhost:8545",
      "from": "0xff93B45308FD417dF303D6515aB04D9e89a750Ca",
      "opts": {
        "bridge": "0x62877dDCd49aD22f5eDfc6ac108e9a4b5D2bD88B",
        "erc20Handler": "0x3167776db165D8eA0f51790CA2bbf44Db5105ADF",
        "erc721Handler": "0x3f709398808af36ADBA86ACC617FeB7F5B7B193E",
        "genericHandler": "0x2B6Ab4b880A45a07d83Cf4d664Df4Ab85705Bc07",
        "gasLimit": "1000000",
        "maxGasPrice": "20000000"
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
- This is an example config file for a single relayer ("Alice") using the contracts we've deployed.

## Fungible Transfers

### Substrate Native Token ⇒ ERC 20

Steps to transfer an ERC-20 token:

1. Select the `Extrinsics` tab in the PolkadotJS Portal
2. Call `ChainBridgeTransfer.transferNative` with parameters such as these:
    - Amount: `1000_000_000_000_000_000_000`
    - Recipient: `0xff93B45308FD417dF303D6515aB04D9e89a750Ca`
    - Dest Id: `0`

You can query the recipients balance on Ethereum with this:

_Query token balance of account: Oxff..750Ca_:
```bash
cb-sol-cli erc20 balance --address "0xff93B45308FD417dF303D6515aB04D9e89a750Ca"
```

### ERC20 ⇒ Substrate Native Token

If necessary, you can mint some tokens:

_Mint 1000 ERC20 tokens_:
```bash
cb-sol-cli erc20 mint --amount 1000
```

Before initiating the transfer we have to approve the bridge to take ownership of the tokens:

_Approve bridge to assume custody of tokens:_
```bash
cb-sol-cli erc20 approve --amount 1000 --recipient "0x3167776db165D8eA0f51790CA2bbf44Db5105ADF"
```

To initiate a transfer on the Ethereum chain use this command (Note: there will be a 10 block delay before the relayer will process the transfer):

_Transfer 1 token to account: 0xd4..da27d_:
```bash
cb-sol-cli erc20 deposit --amount 1 --dest 1 --recipient "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d" --resourceId "0x000000000000000000000000000000c76ebe4a02bbc34786d860b355f5a5ce00"
```

### New Erc20 Token => substrate 

#### 部署一个新的合约代币为usdc
```bash
cb-sol-cli deploy --erc20 --erc20Name "USDC stable Coin" --erc20Symbol USDC --erc20Decimals 18
```

generate result: 
```bash
Deploying contracts...
✓ ERC20 contract deployed

================================================================
Url:        http://localhost:8545
Deployer:   0xff93B45308FD417dF303D6515aB04D9e89a750Ca
Gas Limit:   8000000
Gas Price:   20000000
Deploy Cost: 0.0

Options
=======
Chain Id:    0
Threshold:   2
Relayers:    0xff93B45308FD417dF303D6515aB04D9e89a750Ca,0x8e0a907331554AF72563Bd8D43051C2E64Be5d35,0x24962717f8fA5BA3b931bACaF9ac03924EB475a0,0x148FfB2074A9e59eD58142822b3eB3fcBffb0cd7,0x4CEEf6139f00F9F4535Ad19640Ff7A0137708485
Bridge Fee:  0
Expiry:      100

Contract Addresses
================================================================
Bridge:             Not Deployed
----------------------------------------------------------------
Erc20 Handler:      Not Deployed
----------------------------------------------------------------
Erc721 Handler:     Not Deployed
----------------------------------------------------------------
Generic Handler:    Not Deployed
----------------------------------------------------------------
Erc20:              0xE86AA40918017a65c496d0d2D4E3b99d3473f999
----------------------------------------------------------------
Erc721:             Not Deployed
----------------------------------------------------------------
Centrifuge Asset:   Not Deployed
----------------------------------------------------------------
WETC:               Not Deployed
================================================================
```
#### 部署一个新的erc20handler合约
```bash 
cb-sol-cli deploy --bridgeAddress 0xe41419931199d9f8B4390fd03DaEF524379d6a3a --erc20Handler
```

generate the result:

```bash
Deploying contracts...
✓ ERC20Handler contract deployed

================================================================
Url:        http://localhost:8545
Deployer:   0xff93B45308FD417dF303D6515aB04D9e89a750Ca
Gas Limit:   8000000
Gas Price:   20000000
Deploy Cost: 0.0

Options
=======
Chain Id:    0
Threshold:   2
Relayers:    0xff93B45308FD417dF303D6515aB04D9e89a750Ca,0x8e0a907331554AF72563Bd8D43051C2E64Be5d35,0x24962717f8fA5BA3b931bACaF9ac03924EB475a0,0x148FfB2074A9e59eD58142822b3eB3fcBffb0cd7,0x4CEEf6139f00F9F4535Ad19640Ff7A0137708485
Bridge Fee:  0
Expiry:      100

Contract Addresses
================================================================
Bridge:             0x62877dDCd49aD22f5eDfc6ac108e9a4b5D2bD88B
----------------------------------------------------------------
Erc20 Handler:      0xB9045422E5a58B3C28cF5f7E5B3Ead224A35f25c
----------------------------------------------------------------
Erc721 Handler:     Not Deployed
----------------------------------------------------------------
Generic Handler:    Not Deployed
----------------------------------------------------------------
Erc20:              Not Deployed
----------------------------------------------------------------
Erc721:             Not Deployed
----------------------------------------------------------------
Centrifuge Asset:   Not Deployed
----------------------------------------------------------------
WETC:               Not Deployed
================================================================
```


##### Register Resources Ethereum 注册资源在以太坊
- **NOTE:** The below registrations will **not** notify you upon successful completion.

使用chainbridge-cli生成chainid：0，代币为BAR的resoureid
```bash 
cargo run generate-resource-id 0 USDC
    Finished dev [unoptimized + debuginfo] target(s) in 0.46s
     Running `target/debug/chainbridge-cli generate-resource-id 0 USDC`
👉🏼👉🏼generate resource id: Start!
🎈🎈chain id: 0, token_name: USDC, generate resource id: 0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800
🌈🌈generate resource id: Successfull!🌈🌈
```
*Register fungile resource ID with erc20 contract:*

注册fungible resource ID 和erc20 contract 关联：
```bash 
cb-sol-cli bridge register-resource  \
--bridge "0x62877dDCd49aD22f5eDfc6ac108e9a4b5D2bD88B" \
--handler "0xB9045422E5a58B3C28cF5f7E5B3Ead224A35f25c" \
--targetContract "0xE86AA40918017a65c496d0d2D4E3b99d3473f999" \
--resourceId "0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800"
```

#### Specify Token Semantics 指定令牌语义
To allow for a variety of use cases, the Ethereum contracts support both the `transfer` and the `mint/burn` ERC methods.

For simplicity's sake the following examples only make use of the `mint/burn` method:

*Register the erc20 contract as mintable/burnable:*
```bash 
cb-sol-cli bridge set-burn  \ 
--bridge "0x62877dDCd49aD22f5eDfc6ac108e9a4b5D2bD88B" \
--handler "0xB9045422E5a58B3C28cF5f7E5B3Ead224A35f25c" \
--tokenContract "0xE86AA40918017a65c496d0d2D4E3b99d3473f999"
```
Register the associated handler as a minter:
```bash 
cb-sol-cli erc20 add-minter --erc20Address "0xE86AA40918017a65c496d0d2D4E3b99d3473f999" --minter "0xB9045422E5a58B3C28cF5f7E5B3Ead224A35f25c"
```
#### On-Chain Setup (Substrate) 在substrate侧的配置
##### 创建跨链资产USDC

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Choose the forceCreate method of chainBridgeAssets
    1. id: 0, multiaddress: id, id: Alice, issufficient: true, minbalance: 1
3. choose the forceSetMetadata metod of chainBridgeAssets
    1. id: 0, name: 0x55534443, symbol: 0x55534443, decimals: 18, isFrozen: false

###### **Register Resources Substrate**

Steps to register resources:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `chainBridge.setResource`, passing both the `Id` and `Method` listed below for each of the transfer types you wish to use

**Fungible :**

Id: 0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800

Method: 0x436861696e4272696467655472616e736665722e7472616e73666572 (utf-8 encoding of "ChainBridgeTransfer.transfer")

##### Set token id

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Choose the setTokenId method of chainBridgeTransfer
    - resourceId: 0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800
    - tokenId: 0,
    - tokenName (USDC): 0x55534443

##### 启动relayer为新的erc20handle

 

Steps to run a relayer:

1. Clone the [ChainBridge repository](https://github.com/ChainSafe/ChainBridge)
2. Install the ChainBridge binary
3. Create `config.json` using the sample provided below as a starting point
4. Start relayer as a binary using the default "Alice" key

*Clone repo:*

`git clone git@github.com:octopus-network/ChainBridge.git`

*Build ChainBridge and move it to your GOBIN path:*

`cd ChainBridge && git switch oct-dev && make build`

*Run relayer*:

`./build/chainbridge --config scripts/configs/config3.json --testkey alice --verbosity trace --latest`

Sample `config.json`:
```json 
{
  "chains": [
    {
      "name": "eth",
      "type": "ethereum",
      "id": "0",
      "endpoint": "ws://localhost:8545",
      "from": "0xff93B45308FD417dF303D6515aB04D9e89a750Ca",
      "opts": {
        "bridge": "0xe41419931199d9f8B4390fd03DaEF524379d6a3a",
        "erc20Handler": "0x44D634ED74A00f07c093b78be1A9635a1A2dcC80",
        "erc721Handler": "",
        "genericHandler": "",
        "gasLimit": "1000000",
        "maxGasPrice": "20000000"
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
This is an example config file for a single relayer ("Alice") using the contracts we've deployed.

#### **ERC20(USDC) ⇒ Substrate chain(usdc)**
If necessary, you can mint some tokens:

*Mint 1000 ERC20 tokens*:
```bash 
cb-sol-cli erc20 mint --amount 1000 --erc20Address "0xE86AA40918017a65c496d0d2D4E3b99d3473f999"
```
查看账户的资产
```bash 
cb-sol-cli erc20 balance --address "0xff93B45308FD417dF303D6515aB04D9e89a750Ca" --erc20Address "0xE86AA40918017a65c496d0d2D4E3b99d3473f999"
```
Before initiating the transfer we have to approve the bridge to take ownership of the tokens:

*Approve bridge to assume custody of tokens:*
```bash
cb-sol-cli erc20 approve --amount 100 --recipient "0x7aEB732252fD6D23AE1afa83fC22ed3991F72012" --erc20Address "0xE86AA40918017a65c496d0d2D4E3b99d3473f999"
```
To initiate a transfer on the Ethereum chain use this command (Note: there will be a 10 block delay before the relayer will process the transfer):

*Transfer 1 token to account: 0xd4..da27d*:
```bash 
cb-sol-cli erc20 deposit \
--amount 100 --dest 1 \
--bridge "0x62877dDCd49aD22f5eDfc6ac108e9a4b5D2bD88B" \
--recipient "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d" \
--resourceId "0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800"
```
转移之后查看资产
```bash 
cb-sol-cli erc20 balance --address "0xff93B45308FD417dF303D6515aB04D9e89a750Ca" --erc20Address "0xE86AA40918017a65c496d0d2D4E3b99d3473f999"
```
#### substrate侧查看mint出来的跨链资产

Steps to transfer an ERC-20 token:

1. Select the Chainstate tab in the PolkadotJS Portal
2. 选择chainbridgeAssets的account方法
    1. id： 0
    2. accountid： alice

可以查看跨链过来的0号资产

#### substrate chain （usdc) → Erc20 (usdc)

Steps to transfer an ERC-20 token:

1. Select the `Extrinsics` tab in the PolkadotJS Portal
2. Call ChainBridgeTransfer.genericTokenTransfer with parameters such as these:
    - Amount: 100000000000000000000
    - resource_id: 0x000000000000000000000000000000167fcde18a3d8d4ba09b9b05d5c48ce800
    - Recipient: `0xff93B45308FD417dF303D6515aB04D9e89a750Ca`
    - Dest Id: `0`

可以通过ChainbridgeAssets中的account方法查看所有的资产已经全部销毁转移到了evm 侧


## Non-Fungible Transfers

### Substrate NFT ⇒ ERC721

First, you'll need to mint a token.

Steps to mint an ERC-721 token:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `erc721.mint` with parameters such as these:
    - Owner: `Alice`
    - TokenId: `1`
    - Metadata: `""`

Now the owner of the token can initiate a transfer.

Steps to transfer an ERC-721 token:

1. Select the `Sudo` tab in the PolkadotJS Portal
2. Call `ChainBridgeTransfer.transferErc721` with parameters such as these:
    - Recipient: `0xff93B45308FD417dF303D6515aB04D9e89a750Ca`
    - TokenId: `1`
    - DestId: `0`

You can query ownership of tokens on Ethereum with this:

_Query ownership of ERC721 with token ID: 1_:
```bash
cb-sol-cli erc721 owner --id 0x1
```

### ERC721 ⇒ Substrate NFT

If necessary, you can mint an ERC-721 token like this:

_Mint ERC721 with token ID: 99_:
```bash
cb-sol-cli erc721 mint --id 0x99
```

Before initiating the transfer, we must approve the bridge to take ownership of the tokens:

_Approve bridge to assume custody of ERC721 with token ID: 99_:
```bash
cb-sol-cli erc721 approve --id 0x99 --recipient "0x3f709398808af36ADBA86ACC617FeB7F5B7B193E"
```

Now we can initiate the transfer:

_Transfer ERC721 with token ID: 99 to account: 0xd4..da27d_:
```bash
cb-sol-cli erc721 deposit --id 0x99 --dest 1 --resourceId "0x000000000000000000000000000000e389d61c11e5fe32ec1735b3cd38c69501" --recipient "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"
```

## Generic Data Transfer

To demonstrate a possible use of the generic data transfer, we have a hash registry on Ethereum. We also have a method on the example Substrate chain to emit a hash inside an event, which we can trigger from Ethereum. 

### Generic Data Substrate ⇒ Eth

For this example we will transfer a 32 byte hash to a registry on Ethereum. 

Steps to transfer data to Ethereum:

1. Select the `Extrinsics` tab in the PolkadotJS Portal
2. Call `ChainBridgeTransfer.transferHash` with parameters such as these:
    - Hash: `0x699c776c7e6ce8e6d96d979b60e41135a13a2303ae1610c8d546f31f0c6dc730`
    - Dest ID: `0`

You can verify the transfer with this command:

_Verify transfer of hash_:
```bash
cb-sol-cli cent getHash --hash 0x699c776c7e6ce8e6d96d979b60e41135a13a2303ae1610c8d546f31f0c6dc730
```
