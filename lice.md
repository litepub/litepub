---
title: "Litepub Capability Enforcement (LiCE)"
---

# Abstract

Litepub Capability Enforcement (LiCE) is an extension to the [ActivityPub protocol][ap]
which adds support for verifying and enforcing [object capabilities][wikipedia-ocap].

   [ap]: https://www.w3.org/TR/activitypub
   [wikipedia-ocap]: https://en.wikipedia.org/wiki/Object-capability_model


# Overview

LiCE provides two components that, when combined, define a security model based on the
concept of object capabilities:

 * **Proof objects**: Objects which have been created by an *actor* which *controls*
   a given resource that delegate permissions related to that resource.

 * **Capability URIs**: Well-known or private endpoint URIs which either grant
   implicit permissions or return a proof object that permits delegation of permission
   to perform a given action.


## Security Levels

LiCE implementations make use of both components in different ways, depending on the
configured security level:

 * `disabled`: proof objects are neither verified nor generated upon request.
 * `permissive`: proof objects are not verified, but will be generated upon request.
 * `enforcing`: proof objects are verified and are also generated upon request.


### Why does LiCE have configurable security levels?

*This section is non-normative.*

LiCE has a configurable security level to allow for a transitional period, with the
eventual goal that the entire ActivityPub network eventually adopts LiCE and defaults
to running in an enforcing mode.  It is strongly recommended that a "flag day" be set
where all implementations transition to having a default security level of
`enforcing`.


# The `capabilities` Object

Objects that have been security labeled have a new `capabilities` object that specifies
what capabilities are implicitly granted or require specific permissions.  Capabilities
with implicit grants are defined using well-known URIs, such as `as:Public`.

A basic object that has been labeled with a `capabilities` object looks like this:

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://litepub.social/litepub/lice-v0.0.1.jsonld"
  ],
  "capabilities": {
    "reply": "https://example.social/caps/d4c4d96a-36d9-4df5-b9da-4b8c74e02567",
    "like": "https://www.w3.org/ns/activitystreams#Public",
    "announce": "https://example.social/caps/e379a28e-74ce-ed7a-928c-7c3a4b2158e4"
  },
  "id": "https://example.social/objects/d6cb8429-4d26-40fc-90ef-a100503afb73",
  "type": "Note",
  "content": "I'm really excited about the new capabilities feature!",
  "attributedTo": "https://example.social/users/bob"
}
```

In this example, `Like` activities are allowed from the public, `Announce` and
`Create.inReplyTo` activities require explicit authorization in the form of a
proof object.


## Optional Specifiers

A `capabilities` object at the minimum is an object which contains the names of
capabilities but may also contain additional specifiers in the typical JSON-LD
way:

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://litepub.social/litepub/lice-v0.0.1.jsonld"
  ],
  "capabilities": {
    "reply": {
      "id": "https://www.w3.org/ns/activitystreams#Public",
      "expires": "2019-09-15T12:00:00Z"
    },
    "like": "https://www.w3.org/ns/activitystreams#Public",
    "announce": "https://www.w3.org/ns/activitystreams#Public"
  },
  "id": "https://example.social/objects/d6cb8429-4d26-40fc-90ef-a100503afb73",
  "type": "Note",
  "content": "I'm really excited about the new capabilities feature!",
  "attributedTo": "https://example.social/users/bob"
}
```

> *TODO*: document a list of possible specifiers here.  The `expires` specifier
> is obvious, but there may be other specifiers worth examining.


# Proof Objects

When a URI is used for a capability that is not a well-known URI such as `as:Public`,
then it is assumed to be an *authorization endpoint*.  Authorization endpoints
behave similarly to *inboxes* but return either a *proof object* or a rejection object
as a response.

Proof objects MUST:

 * be publicly accessible at the URI specified in their `id`
 * of type `Accept`
 * reference the allowed activity by either embedding it or referencing it by `id`

