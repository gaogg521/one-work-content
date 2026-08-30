---
name: smart-accounts-kit
description: 使用 MetaMask Smart Accounts Kit 进行 Web3 开发。当用户想要构建带有 ERC-4337 智能账户的 dApp、发送用户操作、批量交易、配置签名者（EOA、passkey、多签）、使用 paymaster 实现 gas 抽象、创建委托或请求高级权限（ERC-7715）时使用。支持 Viem 集成、多种签名者类型（Dynamic、Web3Auth、Wagmi）、无 gas 交易和委托框架。
metadata: None
openclaw: None
emoji: 🦊
homepage: https://docs.metamask.io/smart-accounts-kit
tags:
- 元数据
---

## 快速参考

此技能文件提供对 MetaMask Smart Accounts Kit v0.3.0 的快速访问。有关详细信息，请参阅特定的参考文件。

**📚 详细参考：**

- [智能账户参考](./references/smart-accounts.md) - 账户创建、实现、签名者
- [委托参考](./references/delegations.md) - 委托生命周期、范围、限制条件
- [高级权限参考](./references/advanced-permissions.md) - 通过 MetaMask 实现 ERC-7715 权限

## 包安装

```bash
npm install @metamask/smart-accounts-kit@0.3.0
```

对于自定义限制条件执行器：

```bash
forge install metamask/delegation-framework@v1.3.0
```

## 核心概念摘要

### 1. 智能账户（ERC-4337）

三种实现类型：

| 实现 | 最适合 | 关键特性 |
|---------------|----------|-------------|
| **Hybrid** (`Implementation.Hybrid`) | 标准 dApp 用户 | EOA + passkey 签名者，最灵活 |
| **MultiSig** (`Implementation.MultiSig`) | 金库/DAO 操作 | 基于阈值的安全，Safe 兼容 |
| **Stateless7702** (`Implementation.Stateless7702`) | 拥有现有 EOA 的高级用户 | 保持相同地址，通过 EIP-7702 添加智能账户功能 |

**决策指南：**
- 为普通用户构建？→ Hybrid
- 管理金库或多方控制？→ MultiSig  
- 升级现有 EOA 而不更改地址？→ Stateless7702

### 2. 委托框架（ERC-7710）

从委托者向受委托者授予权限：

- **范围** - 初始权限（支出限制、函数调用）
- **限制条件** - 由智能合约强制执行的限制
- **类型** - 根、开放根、重新委托、开放重新委托
- **生命周期** - 创建 → 签名 → 存储 → 兑换

### 3. 高级权限（ERC-7715）

通过 MetaMask 扩展请求权限：

- 人类可读的 UI 确认
- ERC-20 和原生代币权限
- 需要 MetaMask Flask 13.5.0+
- 用户必须拥有智能账户

## 快速代码示例

### 创建智能账户

```typescript
import { Implementation, toMetaMaskSmartAccount } from '@metamask/smart-accounts-kit'
import { privateKeyToAccount } from 'viem/accounts'

const account = privateKeyToAccount('0x...')

const smartAccount = await toMetaMaskSmartAccount({
  client: publicClient,
  implementation: Implementation.Hybrid,
  deployParams: [account.address, [], [], []],
  deploySalt: '0x',
  signer: { account },
})
```

### 创建委托

```typescript
import { createDelegation } from '@metamask/smart-accounts-kit'
import { parseUnits } from 'viem'

const delegation = createDelegation({
  to: delegateAddress,
  from: delegatorSmartAccount.address,
  environment: delegatorSmartAccount.environment,
  scope: {
    type: 'erc20TransferAmount',
    tokenAddress: '0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238',
    maxAmount: parseUnits('10', 6),
  },
  caveats: [
    { type: 'timestamp', afterThreshold: now, beforeThreshold: expiry },
    { type: 'limitedCalls', limit: 5 },
  ],
})
```

### 签署委托

```typescript
const signature = await smartAccount.signDelegation({ delegation })
const signedDelegation = { ...delegation, signature }
```

