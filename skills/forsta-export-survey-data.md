---
name: Export Forsta survey data without timing out
description: >-
  Pull a survey's datamap and collected response data out of a Forsta Decipher instance, using the
  asynchronous task path so large exports survive the 30-minute synchronous timeout.
api: openapi/forsta-decipher-rest-api-openapi.yml
generated: '2026-09-10'
method: generated
source: derived from openapi/forsta-decipher-rest-api-openapi.yml (operationIds verified against the spec)
operations:
  - getSurveyDatamap
  - getSurveyLayouts
  - getSurveyData
  - getStatus
  - getStatusContent
  - deleteStatus
---

# Export Forsta survey data

## Why this needs a skill

Synchronous Decipher calls carry a **30-minute network timeout**, and converting a large dataset
to a proprietary format (SPSS SAV, for example) routinely takes longer. The provider's answer is
an opt-in task mode. Getting this wrong means a truncated connection and no data.

## Steps

1. **Read the shape first.** `getSurveyDatamap` (`GET /surveys/{survey}/datamap`) describes the
   survey's variables and question structure — you need it to interpret the rows. Optionally list
   the available layouts with `getSurveyLayouts` (`GET /surveys/{survey}/layouts`).
2. **Request the data as a task.** `getSurveyData` (`GET /surveys/{survey}/data`), and add
   **`forceTask=1`**. Any call accepts `forceTask` with any value; it turns the call into a
   background job and returns an `ident` immediately instead of streaming.
3. **Poll.** `getStatus` (`GET /api/v1/status`) with that `ident`. When the job is done, a simple
   JSON result comes back inside the status response itself.
4. **Fetch a file result.** If the result is a zip rather than JSON, call `getStatusContent`
   (`GET /api/v1/status/content`) to retrieve it.
5. **Abandon a job** you no longer need with `deleteStatus` (`DELETE /api/v1/status`).

## Format options that avoid work later

- `select=a,b,c` — projection; every API supports it. Ask only for the fields you will use.
- `top=<n>` — return only the first n objects of an array response.
- `contentType=excel` — any array-returning call can come back as Excel 2007 instead of JSON.
- `accept: application/xml` — any resource can come back as XML in the IBM JSONX serialization.

## Safety

- **Pagination does not exist.** No cursor, no offset, no page parameter, no total-count field
  anywhere in the contract. `top` truncates; it does not page. Large result sets are handled by
  the task mechanism above, not by iterating.
- Data export is a read. It is safe to retry — unlike the write operations elsewhere in this API,
  which have no idempotency protection.
- Watch `x-usage-today` on each response: exports consume resource units as well as calls, and
  exceeding a contractual monthly allowance returns **402**, not 429.
