---
name: Automate submissions with a Compute Prediction Node
description: Register a webhook so Numerai triggers a cloud Prediction Node to compute and submit each round automatically.
api: https://api-tournament.numer.ai
operations: [addModel, setSubmissionWebhook, triggerModelWebhook, triggerLogs, pipelineStatus]
---

# Automate submissions with Numerai Compute

Numerai Compute runs your model in the cloud and **calls a webhook you register each round** so
submissions happen without you being online. The official `numerai-cli` deploys the node (AWS/Azure)
and wires the webhook; the GraphQL mutations below are what it calls under the hood.

## Steps

1. **Create the model** (if needed) — `addModel` mutation registers a named model.
2. **Deploy the node** — `pip install numerai-cli`, then `numerai node config` / `numerai node deploy`
   builds a Docker image and provisions the cloud endpoint (see `cli/numerai-cli.yml`).
3. **Register the webhook** — `setSubmissionWebhook` mutation binds the node's URL to the model.
   Numerai invokes it when a new round opens.
4. **Test / trigger** — `triggerModelWebhook` fires the node manually; `numerai node test` validates it.
5. **Observe** — poll `pipelineStatus` and `triggerLogs` to confirm the node ran and submitted.

## Rules

- Auth: API token pair with `upload_submission` (and `download_submission`) scope.
- Direction is **Numerai -> your node** (outbound webhook); there is no inbound event subscription.
  See `asyncapi/numerai-webhooks.yml`.
- `setSubmissionWebhook` and `triggerModelWebhook` are limited to 50/min per IP.