An example proof object looks like this:

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://litepub.social/litepub/lice-v0.0.1.jsonld"
  ],
  "id": "https://example.social/proofs/fa43926a-63e5-4133-9c52-36d5fc6094fa",
  "type": "Accept",
  "actor": "https://example.social/users/bob",
  "object": {
    "id": "https://example.social/activities/12945622-9ea5-46f9-9005-41c5a2364f9c",
    "type": "Like",
    "object": "https://example.social/objects/d6cb8429-4d26-40fc-90ef-a100503afb73",
    "actor": "https://example.social/users/alyssa",
    "to": [
      "https://example.social/users/alyssa/followers",
      "https://example.social/users/bob"
    ]
  }
}
```


### Attestation of Properties in the Referent Object

An `attestations` object contains properties and values that must match the
properties and values in the referent object.

In the event that an `attestations` object is included, the `id` property
MUST NOT be present.

When verifying proof objects that contain an `attestations` object, the verifier
MUST ensure that the object being authorized against the proof has the same
properties as present in the proof object.  In the event that the proof object's
child fragments and the referent object disagree, the verifier MUST fail the
verification.

An example proof object with an `attestations` object:

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://litepub.social/litepub/lice-v0.0.1.jsonld"
  ],
  "id": "https://example.social/proofs/fa43926a-63e5-4133-9c52-36d5fc6094fa",
  "type": "Accept",
  "actor": "https://example.social/users/bob",
  "object": {
    "id": "https://example.social/activities/12945622-9ea5-46f9-9005-41c5a2364f9c",
    "type": "Like",
    "object": "https://example.social/objects/d6cb8429-4d26-40fc-90ef-a100503afb73",
    "actor": "https://example.social/users/alyssa",
    "to": [
      "https://example.social/users/alyssa/followers",
      "https://example.social/users/bob"
    ]
  }
  "attestations": {
    "type": "Like",
    "object": "https://example.social/objects/d6cb8429-4d26-40fc-90ef-a100503afb73",
    "to": [
      "https://example.social/users/alyssa/followers",
      "https://example.social/users/bob"
    ]
  }
}
```


## Invocation

When a proof object is required in order to prove an activity is authorized, it MUST be
attached as the `proof` field, either by embedding the object or referencing it by URI.
This is known as a capability invocation.

An example invocation where the proof object is embedded:

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://litepub.social/litepub/lice-v0.0.1.jsonld"
  ],
  "id": "https://example.social/activities/12945622-9ea5-46f9-9005-41c5a2364f9c",
  "type": "Like",
  "object": "https://example.social/objects/d6cb8429-4d26-40fc-90ef-a100503afb73",
  "actor": "https://example.social/users/alyssa",
  "to": [
    "https://example.social/users/alyssa/followers",
    "https://example.social/users/bob"
  ],
  "proof": {
    "id": "https://example.social/proofs/fa43926a-63e5-4133-9c52-36d5fc6094fa",
    "type": "Accept",
    "actor": "https://example.social/users/bob",
    "object": {
      "id": "https://example.social/activities/12945622-9ea5-46f9-9005-41c5a2364f9c",
      "type": "Like",
      "object": "https://example.social/objects/d6cb8429-4d26-40fc-90ef-a100503afb73",
      "actor": "https://example.social/users/alyssa",
      "to": [
        "https://example.social/users/alyssa/followers",
        "https://example.social/users/bob"
      ]
    }
  }
}
```

An example where the proof object is referenced:

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://litepub.social/litepub/lice-v0.0.1.jsonld"
  ],
  "id": "https://example.social/activities/12945622-9ea5-46f9-9005-41c5a2364f9c",
  "type": "Like",
  "object": "https://example.social/objects/d6cb8429-4d26-40fc-90ef-a100503afb73",
  "actor": "https://example.social/users/alyssa",
  "to": [
    "https://example.social/users/alyssa/followers",
    "https://example.social/users/bob"
  ],
  "proof": "https://example.social/proofs/fa43926a-63e5-4133-9c52-36d5fc6094fa"
}
```


## Verification

Proof objects SHOULD be verified before accepting any activity which references one.
The default way to do this is to fetch the proof object at the given URI and verify
that there is a cyclical relationship between the accepted activity and the activity
which references the proof.

An optional way of verifying a proof would be to use [Linked Data Signature][lds].
If it is possible to verify the proof using a cryptographic signature, then refetching
the proof object is not necessary.  All implementations MUST support fallback to
fetching the proof object if a valid signature is not present.

   [lds]: https://w3c-dvcg.github.io/ld-signatures/


### Verifying Attestations

*This section is non-normative.*

Implementations MUST check that properties in the `attestations` object match
properties in the referent object.  In most cases, this is done by doing a
literal comparison.

When comparing sets (JSON arrays), an implementation SHOULD iterate over the
values in the attested property and verify set membership in the referent
property for every value.  As an optimization, an implementation MAY normalize
a copy of both properties and match that the sequence is identical.


# Security Considerations

*This section is non-normative.*


## Recursive Object Fetching Denial of Service

A malicious proof object could reference itself, either directly or indirectly.

Implementations SHOULD limit the depth to which they fetch referenced activities and
proof objects.  A valid activity will never have a recursive loop, so the depth limit
can be very low.


## Proof Object/Child Object Swapping

A malicious server could swap the authorized object with a newer version that contains
content that was not authorized by the granting server.

Implementations SHOULD include key properties of the child object when generating a
proof object, such as `content`, `name`, `summary` and `attachment`.


## Fake Direction Spoofing

A malicious server could present a proof object using a third-party domain or third-party
actor.

Implementations SHOULD verify that the proof object is created by the same actor which
created the content being interacted with.

Implementations SHOULD verify that the proof object is at the same domain as the object
being interacted with.
