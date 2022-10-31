# Solidity Cheatsheet

**Solidity Practice 📦 :**

[https://cryptozombies.io/en/course/](https://cryptozombies.io/en/course/)

### Function Types (Solidity)

```solidity
// Pure functions :
// Functions whose output depend on the parameters given not on the contract's variables

function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}

// View functions :
// Function which will access the resources other than the parameters and not modify them will have view property

function _generateRandomDna(string memory _str) private view returns(uint){}

// Public functions :
// Anyone can execute

// Private functions :
// Not anyone can execute, have special naming conventions (Eg. _printHEllo() starts with _ to represent a private function)
```

### Hash Functions

```solidity
// Ethereum has the hash function keccak256 built in, which is a version of SHA3. A hash function basically maps an input into a random 256-bit hexadecimal number. A slight change in the input will cause a large change in the hash.

Eg :

function _generateRandomDna(string memory _str) private view returns (uint) {
	uint rand = uint(keccak256(abi.encodePacked(_str))); // This function generates the hash and then typecast it as uint because the result we get is in hexadecimal form.
	return rand%dnaModulus;
}
```

### Events (Solidity)

```solidity
// Events : They are a way for your contract to communicate that something happened on the blockchain to your app front-end, which can be 'listening' for certain events and take action when they happen.

// event----------------------------------------------------------
event IntegersAdded(uint x, uint y, uint result);
// ---------------------------------------------------------------

// function-------------------------------------------------------
function add(uint _x, uint _y) public returns (uint) {
  uint result = _x + _y;
  // fire the event
  emit IntegersAdded(_x, _y, result);
  return result;
}
// ---------------------------------------------------------------

// js-implimentation----------------------------------------------
YourContract.IntegersAdded(function(error, result) {})
// ---------------------------------------------------------------
```

### Mappings

```solidity
// A mapping is essentially a key-value store for storing and looking up data. In the first example, the key is an address and the value is a uint, and in the second example the key is a uint and the value a string.

mapping(address => uint) public zombieToOwner;
mapping(address => uint) ownerZombieCount;
```

### msg.sender

```solidity
mapping (uint => address) public zombieToOwner;
mapping (address => uint) ownerZombieCount;

function _createZombie(string memory _name, uint _dna) private {
	uint id = zombies.push(Zombie(_name, _dna)) - 1;
	zombieToOwner[id] = msg.sender; // mapping id with address
	/*
		[
			0 => 0xjhdakejhdfkqejdhq3ie12398,
			1 => 0xhebwebcwjeh374rh432i38hrr
		]
	*/
	ownerZombieCount[msg.sender]++;
	/*
		[
			0xjhdakejhdfkqejdhq3ie12398 => 1,
			0xhebwebcwjeh374rh432i38hrr => 5
		]
	*/
	emit NewZombie(id, _name, _dna);
}
```

### Require

```solidity
// Require makes it so that the function will throw an error and stop executing if some condition is not true

function createRandomZombie(string memory _name) public {
	require(ownerZombieCount[msg.sender] == 0); // Throws error if condition not satisfied.
	uint randDna = _generateRandomDna(_name);
	_createZombie(_name, randDna);
}
```

### \***\*Inheritance\*\***

```solidity
// BabyDoge inherits from Doge. That means if you compile and deploy BabyDoge, it will have access to both catchphrase() and anotherCatchphrase() (and any other public functions we may define on Doge).

contract Doge {
  function catchphrase() public returns (string memory) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string memory) {
    return "Such Moon BabyDoge";
  }
}
```

### Storage & Memory

```solidity
// Storage refers to variables stored permanently on the blockchain. Memory variables are temporary, and are erased between external function calls to your contract. Think of it like your computer's hard disk vs RAM.

contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ Seems pretty straightforward, but solidity will give you a warning
    // telling you that you should explicitly declare `storage` or `memory` here.

    // So instead, you should declare with the `storage` keyword, like:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...in which case `mySandwich` is a pointer to `sandwiches[_index]`
    // in storage, and...
    mySandwich.status = "Eaten!";
    // ...this will permanently change `sandwiches[_index]` on the blockchain.

    // If you just want a copy, you can use `memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...in which case `anotherSandwich` will simply be a copy of the
    // data in memory, and...
    anotherSandwich.status = "Eaten!";
    // ...will just modify the temporary variable and have no effect
    // on `sandwiches[_index + 1]`. But you can do this:
    sandwiches[_index + 1] = anotherSandwich;
    // ...if you want to copy the changes back into blockchain storage.
  }
}
```

### Internal & External

```solidity
// **internal** is the same as private, except that it's also accessible to contracts that inherit from this contract. (Hey, that sounds like what we want here!).

