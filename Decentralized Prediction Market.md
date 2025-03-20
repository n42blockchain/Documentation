# **N42 Public Chain Prediction Market Interface Design**

## **1. Overview of Prediction Markets**
As a **high-performance, EVM-compatible public chain**, N42 supports **decentralized prediction markets**, allowing users to bet on **events, financial markets, sports, politics, and more**, with smart contracts ensuring fair transactions.

### **Key Features:**
✅ **Smart Contract-Driven**: Fully decentralized, executed on the blockchain  
✅ **EVM Compatibility**: Can leverage existing Ethereum-based prediction markets (e.g., Augur, Polymarket)  
✅ **Low Gas Fees**: Suitable for large-scale user participation  
✅ **Oracle Support**: Fetches results via **on-chain oracles**  

---

## **2. Core Interfaces of the Prediction Market**
### **(1) Market Creation**
Users or project teams can create new prediction markets and provide bettable event options.

📌 **Example: Who will win the 2024 U.S. Presidential Election?**
- Option 1️⃣: Candidate A  
- Option 2️⃣: Candidate B  
- Option 3️⃣: Others  

#### **Create Prediction Market**
```solidity
function createMarket(
    string memory _eventName,    // Event name
    string[] memory _outcomes,   // Possible outcomes
    uint256 _endTimestamp,       // Market closing time
    address _oracle              // Oracle address
) external returns (uint256 marketId);
```
📌 **Returns: `marketId` (market ID) for future reference**  

---

### **(2) Placing a Bet**
Users can place bets on a market option during the open period by staking their funds.

#### **User Betting**
```solidity
function placeBet(
    uint256 _marketId,   // Prediction market ID
    uint256 _outcomeId,  // Selected outcome ID
    uint256 _amount      // Bet amount
) external;
```
📌 **Once the bet is placed, funds are locked in the smart contract until the event concludes**  

---

### **(3) Querying Market Information**
#### **Get Market Details**
```solidity
function getMarketInfo(uint256 _marketId) external view returns (
    string memory eventName,
    string[] memory outcomes,
    uint256 totalPool,
    uint256 endTimestamp,
    bool isResolved
);
```
📌 **Used to display prediction market details on the frontend**  

#### **Get Total Bet Pool for an Outcome**
```solidity
function getOutcomePool(uint256 _marketId, uint256 _outcomeId) external view returns (uint256);
```
📌 **Displays the total amount bet on each outcome**  

---

### **(4) Market Settlement**
After the event ends, the prediction market requires an **oracle to fetch the result** and distribute funds accordingly.

#### **Oracle Reporting the Final Outcome**
```solidity
function reportOutcome(
    uint256 _marketId,
    uint256 _winningOutcome
) external onlyOracle;
```
📌 **Only the designated oracle address can call this function, ensuring a fair market settlement**  

#### **Claiming Winnings**
```solidity
function claimWinnings(uint256 _marketId) external;
```
📌 **Winners claim their rewards, and the prize pool is distributed proportionally**  

---

### **(5) Refund Mechanism**
If a market is **invalidated** due to unforeseen reasons (oracle failure, market collapse), users can request a refund.

#### **Refunding Users in Case of Market Invalidation**
```solidity
function requestRefund(uint256 _marketId) external;
```
📌 **Funds are returned to all participants**  

---

## **3. Oracle Interface**
The prediction market settlement relies on **oracles for accurate event outcomes**.

✅ **Oracle Options**:
1️⃣ **On-Chain Oracles (Chainlink, Pyth, DIA)**  
2️⃣ **Decentralized Oracles (Augur, Gnosis)**  
3️⃣ **Community-Based Decision (Kleros Arbitration)**  

#### **Fetching Event Results**
```solidity
function fetchEventResult(uint256 _marketId) external view returns (uint256);
```
📌 **Oracles periodically update event statuses, and markets retrieve the final result via this function**  

---

## **4. Web3 Interaction Examples**
Prediction market frontends can interact using **Web3.js / Ethers.js**.

### **(1) Creating a Market**
```javascript
const contract = new ethers.Contract(marketAddress, MarketABI, signer);
await contract.createMarket(
  "Who will win the 2024 U.S. Presidential Election?",
  ["Candidate A", "Candidate B", "Others"],
  Math.floor(Date.now() / 1000) + 604800, // Ends in one week
  oracleAddress
);
```

### **(2) Placing a Bet**
```javascript
await contract.placeBet(marketId, outcomeId, ethers.utils.parseUnits("10", "ether"));
```

### **(3) Querying Market Information**
```javascript
const marketInfo = await contract.getMarketInfo(marketId);
console.log(`Market: ${marketInfo.eventName}`);
console.log(`Options: ${marketInfo.outcomes.join(", ")}`);
```

### **(4) Claiming Winnings**
```javascript
await contract.claimWinnings(marketId);
```

---

## **5. Applicable Use Cases for Prediction Markets**
🔹 **Financial Market Predictions** (Will BTC surpass $100,000?)  
🔹 **Sports Betting** (Who will win the FIFA World Cup?)  
🔹 **Political Events** (Who will win the 2024 election?)  
🔹 **Technological Developments** (When will AI surpass human intelligence?)  

---

## **6. Conclusion**
✅ **N42 supports EVM-compatible smart contracts, enabling quick deployment of prediction markets**  
✅ **Low gas fees and high TPS make it ideal for large-scale betting and settlement**  
✅ **Oracle integration ensures fairness, supporting both on-chain data and community-based decision-making**  
✅ **Applicable to financial, sports, political, and technological prediction scenarios** 🚀  

👉 **Developers can integrate prediction markets seamlessly into the N42 ecosystem using Solidity + Web3!** 🎯
