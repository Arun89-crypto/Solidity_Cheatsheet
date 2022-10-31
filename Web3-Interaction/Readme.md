# Web3️⃣ Interaction

---

Declaring a contract

```jsx
cryptoZombies = new web3js.eth.Contract(cryptoZombiesABI, cryptoZombiesAddress);
```

## **Call function and Send function**

### Call

`call` is used for `view` and `pure` functions. It only runs on the local node, and won't create a transaction on the blockchain.

> Review: view and pure functions are read-only and don't change state on the blockchain. They also don't cost any gas, and the user won't be prompted to sign a transaction with MetaMask.

Using Web3.js, you would `call` a function named `myMethod` with the parameter `123` as follows:

```
myContract.methods.myMethod(123).call()
```

### Send

`send` will create a transaction and change data on the blockchain. You'll need to use `send` for any functions that aren't `view` or `pure`.

> Note: sending a transaction will require the user to pay gas, and will pop up their Metamask to prompt them to sign a transaction. When we use Metamask as our web3 provider, this all happens automatically when we call send(), and we don't need to do anything special in our code. Pretty cool!

Using Web3.js, you would `send` a transaction calling a function named `myMethod` with the parameter `123` as follows:

```
myContract.methods.myMethod(123).send()
```

Payable Functions (Send)

```jsx
cryptoZombies.methods
  .createRandomZombie(name)
  .send({ from: userAccount })
  .on("receipt", function (receipt) {
    $("#txStatus").text("Successfully created " + name + "!");
    // Transaction was accepted into the blockchain, let's redraw the UI
    getZombiesByOwner(userAccount).then(displayZombies);
  });
```

### \***\*Subscribing to Events\*\***

```solidity
// EVENT in smart contract
event NewZombie(uint zombieId, string name, uint dna);
```

```jsx
// Listening to EVENT in js
cryptoZombies.events
  .NewZombie()
  .on("data", function (event) {
    let zombie = event.returnValues;
    console.log(
      "A new zombie was born!",
      zombie.zombieId,
      zombie.name,
      zombie.dna
    );
  })
  .on("error", console.error);
```

### \***\*Using indexed\*\***

---

In order to filter events and only listen for changes related to the current user, our Solidity contract would have to use the `indexed` keyword, like we did in the `Transfer` event of our ERC721 implementation:

```solidity
event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
```

In this case, because `_from` and `_to` are `indexed`, that means we can filter for them in our event listener in our front end:

```jsx
// Use `filter` to only fire this code when `_to` equals `userAccount`
cryptoZombies.events
  .Transfer({ filter: { _to: userAccount } })
  .on("data", function (event) {
    let data = event.returnValues;
    // The current user just received a zombie!
    // Do something here to update the UI to show it
  })
  .on("error", console.error);
```

As you can see, using `event`s and `indexed` fields can be quite a useful practice for listening to changes to your contract and reflecting them in your app's front-end.

### **Querying past events**

We can even query past events using `getPastEvents`, and use the filters `fromBlock` and `toBlock` to give Solidity a time range for the event logs ("block" in this case referring to the Ethereum block number):

```jsx
cryptoZombies
  .getPastEvents("NewZombie", { fromBlock: 0, toBlock: "latest" })
  .then(function (events) {
    // `events` is an array of `event` objects that we can iterate, like we did above
    // This code will get us a list of every zombie that was ever created
  });
```
