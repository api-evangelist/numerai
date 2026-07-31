---
name: Submit Numerai Signals predictions
description: Upload stock-signal predictions to Numerai Signals over the GraphQL API.
api: https://api-tournament.numer.ai
operations: [submissionUploadSignalsAuth, createSignalsSubmission, signalsLeaderboard]
---

# Submit Numerai Signals predictions

Numerai **Signals** is the stock-market tournament: you map your own signals to Numerai tickers and
submit per round via the same GraphQL endpoint (`https://api-tournament.numer.ai`). Auth is the
API token pair (`x-public-id` / `x-secret-key`) with the `upload_submission` scope.

## Steps

1. **Build your submission** — a CSV mapping tickers to your signal values for the open round.
2. **Get an upload slot** — call `submissionUploadSignalsAuth` for your Signals model to get a signed URL.
3. **Upload and register** — PUT the file to the signed URL, then call the `createSignalsSubmission`
   mutation with the model id.
4. **Track rank** — read `signalsLeaderboard` / `signalsLeaderboardOverview` for standing and payouts.

## Rules

- `createSignalsSubmission` is limited to 500/min per IP unless allowlisted
  (`rate-limits/numerai-rate-limits.yml`).
- One active submission per model/round; re-upload replaces it.
- Handle the GraphQL `errors[]` envelope (`errors/numerai-problem-types.yml`).
