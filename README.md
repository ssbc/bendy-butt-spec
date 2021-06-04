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

Example (FIXME: proper data):

```
[
  [
    <Buffer 00 02 e8 20 31 38 8d df f8 b5 0e 56 b6 c0 97 42 1e 9a a8 92 ec 04 e9 42 fa fd 31 dc 3d 2c 2e 3e 52 fd>, // author
    1, // sequence
    <Buffer 01 02>, // previous
    1456154790701, // timestamp
    [
      { 
        type: 'metafeed/add', 
        feedformat: 'classic', 
        feedpurpose: 'main', 
        ...
      }, // content payload
      <Buffer 04 00 ff f7 3e 32 13 e9 45 4c 50 e8 eb 86 c3 c7 0d 1f 95 dd d2 29 95 41 c5 3b fa 50 8c 8b a3 f1 3a 6f ce 33 9d ba 61 70 12 b5 83 99 4f 75 8c 60 a3 fa ... 16 more bytes> // content signature
    ]
  ],
  <Buffer 04 00 8d e9 bc f9 8f 93 7b 49 69 a3 b8 b0 42 b9 08 c8 bc 0c f3 2d 43 50 08 84 20 14 06 e7 06 bc 21 f0 ff 29 a1 fc f9 55 25 9f c1 ac e0 90 17 f1 33 6a ... 16 more bytes> // payload signature
]
```

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
