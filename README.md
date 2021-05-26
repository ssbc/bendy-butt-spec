# Bipfy badger

What is bipfy badger?

> a new feed format for [SSB]

Why a new feed format, we already have [gabby grove] and [bamboo]?

> Because we would like a simple feed format for [meta feeds] that is
> easy to implement in any language

Can't you just use the classic format?

> That would mean clients would have to support classic forever, even
> for applications that only use newer feed formats

What is wrong with classic format?

> There are multiple problems, most of them comes from the fact that
> it is very tied to the internals of the v8 engine for Javascript.

## So what is bipfy badger really?

A feed format based on [bipf] a very simple binary format (Javascript
implementation is less than 500 lines of code) tailored for [meta
feeds].

Uses ed25519 for keys and SHA256 for hashing.

A message in this format is encoded as an array of:

- `author` a binary [TFK] encoded feed id
- `sequence` an integer starting at 1
- `previous` a binary [TFK] encoded message id of the previous message
  on the feed
- `time` an integer representing the UNIX epoch timestamp the message
  was created
- `content` an object consisting of the data relevant to the message
- `content signature` the bytes of `content` signed using the private
  key of the subfeed
- `signature` the bytes of all the fields above concatenated and
  signed using the private key of the meta feed

For signatures we use the same [HMAC signing capability]
(sodium.crypto_auth) as in the classic SSB format.

[SSB]: https://github.com/ssbc/
[gabby grove]: https://github.com/ssbc/ssb-spec-drafts/tree/master/drafts/draft-ssb-core-gabbygrove/00
[bamboo]: https://github.com/AljoschaMeyer/bamboo
[meta feeds]: https://github.com/ssb-ngi-pointer/ssb-meta-feed-spec
[bipf]: https://github.com/ssbc/bipf
[TFK]: https://github.com/ssbc/envelope-spec/blob/master/encoding/tfk.md
[HMAC signing capability]: https://github.com/ssb-js/ssb-keys#signobjkeys-hmac_key-obj
