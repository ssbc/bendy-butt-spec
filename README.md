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

A message in this format is encoded as an array of:

- `author` a binary [TFK] encoded feed id
- `sequence` a 32 bit integer starting at 1
- `previous` a binary [TFK] encoded message id of the previous message
  on the feed. For the first message this must be zero bytes.
- `time` an integer representing the UNIX epoch timestamp the message
  was created
- `content` an object encoded data type consisting of the data
  relevant to the message including some mandatory fields or a binary
  [TFK] encoded box2 message.
- `content signature` concatenation of the bytes of 'metafeeds' and
  the bytes of the `content` field signed using the private key of the
  sub feed, if content is encrypted this will also be encrypted as a
  binary [TFK] encoded box2 message
- `signature` the bytes of all the fields above concatenated and
  signed using the private key of the meta feed

The input for content signature includes specific starting bytes for
domain separation in order to make sure that the signature can only be
used for meta feed signatures and not for anything else.

For signatures we use the same [HMAC signing capability]
(sodium.crypto_auth) and sodium.crypto_sign_detached as in the classic
SSB format.

Uses ed25519 for keys and SHA256 for hashing (FIXME: will be specified
in TFK document).

## Validation

For validation we differentiate between if the message is valid and if
the content is valid. Since the content can be encrypted there is
potentially no way to validate that. This means a message should only
be rejected before it is inserted in the local database if it fails
the message validation rules and not be included in the state of the
meta feed if content is not valid.

Message must conform to the following rules:
 - Must be in the format specified above
 - The previous field must be correct
 - The signature must be valid
 - The maximum size of a message in bytes must not exceed 8192 bytes.

Content must conform to the following rules:
 - a type field with a string value of only the following values
   possible: 'metafeed/add', 'metafeed/update, 'metafeed/tombstone'
 - a subfeed field with a [TFK] encoded feed id
 - a metafeed field with a [TFK] encoded feed id
 - a nonce field with a 32 bit random integer value

[SSB]: https://github.com/ssbc/
[gabby grove]: https://github.com/ssbc/ssb-spec-drafts/tree/master/drafts/draft-ssb-core-gabbygrove/00
[bamboo]: https://github.com/AljoschaMeyer/bamboo
[meta feeds]: https://github.com/ssb-ngi-pointer/ssb-meta-feed-spec
[bipf]: https://github.com/ssbc/bipf
[TFK]: https://github.com/ssbc/envelope-spec/blob/master/encoding/tfk.md
[HMAC signing capability]: https://github.com/ssb-js/ssb-keys#signobjkeys-hmac_key-obj
