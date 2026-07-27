---
name: List, review, and retry submissions
description: Find submissions with keyset pagination and filters, submit review changes, and retry failed submissions.
api: openapi/indico-data-openapi.yml
operations: [refresh_token, get_submissions, submit_review, retry_submission]
---

# List, review, and retry submissions

Use this skill to manage a queue of Indico workflow submissions.

## Auth
Exchange your API token for a JWT via `POST /api/v1/auth/refreshToken` (`refresh_token`, HTTP Basic), then send `Authorization: Bearer <access_token>` (15-minute TTL).

## Steps
1. **List / filter** — `GET /api/v1/submissions/` (`get_submissions`). Filter with `workflowId`, `submissionId`, `inputFilename`, `status`, `retrieved`. Paginate with keyset params: `limit` (default 1000), `after` (cursor), `desc`, `orderBy`. Response is a `Page[Submission]`.
2. **Review** — for submissions in a review status, `POST /api/v1/submissions/{id}/review` (`submit_review`) to submit review changes.
3. **Retry failures** — collect IDs whose `status` is `FAILED` and `POST /api/v1/submissions/retry` (`retry_submission`) with the `submission_ids` query list. It returns the retried submissions.

## Rules
- Errors: `422` = FastAPI validation envelope `{ "detail": [...] }`; `401` = expired/missing JWT.
- Iterate pages by passing the last-seen position as `after`; stop when a page returns fewer than `limit` items.
