---
name: Complete a Ready To Fly passenger check
description: Walk a single airline passenger through a Zamna session — capture consent, recognise and apply their travel document, answer rule-driven checklist questions, and (for US destinations) submit their address — using the Zamna Ready To Fly (PaxCheck) API.
api: openapi/zamna-paxcheck-openapi-original.json
operations:
- getSession
- getPaxSession
- setConsent
- recogniseDocument
- applyRecognisedDocument
- applyTravelDocument
- postAnswers
- addAddress
---

# Complete a Ready To Fly passenger check

Use the Zamna Ready To Fly (PaxCheck) API to bring one passenger to a "ready to fly"
state before they reach the airport. All steps below are grounded in real
operationIds from `openapi/zamna-paxcheck-openapi-original.json`.

## Auth
Every operation here is protected by `BearerAuth` — send the `pax_check_token` JWT
(returned during session initialization via `/start` or `/start2`) as
`Authorization: Bearer <pax_check_token>`. A missing/expired token returns `401`;
re-initialize the session to get a fresh token. See
`conventions/zamna-conventions.yml` and `errors/zamna-problem-types.yml`.

## Steps
1. **Load the session.** Call `getSession` with the `sessionId` to retrieve the
   MultiPax session and its passenger list.
2. **Load the passenger.** Call `getPaxSession` with `sessionId` + `paxSessionId`
   for the specific traveler you are processing.
3. **Capture consent.** Call `setConsent` on the session before handling any
   passenger document data (consent is a first-class, GDPR-driven step).
4. **Recognise the document.** Call `recogniseDocument` with the scanned document
   image, then `applyRecognisedDocument` to attach the recognised result to the
   passenger session. If recognition is unavailable, fall back to
   `applyTravelDocument` (manual travel-document entry).
5. **Answer the checklist.** Call `postAnswers` with the passenger's responses to
   the rule-driven questions returned on the pax-session.
6. **Add address (US destinations).** When the itinerary requires it, call
   `addAddress` with the passenger's US destination address.
7. **Re-check readiness.** Call `getPaxSession` again to confirm the checklist has
   resolved and the passenger is Ready To Fly.

## Rules
- No idempotency-key contract exists; do not blindly retry mutating calls — re-read
  the pax-session with `getPaxSession` to check state before retrying.
- No PII should be persisted client-side; the API is designed to read/verify
  documents once inside airline infrastructure.
