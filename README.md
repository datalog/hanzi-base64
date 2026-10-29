# hanzi-base64

A compact, lossless representation of the GB2312 lookup table for QR Hanzi mode (subset 1).

This repository provides the original table and its compressed Base64 representation. It uses the format and tools from **[kanji-base64](https://github.com/datalog/kanji-base64)**, where the algorithm, command format, encoder, and decoder are documented.

## The Hanzi table

Hanzi mode uses 13-bit values to represent characters from GB2312, a character set containing simplified Chinese ideographs, kana, alphabets, and symbols. Each value identifies a fixed position in the lookup table.

The table has **8192 positions: 7445 characters and 747 empty entries**. Each entry stores a Unicode code point or `0xFFFF` for an unused position. Compression preserves every value and its index.

The encoder uses dynamic programming to minimize the command stream size within the format's supported commands.

## Files

- [hanzi-lookup-table.js](hanzi-lookup-table.js): `HANZI_LOOKUP_TABLE`, the original array mapping QR Hanzi indices to Unicode code points.
- [hanzi-lookup-table-base64.js](hanzi-lookup-table-base64.js): `HANZI_LOOKUP_TABLE_BASE64`, the same table encoded as a Base64 command stream.

## Size comparison

The original binary table uses two bytes per entry. The comparison below measures that table and two ways of encoding it:

- **Delta encoder:** literals, deltas, and empty runs.
- **Full encoder:** the same commands, plus constant-step series and continuations. This produces the bundled Base64 file.

| Method | Binary | Base64 | Gzip of Base64 |
| :--- | ---: | ---: | ---: |
| Original | 16 384 bytes | 21 848 chars | 15 128 bytes |
| Delta encoder | 13 226 bytes | 17 636 chars | 12 281 bytes |
| Full encoder | 12 701 bytes | 16 936 chars | 12 264 bytes |
| **Full encoder savings** | **22.48%** | **22.48%** | **18.93%** |

Sizes cover the data payload only, excluding JavaScript wrappers and decoder code. The original binary uses big-endian byte order; Base64 encodes the bytes directly. gzip compression uses level 9; results may vary with compressor version. Savings compare the Full encoder with the Original in each column.

## Usage

Place [lookup-table-decoder.js](https://github.com/datalog/kanji-base64/blob/main/lookup-table-decoder.js) from the main project alongside `hanzi-lookup-table-base64.js`, then load both files:

```html
<script src="hanzi-lookup-table-base64.js"></script>
<script src="lookup-table-decoder.js"></script>
<script>
var lookup = decodeKanjiTable(HANZI_LOOKUP_TABLE_BASE64);
console.log(lookup[0x4E2D]); // 4655: 中
</script>
```

The function retains its original name, `decodeKanjiTable`, and accepts this table without modification. It returns an `Int16Array(65536)` mapping Unicode code points to QR Hanzi indices, with `-1` for unsupported characters.

To regenerate the Base64 data, load `hanzi-lookup-table.js` and [lookup-table-encoder.js](https://github.com/datalog/kanji-base64/blob/main/lookup-table-encoder.js), then call:

```js
var base64 = encodeTable(HANZI_LOOKUP_TABLE);
```

These tools pack and unpack the lookup table; generating a QR symbol requires a QR encoder with Hanzi mode support.

## Table construction

The character set is defined by [GB/T 2312-1980](https://openstd.samr.gov.cn/bzgk/std/newGbInfo?hcno=5664A728BD9D523DE3B99BC37AC7A2CC). Each 13-bit QR Hanzi index is converted to a GB2312 byte pair, then mapped to Unicode. Undefined pairs remain empty; GBK extensions are excluded.

All 8192 positions were restored after encoding and decoding and compared with the original table.
