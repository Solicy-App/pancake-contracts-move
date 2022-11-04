# aptos-contracts
Aptos contracts

## Dependence
Aptos CLI

## How to use

1. Initialize your aptos account
```shell
$ aptos  init
```
you will get a ".aptos" folder in your current folder.
```shell
config.yaml
profiles:
  default:
    private_key: "0x0000000000000000000000000000000000000000000000000000000000000000"
    public_key: "0x0000000000000000000000000000000000000000000000000000000000000000"
    account: 3add3576f7f3f411a5bd5fbab22dff4747107f25ce8726bf9926542718ff8a26   # your_original_account
    rest_url: "https://fullnode.devnet.aptoslabs.com/v1"
    faucet_url: "https://faucet.devnet.aptoslabs.com/"
```
2. Get test APT
```shell
$ aptos account  fund-with-faucet --account your_original_account --amount 100000000
```
3. Create your resource account
```shell
$ aptos move run --function-id '0x1::resource_account::create_resource_account_and_fund' --args 'string:any string you want' 'hex:your_original_account' 'u64:10000000'
```
4. Get your resourc eaccount 
```shell
$ aptos account list --account your_original_account
```

Or find it on explorer: https://explorer.devnet.aptos.dev/account/your_original_account

```txt
TYPE:
0x1::resource_account::Container
DATA:
{
  "store": {
    "data": [
      {
        "key": "0x929ac1ea533d04f7d98c234722b40c229c3adb1838b27590d2237261c8d52b68",
        "value": {
          "account": "0x929ac1ea533d04f7d98c234722b40c229c3adb1838b27590d2237261c8d52b68"  # your_resource_account
        }
      }
    ]
  }
}
```
5. Replace your_original_account with your_resource_account in config.yaml


6. Edit Move.toml file

  ```shell
[package]
name = "pancake-swap"
version = "0.0.1"
[dependencies]
AptosFramework = { git = "https://github.com/aptos-labs/aptos-core.git", subdir = "aptos-move/framework/aptos-framework/", rev = "72421d32d77f1877ded478e96f5b95914de1df91" }
AptosStdlib = { git = "https://github.com/aptos-labs/aptos-core.git", subdir = "aptos-move/framework/aptos-stdlib/", rev = "72421d32d77f1877ded478e96f5b95914de1df91" }
[addresses]
pancake = "71e609393d30dfacaf477c9a9cd7824ae14b5f8d2a20c0b1917325d41e4a4aac" //repalce this with your_resource_account 
dev = "2b627a16b744c004fee98620f99479f6446494cebd57d2a523e2e6cdb3ee8dc5" // repalce this with your_original_account which you created the resource account 
zero = "0000000000000000000000000000000000000000000000000000000000000000"
default_admin = "2b627a16b744c004fee98620f99479f6446494cebd57d2a523e2e6cdb3ee8dc5" // need to create an admin account, and replace this.
``` 
7. Compile code
```shell
$ aptos move compile --package-dir .\TestCoin\
```
8. Publish package
```shell
$ aptos move publish --package-dir .\TestCoin\
```

9. Change account back to original in config.yaml

10. Set the `TestCoin` address value to the original account in `./TestCoin/Move.toml`. And publish the TestCoin module
```shell
$ aptos move publish --package-dir .\TestCoin\
```

11. At this point you should be able to see the modules in `pancake-swap` package under resource account, and `TestCoin` module under original account in the block explorer or CLI (as shown in the step 6).

### Testing

12. Initialize `TestCoinsV1`:
```shell
$ aptos move run --function-id original_account_hex::TestCoinsV1::initialize
```

13. Mint test USDT (I tested with original_account, but you can mint tokens to any account):
```shell
$ aptos move run --function-id original_account_hex::TestCoinsV1::mint_coin --args address:original_account_hex u64:20000000000000000 --type-args original_account_hex::TestCoinsV1::USDT
```

14. Mint test BTC (I tested with original_account, but you can mint tokens to any account):
```shell
$ aptos move run --function-id original_account_hex::TestCoinsV1::mint_coin --args address:original_account_hex u64:20000000000000000 --type-args original_account_hex::TestCoinsV1::BTC
```

15. Create swap pair.
```shell
$ aptos move run --function-id resource_account_hex::router::create_pair --type-args original_account_hex::TestCoinsV1::BTC original_account_hex::TestCoinsV1::USDT
```

16. Add liquidity to the pair created.
```shell
$ aptos move run --function-id resource_account_hex::router::add_liquidity --args u64:10000 u64:5000 u64:5000 u64:2500 --type-args original_account_hex::TestCoinsV1::BTC original_account_hex::TestCoinsV1::USDT
```

17. Try to swap max 10000 BTC for 100 ETH. (If will need more BTC the transaction reverts)
```shell
$ aptos move run --function-id resource_account_hex::router::swap_exact_output --args u64:100 u64:10000 --type-args original_account_hex::TestCoinsV1::BTC original_account_hex::TestCoinsV1::USDT
```