### 兑换委托

```typescript
import { createExecution, ExecutionMode } from '@metamask/smart-accounts-kit'
import { DelegationManager } from '@metamask/smart-accounts-kit/contracts'
import { encodeFunctionData, erc20Abi } from 'viem'

const callData = encodeFunctionData({
  abi: erc20Abi,
  args: [recipient, parseUnits('1', 6)],
  functionName: 'transfer',
})

const execution = createExecution({ target: tokenAddress, callData })

const redeemCalldata = DelegationManager.encode.redeemDelegations({
  delegations: [[signedDelegation]],
  modes: [ExecutionMode.SingleDefault],
  executions: [[execution]],
})

// 通过智能账户
const userOpHash = await bundlerClient.sendUserOperation({
  account: delegateSmartAccount,
  calls: [{ to: delegateSmartAccount.address, data: redeemCalldata }],
})

// 通过 EOA
const txHash = await delegateWalletClient.sendTransaction({
  to: environment.DelegationManager,
  data: redeemCalldata,
})
```

### 请求高级权限

```typescript
import { erc7715ProviderActions } from '@metamask/smart-accounts-kit/actions'

const walletClient = createWalletClient({
  transport: custom(window.ethereum),
}).extend(erc7715ProviderActions())

const grantedPermissions = await walletClient.requestExecutionPermissions([
  {
    chainId: chain.id,
    expiry: now + 604800,
    signer: {
      type: 'account',
      data: { address: sessionAccount.address },
    },
    permission: {
      type: 'erc20-token-periodic',
      data: {
        tokenAddress,
        periodAmount: parseUnits('10', 6),
        periodDuration: 86400,
        justification: 'Transfer 10 USDC daily',
      },
    },
    isAdjustmentAllowed: true,
  },
])
```

### 兑换高级权限

```typescript
// 智能账户
import { erc7710BundlerActions } from '@metamask/smart-accounts-kit/actions'

const bundlerClient = createBundlerClient({
  client: publicClient,
  transport: http(bundlerUrl),
}).extend(erc7710BundlerActions())

const permissionsContext = grantedPermissions[0].context
const delegationManager = grantedPermissions[0].signerMeta.delegationManager

const userOpHash = await bundlerClient.sendUserOperationWithDelegation({
  publicClient,
  account: sessionAccount,
  calls: [
    {
      to: tokenAddress,
      data: calldata,
      permissionsContext,
      delegationManager,
    },
  ],
})

// EOA
import { erc7710WalletActions } from '@metamask/smart-accounts-kit/actions'

const walletClient = createWalletClient({
  account: sessionAccount,
  chain,
  transport: http(),
}).extend(erc7710WalletActions())

const txHash = await walletClient.sendTransactionWithDelegation({
  to: tokenAddress,
  data: calldata,
  permissionsContext,
  delegationManager,
})
```

## 关键 API 方法

### 智能账户

- `toMetaMaskSmartAccount()` - 创建智能账户
- `aggregateSignature()` - 组合多签签名
- `signDelegation()` - 签署委托
- `signUserOperation()` - 签署用户操作
- `signMessage()` / `signTypedData()` - 标准签名

### 委托

- `createDelegation()` - 创建与受委托者的委托
- `createOpenDelegation()` - 创建开放委托
- `createCaveatBuilder()` - 构建限制条件数组
- `createExecution()` - 创建执行结构体
- `redeemDelegations()` - 编码兑换调用数据
- `signDelegation()` - 使用私钥签署
- `getSmartAccountsEnvironment()` - 解析环境
- `deploySmartAccountsEnvironment()` - 部署合约
- `overrideDeployedEnvironment()` - 覆盖环境

### 高级权限

- `erc7715ProviderActions()` - 用于请求的 Wallet 客户端扩展
- `requestExecutionPermissions()` - 请求权限
- `erc7710BundlerActions()` - Bundler 客户端扩展
- `sendUserOperationWithDelegation()` - 使用智能账户兑换
- `erc7710WalletActions()` - Wallet 客户端扩展
- `sendTransactionWithDelegation()` - 使用 EOA 兑换

