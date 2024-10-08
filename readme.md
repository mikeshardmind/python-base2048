### python-base2048

A pure python implementation of [base2048](https://github.com/ionite34/base2048)


The encoding map used is compatible with libraries available in
[Rust](https://github.com/LLFourn/rust-base2048) and
[js (with jsdoc typings)](https://www.npmjs.com/package/@mikeshardmind/base2048)

[The original python implementation](https://github.com/ionite34/base2048) which
this is intended to replace binds to the rust crate, which may make it marginally
faster on some platforms, but makes it less portable
and harder to use in vendoring situations.

The encoding map used is **not** compatible with the
[original implementation of the idea in js](https://github.com/qntm/base2048)
care was taken to ensure that the characters used display properly on
twitter, discord, and general text editors across major platforms.
These characters are also incidentally url safe, though users may find these are
practically impossible to manually type.

### installing

This is installable from either git or pypi

```
pip install mikeshardmind-base2048
```

The library is self-contained and is safe to vendor. Please note that the library
is licensed under MPL-2, which is a per-file copy-left license. If you do not modify
the files vendored, you have no further obligation as the license is included
in the header. If you would like this made available under another license, please
contact me regarding your use case.


### usage

The library provides two public functions: encode and decode.

encode turns bytes into encoded strings, decode reverses this.

These functions are one-shot and do not offer any options or configuration.

```py
>>> import base2048
>>> base2048.encode(b"data")
'Ҽĝġ'
>>> base2048.decode('Ҽĝġ')
b'data'
```


### typed?

The library provides typings that are known to work with pyright and mypy.
The assumption is that they also work with pyre and pytype based on the typing conformance suite,
however this has not been tested.

CI is set up to test library typings using pyright. any issues that arise with other type checkers
are most likely an issue with those type checkers, but feel free to open an issue if this breaks your
use in typed code.

### Why?

Sometimes you need to transfer data as text. base2048 with modern operating systems
is safe to assume it will display fine, and compares favorably with other
binary-text encodings on a few factors


| encoding     | bits/character\* | markdown-safe | url-safe | requires byte padding\*\* |
| ------------ | ---------------- | ------------- | -------- | ------------------------- |
| base64       | 6                | yes           | no       | yes                       |
| base85\*\*\* | 6.4              | no            | no       | yes                       |
| base2048     | 11               | yes           | no       | no                        |


\* for each of these encodings, this is the optimal bits/character. Whether or not
    you end up with *Wasted* characters will depend on if the encoding requires
    padding and if so, whether or not your data requires it.
    Base2048 has a dedicated set of characters used for residual bits to avoid
    padding. This means no extra characters are ever spent over the bits/character
    estimate.

\*\* All of the compared encodings are byte-aligned, and do not have a means to
    transfer arbitrary bit-length data.

\*\*\* The original base85 is not markdown-safe or url-safe. There are variants
    of it which are at least markdown safe, but none of the common variants are,
    this requires a custom encoding to use base85 on common chat
    platforms which use markdown.


### What uses this?

I use this in a few places, and thought others might find use.
If you do, feel free to let me know, and I may link back to you.

- This is used in multiple discord bots to store state in discord's custom_id field,
  allowing state to be held by discord. This allows for high concurrency without
  shared state synchronization or increasing complexity of worker dispatch.

- This is used in a not-yet-public error handling service to allow users to share
  errors with developers in common chat platforms in a compact representation
  without requiring them to share full logs that may contain identifying data.
  (This is something which is intended to be published in the future, and a
  link will be updated here if and when it is)

- This is being used to transmit public keys
  (Implementation details have not been shared at this time)

- This is used by [Wakforge](https://wakforge.org) to allow users to
  share customized builds with each other in chat apps.


### What's this binary blob in the wheel?

It's a zlib compressed character map to reduce the installed package size on disk
and over network, The file's contents should not change so long as the same character map is used.

It's handled via zlib decompression followed by struct unpacking with a strict
format specifier, malicious replacement's worst case would require a vulnerability
in zlib or python. You can verify the expected output yourself by generating it the same
way I did, with object_generation/gen.py


### Performance considerations

- This contains a binary blob descibed in the previous section for data used.
The file is decompressed and parsed *once*. This is done at import time.
The file is relatively small, and parsing of it is done using a very fast method
as this is struct packed data, but if this is being
used in a cli tool that does not always end up using it, you may want to import this
only within the scope it is used.

- This was negligibly slower than the rust bindings in a naive benchmark on python3.11.
comparing use on 3.10 + pypy vs the rust bindings on cpython 3.11
(no pypi wheel, and the source distribution failed to build on pypi),
this implementation with pypy was negligibly faster.
further benchmarking will be required to make any conclusive statements.

- Due to the nature of how this functions, and the ffi barrier when using a rust crate,
you should perform benchmarking on data representative of your own use if this is a
major concern to your use, and compare the performance to other options including the
linked alternative implementations above, as well as the performance of
less-efficient encodings (at a possible tradeoff)