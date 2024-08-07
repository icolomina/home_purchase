# Deprecated
This repository has been deprecated. New repository is: https://github.com/icolomina/soroban-contracts

# home_purchase
This soroban smart contract contains the rules to manage a house purchase between a buyer and a seller.

## Build the contract

- Install Soroban and Rust SDK following the instructions [here](https://soroban.stellar.org/docs/getting-started/setup)
- Test the contract. To do it, execute the following command from the contract root folder:

```
cargo test
```
This will compile and execute tests. After checking they are successful, you can generate the wasm file
  
- To generate the wasm file, execute the following from the contract root folder

```shell
soroban contract build
```

After that, you will be able to deploy and install it. You can see how to use soroban-cli to deploy contracts in the [docs](https://soroban.stellar.org/docs/getting-started/hello-world). 
Before deploying the contract you will need to create a user. You can do it using "soroban config":

```shell
soroban config identity generate --global <your_user_name>
```
The above simple command will create a user. If you want to see the user's public key you can execute:

```shell
soroban config identity address <your_user_name>
```
For showing the private key you can use:

```shell
soroban config identity show <your_user_name>
```
When you have the user ready, add a network using the following command:

```shell
soroban config network add --global futurenet --rpc-url https://rpc-futurenet.stellar.org:443 --network-passphrase "Test SDF Future Network ; October 2022"
```

And now, you can deploy as follows:

```shell
soroban contract deploy --wasm target/wasm32-unknown-unknown/release/<your_wams_file>.wasm --source the_owner --network futurenet
```

## Contract functions

- **initialize**: Init the contract data:
  - If the first payment amount is equal or greater than de amount an error is returned
  - If the contract has already been initialized an error is returned
 
- **transfer_first_payment**: Transfer the first payment from the seller to the buyer. If the contract has not been initialized an error is returned.
- **seller_propose_meeting**: After the seller receives the first payment, He or She proposes a meeting date to the buyer. The contract emits an event with the meeting date so the buyer can query it and accept or reject it. If the seller tries to propose a meeting date before having received the first payment, an error is returned-
  - If there is already a meeting date established, an error is returned
  - If the meeting date proposed is lower or equal than the current date, an error is returned
 
- **buyer_review_meeting**: After reading the events and reviewing meeting date, the buyer can accept or reject it. If the buyer rejects it, the seller can propose another date
- **transfer_rest_of_payment**: Buyer trasfers the rest of the payment to the seller. This cannot be executed before the meeting date. If so, an error is returned. After the rest of the payment is transferred, the contract changes the proprietary key from seller to buyer

## Authentication

This contract does not use authentication so when the owner signs the transaction to invoke the functions they will be executed with no extra checkings. This means that the users who use the contract trust the platform which is invoking it. If we want to add more authentication, for instance, to ensure only the buyer can execute **transfer_first_payment** and **transfer_rest_of_payment** we could add the following line in the functions code:

### Modification in transfer_first_payment

```rust
pub fn transfer_first_payment(env: Env) -> Result<bool, Error> {
  if let Some(purchase_trading) = get_purchase_trading(&env) {
    purchase_trading.buyer.require_auth();
    let tk = token::Client::new(&env, &purchase_trading.token);
    tk.transfer(&purchase_trading.buyer, &purchase_trading.seller, &purchase_trading.first_payment);
    set_state_as_first_payment_sent(&env);
    return Ok(true);
  } else {
    return Err(Error::PurchaseDataNotStored);
  }
}
```
In this case, if we get *puchase_trading* var then we add line *purchase_trading.buyer.require_auth();* to ensure only buyer can execute this

### Modification in transfer_rest_of_payment

```rust
pub fn transfer_rest_of_payment(env: Env) -> Result<i128, Error> {
   if let Some(mta) = get_meeting_accepted(&env) {
      let current_ts = env.ledger().timestamp();
      if !mta.is_meeting_taking_place(current_ts)  {
         return Err(Error::CannotTransferAmountBeforeMeeting);
      }
   } else {
      return Err(Error::MeetingNotAcceptedYet);
   }

   let purchase_trading: PurchaseTrading = get_purchase_trading(&env).unwrap(); 
   purchase_trading.buyer.require_auth();

   // rest of the code .....  
}
```

In this case, after getting purchase_trading we also add *purchase_trading.buyer.require_auth();*.

## The token

As you can see in the code, the **initialize** method receives and store the token address by which the transfers will be made. This means that a Soroban token (those contracts which follow the [TokenInterface](https://soroban.stellar.org/docs/reference/interfaces/token-interface)) must be deployed so we can get a contract id and send it to the **initialize** function. Furthermore, the buyer and the seller addresses must be minted with tokens.

As a token, the [soroban token example](https://github.com/stellar/soroban-examples/tree/v0.9.2/token) can be used.

## Tutorial

You can read [this tutorial](https://dev.to/icolomina/creating-a-dapp-using-php-and-a-house-purchase-soroban-smart-contract-38f1) which shows how you could use the [php stellar sdk](https://github.com/Soneso/stellar-php-sdk/blob/main/soroban.md) to build a dapp using php and interact with this contract.