## 支持的 ERC-7715 权限类型

### ERC-20 代币权限

| 权限类型 | 描述 |
|----------------|-------------|
| `erc20-token-periodic` | 每个周期重置的周期限制 |
| `erc20-token-stream` | 带有 amountPerSecond 速率的线性流式传输 |

### 原生代币权限

| 权限类型 | 描述 |
|----------------|-------------|
| `native-token-periodic` | 重置的每周期 ETH 限制 |
| `native-token-stream` | 带有 amountPerSecond 速率的线性 ETH 流式传输 |

## 常见委托范围

### 支出限制

| 范围                       | 描述                   |
| --------------------------- | ----------------------------- |
| `erc20TransferAmount`       | 固定 ERC-20 限制            |
| `erc20PeriodTransfer`       | 每周期 ERC-20 限制       |
| `erc20Streaming`            | 线性流式传输 ERC-20       |
| `nativeTokenTransferAmount` | 固定原生代币限制      |
| `nativeTokenPeriodTransfer` | 每周期原生代币限制 |
| `nativeTokenStreaming`      | 线性流式传输原生代币       |
| `erc721Transfer`            | ERC-721 (NFT) 转账        |

### 函数调用

| 范围               | 描述                        |
| ------------------- | ---------------------------------- |
| `functionCall`      | 允许的特定方法/地址 |
| `ownershipTransfer` | 仅所有权转账           |

## 常见限制条件执行器

### 目标和方法

- `allowedTargets` - 限制可调用地址
- `allowedMethods` - 限制可调用方法
- `allowedCalldata` - 验证特定调用数据
- `exactCalldata` / `exactCalldataBatch` - 精确调用数据匹配
- `exactExecution` / `exactExecutionBatch` - 精确执行匹配

### 价值和代币

- `valueLte` - 限制原生代币价值
- `erc20TransferAmount` - 限制 ERC-20 金额
- `erc20BalanceChange` - 验证 ERC-20 余额变化
- `erc721Transfer` / `erc721BalanceChange` - ERC-721 限制
- `erc1155BalanceChange` - ERC-1155 验证

### 时间和频率

- `timestamp` - 有效时间范围（秒）
- `blockNumber` - 有效区块范围
- `limitedCalls` - 限制兑换次数
- `erc20PeriodTransfer` / `erc20Streaming` - 基于时间的 ERC-20
- `nativeTokenPeriodTransfer` / `nativeTokenStreaming` - 基于时间的原生代币

### 安全和状态

- `redeemer` - 限制兑换到特定地址
- `id` - 带 ID 的一次性委托
- `nonce` - 通过 nonce 批量撤销
- `deployed` - 自动部署合约
- `ownershipTransfer` - 仅所有权转账
- `nativeTokenPayment` - 需要支付
- `nativeBalanceChange` - 验证原生余额
- `multiTokenPeriod` - 多代币周期限制

## 执行模式

| 模式            | 链   | 处理  | 失败时 |
| --------------- | -------- | ----------- | ---------- |
| `SingleDefault` | 一条      | 顺序  | 回滚     |
| `SingleTry`     | 一条      | 顺序  | 继续   |
| `BatchDefault`  | 多条 | 交错 | 回滚     |
| `BatchTry`      | 多条 | 交错 | 继续   |

## 合约地址（v1.3.0）

### 核心

| 合约              | 地址                                      |
| --------------------- | -------------------------------------------- |
| EntryPoint            | `0x0000000071727De22E5E9d8BAf0edAc6f37da032` |
| SimpleFactory         | `0x69Aa2f9fe1572F1B640E1bbc512f5c3a734fc77c` |
| DelegationManager     | `0xdb9B1e94B5b69Df7e401DDbedE43491141047dB3` |
| MultiSigDeleGatorImpl | `0x56a9EdB16a0105eb5a4C54f4C062e2868844f3A7` |
| HybridDeleGatorImpl   | `0x48dBe696A4D990079e039489bA2053B36E8FFEC4` |

