# DEX Router Solana V1

[![Anchor Version](https://img.shields.io/badge/Anchor-0.31.1-blue.svg)](https://www.anchor-lang.com/)
[![Solana Version](https://img.shields.io/badge/Solana-2.1.21-purple.svg)](https://solana.com/)

This repository contains the core smart contracts for the Dex Router on Solana Blockchain. The router enables efficient token swaps across multiple liquidity sources and protocols through a unified interface.

## 🚀 Key Features

### Core Features

- **Multi-DEX Aggregation**: Supports 30+ Solana ecosystem DEXs with automatic best path finding
- **Smart Routing Algorithm**: X Routing algorithm automatically selects optimal trading paths to maximize user returns
- **Fee Management**: Flexible fee system supporting proxy trading and platform fees
- **多 DEX 聚合**：支持 30 多个 Solana 生态系统的 DEX，并具备自动寻找最佳路径的功能
- **智能路由算法**：X 路由算法自动选择最优交易路径，以最大化用户收益
- **费用管理**：灵活的费用系统，支持代理交易和平台费用

### Trading Types

- **Basic Swap**: Standard token exchange
- **Proxy Swap**: Trading through proxy accounts
- **Commission Swap**: Trading mode with commission collection
- **Platform Fee Swap**: Trading mode with platform fee collection
- **Wrap/Unwrap**: SOL to wSOL conversion
- **基础交换**：标准的代币兑换
- **代理交换**：通过代理账户进行交易
- **佣金交换**：带有佣金收取的交易模式
- **平台手续费交换**：带有平台手续费收取的交易模式
- **包装/解包**：SOL 与 wSOL 的转换

## 📋 System Requirements

- **Solana CLI**: 2.2.12+
- **Anchor Framework**: 0.31.1+
- **Rust**: 1.85+
- **Node.js**: 23+

## 🛠️ Installation and Deployment

### 1. Environment Setup

#### Quick Installation (Recommended)

For Mac and Linux, run the following single command to install all dependencies:
对于 Mac 和 Linux 系统，请运行以下单个命令来安装所有依赖项：

```bash
curl --proto '=https' --tlsv1.2 -sSfL https://solana-install.solana.workers.dev | bash
```

**Windows users**: You must first install WSL (see [Installation Dependencies](https://solana.com/docs/intro/installation)). Then run the above command in the Ubuntu (Linux) terminal.
**Windows 用户**：您必须先安装 WSL（请参阅[安装依赖项](https://solana.com/docs/intro/installation)）。然后在 Ubuntu（Linux）终端中运行上述命令。

After installation, you should see output similar to:
**安装完成后，您应该会看到类似以下的输出：**

```
Installed Versions:
Rust: rustc 1.86.0 (05f9846f8 2025-03-31)
Solana CLI: solana-cli 2.2.12 (src:0315eb6a; feat:1522022101, client:Agave)
Anchor CLI: anchor-cli 0.31.1
Node.js: v23.11.0
Yarn: 1.22.1
```

#### Manual Installation

**手动安装**

If the quick installation doesn't work, please refer to the [Solana Installation Guide](https://solana.com/docs/intro/installation) for detailed manual installation instructions for each component.
**如果快速安装失败，请参阅[Solana 安装指南](https://solana.com/docs/intro/installation)，获取每个组件的详细手动安装说明。**

#### Environment Verification

**环境验证**

Verify that all components are properly installed:
**验证所有组件是否已正确安装：**

```bash
# Verify Rust
rustc --version

# Verify Solana CLI
solana --version

# Verify Anchor CLI
anchor --version

# Verify Node.js
node --version

# Verify Yarn
yarn --version
```

### 2. Clone Repository

```bash
git clone <repository-url>
cd DEX-Router-Solana-V1
```

### 3. Build Contracts

```bash
# Build all programs
anchor build

# Build specific program
anchor build -p dex-solana
```

### 4. Run Tests

```bash
# Run all tests
anchor test

# Run specific program tests
anchor test -p dex-solana
```

## 📖 Usage Guide

### Parameter Assembly

**参数组装**

Before calling swap methods, you need to assemble the required parameters:
**在调用交换方法之前，您需要组装所需的参数：**

#### SwapArgs Structure

```typescript
interface SwapArgs {
  amountIn: BN // Input amount
  expectAmountOut: BN // Expected output amount
  minReturn: BN // Minimum return amount
  amounts: BN[] // 1st level split amounts
  routes: Route[][] // 2nd level split routes
}

interface Route {
  dexes: Dex[] // DEX types for this route
  weights: number[] // Weights for each DEX
}

enum Dex {
  SplTokenSwap,
  StableSwap,
  Whirlpool,
  MeteoraDynamicpool,
  RaydiumSwap,
  RaydiumStableSwap,
  RaydiumClmmSwap,
  // ... more DEX types
}
```

#### Commission Info Encoding

**佣金信息编码**

For commission and platform fee methods, commission info is encoded as a 32-bit integer:
**对于佣金和平台手续费方法，佣金信息被编码为一个 32 位整数：**

```typescript
// Commission info encoding
const commissionDirection = true // true: from input, false: from output
const commissionRate = 100 // Rate in basis points (0.01%)
const commissionInfo =
  (commissionDirection ? 1 << 31 : 0) | (commissionRate & ((1 << 30) - 1))
```

### Basic Swap

```typescript
// Basic token exchange
const swapArgs = {
  amountIn: new BN(1000000), // 1 token (assuming 6 decimals)
  expectAmountOut: new BN(950000), // Expected output
  minReturn: new BN(900000), // Minimum return
  amounts: [new BN(1000000)], // Single split
  routes: [[{ dexes: [Dex.RaydiumSwap], weights: [100] }]], // Single route
}

const swapTx = await program.methods
  .swap(swapArgs, orderId)
  .accounts(swapAccounts)
  .rpc()
```

### Proxy Swap

```typescript
// Trading through proxy account
const proxySwapTx = await program.methods
  .proxySwap(swapArgs, orderId)
  .accounts(proxySwapAccounts)
  .rpc()
```

### Commission Swap

```typescript
// Trading with commission collection
const commissionSwapArgs = {
  ...swapArgs,
  commissionRate: 100, // 1% commission
  commissionDirection: true, // Commission from input
}

const commissionSwapTx = await program.methods
  .commissionSplSwap(commissionSwapArgs, orderId)
  .accounts(commissionSwapAccounts)
  .rpc()
```

### Platform Fee Swap

```typescript
// Trading with platform fee collection
const platformFeeRate = 50 // 0.5% platform fee
const trimRate = 10 // 1% trim rate

const platformFeeSwapTx = await program.methods
  .platformFeeSolProxySwapV2(
    swapArgs,
    commissionInfo,
    platformFeeRate,
    trimRate,
    orderId
  )
  .accounts(platformFeeAccounts)
  .rpc()
```

### Swap V3 Methods

#### Swap To B (Token to Token)

```typescript
// Swap with commission and optional platform fee
const commissionInfo =
  (commissionDirection ? 1 << 31 : 0) | (commissionRate & ((1 << 30) - 1))
const platformFeeRate = 50 // 0.5% platform fee
const trimRate = 10 // 1% trim rate

const swapToBTx = await program.methods
  .swapTobV3(swapArgs, commissionInfo, trimRate, platformFeeRate, orderId)
  .accounts(swapV3Accounts)
  .rpc()
```

#### Swap To C (Token to Token)

```typescript
// Swap with commission and platform fee (no trim)
const swapToCTx = await program.methods
  .swapV3(swapArgs, commissionInfo, platformFeeRate, orderId)
  .accounts(swapV3Accounts)
  .rpc()
```

### Multi-Hop Routing

```typescript
// Complex routing with multiple hops and DEXs
const complexSwapArgs = {
  amountIn: new BN(1000000),
  expectAmountOut: new BN(950000),
  minReturn: new BN(900000),
  amounts: [
    new BN(600000), // 60% through route 1
    new BN(400000), // 40% through route 2
  ],
  routes: [
    [
      { dexes: [Dex.RaydiumSwap], weights: [100] },
      { dexes: [Dex.Whirlpool], weights: [100] },
    ],
    [{ dexes: [Dex.MeteoraDynamicpool], weights: [100] }],
  ],
}
```

## 🏗️ Project Structure

```
DEX-Router-Solana-V1/
├── Anchor.toml                 # Anchor configuration
├── Cargo.toml                  # Rust workspace configuration
├── programs/
│   └── dex-solana/            # Main program
│       ├── src/
│       │   ├── adapters/      # DEX adapters
│       │   ├── instructions/  # Instruction handlers
│       │   ├── state/         # State definitions
│       │   ├── utils/         # Utility functions
│       │   └── lib.rs         # Program entry point
│       └── Cargo.toml
├── tests/                     # Test files
└── README.md
```

## 🔧 Configuration

### Program ID

- **Mainnet**: `6m2CDdhRgxpH4WjvdzxAYbGxwdGUz5MziiL5jek2kBma`
- **Testnet**: Configure based on deployment environment

## 🧪 Testing

```bash
# Run all tests
anchor test

# Run specific test file
anchor test tests/specific_test.ts

# Run tests with detailed logs
anchor test --skip-lint
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

We welcome contributions! Please see our [Discord community](https://discord.gg/okxdexapi) for technical discussions and support.

### Ways to Contribute

1. **Join Community Discussions** - Help other developers in our Discord
2. **Open Issues** - Report bugs or suggest features
3. **Submit Pull Requests** - Contribute code improvements

### Pull Request Guidelines

- Discuss non-trivial changes in an issue first
- Include tests for new functionality
- Update documentation as needed
- Add a changelog entry describing your changes
