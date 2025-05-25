# BINEX

[![Rust](https://github.com/rtk-rs/binex/actions/workflows/rust.yml/badge.svg)](https://github.com/rtk-rs/binex/actions/workflows/rust.yml)
[![Rust](https://github.com/rtk-rs/binex/actions/workflows/daily.yml/badge.svg)](https://github.com/rtk-rs/binex/actions/workflows/daily.yml) [![crates.io](https://img.shields.io/crates/v/binex.svg)](https://crates.io/crates/binex) 
[![crates.io](https://docs.rs/binex/badge.svg)](https://docs.rs/binex/badge.svg)

[![MRSV](https://img.shields.io/badge/MSRV-1.82.0-orange?style=for-the-badge)](https://github.com/rust-lang/rust/releases/tag/1.82.0)
[![License](https://img.shields.io/badge/license-MPL_2.0-orange?style=for-the-badge&logo=mozilla)](https://github.com/rtk-rs/binex/blob/main/LICENSE)

BINEX is a simple library to decode and encode BINEX messages.  
BINEX stands for BINary EXchange and is the "real time" stream oriented
version of the RINEX format. It is to this day, the only open source protocol
to encode GNSS and navigation data.

While RINEX is readable and based on line termination, BINEX is real-time and
hardware orientated (at the GNSS receiver firmware level).

This library allows easy message encoding and decoding, and aims at providing seamless
convertion from RINEX back and forth.

You have two scenarios to approach a BINEX stream:

* use our Decoder object, which works on I/O interface directly
and can represent a stream of continuous of either Messages (open source)
or undisclosed elements. (private prototypes)

* or use Message::decode to work on your own buffer directly.

Current limitations
===================

+ Big endian streams are fully validated & tested
+ Little endian streams are tested & verified but we don't have a dataset to confirm yet
+ Enhanced CRC (robust messaging) is not supported yet
+ MD5 checksum (very lengthy message prototypes) is implemented but not verified yet

Message Decoding
================

Use the BINEX `Decoder` to decode a `Readable` interface streaming
BINEX messages. Decoder exposes both open source Messages that
were fully interprated and closed source Messages (undisclosed prototypes)
that it cannot interprate:

```rust
use std::fs::File;
use flate2::read::GzDecoder;
use binex::prelude::{Decoder, StreamElement, Provider, Error};

let fd = File::open("data/BIN/mfle20200105.bnx.gz")
    .unwrap();

let mut decoder = Decoder::new(GzDecoder::new(fd));

loop {
    match decoder.next() {
        Some(Ok(StreamElement::OpenSource(msg))) => {
            // fully interprated element
        },
        Some(Ok(StreamElement::ClosedSource(element))) => {
            // grab content you will need to interpate
            let closed_meta = element.closed_meta; 
            let open_meta = closed_meta.open_meta;

            // verify this is your organization
            if closed_meta.provider == Provider::JPL {

                // grab fields that you probably need to decode
                let big_endian = open_meta.big_endian;
                let is_reversed = open_meta.reversed;
                let enhanced_crc = open_meta.enhanced_crc;

                let mid = closed_meta.mid; // message ID
                let mlen = closed_meta.mlen; // total message length
                let chunk_size = closed_meta.size; // chunk length

                // now, proceed to interpretation of this element,
                // using undisclosed method
                element.interprate(&|data| {
                    match mid {
                        _ => {},
                    }
                });
            }
        },
        Some(Err(e)) => {
            // it is possible that some frames may not
            // be supported yet.
            // Any I/O error should not happen.
        },
        None => {
            // end of stream
            break;
        },
    }
}
```

Message Forging
===============

The BINEX library allows easy message forging. Each message can be easily encoded and then
streamed into a `Writable` interface:

```rust
```

License
=======

This library is part of the [RTK-rs framework](https://github.com/rtk-rs) which
is delivered under the [Mozilla V2 Public](https://www.mozilla.org/en-US/MPL/2.0) license.