// **external** is similar to public, except that these functions can ONLY be called outside the contract — they can't be called by other functions inside that contract. We'll talk about why you might want to use external vs public later.
```

### Interface (Using Another Third Party Contract)

```solidity
pragma solidity >=0.5.0 <0.6.0;

import "./zombiefactory.sol";

contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}

contract ZombieFeeding is ZombieFactory {

	// INTERFACE-ADDRESS
	// -----------------
  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  // Initialize kittyContract here using `ckAddress` from above
  KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

}
```

### \***\*Handling Multiple Return Values\*\***

```solidity
contract KittyInterface {
	// FUNCTION WITH 10 RETURN VALUES
	// ------------------------------
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}

function feedOnKitty(uint _zombieId, uint _kittyId) public {
	uint kittyDna;
	// We get the required value
	// -------------------------
	(,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
	feedAndMultiply(_zombieId, kittyDna);
}
```

### \***\*Ownable Contracts\*\***

```solidity
// So the Ownable contract basically does the following:

// When a contract is created, its constructor sets the owner to msg.sender (the person who deployed it)

// It adds an onlyOwner modifier, which can restrict access to certain functions to only the owner

// It allows you to transfer the contract to a new owner

// onlyOwner is such a common requirement for contracts that most Solidity DApps start with a copy/paste of this Ownable contract, and then their first contract inherits from it.

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol";

contract Sample is Ownable {}
```

[openzeppelin-contracts/Ownable.sol at master · OpenZeppelin/openzeppelin-contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol)

# Gas 💨

In Solidity, your users have to pay every time they execute a function on your DApp using a currency called **_gas_**. Users buy gas with Ether (the currency on Ethereum), so your users have to spend ETH in order to execute functions on your DApp.

How much gas is required to execute a function depends on how complex that function's logic is. Each individual operation has a **_gas cost_** based roughly on how much computing resources will be required to perform that operation (e.g. writing to storage is much more expensive than adding two integers). The total **_gas cost_** of your function is the sum of the gas costs of all its individual operations.

Because running functions costs real money for your users, code optimization is much more important in Ethereum than in other programming languages. If your code is sloppy, your users are going to have to pay a premium to execute your functions — and this could add up to millions of dollars in unnecessary fees across thousands of users.

### \***\*Struct packing to save gas\*\***

In Lesson 1, we mentioned that there are other types of `uint`s: `uint8`, `uint16`, `uint32`, etc.

Normally there's no benefit to using these sub-types because Solidity reserves 256 bits of storage regardless of the `uint` size. For example, using `uint8` instead of `uint` (`uint256`) won't save you any gas.

But there's an exception to this: inside `struct`s.

If you have multiple `uint`s inside a struct, using a smaller-sized `uint` when possible will allow Solidity to pack these variables together to take up less storage. For example:

```solidity
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` will cost less gas than `normal` because of struct packing
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30);
```

### \***\*Time Units\*\***

Solidity provides some native units for dealing with time.

The variable `now` will return the current unix timestamp of the latest block (the number of seconds that have passed since January 1st 1970). The unix time as I write this is `1515527488`.

Solidity also contains the time units `seconds`, `minutes`, `hours`, `days`, `weeks`and `years`. These will convert to a `uint` of the number of seconds in that length of time. So `1 minutes` is `60`, `1 hours` is `3600` (60 seconds x 60 minutes), `1 days` is `86400` (24 hours x 60 minutes x 60 seconds), etc.

EG:

```solidity
uint lastUpdated;

// Set `lastUpdated` to `now`
function updateTimestamp() public {
  lastUpdated = now;
}

// Will return `true` if 5 minutes have passed since `updateTimestamp` was
// called, `false` if 5 minutes have not passed
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```

### Modifiers

Modifiers can also take parameters

```solidity
modifier aboveLevel(uint _level, uint _zombieId) {
	require(zombies[_zombieId].level >= _level);
	_;
}

// USAGE
function changeName(uint _zombieId, string calldata _newName) external aboveLevel(2, _zombieId) {
    require(msg.sender == zombieToOwner[_zombieId]);
    zombies[_zombieId].name = _newName;
}
```

### \***\*Saving Gas With 'View' Functions\*\***

This function will only need to read data from the blockchain, so we can make it a `view`
 function.

`view` functions don't cost any gas when they're called externally by a user.

```solidity
function getZombiesByOwner(address _owner) external view returns(uint[] memory) {}
```

- `storage` function is an expensive function in solidity.
- To iterate through the arrays we can use for loop, and to iterate through the mapping it is not possible now whenever we may need something to be using less gas we use Array with loops, The mapping approach may look tempting in vision but it is not the best approach in somecases

## Payable Function Modifier

`payable` functions are part of what makes Solidity and Ethereum so cool — they are a special type of function that can receive Ether.

Let that sink in for a minute. When you call an API function on a normal web server, you can't send US dollars along with your function call — nor can you send Bitcoin.

But in Ethereum, because both the money (_Ether_), the data (_transaction payload_), and the contract code itself all live on Ethereum, it's possible for you to call a function **and** pay money to the contract at the same time.

```solidity
function levelUp(uint _zombieId) external payable {
    require(msg.value == 0.001 ether);
    zombies[_zombieId].level++;
}
```

### \***\*Withdraws\*\***

After you send Ether to a contract, it gets stored in the contract's Ethereum account, and it will be trapped there — unless you add a function to withdraw the Ether from the contract.

You can write a function to withdraw Ether from the contract as follows:

```solidity
function withdraw() external onlyOwner{
    address payable _owner = address(uint160(owner()));
    _owner.transfer(address(this).balance);
}
```

- First we need to convert our address from uint160 to address payable.
- address(this).balance gives the current balance of the wallet address.

### \***\*Random Numbers\*\***

[https://cryptozombies.io/en/lesson/4/chapter/4](https://cryptozombies.io/en/lesson/4/chapter/4)

Using keccak256 for random number generation is vulnerable to attack. One can run the algo locally before broadcasting the change to all the nodes on the ethereum network and then send the desired output needed.

```solidity
function randMod(uint _modulus) internal returns(uint) {
    randNonce++;
    return uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % _modulus;
}
```

# \***\*ERC721 Standard, Multiple Inheritance (NFT\*\*** 🎴\***\*)\*\***

```solidity
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

  function balanceOf(address _owner) external view returns (uint256);
  function ownerOf(uint256 _tokenId) external view returns (address);
  function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
  function approve(address _approved, uint256 _tokenId) external payable;
}
```

This is a ERC721 token contract with some function.

```solidity
pragma solidity >=0.5.0 <0.6.0;

import "./zombieattack.sol";
import "./erc721.sol"; // Importing ERC721

contract ZombieOwnership is ZombieAttack, ERC721 {}
```

Now we will be using the functions and firing the events from ERC721

```solidity
pragma solidity >=0.5.0 <0.6.0;

import "./zombieattack.sol";
import "./erc721.sol";

contract ZombieOwnership is ZombieAttack, ERC721 {

  mapping (uint => address) zombieApprovals;

  function balanceOf(address _owner) external view returns (uint256) {
    return ownerZombieCount[_owner];
  }

  function ownerOf(uint256 _tokenId) external view returns (address) {
    return zombieToOwner[_tokenId];
  }

  function _transfer(address _from, address _to, uint256 _tokenId) private {
    ownerZombieCount[_to]++;
    ownerZombieCount[_from]--;
    zombieToOwner[_tokenId] = _to;
    emit Transfer(_from, _to, _tokenId);
  }

  function transferFrom(address _from, address _to, uint256 _tokenId) external payable {
    require (zombieToOwner[_tokenId] == msg.sender || zombieApprovals[_tokenId] == msg.sender);
    _transfer(_from, _to, _tokenId);
  }

  function approve(address _approved, uint256 _tokenId) external payable onlyOwnerOf(_tokenId) {
    zombieApprovals[_tokenId] = _approved;
    emit Approval(msg.sender, _approved, _tokenId);
  }
}
```

### \***\*Preventing Overflows\*\***

We use SafeMath Library which works something as :

```solidity
function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
}
```

We can use it something like :

```solidity
// TO USE
using SafeMath for uint256;
// Declare using SafeMath32 for uint32
using SafeMath32 for uint32;
// Declare using SafeMath16 for uint16
using SafeMath16 for uint16;

ownerZombieCount[_to] = ownerZombieCount[_to].add(1);
```

### Comments

We have different Syntaxes for Comments

```solidity
/// @title A contract for basic math operations
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice For now, this contract just adds a multiply function
contract Math {
  /// @notice Multiplies 2 numbers together
  /// @param x the first uint.
  /// @param y the second uint.
  /// @return z the product of (x * y)
  /// @dev This function does not currently check for overflows
  function multiply(uint x, uint y) returns (uint z) {
    // This is just a normal comment, and won't get picked up by natspec
    z = x * y;
  }
}

// COMMENT TYPES:
// -------------
// @title
// @author
// @notice
// @dev
```
