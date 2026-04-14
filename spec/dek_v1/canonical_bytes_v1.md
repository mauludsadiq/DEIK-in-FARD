# DEK Canonical Bytes v1

Status: FROZEN
Version: 1.0.0

## Overview

The DEK byte encoding maps every canonical value to a unique byte sequence.
Every value starts with a 1-byte tag. The tag determines the payload layout.

## Tag Table

| Tag | Hex  | Type            | Payload                                              |
|-----|------|-----------------|------------------------------------------------------|
| 0   | 0x00 | null            | (none)                                               |
| 1   | 0x01 | bool false      | (none)                                               |
| 2   | 0x02 | bool true       | (none)                                               |
| 3   | 0x03 | int64           | 8 bytes big-endian two's complement                  |
| 4   | 0x04 | float64         | 8 bytes IEEE 754 big-endian                          |
| 5   | 0x05 | text            | u64_be(len) ++ utf8_bytes                            |
| 6   | 0x06 | bytes           | u64_be(len) ++ raw_bytes                             |
| 7   | 0x07 | list            | u64_be(count) ++ encode(item)*                       |
| 8   | 0x08 | record          | u64_be(count) ++ (encode_text(key) ++ encode(val))*  |
| 9   | 0x09 | error           | encode(record{code,data,message,phase})               |
| 10  | 0x0a | option_none     | (none)                                               |
| 11  | 0x0b | option_some     | encode(inner)                                        |
| 12  | 0x0c | result_ok       | encode(inner)                                        |
| 13  | 0x0d | result_err      | encode(inner)                                        |
| 14  | 0x0e | cid_ref         | u64_be(len) ++ utf8_bytes                            |
| 15  | 0x0f | trace_frame     | encode(record v)                                     |
| 16  | 0x10 | oracle_event    | encode(record v)                                     |
| 17  | 0x11 | bundle          | encode(record v)                                     |
| 18  | 0x12 | digest          | encode(record v)                                     |
| 19  | 0x13 | program_ref     | encode(record v)                                     |
| 20  | 0x14 | dep_list        | encode(record v)                                     |
| 21  | 0x15 | sensor_snapshot | encode(record v)                                     |
| 22  | 0x16 | timestamp       | encode(record v)                                     |
| 23  | 0x17 | bigint          | sign_byte ++ u64_be(mag_len) ++ mag_bytes            |
| 24  | 0x18 | map             | u64_be(count) ++ (encode(key) ++ encode(val))*       |
| 25  | 0x19 | set             | u64_be(count) ++ encode(item)*                       |
| 26  | 0x1a | receipt         | encode(record v)                                     |

## Length Encoding

All lengths and counts are encoded as unsigned 64-bit big-endian integers (8 bytes).

## Bigint Sign Byte

    0x00  zero
    0x01  positive
    0x02  negative

## Known Stable Encodings

    encode(null)         = 00
    encode(false)        = 01
    encode(true)         = 02
    encode(int 0)        = 03 0000000000000000
    encode(int 1)        = 03 0000000000000001
    encode(int -1)       = 03 ffffffffffffffff
    encode(text "")      = 05 0000000000000000
    encode(list [])      = 07 0000000000000000
    encode(record [])    = 08 0000000000000000
    encode(option_none)  = 0a
    encode(text "abc")   = 05 0000000000000003 616263

## CID Derivation

    cid(v) = "sha256:" ++ hex(sha256(encode(v)))

## Known Stable CIDs

    cid(null)         = sha256:6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
    cid(bool false)   = sha256:4bf5122f344554c53bde2ebb8cd2b7e3d1600ad631c385a5d7cce23c7785459a
    cid(bool true)    = sha256:dbc1b4c900ffe48d575b5da5c638040125f65db0fe3e24494b76ea986457d986
