---
name: Produce a Forsta crosstab report
description: >-
  Validate crosstab logic, execute a crosstab against a survey's collected data, and export a
  saved report in the format a stakeholder asked for.
api: openapi/forsta-decipher-rest-api-openapi.yml
generated: '2026-09-10'
method: generated
source: derived from openapi/forsta-decipher-rest-api-openapi.yml (operationIds verified against the spec)
operations:
  - getSurveyCrosstabsConfiguration
  - getSurveyCrosstabsValidate
  - createSurveyCrosstabsExecute
  - getSurveyCrosstabsSaved
  - createSurveyCrosstabsSavedExecute
  - createSurveyCrosstabsSavedExport
  - updateSurveyCrosstabsNets
  - updateSurveyCrosstabsWeightingRim
---

# Produce a Forsta crosstab report

## Steps

1. **Read the configuration.** `getSurveyCrosstabsConfiguration`
   (`GET /surveys/{survey}/crosstabs/configuration`) tells you what the survey's crosstab surface
   already looks like.
2. **Validate your logic before running it.** `getSurveyCrosstabsValidate`
   (`GET /surveys/{survey}/crosstabs/validate`). Cheap, read-only, and it catches the mistakes that
   otherwise surface as a 400 halfway through an execution.
3. **Shape the tables if you need to.** `updateSurveyCrosstabsNets`
   (`PUT /surveys/{survey}/crosstabs/nets`) creates or updates nets;
   `updateSurveyCrosstabsWeightingRim` (`PUT /surveys/{survey}/crosstabs/weighting/rim/{var}`)
   applies rim weighting. Both are PUT replaces.
4. **Execute.** `createSurveyCrosstabsExecute` (`POST /surveys/{survey}/crosstabs/execute`). For a
   large survey add `forceTask=1` and poll `getStatus` rather than holding the connection open —
   the synchronous timeout is 30 minutes.
5. **Or re-run something saved.** `getSurveyCrosstabsSaved`
   (`GET /surveys/{survey}/crosstabs/saved`) lists saved reports;
   `createSurveyCrosstabsSavedExecute`
   (`POST /surveys/{survey}/crosstabs/saved/{saved}/execute`) runs one.
6. **Export.** `createSurveyCrosstabsSavedExport`
   (`POST /surveys/{survey}/crosstabs/saved/{saved}/export/{format}`) — `{format}` is a path
   parameter, so read the spec's enum rather than guessing an extension.

## Safety

- 403 on the crosstab routes is permission-specific: the spec distinguishes
  *"Forbidden - User lacks report.view permission"* from
  *"Forbidden - User lacks report.edit permission"*. Read step 1 and 2 with a view-only key; only
  steps 3-6 need edit rights.
- 409 Conflict appears on report routes when the underlying job is already in flight — read
  before re-executing rather than firing again. There is no idempotency key to protect you.
- `deleteSurveyCrosstabsTablesettings` and `deleteSurveyCrosstabsTargetSetting` have no reversal
  operation. Capture the current settings with step 1 before deleting anything.
