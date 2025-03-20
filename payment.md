### **N42 公链支付接口技术文档**

## **1. 概述**
N42 公链是一种高性能、去中心化、无需许可的区块链网络，支持高效、低成本的全球支付和价值转移。N42 采用**无领导、无排序的共识机制**，结合**零知识证明（zk-settlement）**，使交易能够以接近零延迟的方式进行确认，并确保全局状态的一致性。本文档详细介绍 N42 公链支付接口的架构、API 设计、安全性机制及最佳实践。

---

## **2. 核心架构**
N42 支付系统依赖以下关键组件：
- **用户账户（Vault）**：用户在每个域（Domain）内的资产存储点，可接收跨域支付。
- **状态差异列表（SDL）**：记录用户账户的状态变化，作为交易结算的基础。
- **去中心化验证者网络（Validator Network）**：处理交易验证和状态更新，确保支付安全。
- **zk 结算层（Zero-Knowledge Settlement Layer）**：通过 SNARKs 证明交易的正确性，提高隐私性和效率。

---

## **3. 支付 API 设计**
### **3.1 API 端点**
N42 提供基于 REST 和 WebSocket 的支付接口，允许开发者轻松集成支付功能。

| **接口**           | **方法** | **描述** |
|-------------------|--------|----------|
| `/api/v1/payment/initiate` | `POST` | 发起支付交易 |
| `/api/v1/payment/status` | `GET` | 查询支付状态 |
| `/api/v1/payment/history` | `GET` | 查询历史交易 |
| `/api/v1/payment/cancel` | `POST` | 取消未确认交易 |
| `/api/v1/payment/webhook` | `POST` | 监听支付状态更新 |

---

### **3.2 支付交易接口**
#### **3.2.1 发起支付**
**请求示例**
```json
POST /api/v1/payment/initiate
Content-Type: application/json

{
  "sender": "0xA1B2C3D4E5F6...",
  "recipient": "0xF6E5D4C3B2A1...",
  "amount": "100",
  "token": "N42",
  "memo": "Invoice #12345",
  "fee": "0.0001",
  "domain": "commerce.n42"
}
```

**参数说明**
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `sender` | `string` | 是 | 发送方 N42 地址 |
| `recipient` | `string` | 是 | 接收方 N42 地址 |
| `amount` | `string` | 是 | 交易金额 |
| `token` | `string` | 是 | 交易使用的资产类型（如 `N42`, `USDT`）|
| `memo` | `string` | 否 | 交易备注 |
| `fee` | `string` | 是 | 交易手续费 |
| `domain` | `string` | 是 | 交易所属域（支持跨域支付）|

**返回示例**
```json
{
  "transaction_id": "0x123456789ABCDEF...",
  "status": "pending",
  "timestamp": 1710582937
}
```

---

#### **3.2.2 查询支付状态**
**请求示例**
```json
GET /api/v1/payment/status?transaction_id=0x123456789ABCDEF...
```

**返回示例**
```json
{
  "transaction_id": "0x123456789ABCDEF...",
  "status": "confirmed",
  "block": "7508923",
  "timestamp": 1710582945
}
```

**状态说明**
| 状态 | 说明 |
|------|------|
| `pending` | 交易已提交，等待确认 |
| `confirmed` | 交易已上链并成功执行 |
| `failed` | 交易失败 |
| `canceled` | 交易已取消 |

---

#### **3.2.3 取消支付**
**请求示例**
```json
POST /api/v1/payment/cancel
Content-Type: application/json

{
  "transaction_id": "0x123456789ABCDEF..."
}
```

**返回示例**
```json
{
  "transaction_id": "0x123456789ABCDEF...",
  "status": "canceled"
}
```

---

## **4. 安全性设计**
### **4.1 交易签名**
所有支付交易都必须由**ECDSA（或 Falcon 量子安全签名）**签名，以确保数据完整性。

**示例**
```json
{
  "transaction": {
    "sender": "0xA1B2C3D4E5F6...",
    "recipient": "0xF6E5D4C3B2A1...",
    "amount": "100",
    "token": "N42",
    "timestamp": 1710582937
  },
  "signature": "MEUCIQD...q9yz+Xf==" 
}
```

