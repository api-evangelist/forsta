---
name: Run a Forsta survey email campaign
description: >-
  Build an invitation campaign against a Forsta Decipher survey, load its recipient list, send it,
  cancel a send that has not gone out, and reconcile bounces, opt-outs and spam reports afterwards.
api: openapi/forsta-decipher-rest-api-openapi.yml
generated: '2026-09-10'
method: generated
source: derived from openapi/forsta-decipher-rest-api-openapi.yml (operationIds verified against the spec)
operations:
  - getSurveyCampaigns
  - createSurveyCampaign
  - updateSurveyCampaignList
  - editSurveyCampaignInvite
  - createSurveyCampaignSend
  - getSurveyCampaignSends
  - createSurveyCampaignSendCancel
  - getSurveyCampaignBounces
  - getSurveyCampaignOptouts
  - getSurveyCampaignSpam
  - getSurveyCampaignsSupressions
  - createBulkBounced
---

# Run a Forsta survey email campaign

This is the highest-consequence flow in the Decipher API: it puts mail in front of real people and
**there is no idempotency key anywhere in it**. Read the safety section before you automate it.

## Steps

1. **List or create the campaign.** `getSurveyCampaigns` (`GET /surveys/{survey}/campaigns`), then
   `createSurveyCampaign` (`POST /surveys/{survey}/campaigns`) if you need a new one.
2. **Screen the list before you load it.** `createBulkBounced` (`POST /bulk/bounced`) checks
   addresses against known bounce-backs, and `getSurveyCampaignsSupressions`
   (`GET /surveys/{survey}/campaigns/suppressions`) returns the suppression lists in force.
3. **Load the recipients.** `updateSurveyCampaignList`
   (`PUT /surveys/{survey}/campaigns/{campaign}/list`) creates or replaces the list. Being a PUT
   replace, this one step *is* naturally re-runnable — a property of the operation, not a
   guarantee the provider offers.
4. **Set the invite.** `editSurveyCampaignInvite`
   (`PUT /surveys/{survey}/campaigns/{campaign}/invite`).
5. **Send.** `createSurveyCampaignSend`
   (`POST /surveys/{survey}/campaigns/{campaign}/sends`). **Do not retry this call on a timeout.**
6. **Confirm what actually happened.** `getSurveyCampaignSends`
   (`GET /surveys/{survey}/campaigns/{campaign}/sends`) — this is your read-back after any
   uncertain write.
7. **Cancel if needed.** `createSurveyCampaignSendCancel`
   (`POST /surveys/{survey}/campaigns/{campaign}/sends/{send}/cancel`).
8. **Reconcile.** `getSurveyCampaignBounces`, `getSurveyCampaignOptouts` and
   `getSurveyCampaignSpam` (all `GET /surveys/{survey}/campaigns/{listfile}/…`).

## Safety

- **No idempotency, no dedupe.** A retried step 5 sends the campaign twice to the same list. On any
  network error, go to step 6 and read the send list before deciding anything.
- **Reversibility is partial and unbounded.** `createSurveyCampaignSendCancel` reverses a send
  *that has not yet gone out*. Forsta does not publish where that boundary falls, so never tell a
  user a send can be recalled within some window — no such window is documented.
- 403 here usually means the key lacks rights on this survey, not that the campaign is wrong.
- Every datetime you schedule must carry an explicit timezone offset.
