# 🛡️ Wallet Risk Scoring From Scratch (Compound V2)

## 📚 Overview

This project develops a **risk scoring system** for Ethereum wallets interacting with the **Compound V2 lending protocol**. Given on-chain behavior, each wallet receives a **risk score between 0 (highest risk) and 1000 (lowest risk)**. The score reflects the wallet's historical reliability, transaction patterns, and behavior with respect to lending/borrowing activities.

---

## 📥 Data Collection

- **Source**: [Covalent API](https://www.covalenthq.com/docs/api/)
- For each wallet in the provided list, the following was retrieved:
  - **Transaction history** (`/v1/1/address/{address}/transactions_v2/`)
  - Events involving **Compound V2**, including:
    - Supplying or withdrawing assets
    - Borrowing or repaying
    - Liquidation events

> ✅ Covalent API was used due to its structured access to decoded on-chain data across DeFi protocols.

---

## 🧠 Feature Selection Rationale

We selected features based on their **predictive power for risk** in the context of lending/borrowing behavior:

| Feature | Description | Rationale |
|--------|-------------|-----------|
| `wallet_age_days` | Time since the first on-chain activity | Older wallets are generally more trustworthy due to long history |
| `tx_count` | Total number of all on-chain transactions | High activity shows greater involvement and possibly trust |
| `tx_count_30d` | Recent transactions (last 30 days) | Captures wallet's recent engagement and behavior |
| `average_gas_used` | Mean gas used per tx | Higher gas often implies more complex, higher-value actions |
| `liquidation_events` | Number of liquidations in Compound V2 | Direct negative signal — indicates risky borrowing behavior |

---

## ⚙️ Feature Engineering & Normalization

### Transformations:

- Applied `log1p()` (log(x+1)) to reduce outlier skew:
  - `log(wallet_age_days + 1)`
  - `log(tx_count + 1)`
  - `log(tx_count_30d + 1)`
  - `log(average_gas_used + 1)`
  
- Used **MinMaxScaler** to scale transformed features to the range [0, 1].

---

## 🧮 Scoring Method

We used a **weighted aggregation model** to compute each wallet’s risk score:

### 🔢 Formula:

\[
\text{Score} = 1000 \times \sum (\text{scaled feature}_i \times \text{weight}_i) - (50 \times \text{liquidation events})
\]

### 📊 Feature Weights:

| Feature        | Weight |
|----------------|--------|
| `log_age`      | 0.3    |
| `log_tx`       | 0.2    |
| `log_tx_30d`   | 0.2    |
| `log_gas`      | 0.3    |

### 🧯 Liquidation Penalty:
- **50 points** are subtracted for **each** liquidation event.

### ⛔ Clipping:
- Final score is clipped between **100 (min)** and **1000 (max)** to avoid extreme values.

---

## 📈 What Does the Score Mean?

| Score Range | Interpretation |
|-------------|----------------|
| **900–1000** | Very low risk – stable, old, active wallet with no negative history |
| **700–899**  | Moderate risk – mostly healthy activity, but some inactivity or minor red flags |
| **500–699**  | Elevated risk – limited history, inconsistent usage, or recent liquidations |
| **100–499**  | High risk – recent or frequent liquidations, new wallet, or low activity |

> 🔍 **Use Case:** Lenders, protocols, or DeFi risk engines can use this score to assess trustworthiness before interacting with a wallet.

---


## 📁 Output Format

The final output CSV file is wallet_risk_scoring.csv which u can find it in this repo.
The final output CSV file contains:

```csv
wallet_id,score
0xfaa0768bde629806739c3a4620656c5d26f44ef2,732
...

