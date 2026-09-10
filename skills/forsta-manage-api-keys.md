---
name: Provision and rotate Forsta API keys
description: >-
  Create, restrict, inspect, rekey and retire Decipher API keys through the API itself, so that
  automation runs on a least-privilege credential rather than a human's full-access key.
api: openapi/forsta-decipher-rest-api-openapi.yml
generated: '2026-09-10'
method: generated
source: >-
  derived from openapi/forsta-decipher-rest-api-openapi.yml plus the "API Keys" and "Security"
  sections of https://docs.developer.focusvision.com/docs/decipher/api (operationIds verified
  against the spec)
operations:
  - getHRApiKeys
  - createRHApiKey
  - getRHApiKey
  - updateRHApiKey
  - rekeyRHApiKey
  - deleteRHApiKey
  - getRHUsage
---

# Provision and rotate Forsta API keys

## The key format matters

A Decipher key is 64 characters in two halves. The **first 32 identify the key permanently** and
never change; the **last 32 are the secret**. That is why rekeying preserves the key's identity
and restrictions while replacing the secret — and why the id half is safe to log and the secret
half is not.

## Steps

1. **Inventory.** `getHRApiKeys` (`GET /rh/apikeys`) lists the keys for your company's users.
2. **Create a scoped key.** `createRHApiKey` (`POST /rh/apikeys`). Forsta's own guidance is to
   create a *dedicated user account for API usage* with access to a subset of projects, rather
   than issuing a key against a human's full-access account.
3. **Restrict it.** `updateRHApiKey` (`PUT /rh/apikeys/{apikey}`). The restriction object supports:
   - `networks` — an array of IPv4 addresses or properly-masked CIDR networks
     (`5.6.7.0/24` is accepted; `5.6.7.1/24` is not — the network bits must be cleared)
   - permitted calls (this key may only reach the user list, say)
   - permitted methods on those calls (may list users, may not create them)
   - forced parameters (may use the datamap API, but only for surveys X, Y, Z)
   - an expiry date
   An empty restriction object `{}` means no restrictions.
4. **Inspect.** `getRHApiKey` (`GET /rh/apikeys/{apikey}`).
5. **Rotate.** `rekeyRHApiKey` (`POST /rh/apikeys/{apikey}`) issues a new secret half.
6. **Retire.** `deleteRHApiKey` (`DELETE /rh/apikeys/{apikey}`) disables the key. There is no
   un-delete operation.
7. **Account for consumption.** `getRHUsage` (`GET /rh/usage`) calculates usage; every individual
   response also carries `x-usage-today: <calls> <resource units>`.

## Safety

- **A key used over HTTP is dead.** Forsta deactivates any key it sees on an unencrypted request,
  treating it as compromised. Pin HTTPS in your client.
- **Rotation is not reversible.** Neither `rekeyRHApiKey` nor `deleteRHApiKey` has an undo
  operation. Roll forward: create the replacement, cut traffic over, then retire the old key.
- **Impersonation is audited.** A staff or supervisor key may act as another user for one call by
  sending `x-impersonate: <email or numeric id>`. It is written to the audit log of both the
  impersonator and the subject. Use it deliberately, never as a convenience.
