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
- Replay with all 119 rings: 1.47 percent of laundering money blocked, but about 6 times more normal money wrongly blocked than laundering money blocked.
- Replay with the top 30 rings by average model risk: 1.47 percent of laundering money blocked, and wrongly blocked normal money dropped by about 97 percent. This rule was chosen after seeing the results.
- The replay ignores payments from an account to itself.

## Limitations
- The account features use the whole dataset, including transactions after the moment a ring is detected, so the replay is a best-case illustration, not a live result.
- One LightGBM model was trained on all accounts together, not one model per bank.
- Rings were found from payment records and then shown as tokens, so this simulates the privacy design and is not a full privacy-preserving protocol.

## Files
- `krash.ipynb`: the full notebook with outputs
- `screenshots/`: proof of the run on LinuxONE
