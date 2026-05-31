# liquidityOrchestrate


A production-oriented Uniswap v4 hook that automatically moves idle LP liquidity into yield-generating strategies such as Aave and ERC4626 vaults, then restores liquidity when positions become active again.

The hook combines:

* **Uniswap v4 Hooks**
* **Chainlink Automation**
* **Aave V3**
* **ERC4626 Vaults**
* **Oracle-based safety checks**
* **Modular strategy execution**

to create an automated idle liquidity management system for concentrated liquidity LPs.

---



                    ┌──────────────────┐
                    │ Uniswap v4 Pool  │
                    └─────────┬────────┘
                              │
                              ▼
                  ┌──────────────────────┐
                  │ Idle Liquidity Hook  │
                  └─────────┬────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
 ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
 │ LP Registry  │  │ Tick Monitor │  │ Yield Engine │
 └──────────────┘  └──────────────┘  └──────────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Rebalance Engine    │
                 └─────────┬───────────┘
                           │
                           ▼
                 ┌─────────────────────┐
                 │ Strategy Manager    │
                 └─────────┬───────────┘
                           │
            ┌──────────────┴──────────────┐
            ▼                             ▼
      ┌───────────┐               ┌───────────┐
      │   Aave    │               │ ERC4626   │
      └───────────┘               └───────────┘







## Architecture Overview

### Core Flow

1. LP adds liquidity to a Uniswap v4 pool
2. Hook tracks the LP position
3. Swaps trigger update requests
4. Chainlink Automation calls rebalance
5. Out-of-range liquidity is moved to:

   * Aave V3
   * ERC4626 vaults
   * Custom strategies
6. Yield accrues while liquidity is idle
7. Liquidity is restored when position becomes active again

---

# Features

## Automated Idle Liquidity Management

Detects inactive LP positions and reallocates assets into yield strategies automatically.

## Strategy Support

### Aave V3

Supports depositing idle liquidity into lending markets.

* aToken accounting
* principal tracking
* withdrawal synchronization

### ERC4626 Vaults

Supports vault-based yield strategies.

* share accounting
* asset conversion handling
* vault failure protection

### Custom Strategies

Modular strategy architecture using delegatecall executors.

---

## Chainlink Automation Integration

Uses keeper-compatible automation for:

* periodic rebalancing
* pool scanning
* batched maintenance
* decentralized execution

---

## Oracle Safety

Chainlink price feeds are used for:

* price validation
* deviation checks
* stale data protection

---

## Security Features

### Trusted Counterparties

Whitelisted:

* Aave pools
* ERC4626 vaults

to reduce griefing and malicious integrations.

### Emergency Pause

Protocol owner can pause operations during emergencies.

### Reentrancy Protection

Critical operations use OpenZeppelin ReentrancyGuard.

### Storage Spam Protection

Pools must be explicitly approved before tracking.

### LP Limits

Caps maximum tracked LPs per pool.

---

# Contract Inheritance

```solidity
contract IdleLiquidityHookEnterprise is
    BaseHook,
    IdleLiquidityRebalanceEngine,
    ChainlinkAutomation
```

---

# Hook Permissions

The hook enables:

| Hook                  | Enabled |
| --------------------- | ------- |
| afterSwap             | ✅       |
| afterAddLiquidity     | ✅       |
| beforeSwap            | ❌       |
| beforeAddLiquidity    | ❌       |
| beforeRemoveLiquidity | ❌       |
| afterRemoveLiquidity  | ❌       |
| donate hooks          | ❌       |

---

# Key Components

## Position Tracking

Tracks:

* liquidity amounts
* tick ranges
* vault shares
* Aave principal
* accumulated yield
* LP status

---

## Rebalance Engine

Responsible for:

* detecting inactive liquidity
* moving assets into yield strategies
* restoring liquidity when active again

---

## Yield Accounting

Supports:

* global yield indexes
* LP-specific accrual
* protocol revenue share
* strategy accounting

---

# Supported Strategies

## Aave Strategy

Idle liquidity can be supplied into Aave lending markets.

### Accounting

```solidity
totalATokenPrincipal
```

Tracks total supplied principal.

---

## ERC4626 Strategy

Supports vault-based strategies with share accounting.

### Accounting

```solidity
totalVaultShares
```

Tracks deposited vault shares.

---

# Events

## RebalanceAttempt

```solidity
event RebalanceAttempt(PoolId indexed pid, address indexed lp);
```

Emitted when a rebalance starts.

---

## PositionRegistered

```solidity
event PositionRegistered(
    PoolId indexed pid,
    address indexed lp,
    address caller,
    uint128 liquidity0,
    uint128 liquidity1,
    int24 lower,
    int24 upper
);
```

Emitted when an LP position is tracked.

