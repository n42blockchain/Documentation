# **N42 Name Service (ENS) Interface Technical Documentation**

## **1. Overview**
The **N42 Name Service (N42-NS)** is a decentralized naming system built on the N42 blockchain. Similar to Ethereum's ENS, N42-NS maps **human-readable names** (e.g., `alice.n42`) to various types of resources such as **addresses, smart contracts, content hashes, metadata, or identity credentials**.

By integrating **N42-NS**, users and developers benefit from simplified identity, better UX, and seamless interoperability across DApps, wallets, and domains. This documentation details the **architecture**, **API specification**, **contract functions**, and **security mechanisms** of the N42 ENS interface.

---

## **2. Key Features**

- **Name-to-Address Resolution**: Map human-readable `.n42` names to N42 addresses.
- **Reverse Resolution**: Retrieve the name associated with a given N42 address.
- **Multitype Record Support**: Map names to content hashes (IPFS/Arweave), ABI, public keys, social handles, etc.
- **Subdomain Management**: Support for hierarchical naming (e.g., `team.alice.n42`).
- **Ownership & Permissions**: Each name is an NFT representing ownership, transferrable and upgradable.
- **Decentralized Registry**: On-chain registry with tamper-proof history and expiration controls.
- **Integration with N42 Wallets & Browsers**: Seamless user resolution and navigation.

---

## **3. Architecture**

```
          +----------------+
          |  User/DApp     |
          +-------+--------+
                  |
           ENS Interface (API / Contract Call)
                  |
          +-------v--------+
          |  Resolver      | <-------+
          +-------+--------+         |
                  |                  |
          +-------v--------+         |
          |  ENS Registry  |         |
          +----------------+         |
                                     |
                  +-----------------+
                  |  Reverse Resolver
                  +-----------------+
```

- **ENS Registry**: Stores ownership and resolver address of each name.
- **Resolver**: Stores the actual records (e.g., address, content hash).
- **Reverse Resolver**: Used to find the ENS name linked to an address.

---

## **4. Core Smart Contracts**

### **4.1 ENSRegistry**

Stores the mapping of names to:
- Owner
- Resolver
- TTL
- Expiry

```solidity
function owner(bytes32 node) external view returns (address);
function resolver(bytes32 node) external view returns (address);
function setOwner(bytes32 node, address owner) external;
function setResolver(bytes32 node, address resolver) external;
function setSubnodeOwner(bytes32 node, bytes32 label, address owner) external returns (bytes32);
```

### **4.2 PublicResolver**

Handles record resolution.

```solidity
function addr(bytes32 node) external view returns (address);
function setAddr(bytes32 node, address addr) external;

function contenthash(bytes32 node) external view returns (bytes memory);
function setContenthash(bytes32 node, bytes calldata hash) external;

function text(bytes32 node, string calldata key) external view returns (string memory);
function setText(bytes32 node, string calldata key, string calldata value) external;
```

### **4.3 ReverseRegistrar**

Allows reverse resolution (address → name).

```solidity
function setName(string memory name) external returns (bytes32);
function node(address addr) public pure returns (bytes32);
```

---

## **5. API Specification**

### **5.1 Register a New ENS Name**
```json
POST /api/v1/ens/register
{
  "name": "alice.n42",
  "owner": "0x123...",
  "resolver": "0xResolverAddress...",
  "duration": 31536000,
  "signature": "0x..."
}
```

**Response**
```json
{
  "status": "success",
  "node": "0xf0d1e3...",
  "expires": 1789343200
}
```

---

### **5.2 Resolve Name to Address**
```http
GET /api/v1/ens/resolve?name=alice.n42
```

**Response**
```json
{
  "name": "alice.n42",
  "address": "0x123456..."
}
```

---

### **5.3 Set Reverse Record (Address to Name)**
```json
POST /api/v1/ens/reverse
{
  "address": "0x123...",
  "name": "alice.n42",
  "signature": "0x..."
}
```

---

### **5.4 Get Text Record (e.g., social handle)**
```http
GET /api/v1/ens/text?name=alice.n42&key=twitter
```

**Response**
```json
{
  "twitter": "@alice_web3"
}
```

---

## **6. Record Types Supported**

| **Type**       | **Description**                                |
|----------------|------------------------------------------------|
| Address        | Maps to an N42 account address (EVM compatible)|
| Contenthash    | Maps to IPFS/Arweave hash (for websites, NFTs) |
| Text           | Generic key-value record (e.g., email, Twitter)|
| ABI            | Smart contract interface specification          |
| Public Key     | Encryption and verification public keys         |
| DNS Records    | (future support) domain interoperability        |

---

## **7. Security Design**

- **Name Ownership = NFT**: Every name is an ERC-721 token; users can transfer and trade names.
- **Signature Verification**: All write operations must be signed by the name owner.
- **Time-to-Live (TTL)**: Allows resolvers to cache results safely.
- **Name Expiry Protection**: Expired names can be reclaimed unless renewed.

---

## **8. Integration Guide**

### **8.1 In Smart Contracts (Solidity)**
```solidity
ENSRegistry registry = ENSRegistry(0xRegistryAddress);
bytes32 node = keccak256(abi.encodePacked(baseNode, keccak256("alice")));
address resolverAddr = registry.resolver(node);
address resolved = Resolver(resolverAddr).addr(node);
```

### **8.2 In Frontend (JavaScript/Web3.js)**
```javascript
const ens = new web3.eth.Contract(ensAbi, ENSRegistryAddress);
const namehash = web3.utils.namehash('alice.n42');
const resolverAddress = await ens.methods.resolver(namehash).call();
const resolver = new web3.eth.Contract(resolverAbi, resolverAddress);
const address = await resolver.methods.addr(namehash).call();
```

---

## **9. Future Extensions**

- **Name Auctions & Bidding**
- **Name Leasing and Subdomain Rentals**
- **Integration with N42 Social Graph**
- **Cross-chain ENS Resolution (via ZK Bridge)**

---

## **10. Conclusion**
N42 Name Service (N42-NS) provides a **secure, user-friendly, and decentralized identity layer** for the N42 ecosystem. It enables **simplified blockchain access, user-centric DApp integration, and universal resource mapping**, making it a key infrastructure component for a fully decentralized internet.

For integration, see source:  
🔗 [GitHub - N42 ENS Contracts](https://github.com/n42blockchain)
