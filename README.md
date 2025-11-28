# 🔮 Oracles

This repository contains a Foundry-based workshop project focused on **integrating and interacting with decentralized oracles**. The goal is to explore how to use Chainlink and Pyth oracles in Solidity smart contracts, both with and without additional safety checks.
It also includes a theoretical overview of what oracles are, why they are essential in blockchain systems, how they work, and a comparative analysis of the three most widely used oracle mechanisms in DeFi: **Chainlink**, **Pyth Network**, and **TWAP**.

## 🤔 What is an oracle?

An oracle is a **communication bridge** between on-chain smart contracts and the off-chain world, making external data available inside blockchain environments.

Smart contracts cannot access external information by themselves. They can only read the **deterministic** state of the blockchain they run on. This limitation is necessary for determinism and consensus, but it also restricts the usefulness of smart contracts in real-world scenarios.

However, many applications require real-world data, such as: cryptocurrency or asset prices, weather conditions, flight delays, sports results, market volatility, randomness, etc.

An oracle solves this limitation by **reading data from off-chain sources and making it available on-chain**, enabling smart contracts to react to real-world events.

Without this bridge, blockchain applications would be isolated systems incapable of interacting with real-world data.

## 🧩 Why are oracles important?

Oracles are fundamental because they enable blockchain systems to interact with the outside world.

If smart contracts only had access to on-chain data, their utility would be extremely limited.
Many of the most powerful use cases of blockchain originate precisely from the ability to automate logic based on external events.

Another practical case is flight insurance: a smart contract can automatically refund users if a flight arrives late. The logic is deterministic and transparent, but the actual event—“did the flight arrive late?”—comes from the off-chain world.
Only an oracle can deliver that information to the contract.

This is why oracles are not optional components but **core infrastructure for DeFi and real-world applications**. If the oracle fails, becomes manipulated or delivers outdated data, the smart contract logic becomes meaningless or even dangerous. A single compromised oracle has historically led to massive exploits, incorrect liquidations, and complete protocol collapses.

## ⚙️ How oracles work?

The main purpose of an oracle is always to **extract information that exists off-chain and make it available on-chain**. However, the way this is done varies depending on the oracle architecture.

Some oracles operate in a **pull-based model**: the network periodically publishes values on-chain and smart contracts simply read them when needed. Chainlink or TWAP follows this model.

Others use a **push-based model**: trusted actors or authorized updaters push new data to the oracle contract. Pyth Network follows this model.

Regardless of the mechanism, the sequence is conceptually the same:

1. An entity (node, signer, or market mechanism) obtains off-chain information.
2. It validates or produces a trustworthy representation of that information.
3. It publishes or makes the information available on-chain.
4. Smart contracts consume that value to execute logic or enforce rules.

Then, security, decentralization and reliability depend entirely on how these steps are implemented.

## 🔗 How Chainlink works?

Chainlink is the most **widely adopted oracle solution** in the blockchain industry and is considered the standard for DeFi. It was designed specifically to avoid the inherent weaknesses of centralized oracles, where a single source of truth could be manipulated or corrupted.

Chainlink operates through a **decentralized network of independent nodes**. Each of these nodes retrieves data from different sources like centralized exchanges, decentralized exchanges, market makers and other reliable providers. Because every node uses different sources, no single actor can control the final result. Once nodes report their values, Chainlink computes an aggregated price, usually through a form of **median or weighted average**, and this aggregated result is what gets published on-chain.

This design makes it extremely difficult for an attacker to manipulate the price, because they would need to corrupt a majority of the Chainlink node network and the underlying data sources. Additionally, the network incorporates an economic incentive layer: **Chainlink nodes earn LINK tokens** for their honest reporting. Attempting to cheat the system is more costly than reporting data correctly, following game-theoretic principles.

Because Chainlink publishes data periodically, smart contracts simply read the most recent, validated price using the `.latestRoundData()` function. This architecture makes Chainlink robust, predictable and resistant to the types of attacks that affect centralized or simplistic oracle models.

## ⚡ Comparison between Chainlink vs Pyth Network vs TWAP

