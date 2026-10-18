# RingFence by NeuroNova

Cross-bank money-mule detection with minimal data sharing, built on IBM LinuxONE for the IBM Z Datathon 2026.

Team: Ritika S,Lavanya A,Krashanya G,Varshini K M, Saveetha Engineering College, Chennai

## Problem
Money mules split stolen cash across accounts in many banks. Each bank sees only a small, normal-looking transfer, so rings go unseen. Banks cannot share customer data because of privacy rules.

## Our approach
1. Each account gets behaviour features: number of payments, fan-in and fan-out (how many different accounts it pays or is paid by), and total amounts.
2. A LightGBM model scores each account for laundering risk.
3. A bank shares only a keyed-hash token and a risk flag. No names, account numbers, balances or histories are shared.
4. Matching tokens that paid each other are linked across banks to reveal possible rings.
5. We replay the transactions in time order and measure how much money a ring block would stop.

## Where it runs
Everything ran in JupyterLab on an IBM LinuxONE Community Cloud virtual machine (s390x processor), using Docker and the LinuxONE JupyterLab image. Screenshots are in the `screenshots` folder.

## Data
IBM synthetic AML transactions (HI-Small version) from Kaggle, "Transactions for Anti Money Laundering", licence CDLA-Sharing-1.0. The file is not included here because of its size. Download it from Kaggle and upload it to JupyterLab.
No real customer data is used. Bank IDs in the dataset act as our simulated banks.

## Results (synthetic data)
- Privacy attack test: an attacker who knows all 515,088 account IDs cracked 2000 of 2000 plain SHA-256 tokens but 0 of 2000 keyed (HMAC) tokens. This tests dictionary guessing only, and keyed hashing is safe only while the key stays secret.


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