## 关键规则

### 始终需要

1. **始终使用限制条件** - 切勿创建无限制的委托
2. **首先部署委托者** - 账户必须在兑换前部署
3. **检查智能账户状态** - ERC-7715 需要用户拥有智能账户

### 行为

4. **限制条件是累积的** - 在委托链中，限制会叠加
5. **函数调用默认值** - v0.3.0 默认无原生代币（使用 `valueLte`）
6. **批量模式限制条件** - 无兼容的限制条件执行器可用

### 要求

7. **ERC-7715 要求** - MetaMask Flask 13.5.0+、智能账户
8. **多签阈值** - 需要至少阈值个签名者
9. **7702 升级** - Stateless7702 需要首先通过 EIP-7702 升级

## 高级模式

### 并行用户操作（Nonce 密钥）

智能账户使用 256 位 nonce 结构：192 位密钥 + 64 位序列。每个唯一密钥都有自己的独立序列，实现并行执行。这对于后端服务并发处理多个委托至关重要。

#### 安装

为了正确处理 nonce，请 alongside Smart Accounts Kit 安装 permissionless SDK：

```bash
npm install permissionless
```

#### 并行 Nonce 的工作原理

ERC-4337 使用单个 uint256 nonce，其中：
- **192 位** = 密钥标识符（允许并行流）
- **64 位** = 序列号（每个密钥递增）

每个密钥都有独立的序列，因此具有不同密钥的 UserOp 可以在没有排序约束的情况下并行执行。

#### 使用 Permissionless 获取 Nonce

```typescript
import { getAccountNonce } from 'permissionless'
import { entryPoint07Address } from 'viem/account-abstraction'

// 获取特定密钥的 nonce
const parallelNonce = await getAccountNonce(publicClient, {
  address: smartAccount.address,
  entryPointAddress: entryPoint07Address,
  key: BigInt(Date.now()), // 用于并行执行的唯一密钥
})

const userOpHash = await bundlerClient.sendUserOperation({
  account: smartAccount,
  calls: [redeemCalldata],
  nonce: parallelNonce, // 正确编码的 256 位 nonce
})
```

#### 并行执行模式

```typescript
import { getAccountNonce } from 'permissionless'
import { entryPoint07Address } from 'viem/account-abstraction'

// 并行执行多个兑换 UserOp
const redeems = await Promise.all(
  delegations.map(async (delegation, index) => {
    // 为此操作生成唯一密钥
    const nonceKey = BigInt(Date.now()) + BigInt(index * 1000)
    
    // 获取此密钥的正确编码 nonce
    const nonce = await getAccountNonce(publicClient, {
      address: backendSmartAccount.address,
      entryPointAddress: entryPoint07Address,
      key: nonceKey,
    })
    
    const redeemCalldata = DelegationManager.encode.redeemDelegations({
      delegations: [[delegation]],
      modes: [ExecutionMode.SingleDefault],
      executions: [[execution]],
    })
    
    return bundlerClient.sendUserOperation({
      account: backendSmartAccount,
      calls: [{ to: backendSmartAccount.address, data: redeemCalldata }],
      nonce, // 通过唯一密钥启用并行执行
    })
  })
)
```

#### 不使用 Permissionless（手动方法）

EntryPoint 合约将 nonce 编码为：`sequence | (key << 64)`

如果不使用 permissionless，请手动编码：

```typescript
// EntryPoint: nonceSequenceNumber[sender][key] | (uint256(key) << 64)
const key = BigInt(Date.now())
const sequence = 0n // 新密钥从序列 0 开始
const nonce = sequence | (key << 64n)
// 或等效地：(key << 64n) | sequence
```

然而，推荐使用 permissionless 的 `getAccountNonce`，因为它：
- 从 EntryPoint 获取密钥的当前序列
- 正确编码 256 位值
- 处理边缘情况和验证

#### 关键点

