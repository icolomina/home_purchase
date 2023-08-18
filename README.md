# home_purchase
This soroban smart contract contains the rules to manage a house purchase between a buyer and a seller.

## Build the contract

- Install Soroban and Rust SDK following the instructions [here](https://soroban.stellar.org/docs/getting-started/setup)
- Test the contract. To do it, execute the following command from the contract root folder:

```
cargo test
```
This will compile and execute tests. After checking they are successfull, you can generate the wasm file
  
- To generate the wasm file, execute the following from the contract root folder

```shell
soroban contract build
```

After that, you will be able to deploy and install it. You can see how to use soroban-cli to deploy contracts in the [docs](https://soroban.stellar.org/docs/getting-started/hello-world)

## Contract functions

- 
