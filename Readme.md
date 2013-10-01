# json64: Web-safe and binary formats for exchanging JSON

JavaScript Object Notation (JSON) is a text format for data-interchange.
JSON can use almost any Unicode character. JSON does not support binary data
(byte arrays) directly so they are often exchanged as base64url-encoded strings.
These two aspects can make JSON awkward and/or inefficient to transport data
directly in text-based and binary protocols. `web64` and `bin64` are
transformations to make JSON more suitable for text-based and binary protocols
respectively.

## Web64

`web64` converts a JSON text to a different text format that is suitable for
text-based web protocols. It escapes JSON text to a web-safe form that uses a
repertoire of 65 characters: `a`-`z` `A`-`Z` `0`-`9` `_` `-` `.` (64 characters
that can be used in a base64url-encoding, plus a dot). These characters can be
used without further escaping in many web contexts (eg HTTP headers, URI query
parameters, XML attributes).

`JSON.web64(text)` takes a string as input. The string is split into 1 or more
segments. Even segments (ie 2nd, 4th, 6th segment etc) MUST only consist of
characters from the repertoire of 64 used by a base64url-encoding. There are
no other restrictions on segments. An empty string is a valid segment, for
instance. Each odd segment (ie 1st, 3rd, 5th segment etc) is replaced with the
base64url-encoding (without `=` padding) of the UTF-8 encoding of the segment.
All segments are then joined, separated by a dot, to create the output string.

`JSON.deweb64(text)` reverses the transformation. It splits its input into 1
or more dot-separated segments. Each odd segment (ie 1st, 3rd, 5th segment etc)
is replaced with the UTF-8 decoding of the base64url-decoding of the segment.
All segments are then concatenated to create the output string.

`JSON.web64v(value)` is a shortcut for `JSON.web64(JSON.stringify(value))`.
`JSON.deweb64v(text)` is a shortcut for `JSON.parse(JSON.deweb64(text))`.

## Bin64

`bin64` converts a JSON text to a byte array, suitable for a binary protocol.
Segments of the JSON text that are a base64url-encoding can be decoded to
reduce their size by 25%.

`JSON.bin64(text)` takes a string as input. The string is split into 1 or more
segments. Even segments (ie 2nd, 4th, 6th segment etc) MUST only consist of
characters from the repertoire of 64 used by a base64url-encoding. The length
of each even segment modulo 4 MUST NOT be 1. A byte array is created for each
segment. Odd segments are UTF-8-encoded to give a byte array. Even segments
are base64url-decoded to give a byte array. Each byte array is prefixed by its
length encoded as a VARINT (variable-length integer). The output is the
concatenation of the length-prefixed byte arrays in order.

Each byte in a VARINT, except the last, has the most significant bit (msb) set.
The lower 7 bits of each byte store the binary representation of the number in
groups of 7 bits, least significant group first. For example, 50 is encoded
as `32` (hex); 500 is encoded as `F4 03` (hex).

`JSON.debin64(buf)` reverses the transformation of `JSON.bin64(text)`.

`JSON.bin64v(value)` is a shortcut for `JSON.bin64(JSON.stringify(value))`.
`JSON.debin64v(buf)` is a shortcut for `JSON.parse(JSON.debin64(buf))`.

## Example

...