- **不同的密钥 = 并行执行** — 不同密钥之间没有排序保证
- **相同的密钥 = 顺序执行** — 每个密钥的序列单调递增
- **使用场景：** 后端兑换服务、DCA 应用、高频交易、批量操作
- **Nonce 生成：** `getAccountNonce` 返回正确编码的完整 256 位 nonce

#### 常见错误

| 错误 | 结果 |
|---------|--------|
| 重复使用相同的 nonce 密钥 | 顺序执行（违背目的） |
| 使用不带偏移的 `Date.now()` | 如果多个操作同时触发可能发生冲突 |
| 不使用 `getAccountNonce` | 可能错过当前序列，导致替换而非新操作 |
| 假设排序 | 依赖操作中的竞态条件 |

#### 错误处理

```typescript
const results = await Promise.allSettled(redeems)

results.forEach((result, index) => {
  if (result.status === 'rejected') {
    // 检查特定错误
    if (result.reason.message?.includes('AA25')) {
      console.error(`Nonce collision for op ${index}`)
    }
    // 处理或重试
  }
})
```

### 后端委托兑换

对于服务器端自动化（DCA 机器人、keeper 服务、自动交易）：

```typescript
// 1. 后端创建自己的智能账户作为受委托者
const backendAccount = await toMetaMaskSmartAccount({
  client: publicClient,
  implementation: Implementation.Hybrid,
  deployParams: [backendOwner.address, [], [], []],
  deploySalt: '0x',
  signer: { account: backendOwner },
})

// 2. 后端通过从其账户发送 UserOp 来兑换
const userOpHash = await bundlerClient.sendUserOperation({
  account: backendAccount,
  calls: [{
    to: backendAccount.address,
    data: DelegationManager.encode.redeemDelegations({
      delegations: [[userDelegation]],
      modes: [ExecutionMode.SingleDefault],
      executions: [[swapExecution]],
    })
  }],
})
```

**使用场景：** 基于市场信号或计划间隔自动兑换交换委托的自动定投（DCA）机器人。

### 反事实账户部署

委托者账户必须在委托可以兑换之前部署。DelegationManager 对反事实账户回滚并返回 `0xb9f0f171`。

**解决方案：** 通过第一个 UserOp 自动部署：

```typescript
// 构建兑换调用数据
const redeemCalldata = DelegationManager.encode.redeemDelegations({
  delegations: [[signedDelegation]],
  modes: [ExecutionMode.SingleDefault],
  executions: [[execution]],
})

// 第一次兑换通过 initCode 自动部署账户
const userOpHash = await bundlerClient.sendUserOperation({
  account: smartAccount, // 如果是反事实将部署
  calls: [{
    to: smartAccount.address,
    data: redeemCalldata,
    value: 0n,
  }],
})
```

### AI 代理的会话账户

对于自动化服务，会话账户充当隔离的签名者，只能在授予的委托范围内操作。私钥可以临时生成、存储在环境变量中，或通过 HSM/服务器钱包管理：

```typescript
// 从各种来源创建的会话账户
const sessionAccount = privateKeyToAccount(
  process.env.SESSION_KEY || generatePrivateKey() || hsmWallet.key
)

// 从用户向会话账户请求委托
const delegation = createDelegation({
  to: sessionAccount.address,
  from: userSmartAccount.address,
  environment,
  scope: { type: 'erc20TransferAmount', tokenAddress, maxAmount: parseUnits('100', 6) },
  caveats: [
    { type: 'timestamp', afterThreshold: now, beforeThreshold: expiry },
    { type: 'limitedCalls', limit: 10 },
  ],
})
// 会话账户只能在委托约束范围内操作
```

## 常见模式

### 模式 1：带时间限制的 ERC-20

```typescript
const delegation = createDelegation({
  to: delegate,
  from: delegator,
  environment,
  scope: {
    type: 'erc20TransferAmount',
    tokenAddress,
    maxAmount: parseUnits('100', 6),
  },
  caveats: [
    { type: 'timestamp', afterThreshold: now, beforeThreshold: expiry },
    { type: 'limitedCalls', limit: 10 },
    { type: 'redeemer', redeemers: [delegate] },
  ],
})
```

