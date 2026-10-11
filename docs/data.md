# Data

**Source:** [PaySim synthetic financial dataset](https://www.kaggle.com/datasets/ealaxi/paysim1) (Kaggle), licensed **CC BY-SA 4.0**. PaySim simulates one month of mobile-money transactions from an African mobile-money service. The data is synthetic, so it contains no real customer information.

**Location:** `data/paysim.csv` (gitignored, about 494 MB). To download it:

```bash
# Needs KAGGLE_USERNAME and KAGGLE_KEY in .env
set -a && . ./.env && set +a
uvx kaggle datasets download -d ealaxi/paysim1 -p data --unzip
mv data/PS_20174392719_1491204439457_log.csv data/paysim.csv
```

## Profile

| | |
|---|---|
| Rows | 6,362,620 |
| Columns | 11 |
| Null values | 0 |
| Fraud rows (`isFraud = 1`) | 8,213 (**0.129 %**, about 1 in 775) |
| Rows flagged by the simulator's rule (`isFlaggedFraud = 1`) | 16 |
| Time span | `step` 1 to 743. One step is one hour, so the data covers about 31 days |

## Columns

| Column | Type | Meaning |
|---|---|---|
| `step` | int | Hour of the simulation (1 to 743) |
| `type` | str | `CASH_IN`, `CASH_OUT`, `DEBIT`, `PAYMENT`, `TRANSFER` |
| `amount` | float | Transaction amount |
| `nameOrig` | str | Sending account (`C…` = customer) |
| `oldbalanceOrg`, `newbalanceOrig` | float | Sender balance before and after the transaction |
| `nameDest` | str | Receiving account (`C…` = customer, `M…` = merchant) |
| `oldbalanceDest`, `newbalanceDest` | float | Receiver balance before and after (always 0 for merchants) |
| `isFraud` | int | **Label**: 1 = fraudulent |
| `isFlaggedFraud` | int | The simulator's built-in rule (transfers over 200,000). Used only as a baseline to compare against, never as a model feature |

## Fraud by transaction type

| type | legit | fraud |
|---|---:|---:|
| CASH_IN | 1,399,284 | 0 |
| CASH_OUT | 2,233,384 | 4,116 |
| DEBIT | 41,432 | 0 |
| PAYMENT | 2,151,495 | 0 |
| TRANSFER | 528,812 | 4,097 |

All fraud happens in `TRANSFER` and `CASH_OUT` transactions. The usual pattern is that an account is taken over, its money is transferred to a mule account, and the mule cashes it out.

## Implications for the platform

- **Use precision–recall metrics, not accuracy.** With 0.13 % fraud, a model that always predicts "legit" is 99.87 % accurate and useless. Models are judged on PR-AUC and on recall at a fixed 1 % false-positive rate, which is what an operations team can review.
- **Split by time, not at random.** Train on earlier steps and test on later ones. Real models score future transactions, and a random split leaks future patterns into training.
- **Compute account aggregates on the destination account.** Of 6.35 M senders, only 9,298 appear more than once, so velocity features computed per sender (for example, transactions in the last hour) would almost always be empty. Destination accounts repeat far more often (2.7 M unique accounts across 6.36 M rows), so the Phase 2 feature store keys its 1h/24h aggregates on `nameDest`. ADR-002 will record this decision.
