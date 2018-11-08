---
title: "LitePub protocol suite"
---

LitePub is a suite of protocols which provide a federated social network.  They constitute various profiles
of the [ActivityPub][ap] specification.  It is intended that LitePub implementations provide compatibility
with [ActivityPub][ap], but there are some core behavioral differences.

   [ap]: https://www.w3.org/TR/activitypub/


## LitePub core

New implementors of LitePub are encouraged to begin by reading the [high-level LitePub overview](/litepub/overview.html),
as well as reading about the specific [LitePub profiles](/litepub/profiles.html) which are composed using the
[LitePub core vocabulary](/litepub/vocabulary.html).

ActivityPub implementors looking to improve compatibility with the LitePub ecosystem should read the
[LitePub for ActivityPub implementors](/litepub/ap-compat.html) document.


## LitePub discovery

The primary method of interaction with LitePub services is through an `inbox` URI, but discovery of that
`inbox` URI is not human friendly.  Accordingly, supporting the [LitePub discovery](/litepub/discovery.html)
WebFinger profile will help to provide a mapping of email-style addresses to `inbox` URIs.


## LitePub implementations

 * [Pleroma](https://pleroma.social)
 * [PixelFed](https://pixelfed.org)