### 模式 2：带价值的函数调用

```typescript
const delegation = createDelegation({
  to: delegate,
  from: delegator,
  environment,
  scope: {
    type: 'functionCall',
    targets: [contractAddress],
    selectors: ['transfer(address,uint256)'],
    valueLte: { maxValue: parseEther('0.1') },
  },
  caveats: [{ type: 'allowedMethods', selectors: ['transfer(address,uint256)'] }],
})
```

### 模式 3：周期性原生代币

```typescript
const delegation = createDelegation({
  to: delegate,
  from: delegator,
  environment,
  scope: {
    type: 'nativeTokenPeriodTransfer',
    periodAmount: parseEther('0.01'),
    periodDuration: 86400,
    startDate: now,
  },
})
```

### 模式 4：重新委托链

```typescript
// Alice → Bob (100 USDC)
const aliceToBob = createDelegation({
  to: bob,
  from: alice,
  environment,
  scope: { type: 'erc20TransferAmount', tokenAddress, maxAmount: parseUnits('100', 6) },
})

// Bob → Carol (50 USDC，权限子集)
const bobToCarol = createDelegation({
  to: carol,
  from: bob,
  environment,
  scope: { type: 'erc20TransferAmount', tokenAddress, maxAmount: parseUnits('50', 6) },
  parentDelegation: aliceToBob,
  caveats: [{ type: 'timestamp', afterThreshold: now, beforeThreshold: expiry }],
})
```

## 故障排除快速修复

| 问题                    | 解决方案                                                     |
| ------------------------ | ------------------------------------------------------------ |
| 账户未部署     | 使用 `bundlerClient.sendUserOperation()` 部署            |
| 签名无效        | 验证链 ID、委托管理器、签名者权限      |
| 限制条件执行器回滚 | 检查限制条件参数是否匹配执行，验证顺序        |
| 兑换失败        | 检查委托者余额、调用数据有效性、目标合约 |
| ERC-7715 不工作     | 升级到 Flask 13.5.0+，确保用户拥有智能账户      |
| 权限被拒绝        | 优雅处理，提供手动回退                   |
| 阈值未达到        | 为多签添加更多签名者                                |
| 7702 不工作         | 确认 EOA 首先通过 EIP-7702 升级                      |

## 错误代码参考