---

## EmergencyPauseSet

```solidity
event EmergencyPauseSet(bool paused);
```

Emitted when emergency mode changes.

---

# Pool Lifecycle

## 1. LP Adds Liquidity

`afterAddLiquidity()` registers the position.

---

## 2. Swaps Trigger Updates

`afterSwap()` marks the pool for rebalancing.

---

## 3. Keeper Executes Rebalance

Chainlink Automation calls:

```solidity
performUpkeep()
```

---

## 4. Idle Liquidity Gets Deposited

Assets are routed into:

* Aave
* ERC4626 vaults
* custom strategies

---

## 5. Yield Accrues

Yield indexes are updated and LPs accumulate rewards.

---

## 6. Liquidity Returns Active

Funds are withdrawn from strategies and restored.

---

# Admin Functions

## Strategy Configuration

```solidity
setAaveStrategy(address)
setERC4626Strategy(address)
setStrategyManager(address)
```

---

## Pool Configuration

```solidity
setPoolConfigAave(...)
setPoolConfigAaveSelective(...)
updateRates(...)
updateRatesBatch(...)
```

---

## Trusted Counterparties

```solidity
setTrustedAavePool(address,bool)
setTrustedERC4626Vault(address,bool)
```

---

## Emergency Controls

```solidity
setEmergencyPause(bool)
```

---

# Keeper Automation

The contract implements automated upkeep scanning.

## Upkeep Logic

```solidity
_checkUpkeep()
_performUpkeep()
```

### Features

* rotating pool cursor
* bounded scanning
* gas-aware batching
* anti-spam update throttling

---

# Oracle System

Uses Chainlink aggregators for:

* price retrieval
* deviation checks
* stale feed protection

## Configuration

```solidity
setPriceFeed(asset, feed)
```

---

# Testing Utilities

Includes helper functions for testing environments:

```solidity
registerPosition()
registerLP()
clearPosition()
setAccountingForTest()
setPositionStatusForTest()
```

These simplify integration and simulation testing.

---

# Security Considerations

## Delegatecall Usage

Strategies execute via delegatecall.

Only trusted and audited strategy implementations should be used.

---

## Oracle Dependency

Incorrect oracle configuration can impact rebalancing logic.

Use reliable Chainlink feeds only.

---

## Vault Risk

ERC4626 vault behavior depends on external implementations.

Always whitelist trusted vaults.

---

# Example Integration Flow

```solidity
// 1. Configure strategies
hook.setStrategyManager(strategyManager);

// 2. Configure pool
hook.setPoolConfigAave(
    pid,
    0,
    asset,
    aavePool,
    aToken,
    9000,
    1000
);

// 3. Add Chainlink feed
hook.setPriceFeed(asset, feed);

// 4. LP adds liquidity
poolManager.modifyLiquidity(...);

// 5. Keeper performs rebalance
performUpkeep(...);
```

---

# Dependencies

Built with:

* Uniswap v4
* Aave
* Chainlink
* OpenZeppelin

---

# Use Cases

## Yield-Optimized LP Infrastructure

Improve capital efficiency for concentrated liquidity providers.

---

## Automated Treasury Liquidity

Protocols can automate treasury LP positions.

---

## Institutional LP Management

Enable passive yield generation for inactive liquidity.

---

# Future Improvements

* multi-strategy routing
* dynamic risk scoring
* cross-chain automation
* AI-assisted rebalance optimization
* protocol fee distribution
* governance-controlled strategy allocation

---

# License

MIT

---

# Disclaimer

This protocol interacts with external DeFi systems and uses delegatecall-based strategy execution.

Use only audited integrations and thoroughly test before deploying to production environments.




## Foundry

**Foundry is a blazing fast, portable and modular toolkit for Ethereum application development written in Rust.**

Foundry consists of:

- **Forge**: Ethereum testing framework (like Truffle, Hardhat and DappTools).
- **Cast**: Swiss army knife for interacting with EVM smart contracts, sending transactions and getting chain data.
- **Anvil**: Local Ethereum node, akin to Ganache, Hardhat Network.
- **Chisel**: Fast, utilitarian, and verbose solidity REPL.

## Documentation

https://book.getfoundry.sh/

## Usage

### Build

```shell
$ forge build
```

### Test

```shell
$ forge test
```

### Format

```shell
$ forge fmt
```

### Gas Snapshots

```shell
$ forge snapshot
```

### Anvil

```shell
$ anvil
```

### Deploy

```shell
$ forge script script/Counter.s.sol:CounterScript --rpc-url <your_rpc_url> --private-key <your_private_key>
```

### Cast

```shell
$ cast <subcommand>
```

### Help

```shell
$ forge --help
$ anvil --help
$ cast --help
```
