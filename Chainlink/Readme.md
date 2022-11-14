# Chainlink Tutorial & Oracles

**This is a resource for chainlink contract interaction and importing the functionalities in our contract.**

Now, let's suppose you're building a DeFi dapp, and want to give your users the ability to withdraw ETH worth a certain amount of USD. To fulfill this request, your smart contract (for simplicity's sake we'll call it the "caller contract" from here onwards) must know how much one Ether is worth.

And here's the thing: a JavaScript application can easily fetch this kind of information, making requests to the Binance public API (or any other service that publicly provides a price feed). But, a smart contract can't directly access data from the outside world.

Now we could build a JavaScript application ourselves, but then we are introducing a centralized point of failure! We also couldn't just pull from the Binance API, because that would be a centralized point of failure!

So what we want to do, is get our data from both a **decentralized oracle network (DON)** and **decentralized data sources**.

Chainlink is a framework for decentralized oracle networks (DONs), and is a way to get data in from multiple sources across multiple oracles. This DON aggregates data in a decentralized manner and places it on the blockchain in a smart contract (often referred to as a "price reference feed" or "data feed") for us to read from. So all we have to do, is read from a contract that the Chainlink network is constantly updating for us!

For Eg:

Decentralised Data Feeds

[Decentralized Data Feeds | Chainlink](https://data.chain.link/)

Using Data Feed by importing

```solidity
pragma solidity ^0.6.7;

import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";

contract PriceConsumerV3 {
  AggregatorV3Interface public priceFeed;

  constructor() public {
    priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
  }

  function getLatestPrice() public view returns (int) {
    (,int price,,,) = priceFeed.latestRoundData();
    return price;
  }

  function getDecimals() public view returns (uint8) {
    uint8 decimals = priceFeed.decimals();
    return decimals;
  }
}
```

### Generating Randomness

---

the Chainlink VRF follows the basic request model, so we need to define:

1. A function to request the random number
2. A function for the the Chainlink node to return the random number to.

Remember, the Chainlink node is actually going to call the VRF Coordinator first to verify the number is random, then the VRF Coordinator will be the one to call our `ZombieFactory` contract.

Since we are importing the `VRFConsumerbase` contract, we can use the two built in functions that do both of these!

A. `requestRandomness`

1. This function checks to see that our contract has `LINK` tokens to pay a Chainlink node
2. Then, it sends some `LINK` tokens to the Chainlink node
3. Emits an event that the Chainlink node is looking for
4. Assigns a `requestId` to our request for a random number on-chain

B. `fulfillRandomness`

1. The Chainlink node first calls a function on the VRF Coordinator and includes a random number
2. The VRF Coordinator checks to see if the number is random
3. Then, it returns the random number the Chainlink node created, along with the original requestID from our request

```solidity
pragma solidity ^0.6.6;
import "@chainlink/contracts/src/v0.6/VRFConsumerBase.sol";

contract ZombieFactory is VRFConsumerbase {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    bytes32 public keyHash;
    uint256 public fee;
    uint256 public randomResult;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    constructor() VRFConsumerBase(
        0xb3dCcb4Cf7a26f6cf6B120Cf5A73875B7BBc655B, // VRF Coordinator
        0x01BE23585060835E02B77ef475b0Cc51aA1e0709  // LINK Token
    ) public{
        keyHash = 0x2ed0feb3e7fd2022120aa84fab1945545a9f2ffc9076fd6156fa96eaff4c1311;
        fee = 100000000000000000;

    }

    function _createZombie(string memory _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    }

    function getRandomNumber() public returns (bytes32 requestId) {
        return requestRandomness(keyHash, fee);
    }

    function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
        randomResult = randomness;
    }

}
```