来自 MetaMask Delegation Framework 合约（v1.3.0）的错误代码。使用像 [calldata.swiss-knife.xyz](https://calldata.swiss-knife.xyz/decoder) 这样的解码器来识别错误签名。

### DelegationManager 错误（已验证）

| 错误代码 | 错误名称 | 含义 |
|------------|-----------|---------|
| `0xb5863604` | `InvalidDelegate()` | **调用者不是受委托者** — 最常见的错误 |
| `0xb9f0f171` | `InvalidDelegator()` | 调用者不是委托者 |
| `0x05baa052` | `CannotUseADisabledDelegation()` | 尝试兑换已禁用的委托 |
| `0xded4370e` | `InvalidAuthority()` | 委托链权限验证失败 |
| `0x1bcaf69f` | `BatchDataLengthMismatch()` | 批量中的数组长度不匹配 |
| `0x005ecddb` | `AlreadyDisabled()` | 委托已被禁用 |
| `0xf2a5f75a` | `AlreadyEnabled()` | 委托已启用 |
| `0xf645eedf` | `ECDSAInvalidSignature()` | 无效的 ECDSA 签名格式 |
| `0xfce698f7` | `ECDSAInvalidSignatureLength(uint256)` | 签名长度不正确 |
| `0xd78bce0c` | `ECDSAInvalidSignatureS(bytes32)` | 签名 S 值无效 |
| `0xac241e11` | `EmptySignature()` | 签名为空 |
| `0xd93c0665` | `EnforcedPause()` | 合约已暂停 |
| `0x3db6791c` | `InvalidEOASignature()` | EOA 签名验证失败 |
| `0x155ff427` | `InvalidERC1271Signature()` | 智能合约签名（ERC1271）失败 |
| `0x118cdaa7` | `OwnableUnauthorizedAccount(address)` | 未经授权的账户尝试仅所有者操作 |
| `0x1e4fbdf7` | `OwnableInvalidOwner(address)` | 所有权转让中的无效所有者地址 |
| `0xf6b6ef5b` | `InvalidShortString()` | 字符串参数太短 |
| `0xaa0ea2d8` | `StringTooLong(string)` | 字符串参数超过最大长度 |

### DeleGatorCore 错误（已验证）

| 错误代码 | 错误名称 | 含义 |
|------------|-----------|---------|
| `0xd663742a` | `NotEntryPoint()` | 调用者不是 EntryPoint 合约 |
| `0x0796d945` | `NotEntryPointOrSelf()` | 调用者既不是 EntryPoint 也不是此合约 |
| `0x1a4b3a04` | `NotDelegationManager()` | 调用者不是 DelegationManager |
| `0xb96fcfe4` | `UnsupportedCallType(bytes1)` | 不支持的执行调用类型 |
| `0x1187dc06` | `UnsupportedExecType(bytes1)` | 不支持的执行类型 |
| `0x29c3b7ee` | `NotSelf()` | 调用者不是此合约本身 |

### 常见限制条件执行器错误（回滚字符串）

| 错误字符串 | 含义 |
|--------------|---------|
| `AllowedTargetsEnforcer:target-address-not-allowed` | 目标合约不在允许列表中 |
| `AllowedTargetsEnforcer:invalid-terms-length` | 条款长度不是 20 字节的倍数 |
| `ERC20TransferAmountEnforcer:invalid-terms-length` | 条款必须是 52 字节 |
| `ERC20TransferAmountEnforcer:invalid-contract` | 目标与允许的代币不匹配 |
| `ERC20TransferAmountEnforcer:invalid-method` | 方法不是 `transfer` |
| `ERC20TransferAmountEnforcer:allowance-exceeded` | 转账超过委托限制 |
| `CaveatEnforcer:invalid-call-type` | 必须使用单调用类型 |
| `CaveatEnforcer:invalid-execution-type` | 必须使用默认执行类型 |

### 生产中最常见的错误

**`0xb5863604` — InvalidDelegate()**
- **原因：** 调用者与委托中的受委托者地址不匹配
- **修复：** 验证 `msg.sender` 等于委托中的 `to` 地址

**`0xb9f0f171` — InvalidDelegator()**
- **原因：** 尝试从错误地址启用/禁用，或反事实账户
- **修复：** 只有委托者可以启用/禁用；对于反事实账户，第一个 UserOp 自动部署

**`0x05baa052` — CannotUseADisabledDelegation()**
- **原因：** 委托已被委托者禁用
- **修复：** 要求委托者重新启用，或使用不同的委托

**`0xded4370e` — InvalidAuthority()**
- **原因：** 委托链断裂（重新委托父级不匹配）
- **修复：** 确保重新委托链正确排序（叶 → 根）

**`0x1bcaf69f` — BatchDataLengthMismatch()**
- **原因：** `redeemDelegations` 调用中的数组长度不匹配
- **修复：** 确保 `permissionContexts`、`modes`、`executionCallDatas` 长度相等

**`0x3db6791c` — InvalidEOASignature()**
- **原因：** EOA 签名无效、错误的链或错误的委托管理器
- **修复：** 验证签名是使用正确的链 ID 和委托管理器创建的

## 资源

- **NPM：** `@metamask/smart-accounts-kit`
- **合约：** `metamask/delegation-framework@v1.3.0`
- **ERC 标准：** ERC-4337、ERC-7710、ERC-7715、ERC-7579
- **MetaMask Flask：** https://metamask.io/flask

## 版本信息

- **工具包：** 0.3.0
- **委托框架：** 1.3.0
- **破坏性变更：** 函数调用范围默认无原生代币转账

---

**有关详细文档，请参阅 `/references` 目录中的参考文件。**