| Feature                     | **Chainlink**                                              | **Pyth Network**                                           | **TWAP**                                              |
| --------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------- |
| **Source of Data**          | Multiple off-chain providers (highly decentralized)        | Trusted market makers / exchanges (signed data)            | On-chain DEX liquidity pools                          |
| **Update Mechanism**        | **Pull-based** (feeds updated periodically by the network) | **Push-based** (requires signing + manual price update tx) | **Pull-based** (computed from on-chain price history) |
| **Manipulation Resistance** | Very high (hard to corrupt majority of nodes)              | Very high (data is signed and verified)                    | Moderate (depends on DEX liquidity + time window)     |
| **Freshness Guarantees**    | Designed uptime + heartbeat                                | Depends on who pushes updates                              | Depends on configured time window                     |
| **Primary Use Case**        | DeFi protocols, lending, liquidation checks                | High-frequency trading, derivatives                        | On-chain DEX price smoothing                          |
| **Weaknesses / Risks**      | Higher gas cost, external dependency                       | Risk from underlying bridge (Wormhole)                     | Vulnerable to MEV and short window manipulation       |
| **Decentralization Level**  | High                                                       | Medium                                                     | Depends on the DEX                                    |

### 🔍 Summary of Differences

- Chainlink → Most decentralized & secure. Ideal for lending, liquidations, collateral checks.
- Pyth → Fast data, cryptographically signed, but freshness depends on update transactions.
- TWAP → Fully on-chain, useful for smoothing volatility but vulnerable to manipulation if the window is too short (less than 30 minutes).

---

## 📂 Project Structure

The project includes four main Solidity contracts:

| Contract Name               | Description                                                                                        |
| --------------------------- | -------------------------------------------------------------------------------------------------- |
| `ChainlinkOracle.sol`       | Implements basic interaction with Chainlink oracle to fetch prices, **without any safety checks**. |
| `PythOracle.sol`            | Implements basic interaction with Pyth oracle to fetch prices, **without any safety checks**.      |
| `ChainlinkOracleChecks.sol` | Extends Chainlink oracle usage by adding **price validity and freshness checks** to ensure safety. |
| `PythOracleChecks.sol`      | Extends Pyth oracle usage by adding **price validity and freshness checks** for safer price reads. |

---

## 📄 Contract Details

### ChainlinkOracle.sol

- Simple price feed consumer for Chainlink.
- Calls Chainlink AggregatorV3Interface `latestRoundData()` directly.
- No validation or freshness checks.

### ChainlinkOracleChecks.sol

Wraps Chainlink oracle calls with checks:

- Do not use `.latestAnswer()`
- Wrap calls to `.latestRoundData()` inside a `try-catch` block
- Ensure the returned price is > 0
- Checks the timestamp to avoid using stale price
- Validates round IDs to prevent stale data
- If deploying on an L2 network, check the sequencer feed status.
- Do not assume that a stablecoin price is 1, check it with a priceFeed.
- Do not assume that the price of a wrapped coin is equal to its native asset, check it through a price feed.
- Do not assume that a price feed has 8 decimals, always check it with the `.decimals()` function
- Consider implementing a backup oracle in the `try-catch` block

### PythOracle.sol

- Basic integration with Pyth Oracle contract.
- Calls `getPrice()` without extra validations.
- Suitable for learning data fetching.

### PythOracleChecks.sol

Adds validation layers on top of `PythOracle.sol`:

- Checks for price validity with an `updatePrice()` function
- `pyth.updatePriceFeeds{value: fee}(updateData)` functions requires a fee. Implement first the `.getUpdateFee()` function to get the value of the fee.
- Do not use `getPriceUnsafe()` as it not checks if the price is stale
- Use instead the `.getPriceNoOlderThan()` with the priceId and the number of seconds.
- Check the pyth response with an internal `_validatePrice` function:
- Ensure the price is greater than 0 - Ensure the exponent is greater than the minimum one - Ensure the confidence ratio is greater than the minimum one

---

## 📚 Additional Resources

- [Chainlink Docs](https://docs.chain.link/docs)
- [Pyth Network Docs](https://docs.pyth.network/)
- [Zokyo Oracle Security Workshop](https://www.youtube.com/watch?v=G8Oj3y6QZKQ&t=61s)
- [Foundry Book](https://book.getfoundry.sh/)
- [Solidity Docs](https://docs.soliditylang.org/)
