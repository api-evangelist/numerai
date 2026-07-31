---
name: Download data and submit tournament predictions
description: Authenticate, download the current round dataset, and upload a model's predictions to the Numerai Classic tournament.
api: https://api-tournament.numer.ai
operations: [listDatasets, dataset, submissionUploadAuth, createSubmission, submissionScores]
---

# Submit Numerai tournament predictions

The Numerai API is a single GraphQL endpoint at `https://api-tournament.numer.ai` (POST, `application/json`).
Programmatic access uses an **API token pair** (public id + secret key) created in Account > Settings,
sent as `x-public-id` / `x-secret-key` headers. The token needs the `upload_submission` scope
(and `download_submission` to fetch prior submissions). See `scopes/numerai-scopes.yml`.

The `numerapi` Python client wraps every step below.

## Steps

1. **List the current datasets** — call `listDatasets` (and `dataset`) to resolve the training/live
   feature files for the open round (Parquet).
2. **Download the live data** — fetch the resolved dataset URL and run your trained model to produce
   a CSV of `id,prediction` rows.
3. **Get an upload slot** — call `submissionUploadAuth` for the target model to obtain a signed upload URL.
4. **Submit** — call the `createSubmission` mutation with the model id and uploaded file to register
   the prediction for the current round. Only one active submission per model/round — a new upload
   replaces the prior one.
5. **Check scoring** — poll `submissionScores` / `submissions` after the round resolves to read
   correlation and performance.

## Rules

- **Rate limits**: `createSubmission` allows 500/min per IP (unless allowlisted). Back off on throttle.
  See `rate-limits/numerai-rate-limits.yml`.
- **No idempotency key**: submissions are round-scoped and last-write-wins; do not retry blindly.
- **Errors** arrive in the GraphQL `errors[]` envelope (HTTP 200). See `errors/numerai-problem-types.yml`.
