# 📦 VaultLayer Protocol

**Unlock the dormant value of your Bitcoin. Borrow against your BTC holdings without surrendering ownership.**

VaultLayer is a Clarity smart contract that enables non-custodial, overcollateralized borrowing against Bitcoin (BTC) using synthetic stablecoins. It is built on the Stacks blockchain, inheriting Bitcoin's finality and settlement assurances.

---

## 🔍 Overview

VaultLayer allows users to:

* **Deposit BTC** as collateral (off-chain, verified via bridge or proof).
* **Mint synthetic stablecoins** by opening a loan vault.
* **Maintain ownership and upside exposure** to BTC price appreciation.
* **Automatically manage liquidation risk** based on real-time price feeds.
* **Securely repay and reclaim collateral**, or allow liquidation when undercollateralized.

---

## ⚙️ System Architecture

```
+-------------+      +--------------------+       +-------------------+
|  BTC Holder | ---> |  VaultLayer        | <---> |  Price Oracle     |
+-------------+      |  (Clarity Contract)|       +-------------------+
                         |        ^
                         v        |
                    +--------------------+
                    |  Data Maps & State |
                    +--------------------+
```

* **BTC Holder** interacts with VaultLayer via Clarity functions (`deposit-collateral`, `request-loan`, `repay-loan`).
* **VaultLayer** manages loan positions, enforces collateral ratios, and processes repayments or liquidations.
* **Price Oracle** feeds the real-time BTC price for accurate collateral valuations (admin-controlled).

---

## 🏗️ Contract Architecture

### ✅ Constants

Defined constants cover protocol defaults, asset whitelist, and structured error codes for:

* Authorization errors
* Loan management issues
* Oracle input validation
* Protocol integrity assertions

### 🧠 State Variables

```clarity
platform-initialized           ;; Protocol setup flag
minimum-collateral-ratio       ;; e.g., 150%
liquidation-threshold          ;; e.g., 120%
platform-fee-rate              ;; Placeholder for protocol fees (future use)
total-btc-locked               ;; Aggregate collateral locked
total-loans-issued             ;; Sequential ID counter
```

### 🗂️ Data Maps

```clarity
loans             ;; All loan vaults, keyed by ID
user-loans        ;; Mapping of users to active loan IDs
collateral-prices ;; Oracle price feed (BTC, STX, etc.)
```

Each loan vault tracks:

* Borrower address
* Collateral amount (in BTC units)
* Loan amount (in synthetic stablecoin units)
* Interest rate
* Timestamps for interest accrual
* Status: "active", "repaid", or "liquidated"

### 🧾 Read-Only Functions

Used to query loan details, user positions, protocol stats, or valid assets:

* `get-loan-details`
* `get-user-loans`
* `get-platform-stats`
* `get-valid-assets`

---

## 🔁 Protocol Flow

### 1. Platform Initialization

Only the contract owner can call:

```clarity
(initialize-platform)
```

Initializes the protocol and allows collateral deposits and loan requests.

---

### 2. Collateral Deposit

```clarity
(deposit-collateral amount)
```

Tracks BTC deposited to the protocol (off-chain validation required).

---

### 3. Loan Issuance

```clarity
(request-loan collateral loan-amount)
```

* Uses on-chain BTC price from oracle
* Enforces minimum collateral ratio
* Registers loan ID and updates borrower state

---

### 4. Repayment

```clarity
(repay-loan loan-id amount)
```

* Checks authorization and active status
* Calculates interest since last block height
* Closes loan and releases collateral

---

### 5. Liquidation Check

Internal function used to monitor undercollateralized loans:

```clarity
(check-liquidation loan-id)
```

If a loan's collateral ratio drops below the `liquidation-threshold`, it is automatically:

* Marked as "liquidated"
* Removed from borrower's active loans
* Collateral retained for platform (future auction system could be implemented)

---

### 6. Oracle Price Feed (Admin-only)

```clarity
(update-price-feed asset price)
```

Enables the contract owner to update the BTC price.

---

## 🛡️ Risk Management

| Mechanism             | Description                                      |
| --------------------- | ------------------------------------------------ |
| Overcollateralization | Enforced via `minimum-collateral-ratio`          |
| Liquidation Threshold | Ensures timely liquidation to prevent insolvency |
| Interest Accrual      | Compound interest per block on outstanding loans |
| Oracle Dependency     | Only admin-controlled updates accepted           |

---

## 🔐 Access Control

* Only the contract **owner (deployer)** can:

  * Initialize the platform
  * Update critical risk parameters
  * Feed price data

All user-facing loan operations are **permissionless** once the protocol is initialized.

---

## 📊 Metrics & Telemetry

Use the following read-only functions to access protocol-wide metrics:

```clarity
(get-platform-stats)
```

Returns:

* Total BTC locked
* Total loans issued
* Current collateral ratio & liquidation threshold

---

## 🚧 Future Enhancements

This contract represents a core MVP architecture. Future upgrades may include:

* BTC peg verification (e.g., via sBTC, ZK-proofs)
* Vault liquidation auctions
* Multi-asset collateral support
* Decentralized price oracle integrations
* Fee-sharing or staking mechanics

---

## 📚 Developer Notes

* Written in **Clarity**, a decidable smart contract language for the Stacks blockchain.
* Loan amounts and collateral are tracked in **uint**. Use appropriate scaling off-chain.
* Ensure correct BTC proof-of-deposit via auxiliary mechanisms or sBTC bridges.
* No actual token minting occurs in this contract (loan "amounts" are synthetic representations).

---

## ✅ Deployment Checklist

| Step                     | Complete? |
| ------------------------ | --------- |
| Deploy contract          | ✅         |
| Initialize platform      | ⬜         |
| Set BTC price via oracle | ⬜         |
| Deposit collateral       | ⬜         |
| Request & repay loans    | ⬜         |
| Monitor liquidations     | ⬜         |

---

## 🧾 License

This protocol is provided for educational and experimental use. No warranty or guarantees implied. Auditing and formal verification recommended before mainnet deployment.

---

## 📫 Contact & Contribution

For issues, contributions, or integration help:

* **Stacks Discord**: [https://discord.gg/stacks](https://discord.gg/stacks)
* **GitHub Issues**: Open an issue or PR in this repo
* **Stackers.dev Forum**: Share feedback or request features
