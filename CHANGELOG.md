# Changelog

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
the project follows [Semantic Versioning](https://semver.org/).

## Unreleased

- Characters above U+FFFF now round-trip through `Form.str`. They are
  rendered as `\U` followed by eight hex digits instead of `\u` with
  five or six digits, which could not be re-parsed. Character literals
  also accept `\U` escapes when reading.
- The `\x` escape inside string literals now consumes at most two hex
  digits (one byte), matching C. Previously trailing hex digits were
  silently folded into the low byte (e.g. `\x413` produced a single
  byte instead of `A` followed by `3`).

## [0.3.6]

- String parser now rejects invalid escape sequences (e.g. `\z`,
  `\0`) with a parse error instead of silently emitting raw bytes.
  `\u` and `\U` escapes also validate that the required hex digits
  are actually hex characters.
- `Form.str` uses `StringBuf` for `join-children`, `join-segments`,
  and `escape-string`, replacing O(n²) `String.append` loops with
  amortised O(n) appends. Adds a dependency on `strbuf@0.1.0`.

## [0.3.0]

- `Located` gains an `end Info` field tracking the position just past
  the form's last byte. Useful for tools (e.g. formatters) that need
  to detect gaps between siblings in the original source. Breaking:
  `Located.init` now takes three arguments and the deftype has three
  fields.
- `Form.str` emits literal newlines inside string literals rather
  than escaping them as `\n`. Multi-line string literals in source
  round-trip readably. Breaking for consumers relying on
  fully-escaped output.
- String parser now preserves UTF-8 byte sequences verbatim. The
  previous accumulator-as-`Array Char` plus `String.from-chars`
  double-encoded multi-byte UTF-8 input.
- Number parser accepts scientific notation: `1e6`, `1e+06`,
  `1.5e-3`, `1e6f`. Matches the output of `Double.str` so doubles
  round-trip cleanly.

## [0.2.0]

- Every parsed node carries an `Info` (byte offset, line, column). The
  parser now emits `Located` (a `Form` paired with `Info`) at every
  level; `Form`'s recursive variants hold `(Array (Box Located))`.
  `Reader.parse` and `Reader.parse-form` return `Located` accordingly.
  Breaking.

## [0.1.0]

- Initial release.
