# **N42 公链预测市场接口设计**

## **1. 预测市场概述**
N42 作为**兼容 EVM 的高性能公链**，可支持去中心化预测市场（Decentralized Prediction Market），允许用户对**事件、金融市场、体育、政治等**未来结果进行押注，并通过智能合约确保公平交易。  

**主要特点：**
✅ **智能合约驱动**：完全去中心化，依赖区块链执行  
✅ **EVM 兼容**：可复用现有 Ethereum 预测市场（如 Augur、Polymarket）  
✅ **低 Gas 费用**：适合大规模用户参与  
✅ **预言机支持**：通过**链上预言机（Oracle）**获取结果  

---

## **2. 预测市场的核心接口**
### **（1）市场创建**
用户或项目方可以创建新的预测市场，提供可押注的事件选项。

📌 **示例：2024年美国大选赢家？**
- 选项 1️⃣：候选人 A  
- 选项 2️⃣：候选人 B  
- 选项 3️⃣：其他  

#### **创建预测市场**
```solidity
function createMarket(
    string memory _eventName,    // 事件名称
    string[] memory _outcomes,   // 可能的结果
    uint256 _endTimestamp,       // 市场结束时间
    address _oracle              // 预言机地址
) external returns (uint256 marketId);
```
📌 **返回：`marketId`（市场编号），用于后续查询**  

---

### **（2）参与预测**
用户可以在市场开放期间押注某个选项，并存入押金（Stake）。

#### **用户押注**
```solidity
function placeBet(
    uint256 _marketId,   // 预测市场 ID
    uint256 _outcomeId,  // 选项 ID
    uint256 _amount      // 押注金额
) external;
```
📌 **押注后，资金锁定在智能合约，等待事件结果**  

---

### **（3）查询市场信息**
#### **获取市场详情**
```solidity
function getMarketInfo(uint256 _marketId) external view returns (
    string memory eventName,
    string[] memory outcomes,
    uint256 totalPool,
    uint256 endTimestamp,
    bool isResolved
);
```
📌 **用于前端展示预测市场信息**  

#### **获取市场选项的当前投注总额**
```solidity
function getOutcomePool(uint256 _marketId, uint256 _outcomeId) external view returns (uint256);
```
📌 **用于显示每个选项的资金池**  

---

### **（4）市场结算**
预测市场在**事件结束后**，需要从预言机获取结果并分配资金。

#### **预言机推送最终结果**
```solidity
function reportOutcome(
    uint256 _marketId,
    uint256 _winningOutcome
) external onlyOracle;
```
📌 **只有预言机（Oracle）地址可以调用，确保市场公平结算**  

#### **分配资金**
```solidity
function claimWinnings(uint256 _marketId) external;
```
📌 **胜者领取奖励，资金池按比例分配**  

---

### **（5）退款机制**
若市场因特殊原因**无效**（预言机失败、市场崩溃），用户可申请退款。

#### **市场无效时退款**
```solidity
function requestRefund(uint256 _marketId) external;
```
📌 **资金将退还给所有参与者**  

---

## **3. 预言机（Oracle）接口**
预测市场的结算依赖**预言机提供的真实数据**。

✅ **可选方案**：
1️⃣ **链上预言机（Chainlink、Pyth、DIA）**  
2️⃣ **去中心化预言机（Augur、Gnosis）**  
3️⃣ **社区投票（如 Kleros 仲裁）**  

#### **获取事件结果**
```solidity
function fetchEventResult(uint256 _marketId) external view returns (uint256);
```
📌 **预言机定期更新事件状态，市场根据该接口获取最终结果**  

---

## **4. Web3 交互示例**
预测市场的前端可以使用 **Web3.js / Ethers.js** 进行交互：

### **（1）创建市场**
```javascript
const contract = new ethers.Contract(marketAddress, MarketABI, signer);
await contract.createMarket(
  "2024年美国大选赢家？",
  ["候选人 A", "候选人 B", "其他"],
  Math.floor(Date.now() / 1000) + 604800, // 一周后结束
  oracleAddress
);
```

### **（2）用户押注**
```javascript
await contract.placeBet(marketId, outcomeId, ethers.utils.parseUnits("10", "ether"));
```

### **（3）查询市场信息**
```javascript
const marketInfo = await contract.getMarketInfo(marketId);
console.log(`市场：${marketInfo.eventName}`);
console.log(`选项：${marketInfo.outcomes.join(", ")}`);
```

### **（4）领取奖励**
```javascript
await contract.claimWinnings(marketId);
```

---

## **5. 适用的预测市场应用**
🔹 **金融市场预测**（BTC 价格是否突破 $100,000？）  
🔹 **体育比赛**（世界杯冠军预测）  
🔹 **政治事件**（2024年大选赢家？）  
🔹 **科技发展**（AI 什么时候超越人类智能？）  

---

## **6. 结论**
✅ **N42 支持 EVM 兼容智能合约，可快速部署预测市场**  
✅ **低 Gas 费、高 TPS 适用于大规模投注和结算**  
✅ **预言机接口保障结果公正，可支持链上数据与社区治理**  
✅ **适用于金融、体育、政治等多种预测应用场景** 🚀

👉 **开发者可以直接使用 Solidity + Web3 进行集成，让预测市场无缝运行在 N42 生态！** 🎯
