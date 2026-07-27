---
name: Submit a document and retrieve results
description: Submit one or more files to an Indico workflow, poll until processing completes, then retrieve the structured JSON result.
api: openapi/indico-data-openapi.yml
operations: [refresh_token, submit_to_workflow, get_submission, get_submission_result, mark_submission_retrieved]
---

# Submit a document and retrieve results

Use this skill to run a document through an Indico intelligent-document-processing workflow and get structured data back.

## Auth
1. Obtain a JWT: `POST /api/v1/auth/refreshToken` (`refresh_token`) using HTTP Basic Auth with your Indico API token. Read `access_token` from the response — it expires after 15 minutes, so refresh when a call returns 401.
2. Send `Authorization: Bearer <access_token>` on every subsequent call.

## Steps
1. **Submit** — `POST /api/v1/workflows/{id}/submissions/` (`submit_to_workflow`) with `multipart/form-data` `files[]`. The response is an array of integer submission IDs.
2. **Poll** — call `GET /api/v1/submissions/{id}` (`get_submission`) until `status` is `COMPLETE` (or a review status). Statuses progress `PROCESSING -> PENDING_REVIEW -> COMPLETE`; treat `FAILED` as terminal. There are no webhooks — polling is the model.
3. **Retrieve result** — `GET /api/v1/submissions/{id}/result` (`get_submission_result`) to generate/fetch the result file, then download it via the storage proxy if it returns a storage URL.
4. **Acknowledge** — `PUT /api/v1/submissions/{id}/retrieved` (`mark_submission_retrieved`) so the submission is marked retrieved.

## Rules
- Errors: `422` returns a FastAPI validation envelope `{ "detail": [ {loc,msg,type} ] }`; `401` means the JWT is missing/expired.
- No idempotency key exists — do not blind-retry `submit_to_workflow`; it creates a new submission each call.
