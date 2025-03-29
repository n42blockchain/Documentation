'''
N42 公链 AI Agent 接口规范

本文档定义了 N42 公链支持的 AI Agent 接口规范，旨在为开发者提供标准化的通信、控制与执行方式，确保 AI Agent 在链上环境中的高效部署、交互与协作。
'''

# 一、概述

N42 是新一代高性能去中心化公链，具备高度并行、异步共识、分层分片、低延迟、高吞吐等特性。AI Agent 是部署于链上或链下、具备自学习和执行能力的智能模块，能够完成任务处理、状态感知、数据处理、合约调用等功能。

本接口规范定义 AI Agent 与 N42 公链交互的通用协议，涵盖账户结构、数据流、合约交互、任务执行、权限控制、安全机制等方面。

# 二、接口架构设计

## 1. 架构组成

- **Agent Core**：Agent 的主逻辑模块，具备状态感知、策略执行、模型调用能力。
- **Chain Gateway**：链接入模块，负责与 N42 区块链的数据读写、合约交互。
- **Agent Hub**：AI Agent 管理中心，管理注册、调度、协作等元信息。
- **State Cache**：链下状态缓存层，提升数据访问性能。
- **Event Monitor**：监听链上事件，用于触发 Agent 行为。

## 2. 技术标准

- 通讯协议：gRPC / RESTful / WebSocket（链下通信）
- 数据结构：JSON-RPC / Protobuf（可拓展）
- 编程语言：支持 Rust、Go、Python SDK 接入

# 三、接口模块规范

## 1. Agent 注册与身份管理

### 1.1 注册接口
```json
POST /api/agent/register
{
  "agent_id": "string",
  "pubkey": "hex",
  "metadata": { "model": "gpt-4", "domain": "nft_trading" },
  "signature": "hex"
}
```

### 1.2 查询 Agent 信息
```json
GET /api/agent/{agent_id}
```

### 1.3 注销 Agent
```json
POST /api/agent/{agent_id}/deactivate
```

## 2. 状态访问与链上数据读取

### 2.1 读取账户状态
```json
GET /api/chain/account/{address}
```

### 2.2 读取资源/资产状态
```json
GET /api/chain/assets/{token_id}
```

### 2.3 查询事件日志
```json
GET /api/chain/events?from_block=100&to_block=200&type=Transfer
```

## 3. 合约调用与任务执行

### 3.1 调用合约函数
```json
POST /api/chain/contract/call
{
  "from": "agent_address",
  "to": "contract_address",
  "method": "transfer",
  "params": ["0xabc", "1000"],
  "gas_limit": 300000,
  "signature": "hex"
}
```

### 3.2 执行链上任务（Task Intent）
```json
POST /api/agent/intent
{
  "agent_id": "agent123",
  "intent_type": "execute",
  "contract": "0xcontract",
  "action": "buy_token",
  "params": {"token": "nft001", "price": "20"}
}
```

## 4. Agent 间协作接口

### 4.1 协议定义
- 多 Agent 使用统一协作协议协议（Agent-Interlink Protocol, AIP）
- 消息格式：Protobuf / JSON，使用 P2P 消息通道（libp2p）

### 4.2 任务分发
```json
POST /api/agent/delegate
{
  "delegator": "agent123",
  "delegatee": "agent456",
  "task": "price_oracle",
  "parameters": {"symbol": "ETH/USD"}
}
```

## 5. 权限与安全模型

### 5.1 权限声明（Policy Contract）
- 使用链上策略合约定义 Agent 权限
- 支持 ACL / RBAC 模型声明

```solidity
contract AgentPolicy {
  mapping(address => string[]) public permissions;
  function hasPermission(address agent, string calldata action) public view returns (bool);
}
```

### 5.2 零知识签名与身份验证
- 支持 zkLogin / zkAuth 等匿名身份接入
- 所有敏感操作需多签 + ZKP 验证（如数据隐私处理）

## 6. 数据缓存与异步任务调度

### 6.1 数据订阅与缓存
```json
POST /api/agent/subscribe
{
  "topics": ["Transfer", "PriceUpdate"],
  "filters": {"token": "nft001"}
}
```

### 6.2 异步任务注册
```json
POST /api/agent/task/schedule
{
  "task_type": "periodic",
  "interval": 3600,
  "payload": {
    "contract": "oracle_contract",
    "method": "update_price"
  }
}
```

## 7. Agent 状态监控与生命周期管理

### 7.1 状态查询
```json
GET /api/agent/status/{agent_id}
```

### 7.2 异常报告
```json
POST /api/agent/report
{
  "agent_id": "agent789",
  "type": "execution_error",
  "message": "out of gas"
}
```

# 四、最佳实践

- 所有 Agent 应使用链上注册机制绑定身份，确保调用的可审计性
- 优先使用异步接口与状态订阅，提升性能
- 使用 Layer2 / Off-chain 计算模块处理大模型推理，链上进行任务认证与结果回传
- 利用策略合约和多签控制敏感操作
- 结合 N42 的 CRDT 状态模型构建 Agent 状态同步机制

# 五、未来拓展

未来将支持：
- 模型链上注册与追踪（Model Provenance）
- 模型输出验证与质押机制（Agent Slashing）
- 链上 Agent Marketplace 与服务互操作标准（Agent Service Fabric）
- 面向 DePIN / DeAI 场景的 Agent 网络协调机制

'''
以上为 N42 公链 AI Agent 接口规范的详细说明，可作为链上 AI 模块设计与开发的基础标准。
'''
