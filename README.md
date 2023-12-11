# Wencash Genesis Deposit Contract

This is a rewrite of Ethereum and Gnosis beacon chains deposit contracts, available here: 

- [Eth 2.0 deposit contract](https://github.com/ethereum/consensus-specs/blob/dev/solidity_deposit_contract/deposit_contract.sol)
- [GBC deposit contract](https://github.com/gnosischain/deposit-contract)

The following things were changed from the Ethereum beacon chain deposit contract:

- Upgraded to Solidity 0.8.22
- Deposit is made via ERC20 tokens (WEN) instead of native ETH
- Deposits can be done in batches (max size: 128)
- Deposits cannot be made after block 18909332 (~Monday January 1st 2024)

## About 

Wencash Genesis Deposit Contract is deployed on Ethereum: [0xCAE7A9d478Ea404210de26952015B8F5D3B4194A](https://etherscan.io/address/0xCAE7A9d478Ea404210de26952015B8F5D3B4194A#code)
This contract allows anyone to register Genesis Validators for the Wencash Blockchain.

To become a Genesis Validator, a participant must send the deposit data (using the `deposit` method):
- public key,
- withdrawal credentials,
- signature,
- amount,
- deposit data root.
 
This smart contract allows deposits until block #18,909,332 (Monday Jan 01 2024). After this delay, this contract is locked, it will be only used as a historical reference and all WEN in it will be forever locked. But the WEN deposit will be withdrawable on the Wencash Blockchain after its launch.

The `genesis.ssz` config file for the Wencash Consensus Layer will be generated out of this smart contract using the `get_deposit_root()` function and Genesis Validators will have their WEN balance on the Wencash Consensus Layer after the network start.

## Usage

**/!\ No GUI launchpad planned to make the genesis deposits. Recommended for advanced devs knowing what they are doing. The following instructions are unsafe as they do not perform safety checks on the inputs you will provide to the Smart Contract and might cause a loss up to 100% of assets./!\\**

There are several ways to make deposits to this contract (web3, ethers scripts, etc). The following instructions represent just an example:

1. Generate keys and `deposit_data-*.json` by using [Wencash Staking Deposit Cli](https://github.com/wencash/staking-deposit-cli) and by following the instructions there. Just note that the tool has to be launched absolutely with this option: `--devnet_chain_setting '{"network_name": "wencash", "genesis_fork_version": "88000000", "genesis_validator_root": "0000000000000000000000000000000000000000000000000000000000000000"}'`. The most important option here is the `"genesis_fork_version": "88000000"` used to create the signature and deposit data root and which will help to identify the future Wencash Beacon Chain. Here is an example of command to build the tool and generate your keys:

```sh
python3 -V
pip3 install virtualenv
virtualenv venv
make build_linux
./dist/deposit new-mnemonic --devnet_chain_setting '{"network_name": "wencash", "genesis_fork_version": "88000000", "genesis_validator_root": "0000000000000000000000000000000000000000000000000000000000000000"}' --execution_address '[YOUR_ETHEREUM_ADDRESS]'
```

2. Using [Remix](https://remix.ethereum.org/), interact with [Wencash Token Contract](https://etherscan.io/token/0xEBA6145367b33e9FB683358E0421E8b7337D435f#code) to call `approve` method allowing spending WEN for your ethereum address. Do the same for the Wencash Genesis Deposit Contract too (unlimited or not):

```code
approve(
address: [YOUR_ETHEREUM_ADDRESS]
value: 115792089237316195423570985008687907853269984665640564039457584007913129639935  (max possible value in wei for unlimited spending)
)
```

```code
approve(
address: 0xCAE7A9d478Ea404210de26952015B8F5D3B4194A     (Wencash Genesis Deposit Contract)
value: 115792089237316195423570985008687907853269984665640564039457584007913129639935  (max possible value in wei for unlimited spending)
)
```

3. Using [Remix](https://remix.ethereum.org/), interact with [Wencash Genesis Deposit Contract](https://etherscan.io/address/0xCAE7A9d478Ea404210de26952015B8F5D3B4194A#code) to call `deposit` method in order to make 32 WEN deposit for your future genesis validator with the data provided in your `deposit_data-*.json` (previously created in step 1):

```code
deposit(
pubkey: 0x[YOUR_PUBKEY]
withdrawal_credentials: 0x[YOUR_WITHDRAWAL_CREDENTIALS]
signature: 0x[YOUR_SIGNATURE]
deposit_data_root: 0x[YOUR_DEPOSIT_DATA_ROOT]
stake_amount: 32000000000000000000     (32 WEN in wei)
)
```

4. [**OPTIONAL**] Using [Remix](https://remix.ethereum.org/), interact with [Wencash Genesis Deposit Contract](https://etherscan.io/address/0xCAE7A9d478Ea404210de26952015B8F5D3B4194A#code) to call `batchDeposit` method in order to make 32 WEN to 4096 WEN deposits for your future genesis validators (1 to 128) with the data provided in your `deposit_data-*.json` (previously created in step 1). Again, there are several ways to help you generate the input arguments for `batchDeposit` method, here is an example:

```sh
cd ./validator_keys
jq '([.[].pubkey] | "0x" + join("")), "0x" + .[0].withdrawal_credentials, ([.[].signature] | "0x" + join("")), ["0x" + .[].deposit_data_root]' ./deposit_data-*.json > ./output.json
```

Then you can copy past (without the quotes except for the deposit data roots) the arguments previoulsy generated into the `batchDeposit` method in Remix GUI: 

```code
batchDeposit(
pubkeys: [YOUR_PUBKEYS]
withdrawal_credentials: [YOUR_WITHDRAWAL_CREDENTIALS]
signatures: [YOUR_SIGNATURES]
deposit_data_roots: [YOUR_DEPOSIT_DATA_ROOTS]
)
```

**NB**: **`batchDeposit` has been made in order to save you some gas fees if you want to make multiple deposits linked to ONE unique withdrawal credentials but the fees of this method remains quite high (if max size of 128 used). So set carefully your gas fees parameters in order to avoid spending a lot of ether for nothing (causing the running out of gas error).**



