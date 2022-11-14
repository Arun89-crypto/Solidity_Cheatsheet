# Truffle Tutorial

**FOLDER STRUCTURE**

```
├── build
  ├── contracts
      ├── Migrations.json
      ├── CryptoZombies.json
      ├── erc721.json
      ├── ownable.json
      ├── safemath.json
      ├── zombieattack.json
      ├── zombiefactory.json
      ├── zombiefeeding.json
      ├── zombiehelper.json
      ├── zombieownership.json
├── contracts
  ├── Migrations.sol
  ├── CryptoZombies.sol
  ├── erc721.sol
  ├── ownable.sol
  ├── safemath.sol
  ├── zombieattack.sol
  ├── zombiefactory.sol
  ├── zombiefeeding.sol
  ├── zombiehelper.sol
  ├── zombieownership.sol
├── migrations
└── test
. package-lock.json
. truffle-config.js
. truffle.js
```

Simple test code in JavaScript

```jsx
contract("MyAwesomeContract", (accounts) => {
  it("should be able to receive Ethers", () => {});
});
```

### Real World Example

Suppose there is a contract called createRandomZombie

```solidity
function createRandomZombie(string _name) public {
   require(ownerZombieCount[msg.sender] == 0);
   uint randDna = _generateRandomDna(_name);
   randDna = randDna - randDna % 100;
   _createZombie(_name, randDna);
 }
```

Usually, every test has the following phases:

1. **_set up_**: in which we define the initial state and initialize the inputs.
2. **_act_**: where we actually test the code. Always make sure you *test only one thing*.
3. **_assert_:** where we check the results.

Lets look at what our test should do in some more detail.

```solidity
const CryptoZombies = artifacts.require("CryptoZombies");
const zombieNames = ["Zombie 1", "Zombie 2"];
contract("CryptoZombies", (accounts) => {
    let [alice, bob] = accounts;

    it("should be able to create a new zombie", async () => {
        const contractInstance = await CryptoZombies.new();
        const result = await contractInstance.createRandomZombie(zombieNames[0], {from: alice});
        assert.equal(result.receipt.status, true);
        assert.equal(result.logs[0].args.name,zombieNames[0]);
    })

})
```

## **The context function**

To group tests, *Truffle* provides a function called `context`. Let me quickly show you how use it in order to better structure our code:

```solidity
context("with the single-step transfer scenario", async () => {
    it("should transfer a zombie", async () => {
      // TODO: Test the single-step transfer scenario.
    })
})

context("with the two-step transfer scenario", async () => {
    it("should approve and then transfer a zombie when the approved address calls transferFrom", async () => {
      // TODO: Test the two-step scenario.  The approved address calls transferFrom
    })
    it("should approve and then transfer a zombie when the owner calls transferFrom", async () => {
        // TODO: Test the two-step scenario.  The owner calls transferFrom
     })
})

```

If we add this to our `CryptoZombies.js` file and then run `truffle test` the output would look similar to this:

```
Contract: CryptoZombies
    ✓ should be able to create a new zombie (100ms)
    ✓ should not allow two zombies (251ms)
    with the single-step transfer scenario
      ✓ should transfer a zombie
    with the two-step transfer scenario
      ✓ should approve and then transfer a zombie when the owner calls transferFrom
      ✓ should approve and then transfer a zombie when the approved address calls transferFrom

  5 passing (2s)

```

Well?

Hmm...

Take a look again - there's an issue with the above output. It looks like all tests have passed which is obviously false since we didn't even write them yet!!

Fortunately, there's an easy solution- if we just place an `x` in front of the `context()` functions as follows: `xcontext()`, `Truffle` will skip those tests.

## **Chai Assertion Library**

`Chai` is very powerful and, for the scope of this lesson, we're just going to scratch the surface. Once you've finished this lesson, feel free check out [their guides](https://www.chaijs.com/guide/) to further your knowledge.

That said, let's take a look at the three kinds of assertion styles bundled into `Chai`:

- _expect_: lets you chain natural language assertions as follows:
  ```jsx
  let lessonTitle = "Testing Smart Contracts with Truffle";
  expect(lessonTitle).to.be.a("string");
  ```
- _should_: allows for similar assertions as `expect` interface, but the chain starts with a `should` property:
  ```jsx
  let lessonTitle = "Testing Smart Contracts with Truffle";
  lessonTitle.should.be.a("string");
  ```
- _assert_: provides a notation similar to that packaged with node.js and includes several additional tests and it's browser compatible:
  ```jsx
  let lessonTitle = "Testing Smart Contracts with Truffle";
  assert.typeOf(lessonTitle, "string");
  ```

In this chapter, we are going to show you how to improve your assertions with `expect`.

# Truffle Config File

```jsx
const HDWalletProvider = require("truffle-hdwallet-provider");
const LoomTruffleProvider = require("loom-truffle-provider");
const mnemonic = "YOUR MNEMONIC HERE";
module.exports = {
  // Object with configuration for each network
  networks: {
    //development
    development: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*",
      gas: 9500000,
    },
    // Configuration for Ethereum Mainnet
    mainnet: {
      provider: function () {
        return new HDWalletProvider(
          mnemonic,
          "https://mainnet.infura.io/v3/<YOUR_INFURA_API_KEY>"
        );
      },
      network_id: "1", // Match any network id
    },
    // Configuration for Rinkeby Metwork
    rinkeby: {
      provider: function () {
        // Setting the provider with the Infura Rinkeby address and Token
        return new HDWalletProvider(
          mnemonic,
          "https://rinkeby.infura.io/v3/<YOUR_INFURA_API_KEY>"
        );
      },
      network_id: 4,
    },
    // Configuration for Loom Testnet
    loom_testnet: {
      provider: function () {
        const privateKey = "YOUR_PRIVATE_KEY";
        const chainId = "extdev-plasma-us1";
        const writeUrl = "wss://extdev-basechain-us1.dappchains.com/websocket";
        const readUrl = "wss://extdev-basechain-us1.dappchains.com/queryws";
        // TODO: Replace the line below
        const loomTruffleProvider = new LoomTruffleProvider(
          chainId,
          writeUrl,
          readUrl,
          privateKey
        );
        loomTruffleProvider.createExtraAccountsFromMnemonic(mnemonic, 10);
        return loomTruffleProvider;
      },
      network_id: "9545242630824",
    },
  },
  compilers: {
    solc: {
      version: "0.4.25",
    },
  },
};
```
