---
title: "LitePub top-level overview"
---

LitePub is a family of similar server-to-server protocols which each constitute a profile
of [ActivityPub][ap].  They have been stripped down and are intended to be lightweight
with easily understood security properties.

   [ap]: https://www.w3.org/TR/activitypub/

Information about differences between LitePub and ActivityPub can be found in the
[LitePub for ActivityPub implementors][ap-compat] document.

   [ap-compat]: ap-compat.html


## Overview

In LitePub, operations are modelled as interactions between `actors`.  An actor could be,
amongst other things, an user account, a bot, or a service.  LitePub implementations
process messages which are delivered to `inboxes` and apply side effects based on the
interpretation of those messages.  The exact behavior an implementation may take when
presented with a message is implementation-defined.


### Message delivery

Messages are addressed to either `actors` or `collections`.  Every `actor` MUST have an
`inbox` property and MAY have a `sharedInbox` property.  If an `actor` has a `sharedInbox`
property, then an implementation MAY choose to deliver the message to the `sharedInbox`
instead of directly to the actor's inbox.

Handling of `collections` SHOULD be treated as aliases for a group of known `actors`.
Implementations MUST NOT attempt delivery directly to a `collection`.

Delivered messages MUST be signed and then delivered to `inboxes`.  The receiving server
MUST authenticate the message following the rules below in the [Message authentication][auth]
section.

   [auth]: #message-authentication


### Message authentication

LitePub messages are authenticated using a construction based on [HTTP Signatures][httpsigs].
The `date` and `digest` headers MUST be signed, additional headers MAY be signed.

   [httpsigs]: https://tools.ietf.org/html/draft-cavage-http-signatures-10


#### The `digest` header

LitePub messages MUST have a `digest` header of the following construction:

  * The hash method used
  * An equals sign (`=`)
  * The checksum from the hash method used in [base64 encoding][base64].

At a minimum, the SHA2-256 hash method MUST be used, using the identifier `SHA-256`.
Implementations MAY send other digests by separating them with a space and using the
same construction.

To generate the digest, the binary message body is hashed using the selected hash method.
Implementations MUST verify that the signed digest header matches the received message
body.

An example digest header:

```
Digest: SHA-256=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=
```

An example digest header with multiple digests:

```
Digest: SHA-256=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=
  SHA-512=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=
```

   [base64]: https://tools.ietf.org/html/rfc4648


### Message transience

*This section is non-normative.*

An important property of LitePub verses ActivityPub is that messages are intended to be
transient, when message handling is properly implemented in this way, significant security
advantages over the original ActivityPub approach are achieved, such as message deniability.

In respect to the LitePub specification process, this means that care has been taken to
minimize the leakage of both message contents and data which can be used to forensically
authenticate those messages.

It should be considered a significant violation of this specification to handle messages
in a way that could result in the leakage of message contents or signature data to third
parties.  However, strict adherence to the LitePub core protocol should result in an
implementation which handles messages in a secure manner.


### Object Capabilities

*This section is non-normative.*

Another important property of LitePub verses ActivityPub is that message data integrity
is controlled by the server which publishes the original message.  Accordingly, all
message which reference another message MUST do so using the message's `id`.  The `id`
of the message may be treated as a *capability URI*.

When a message is deleted, the URI associated with the message `id` MUST NOT respond
with the message contents anymore, which does two things:

 * prevents further dissemination of the message through the federated graph
 * provides plausible deniability about the existence of a message (spoofing argument)

More information about object capability URIs can be found at these resources:

 * [Overview post written by one of the SecondLife server authors][ocap-uri-1].
 * [W3C: Good practices for Capability URLs][ocap-uri-2].

   [ocap-uri-1]: https://mrtopf.de/second-life/slga-capabilities-explained-technical/
   [ocap-uri-2]: https://www.w3.org/TR/capability-urls/
