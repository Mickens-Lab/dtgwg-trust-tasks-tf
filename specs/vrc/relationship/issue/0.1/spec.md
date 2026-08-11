---
slug: vrc/relationship/issue
version: "0.1"
title: VRC Relationship — Issue
summary: Delivers one party's signed Verifiable Relationship Credential to the other within an accepted relationship exchange, and returns a delivery receipt. Performed once in each direction — the exchange is mutual.
status: draft
targetFrameworkVersion: "0.4"
category: credentials
keywords:
  - vrc
  - relationship
  - issuance
  - delivery
authors:
  - Alberto L (https://github.com/albertoleon7794)
parties:
  - role: issuing party
    requirement: REQUIRED
    member: issuer
  - role: receiving party
    requirement: REQUIRED
    member: recipient
proofRequirement:
  request: REQUIRED
  response: OPTIONAL
  rationale: >-
    On the request, because it delivers a credential the receiving party
    retains and later presents — the SPEC.md §4.7.1 retained-and-relied-upon
    condition; the credential carries its own issuer signature, but the
    envelope proof is what attributes the delivery itself on a relayed path.
    On the response, OPTIONAL: the receipt is consumed inside the exchange by
    the connected peer, and the transport's authcrypt authenticates it.
sideEffects:
  level: mutating
  rationale: "The receiving party stores a credential. Not compensatable by this exchange; revocation is the issuer's own act."
exposure:
  discloses: none
  actsAsSubject: false
errorCodes:
  - code: vrc/relationship/issue:notAccepted
    meaning: The receiving party refuses the delivery — the credential does not match the accepted proposal (wrong parties, wrong relationship DIDs) or arrives outside an accepted exchange.
    retryable: false
related:
  - vrc/relationship/propose
  - vtc/relationships/publish
---

## Abstract

Within an accepted relationship exchange
([`vrc/relationship/propose`](../../propose/0.1/spec.md)), each party issues
the other a **Verifiable Relationship Credential** naming the pairwise
relationship DIDs the proposal established. This specification carries one
such delivery; a conforming exchange performs it **twice, once in each
direction** — the relationship is mutual, and neither delivery is a response
to the other.

The delivery idiom is the DTG one — the signed credential in the payload with
a receipt in the `#response` — following `vtc/members/vmc` and
`vtc/join-requests/accept`: the receipt names what was received and does not
echo the credential back.

## Conformance

A conforming **issuing party** (`issuer`):

1. Emits a document whose `type` is `https://trusttasks.org/spec/vrc/relationship/issue/0.1`, on the relationship exchange's thread (`threadId` = the `propose` document's `id`).
2. Carries in `vrc` a signed credential whose issuer is the issuing party's relationship DID and whose credential subject names the receiving party's relationship DID — the values the proposal exchanged.
3. **SHOULD** set `vrcDigestMultibase` — a multibase-encoded multihash over the credential, the registry's converged digest form — so the delivery can be tied to later references to the credential without re-hashing.

A conforming **receiving party** (`recipient`):

1. Applies the [SPEC.md §7.2](../../../../../SPEC.md#72-consumer-requirements) pipeline.
2. Verifies the credential's own proof and its party bindings against the accepted proposal **before** storing it or returning a receipt.
3. On acceptance, returns the `#response` receipt. On refusal, returns a `trust-task-error` with `vrc/relationship/issue:notAccepted`.

## Witnessing

Where the exchange is witnessed, the ceremony
([`witness/session`](../../../../witness/session/0.1/spec.md)) completes
before the deliveries: what the witness attests includes that the parties
exchanged credentials under its observation, and a delivery that precedes the
ceremony is outside that attestation. Nothing in this document references the
ceremony — the nesting lives on the ceremony's side, via `parentThreadId`.

## Security & Privacy

**Two proofs, two jobs.** The credential's inner proof makes the *credential*
verifiable indefinitely; the envelope proof (REQUIRED) attributes the
*delivery*. Conflating them — accepting an unproofed envelope because the
credential inside verifies — attributes storage-changing action to a document
nobody signed.

**The receipt asserts receipt, not validity.** A `#response` here means the
receiving party accepted and stored the delivery. It is not an endorsement a
third party may rely on; the credential stands on its own proof.
