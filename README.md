# Wallet Risk Scoring

This project implements a comprehensive Wallet Risk Scoring solution that analyzes Ethereum wallet activity on the Compound V2 and V3 lending protocols. The aim is to compute a risk score (0 to 1000) for each wallet based on their borrowing, repayment, liquidation, and overall protocol usage behavior.

---

## Table of Contents

- [Project Overview](#project-overview)  
- [Data Collection Method](#data-collection-method)  
- [Feature Selection Rationale](#feature-selection-rationale)  
- [Scoring Method](#scoring-method)  
- [Justification of Risk Indicators](#justification-of-risk-indicators)  
- [Usage Instructions](#usage-instructions)  
- [Repository Structure](#repository-structure)  
- [Dependencies](#dependencies)  
- [Limitations and Future Improvements](#limitations-and-future-improvements)  

---

## Project Overview

This system fetches transaction histories for a set of Ethereum wallets interacting with Compound V2 and V3 subgraphs using The Graph Protocol. It extracts meaningful on-chain events such as borrows, repayments, and liquidations, and processes these into key features. These features are normalized and aggregated through a weighted scoring model to output a wallet risk score indicating the likelihood of risky borrowing behavior.

---

## Data Collection Method

- **Wallet List:** The input to the system is a CSV file containing 100 wallet addresses.  
- **Data Source:** For each wallet, data is fetched via GraphQL queries to Compound V2 and V3 subgraphs hosted on The Graph Protocol.  
- **Events Queried:** BorrowEvents, RepayEvents, LiquidateEvents, and overall transaction timestamps.  
- **Tools:** Python scripts using `requests` to send queries and `pandas` to handle data.

---

## Feature Selection Rationale

The features engineered to characterize wallet risk are:

- **Number of Liquidations:** Direct count of times the wallet was liquidated—strongest risk indicator.  
- **Late Repayment Ratio:** Proportion of borrowed amounts not repaid promptly, reflecting reliability.  
- **Average Collateral Utilization:** Proxy for leverage intensity, estimated via borrow-to-repay ratio.  
- **Borrow-to-Repay Ratio:** High values indicate possible overextension or negligence in repayments.  
- **Largest Single Borrow:** Largest loan amount taken, showing exposure concentration.  
- **Wallet Activity Duration:** Time span (in days) between wallet’s first and last interaction with the protocol; longer duration often indicates safer behavior.

---

## Scoring Method

- **Normalization:** All features are scaled to the range \[0,1\] using min-max normalization; features like activity duration are inverted so that higher values consistently indicate higher risk.  
- **Weighted Aggregation:** Each feature is assigned an empirically chosen weight reflecting its impact:

| Feature                  | Weight  |
|--------------------------|---------|
| Number of Liquidations   | 30%     |
| Late Repayment Ratio      | 20%     |
| Average Collateral Utilization | 15% |
| Borrow-to-Repay Ratio    | 15%     |
| Largest Single Borrow    | 10%     |
| Activity Duration        | 10%     |

- **Formula:**

  $$
  \text{Risk Score} = 1000 \times \sum_i (w_i \cdot f_i)
  $$

  where $w_i$ is the weight and $f_i$ is the normalized feature value.

- Scores range from 0 (lowest risk) to 1000 (highest risk) per wallet.

---

## Justification of Risk Indicators

- **Liquidations:** Direct signal of financial failure and high risk.  
- **Repayment Behavior:** Late or missed repayments indicate potential default risk.  
- **Collateral Utilization & Borrow Ratios:** Reflect the wallet’s leverage and prudence in borrowing.  
- **Largest Borrow:** Large single borrow events increase exposure risk.  
- **Activity Duration:** Experienced borrowers generally exhibit safer, more responsible behavior.

---

## Usage Instructions

1. **Prepare Wallet List:** Put your list of wallet addresses in a CSV file named `Wallet-id-Sheet1.csv` under the `data/` directory with a column named `wallet_id`.  
2. **Install Dependencies:**
pip install -r requirements.txt


3. **Run the Risk Scoring Script:**

python src/wallet_risk_scoring.py


4. **Results:** The script outputs `wallet_risk_scores.csv` into the `data/` directory containing two columns: `wallet_id` and `score`.

---

## Repository Structure
Wallet-Scoring/
│
├── README.md
├── requirements.txt
├── data/
│ ├── Wallet-id-Sheet1.csv # Input wallet addresses
│ └── wallet_risk_scores.csv # Output risk scores
├── src/
│ └── wallet_risk_scoring.py # Main risk scoring script
└── docs/
└── methodology.md # Detailed explanation (optional)

---

## Dependencies

- Python 3.x  
- `pandas`  
- `requests`

Install dependencies via:

pip install -r requirements.txt

---

## Limitations and Future Improvements

- **Price Conversion:** Currently, borrow amounts are raw token values. For better accuracy, integrating USD price feeds or oracles is recommended.  
- **Pagination & API Rate Limits:** For wallets with large histories, GraphQL query pagination and rate limit handling should be added.  
- **Extended Features:** Additional features (collateral supplied, market diversification, historical volatility) can improve risk stratification.  
- **Error Handling & Performance:** Advanced retry logic, parallel requests, and logging would enhance robustness.

---

*For questions or contributions, feel free to open issues or pull requests.*

---

**License:** MIT (or specify your license)

---

*Thank you for exploring Wallet Risk Scoring!*
