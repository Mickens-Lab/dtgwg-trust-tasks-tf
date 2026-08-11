---
slug: witness/session
version: "0.1"
title: Witness — Session
summary: Opens a witness ceremony as its own exchange, nested in a relationship exchange via parentThreadId. The witness's response issues the session challenge; this document's id is the value a Verifiable Witness Credential later carries as taskContext.
status: draft
targetFrameworkVersion: "0.4"
category: credentials
keywords:
  - witness
  - ceremony
  - vwc
  - session
  - challenge
authors:
  - Alberto L (https://github.com/albertoleon7794)
parties:
  - role: participating party
    requirement: REQUIRED
    member: issuer
  - role: witness
    requirement: REQUIRED
    member: recipient
proofRequirement:
  request: OPTIONAL
  response: REQUIRED
  rationale: >-
    The request is consumed in-exchange by the witness under transport
    authentication (SPEC.md §4.7.1). The response is REQUIRED because it
    issues the session challenge — the value every later presentation in the
    ceremony binds to — and because the ceremony's documents are retained as
    the context of outcome evidence relied on by parties outside the
    exchange: a forged or unattributable challenge poisons everything built
    on it.
sideEffects:
  level: mutating
  rationale: "The witness opens session state it must hold until the ceremony terminates or expires."
exposure:
  discloses: metadata
  actsAsSubject: false
errorCodes:
  - code: witness/session:refused
    meaning: The witness declines to open a session — policy, capacity, or the named exchange is not one it will witness.
    retryable: false
related:
  - witness/session/submit
  - vrc/relationship/propose
---

## Abstract

A **witness ceremony**: a third party observes a relationship exchange and
later attests, in a Verifiable Witness Credential, that it did. This
specification opens the ceremony. The witness's `#response` issues the
**session challenge** the participating parties will bind their presentations
to under [`witness/session/submit`](../../session/submit/0.1/spec.md).

## The nesting, and what a `taskContext` names

The ceremony is **its own exchange**, conducted inside a relationship
exchange. Two rules follow, and they are the reason this specification
exists in this exact shape:

1. **This document's `id` is the ceremony's name.** Per
   [SPEC.md §4.9.1](../../../../SPEC.md#491-citing-an-exchange-as-evidence),
   a citation naming an exchange as evidence names the *innermost* exchange
   that attests the event, by the `id` of the document that initiated it. The
   witnessing is attested by *this* exchange — not by the surrounding
   relationship exchange, whose own responses say nothing about whether a
   witness observed anything. A Verifiable Witness Credential's `taskContext`
   therefore carries **this document's `id`**, never the outer exchange's
   thread.
2. **Every document of the ceremony carries `parentThreadId`.** A *producer*
   **MUST** set `parentThreadId`
   ([§4.9.2](../../../../SPEC.md#492-the-parentthreadid-member)) to the
   containing relationship exchange's `threadId` on every document of this
   exchange, including responses and error responses. Per §4.9.2 the member
   is navigation only: a *consumer* **MUST NOT** reject a document solely for
   its absence.

## Conformance

A conforming **participating party** (`issuer`):

1. Emits a document whose `type` is `https://trusttasks.org/spec/witness/session/0.1`, addressed to the witness, with a fresh `id`, `threadId` equal to that `id`, and `parentThreadId` per the rule above.
2. Names in `parties` the relationship DIDs of the exchange to be witnessed.

A conforming **witness** (`recipient`):

1. Applies the [SPEC.md §7.2](../../../../SPEC.md#72-consumer-requirements) pipeline.
2. On accepting, returns the `#response` carrying a fresh, unpredictable `challenge` and its `domain`, under the REQUIRED proof.
3. On declining, returns a `trust-task-error` with `witness/session:refused`.

## Security & Privacy

**The challenge is the ceremony's binding value.** Presentations under
`submit` are bound to `{challenge, domain}`. A challenge is single-use and
unpredictable; a witness **MUST NOT** reuse one across sessions. *The pair may
be superseded by a canonical session transcript* — binding protocol and
profile versions, context, purpose, scope, session and epoch — as that work
is ratified; this specification deliberately isolates the binding material in
one place so that upgrade replaces a member rather than the ceremony.

**A witness learns who is forming relationships.** Opening a session
discloses the parties' relationship DIDs to the witness by necessity. Those
are pairwise values, but the witness can correlate the sessions it serves;
parties choose witnesses with that in view, and a witness's own retention is
governed by the policies it publishes, not by this specification.
