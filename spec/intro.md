---
title: "LitePub (Introduction)"
status: "Living Standard"
last_updated: "June 26, 2019"
---

### Where does this specification fit?

This specification defines a significant part of the Social Web Platform.  The
Social Web Platform is a platform which is built on top of a set of federated
protocols.  LitePub is one of the most commonly used protocols in the Social
Web Platform.


### Is this ActivityPub?

*This section is non-normative.*

In short: Yes.

In more length: ActivityPub is a less rigid specification of the same basic
protocol.  It is maintained by the [W3C Social Web Community Incubator Group][w3c-swicg].
LitePub is maintained by the LitePub community, which primarily consists of
developers who are shipping LitePub implementations.  Accordingly, the LitePub
specification presents the same basic technology in a context that does not
depend on an understanding of other W3C technologies, such as the Linked Data
Platform.

   [w3c-swicg]: https://www.w3.org/wiki/SocialCG


### Background

*This section is non-normative.*

LitePub is a more rigid, vendor-oriented specification of ActivityPub.
In this way, LitePub defines a specific profile of ActivityPub.

ActivityPub is the Social Web's core federation protocol.  Originally,
ActivityPub was created as a formal specification of the pump.io
federation protocol reworked to leverage [ActivityStreams 2.0][as2] for
the purpose of federated microblogging.  However, it's generic design
has enabled it to be leveraged for many different kinds of applications.

   [as2]: https://www.w3.org/TR/activitystreams-core/


### Audience

*This section is non-normative.*

This specification is targeted at developers who intend to write software
which federates using the LitePub or ActivityPub protocol.

This document is probably not suited for readers who do not already have
a passing familiarity with the underlying concepts of federated protocols,
as it sacrifices clarity for precision and strives for completeness.

There are better guides and tutorials that demonstrate how to build a
federated service using the protocol described by this document.


### Scope

*This section is non-normative.*

This specification is limited to describing the core federation protocol
used for LitePub implementations to federate with other LitePub
implementations.  Many use-cases are more appropriately solved by using
the [LitePub extension process](/litepub/lep/index.html).


### Compatibility with other specifications

*This section is non-normative.*

This specification interacts with other specifications, namely:

 * [ActivityPub][ap]
 * [ActivityStreams 2.0][as2]
 * [Activity Vocabulary][as2-vocab]
 * [WebFinger][webfinger]
 * [W3C Web Payments Security Vocabulary][security-vocab]

In certain cases, unfortunately, due to conflicting requirements, this
specification violates certain requirements of these other specifications.
We have attempted to mark these violations as **willful violations** and
note the reason why our specified behaviour differs.

   [ap]: https://www.w3.org/TR/activitypub
   [as2-vocab]: https://www.w3.org/TR/activitystreams-vocabulary
   [webfinger]: https://tools.ietf.org/html/rfc7033
   [security-vocab]: https://web-payments.org/vocabs/security


### Extensibility

*This section is non-normative.*

ActivityPub treats extensibility as a function of JSON-LD, by extending the
JSON-LD `@context` in a given ActivityPub message.

LitePub treats extensibility by requiring that extensions namespace their
keywords and enumerable values in an IRI namespace.  This is functionally
equivalent to the JSON-LD way when processed by a JSON-LD implementation.
A full discussion of this is available in the [extensions](extensions.html)
section.


### Security

*This section is non-normative.*

ActivityPub does not define a security model.  In this absence, the community
has built a security model based on signature-based authentication.  This
security model is highly inefficient and there are significant concerns with
it's usage of signature-based authentication as a substitute for authorization.

LitePub defines two security models:

 * the [legacy security model](security-legacy.html), which is a formal
   description of the defacto ActivityPub security model

 * the [OCAP security model](security-ocap.html), which is a new security
   model we intend to replace the legacy security model with.


### Privacy Concerns

*This section is non-normative.*

All messages in LitePub are associated with a given actor.  This means that
messages are always linkable to their actor.  Additionally, when the legacy
security model is used, messages may contain either [LDS signatures][lds] or
[HTTP signatures][httpsigs].  The presence of these signatures may be used
to cryptographically link the message to the actor, creating secondary
metadata leakage.  [Blind Key Rotation][bkr] has been created as a partial
mitigation for this problem.

Because of the actor-oriented protocol design, LitePub should not be used
in environments where anonymity is required.

   [lds]: https://w3c-dvcg.github.io/ld-signatures/
   [httpsigs]: https://tools.ietf.org/html/draft-cavage-http-signatures-10
   [bkr]: https://blog.dereferenced.org/the-case-for-blind-key-rotation


### A Quick Introduction To LitePub

*This section is non-normative.*

LitePub is a server to server federation protocol, which enables websites
to exchange messages.  In LitePub, a user account is represented by an
"[actor](actors.html)".  Accounts on different servers correspond to
different actors.

An actor is required to have:

 * an `inbox`: an endpoint for receiving messages
 * an `outbox`: an endpoint which enumerates messages that have been sent

These are endpoints (URLs) that are listed in the actor's ActivityStreams
description.

Here's an example actor object representing a person named Alyssa P. Hacker:

<figure>
<figcaption>Example: Actor object for Alyssa P. Hacker</figcaption>
<pre>{
  "@context": "https://litepub.social/litepub/litepub-v0.1.jsonld",
  "type": "Person",
  "id": "https://social.example/~alyssa",
  "name": "Alyssa P. Hacker",
  "preferredUsername": "alyssa",
  "summary": "Lisp enthusiast hailing from MIT",
  "inbox": "https://social.example/~alyssa/inbox",
  "outbox": "https://social.example/~alyssa/outbox",
  "followers": "https://social.example/~alyssa/followers",
  "following": "https://social.example/~alyssa/following",
  "liked": "https://social.example/~alyssa/liked"
}</pre>
</figure>

