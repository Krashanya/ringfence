# RingFence by NeuroNova

Cross-bank money-mule detection with minimal data sharing, built on IBM LinuxONE for the IBM Z Datathon 2026.

Team: Ritika S, Lavanya A, Krashanya G, Varshini K M, Saveetha Engineering College, Chennai

## Problem
Money mules split stolen cash across accounts in many banks. Each bank sees only a small, normal-looking transfer, so rings go unseen. Banks cannot share customer data because of privacy rules.

## Our approach
1. Each account gets behaviour features: number of payments, fan-in and fan-out (how many different accounts it pays or is paid by), and total amounts.
2. A LightGBM model scores each account for laundering risk.
3. A bank shares only a keyed-hash token and a risk flag. No names, account numbers, balances or histories are shared.
4. Matching tokens that paid each other are linked across banks to reveal possible rings.
5. We replay the transactions in time order and measure how much money a ring block would stop.

## Where it runs
Everything ran in JupyterLab on an IBM LinuxONE Community Cloud virtual machine (s390x processor), using Docker and the LinuxONE JupyterLab image.

## Data
IBM synthetic AML transactions (HI-Small version) from Kaggle, "Transactions for Anti Money Laundering", licence CDLA-Sharing-1.0. The file is not included here because of its size. Download it from Kaggle and upload it to JupyterLab.
No real customer data is used. Bank IDs in the dataset act as our simulated banks.

## Results (synthetic data)

**Data and model**
- 515,088 accounts, of which 6,357 (about 1.2 percent) are linked to laundering.
- LightGBM with 6 account features (payment counts, fan-in, fan-out, amounts), trained on 70 percent of accounts and tested on 30 percent: AUC 0.842.

| Cutoff | Accounts flagged (of 154,527 test accounts) | Precision | Recall |
|---|---|---|---|
| 0.5 | 32,471 | 0.042 | 0.719 |
| 0.8 (used for the demo) | 7,964 | 0.078 | 0.325 |
| 0.9 | 892 | 0.145 | 0.068 |

**Privacy design**
- A bank shares only a keyed-hash (HMAC-SHA256) token and a risk flag. No names, account numbers, balances or histories are shared.
- Attack test: an attacker who knows all 515,088 account IDs cracked 2000 of 2000 plain SHA-256 tokens but 0 of 2000 keyed tokens. This tests dictionary guessing only.

**Cross-bank rings**
- Linking flagged tokens that paid each other gave 120 possible rings with 3 or more accounts, and 119 of them cross 2 or more banks.
- Example ring: 12 accounts across 9 banks, and 10 of the 12 are truly laundering (checked against the dataset labels).

**Replay in time order** (a ring is detected at its second linking payment, later payments to or from ring accounts are blocked, payments from an account to itself are ignored)

| Rings blocked | Laundering money blocked | Normal money wrongly blocked |
|---|---|---|
| All 119 rings | 2.75 billion (1.47 percent of all laundering money) | 16.82 billion |
| Top 60 rings by average model risk | 2.75 billion (1.47 percent) | 2.25 billion |
| Top 30 rings by average model risk | 2.75 billion (1.47 percent) | 0.48 billion |

- Blocking only the top 30 rings keeps the same laundering money blocked and cuts wrongly blocked normal money by about 97 percent. This rule was chosen after seeing the results.
- Amounts are money units from mixed currencies, not rupees.
- The replay is a best-case illustration, because the features use the whole dataset (see Limitations).

**Where it ran**
- All steps ran in JupyterLab on an IBM LinuxONE Community Cloud virtual machine (s390x processor). The notebook with all outputs is `krash.ipynb`, and screenshots are in the `screenshots.pdf'.


## Limitations
- This is a first prototype on synthetic data. It shows the approach works, not production performance.
- Precision is low, so many alerts are false alarms. At cutoff 0.5 the model had precision 0.042 and recall 0.719.
- Most laundering money moves before a ring becomes visible, so only about 1.47 percent of laundering money was blocked in the replay.
- Ranking rings by average model risk helped, but the "top 25 percent" rule was chosen after seeing the results.
- The account features use the whole dataset, including transactions after the moment a ring is detected, so the replay is a best-case illustration, not a live result.
- One LightGBM model was trained on all accounts together, not one model per bank.
- Rings were found from payment records and then shown as tokens, so this simulates the privacy design and is not a full privacy-preserving protocol.
- Only the held-out 30 percent of accounts were scored, so ring coverage is partial.
- The privacy attack test covers dictionary guessing only. Keyed hashing is safe only while the secret key stays secret, and tokens use one key inside the notebook. A real consortium would share a key through a secure channel.
- Amounts are money units from mixed currencies, not rupees.
- Planned next: one model per bank, a test of the smallest data that still breaks a ring, and a wider set of privacy attacks (amounts and timing).

## Files
- `krash.ipynb`: the full notebook with outputs
- `screenshots.pdf`: proof of the run on LinuxONE

