---
title: "LitePub (Objects)"
status: "Living Standard"
last_updated: "June 26, 2019"
---

### Architectural Introduction

*This section is non-normative.*

LitePub and other applications of [ActivityStreams][as2] are built on top of an
object-oriented architecture.  Unlike previous specifications concerning this
protocol, we present the object model in a flat way, where all objects are
derived from the base Object type.  It is our hope that this presentation is
more easily understood by the reader as well as more reflective of reality.

   [as2]: https://www.w3.org/TR/activitystreams-core

In production, we have learned that maintaining an artificial partition between
the notion of regular objects (which reflect data) and activities (which reflect
mutations on data) significantly increases the complexity involved in walking
a graph of activities.  Accordingly, we have come to see the artificial partition
as more harmful than helpful, especially when storing graphs in normalized form
inside so-called "NoSQL" and traditional SQL databases.


### Handling of Objects

LitePub implementations process [ActivityStreams][as2] objects and store
them and/or interpret them to perform mutations on other objects.  To facilitate
this, LitePub defines additional vocabulary on top of the [ActivityStreams
core vocabulary][as2-vocab].

   [as2-vocab]: https://www.w3.org/TR/activitystreams-vocabulary

Information about handling the full [LitePub vocabulary][lp-vocab] is in
available in the LitePub vocabulary section.

   [lp-vocab]: vocabulary.html


#### Processing of Referenced Objects

LitePub objects reference other objects.  Objects are referenced by IRI
([RFC3987][rfc3987]).  Every URI ([RFC3986][rfc3986]) is also usable as
an IRI.

Relative IRI references MUST NOT be used inside a LitePub object.

   [rfc3986]: https://tools.ietf.org/html/rfc3986
   [rfc3987]: https://tools.ietf.org/html/rfc3987

Servers MUST validate the content they receive to avoid content spoofing
attacks as well as to verify that processing the object does not induce
any mutations which would be disallowed by the server's configured policy.
There are two security models specified by this specification in the
[security considerations][lp-security] section.

   [lp-security]: security.html


#### Object Requirements

All LitePub objects MUST have the following fields:

* `id`: the object's identifier (as an IRI)
* `type`: the object's type


#### Object Identifiers and Transience

All LitePub objects MUST have unique global identifiers.  If an object
is received without an identifier, the receiver MAY assign the object an
internally generated identifier.

Servers MAY respond with an HTTP 410 response code when fetched by their
global identifier to indicate their transcience.


#### Object Deletion

Servers SHOULD replace the deleted object with a `Tombstone` to prevent
re-use of the identifier or possible refetching of remote objects.

Servers MUST respond with an HTTP 404 response code when a `Tombstone`
object is encountered.

Servers MUST NOT serve the contents of the `Tombstone` object.

Servers SHOULD delete any local copies of an object if refetching the
object results in either a 404 or 410 response code being returned.

> **Note**: ActivityPub implementations are allowed to respond with
> either 404 or 410 as well as serving the underlying `Tombstone`
> object.  This behaviour is flawed because it results in metadata
> leakage -- namely, it leaks that an object did at one time exist
> with the given identifier.


#### Object Delivery

Servers SHOULD track which peers they have successfully delivered a
given object to.

Servers SHOULD broadcast `Delete` messages to all peers they
successfully delivered the message to.


#### Object Serving and Fetching

Servers MAY require authorization to serve any object, including
objects addressed to `https://www.w3.org/ns/activitystreams#Public`.

Servers MUST require authorization to serve any object not
addressed to `https://www.w3.org/ns/activitystreams#Public`.

Servers SHOULD treat authorized object fetch requests as deliveries
to the authorized peer and record them as a successful delivery.
