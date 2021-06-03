# Meta feed message encoding

Why a new feed format, we already have [gabby grove] and [bamboo]?

> Because we would like a simple feed format for [meta feeds] that is
> easy to implement in any programming language by reusing an existing
> encoder in [bencode]. Furthermore as meta feeds makes it a easier to
> have multiple feed formats we wanted to show that creating a new
> feed format does not have to be hard.

Can't you just use the classic format?

> That would mean clients would have to support classic forever, even
> for applications that only use newer feed formats

What is wrong with classic format?

> There are multiple problems, most of them comes from the fact that
> it is very tied to the internals of the v8 engine for Javascript.

## Details

A message is encoded in [bencode] and uses SSB binary field encodings
([SSB-BFE]) as an array of message payload and signature:

Message payload, an array in this specific order of:

- `author` a binary [SSB-BFE] encoded feed id
- `sequence` a 32 bit integer starting at 1
- `previous` a binary [SSB-BFE] encoded message id of the previous message
  on the feed. For the first message this must be zero bytes.
- `time` an integer representing the UNIX epoch timestamp the message
  was created
- `content` an array of a dictionary encoded the data relevant to the
  meta feed and a signature. The signature is the concatenation of the
  string 'metafeeds' encoded as bytes and the bytes of the content
  payload, signed using the private key of the sub feed. If content is
  encrypted this will also be encrypted as a binary [SSB-BFE] encoded
  box2 message

Signature:

- the bytes of the message payload entry in the array signed using the
  private key of the meta feed

The input for the content signature includes specific starting bytes
for domain separation in order to make sure that the signature can
only be used for meta feed signatures and not for anything else.

For signatures we use the same [HMAC signing capability]
(sodium.crypto_auth) and sodium.crypto_sign_detached as in the classic
SSB format (ed25519).

The key or id of a message is the SHA256 hash of the array consisting
of message payload and signature as bytes.

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
 - The signature field must be valid
 - The [SSB-BFE] format for `author` and `previous` must stay the same,
   there is no upgradeability in the middle of a feed
 - The maximum size of a message in bytes must not exceed 8192 bytes.

Content must conform to the following rules:
 - a type field with a string value of only the following values
   possible: 'metafeed/add', 'metafeed/update, 'metafeed/tombstone'
 - a subfeed field with a [SSB-BFE] encoded feed id
 - a metafeed field with a [SSB-BFE] encoded feed id
 - a nonce field with a 32 bit random integer value
 - the content signature must be correct

[SSB]: https://github.com/ssbc/
[gabby grove]: https://github.com/ssbc/ssb-spec-drafts/tree/master/drafts/draft-ssb-core-gabbygrove/00
[bamboo]: https://github.com/AljoschaMeyer/bamboo
[meta feeds]: https://github.com/ssb-ngi-pointer/ssb-meta-feed-spec
[SSB-BFE]: https://github.com/ssb-ngi-pointer/ssb-binary-field-encodings
[HMAC signing capability]: https://github.com/ssb-js/ssb-keys#signobjkeys-hmac_key-obj
[bencode]: https://en.wikipedia.org/wiki/Bencode
