# Clover Accounts Binding

Clover provides an account binding feature. You can bind your Clover address to a Clover EVM address. Once you bind the accounts,  you can use one account to interact with both EVM-based blockchains as well as Substrate-based blockchains. 

Here are the features of Clover accounts binding \(suppose you have done the accounts binding, for example bind your Clover address: **5DyCG7icXFuq8WtLd7iLi3umczKf84SAvp7w7Rw44RFEU8Yf** with EVM address: **0xe6206C7f064c7d77C6d8e3eD8601c9AA435419cE**\)

1. The above addresses will have the same balance. Other people can send CLV to any of the above account, and you can receive the CLV.  At the same time, you can send CLV to other people using any of the above account.
2. If you only have CLV in **0xe6206C7f064c7d77C6d8e3eD8601c9AA435419cE**,  you can't deploy or interact with Clover smart contract. Once you bind the account with a EVM address, you are able to deploy and interact with Clover smart contract.

## How to Bind the Account

1. Using Clover Wallet, the version should be above 1.0.3
2. Using the following sample code：

```text
const API = require("@polkadot/api")
const Web3 = require("web3")
const web3 = new Web3("https://rpc-2.clover.finance")

const PUB_KEY = "0xe6206C7f064c7d77C6d8e3eD8601c9AA435419cE"
const PRIV_KEY = "0xa504b64992e478a6846670237a68ad89b6e42e90de0490273e28e74f084c03c8"
const CLOVER_SEEDS = "your 12 seed words"

async function run() {
    const wsProvider = new API.WsProvider('wss://api-2.clover.finance');
    const api = await API.ApiPromise.create({
        provider: wsProvider,
        types: {
            Amount: 'i128',
            Keys: 'SessionKeys3',
            AmountOf: 'Amount',
            Balance: 'u128',
            Rate: 'FixedU128',
            Ratio: 'FixedU128',
            EcdsaSignature: '[u8; 65]',
            EvmAddress: 'H160',
        },
    });
    const keyring = new API.Keyring({ type: 'sr25519' });
    const alice = keyring.addFromUri(CLOVER_SEEDS);
    let nonce = await api.rpc.system.accountNextIndex(alice.address);
    web3.eth.accounts.wallet.add(PRIV_KEY);
    let signature = await web3.eth.sign(`clover evm:${web3.utils.bytesToHex(alice.publicKey).slice(2)}`, PUB_KEY);

    await api.tx.evmAccounts
        .claimAccount(PUB_KEY, web3.utils.hexToBytes(signature))
        .signAndSend(alice, {
            nonce,
        }, ({ events = [], status }) => {
            if (status.isFinalized) {
                console.log(`${alice.address} has bound with EVM address: ${PUB_KEY}`)
            }
        });
}

run()
```

 





