---
name: Launch and monitor a Forsta (Decipher) survey
description: >-
  Create or locate a survey in a Forsta Decipher instance, read its pre-launch warnings, take it
  live, watch completions, and close it. Every step is a verified operation in the published
  Decipher REST API contract.
api: openapi/forsta-decipher-rest-api-openapi.yml
generated: '2026-09-10'
method: generated
source: derived from openapi/forsta-decipher-rest-api-openapi.yml (operationIds verified against the spec)
operations:
  - getHello
  - getRHCompanySurveys
  - createRHCompanySurvey
  - getSurveyStateLiveWarnings
  - createSurveyStateLive
  - getSurveySummaryCompletions
  - getSurveyQuota
  - createSurveyStateClose
---

# Launch and monitor a Forsta survey

## Before you start

- **Base URL** is `https://{your-instance}/api/v1`. The spec's `{server}` variable defaults to
  `selfserve.decipherinc.com`; substitute your own instance domain. There is no shared public host.
- **Auth** is a single header: `x-apikey: <64-character key>`. Create it in the Research Hub /
  Portal under *API access*. Restrict it (permitted calls, permitted methods, forced parameters,
  IP network, expiry) before using it for automation.
- **Never call over plain HTTP.** A key seen on an unencrypted request is deactivated immediately
  as compromised and must be regenerated.
- **Survey ids are path-shaped**, e.g. `demo/bapi20`, not opaque tokens.
- Sanity-check credentials with `getHello` (`GET /hello`) before anything else.

## Steps

1. **Find the survey.** `getRHCompanySurveys` (`GET /rh/companies/{company}/surveys`). Narrow the
   response with the universal projection argument, e.g. `select=id,title,state`.
2. **Or create one.** `createRHCompanySurvey` (`POST /rh/companies/{company}/surveys`). This is a
   write with no idempotency key — see *Safety* below.
3. **Rehearse the launch.** `getSurveyStateLiveWarnings`
   (`GET /surveys/{survey}/state/live/warnings`) returns the warnings that a launch would raise.
   Read these and resolve them BEFORE step 4; this is the only dry run the API offers.
4. **Go live.** `createSurveyStateLive` (`POST /surveys/{survey}/state/live`).
5. **Watch it.** Poll `getSurveySummaryCompletions` (`GET /surveys/{survey}/summary/completions`)
   for completes, and `getSurveyQuota` (`GET /surveys/{survey}/quota`) to see whether cells are
   filling as planned.
6. **Close it.** `createSurveyStateClose` (`POST /surveys/{survey}/state/close`). Closing stops
   collection; it does not delete data.

## Safety

- **There is no idempotency contract.** No `Idempotency-Key`, no dedupe window, no ETag or
  conditional requests — the documentation says so explicitly. A retried `POST` executes twice.
  Do not blind-retry step 2 or step 4 on a timeout; re-read with `getRHCompanySurveys` first.
- **Reversibility.** `createSurveyStateClose` reverses `createSurveyStateLive`, but no document
  states a window, so do not promise a user one.
- **Errors** are a flat JSON object `{"$error": "...", "$code": <status>}`, not RFC 9457. Branch on
  the HTTP status, never on the message text: 401 invalid key, 402 monthly call allowance
  exceeded, 403 not permitted for this survey, 404 unknown survey, 428 survey hibernated and needs
  reactivation, 429 too many concurrent requests.
- **429 carries no `Retry-After`.** Choose your own backoff. Every response returns
  `x-usage-today: <calls> <resource units>` — read it to pace yourself.
- **Dates** must carry a timezone. `2014-01-10T20:08:08Z` is valid; a bare local datetime is not.
