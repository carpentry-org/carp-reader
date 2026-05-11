# Changelog

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
the project follows [Semantic Versioning](https://semver.org/).

## [0.2.0]

- Every parsed node carries an `Info` (byte offset, line, column). The
  parser now emits `Located` (a `Form` paired with `Info`) at every
  level; `Form`'s recursive variants hold `(Array (Box Located))`.
  `Reader.parse` and `Reader.parse-form` return `Located` accordingly.
  Breaking.

## [0.1.0]

- Initial release.
