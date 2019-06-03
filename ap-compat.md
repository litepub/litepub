---
title: "LitePub for ActivityPub Implementors"
---

## JSON-LD context

LitePub implementations are not required to use `@context` properties on their messages.  
A conformant ActivityPub implementation is required to process these messages with an
injected `@context` of `"https://www.w3.org/ns/activitystreams"` as described in the
[ActivityStreams 2.0 Core Specification][AS2-CORE-JSON-LD].

   [AS2-CORE-JSON-LD]: https://www.w3.org/TR/activitystreams-core/#jsonld

However, the LitePub Core Vocabulary differs from the ActivityStreams 2.0 Vocabulary.
It is suggested that LitePub implementations supply a locally hosted version of the
[LitePub JSON-LD Context][litepub-jsonld] as their `@context`.  It may be useful to
inject a local copy of the LitePub JSON-LD Context instead of the default ActivityStreams
2.0 context when a message is received without a `@context` as it defines the full
LitePub Core Vocabulary in a way that is useful to JSON-LD processors.

   [litepub-jsonld]: https://litepub.social/litepub/context.jsonld


## Signatures

LitePub implementations MUST use HTTP Signatures to verify the authenticity of
messages being delivered to or from peering nodes.  The details surrounding
the way HTTP Signatures are implemented in LitePub are discussed on the Overview
page.