LitePub uses [ActivityStreams 2.0][as2] for it's core vocabulary, which
describes almost everything you would ever need for any social networking
application.  LitePub can also be extended, read about it in the
[extensions section](extensions.html).

Alyssa can talk to her friends by using the `inbox` and `outbox` endpoints
associated with her profile:

 * Messages are *sent* to other actors by *POST*-ing to the other actor's
   inbox.

 * Messages are *fetched* from other users by *GET*-ing the other actor's
   outbox.

Typically federation happens by pushing messages to actor inboxes, however.
There are other methods beyond using HTTP POST and GETs to federate as well,
but these aren't considered part of the core protocol.

So, let's say that Alyssa wants to say hello to her friend Bob.  To accomplish
this, her LitePub implementation would push a `Create` message to Bob's inbox:

<figure>
<figcaption>Example: Create message used to send a note</figcaption>
<pre>{
  "@context": "https://litepub.social/litepub/litepub-v0.1.jsonld",
  "type": "Create",
  "id": "https://social.example/~alyssa/posts/a29a6843-9feb-4c74-a7f7-081b9c9201d3",
  "to": ["https://other.example/~bob"],
  "actor": "https://social.example/~alyssa",
  "object": {
    "type": "Note",
    "attributedTo": "https://social.example/~alyssa",
    "id": "https://social.example/~alyssa/posts/49e2d03d-b53a-4c4c-a95c-94a6abf45a19",
    "to": ["https://other.example/~bob"],
    "content": "hey bob!"
  }
}
</pre>
</figure>

In order to push this `Create` message, however, Alyssa's server must first
find Bob.  To facilitate this, we use [WebFinger][webfinger] to look up his
ActivityStreams actor.  Leveraging WebFinger, Alyssa can find Bob simply by
looking up `bob@other.example`, fetch his actor object and then find the
inbox to push the message to.

Later on, Bob replies back to Alyssa and she receives the reply in her inbox:

<figure>
<figcaption>Example: Create message used to send a note in reply to another note</figcaption>
<pre>{
  "@context": "https://litepub.social/litepub/litepub-v0.1.jsonld",
  "type": "Create",
  "id": "https://other.example/~bob/p/56783",
  "to": ["https://social.example/~alyssa"],
  "actor": "https://other.example/~bob",
  "object": {
    "type": "Note",
    "attributedTo": "https://other.example/~bob",
    "id": "https://other.example/~bob/p/56784",
    "inReplyTo": "https://social.example/~alyssa/posts/49e2d03d-b53a-4c4c-a95c-94a6abf45a19",
    "to": ["https://social.example/~alyssa"],
    "content": "hi alyssa!"
  }
}</pre>
</figure>

Finally, Alyssa publishes a public note to her followers collection and
the general public.  The special `https://www.w3.org/ns/activitystreams#Public`
(`as:Public`) security label is used to grant access to the public.

<figure>
<figcaption>Example: Create message addressed to followers + <code>as:Public</code></figcaption>
<pre>{
  "@context": "https://litepub.social/litepub/litepub-v0.1.jsonld",
  "type": "Create",
  "id": "https://social.example/~alyssa/posts/9282e9cc-14d0-42b3-a758-d6aeca6c876b",
  "to": ["https://social.example/~alyssa/followers",
         "https://www.w3.org/ns/activitystreams#Public"],
  "actor": "https://social.example/~alyssa",
  "object": {
    "type": "Note",
    "attributedTo": "https://social.example/~alyssa",
    "id": "https://social.example/~alyssa/posts/d18c55d4-8a63-4181-9745-4e6cf7938fa1",
    "to": ["https://social.example/~alyssa/followers",
           "https://www.w3.org/ns/activitystreams#Public"],
    "content": "federated social networking is so much fun!"
  }
}</pre>
</figure>


### Conformance

Sections that are explicitly marked non-normative, as well as diagrams, examples and
notes in this specification are non-normative.  Everything else in this specification
is normative.

The keywords MAY, MUST, MUST NOT, SHOULD and SHOULD NOT are to be interpreted as
described in [RFC2119][rfc2119].

   [rfc2119]: https://tools.ietf.org/html/rfc2119


### Suggested Reading

*This section is non-normative.*

The following documents may be of interest to readers of this specification:

<cite>Unicode Security Considerations</cite> ([UTR36][utr36])

> LitePub is based on UTF-8, an encoding of Unicode.  There are several security
> concerns involved with handling Unicode input.  This document describes those
> security concerns as well as suggests mitigations.

   [utr36]: https://www.unicode.org/reports/tr36/

<cite>ActivityStreams 2.0</cite> ([AS2][as2]), <cite>Activity Vocabulary</cite> ([AS2-VOCAB][as2-vocab])

> LitePub is built ontop of ActivityStreams 2.0 and the Activity Vocabulary.

<cite>WebFinger</cite> ([RFC7033][webfinger])

> LitePub implementations typically use WebFinger for mapping user@domain
> identifiers to actors.

<cite>HTTP Signatures</cite> ([HTTPSIGS][httpsigs])

> LitePub implementations sign and verify messages using HTTP signatures
> when implementing the legacy security model, as well as when establishing
> a new connection in the OCAP security model.
