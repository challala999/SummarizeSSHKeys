# AI Agent Instructions for SummarizeSSHKeys

## Project Overview
A Haskell utility that parses SSH `authorized_keys` files and produces human-readable summaries. The core value is making it easy to audit SSH key lists by extracting key metadata (type, hash, comment) and filtering option noise.

## Architecture

### Core Components
- **[SSHKeys.hs](../SSHKeys.hs)**: Parser module using Parsec that defines the OpenSSH public key format grammar
  - `Line` ADT with three variants: `Entry` (key metadata), `Comment`, `EmptyLine`
  - `Entry` tuple structure: `([Option], String, String, String)` = (options, key-type, hash, comment)
  - Parser combinators follow standard Parsec patterns with `<|>` for alternatives and `sepEndBy` for repetition
- **[SummarizeSSHKeys.hs](../SummarizeSSHKeys.hs)**: CLI executable that orchestrates parsing and formatting
  - Uses `optparse-applicative` for command-line arg parsing
  - Reads from stdin by default (`/dev/stdin`) or accepts file arguments
  - Terminal-aware formatting with `terminal-size` library

### Data Flow
1. CLI accepts files (defaults to stdin)
2. `parseFile` function (Parsec) parses SSH key lines into `Line` ADT
3. Pattern matching filters for `Entry` type
4. `summOpts`, `summKind`, `summHash` functions format output columns

## Key Patterns

### Parsec Parser Design
- **Custom type alias**: `type Parser = Parsec String ()` (no parser state, String input)
- **Composition strategy**: Build complex parsers from simple primitives using monadic do-notation
- **Error handling**: Left/Right pattern for `Either ParseError` results
- **Test-driven validation**: Hardcoded `testData` and `testResult` in SSHKeys.hs verify parser correctness

### Testing Framework
- Uses `HUnit` test framework embedded in source files
- [QuietTesting.hs](../QuietTesting.hs) provides `runTestTTquiet` wrapper to suppress verbose HUnit output (write errors/failures only to stderr)
- [TestMain.hs](../TestMain.hs) aggregates tests and exits with status code if failures exist
- Test suite runs via `cabal test`

### Build & Packaging
- **Cabal**: Primary build tool - `cabal build` compiles, `cabal test` runs tests
- **Nix**: Alternative reproducible build using [default.nix](../default.nix) with `nix-build`
- **Debian**: Packaged via [debian/](../debian/) with source format 3.0 (quilt)

## Development Workflows

### Build
```bash
cabal build                    # Compile executable
cabal test                     # Run test suite
```

### Add Features
1. **Parser changes**: Modify combinators in [SSHKeys.hs](../SSHKeys.hs), update `testData`/`testResult` in same file
2. **CLI changes**: Extend `Options` record in [SummarizeSSHKeys.hs](../SummarizeSSHKeys.hs), adjust `optparse-applicative` parsers
3. **Output formatting**: Adjust `summOpts`, `summKind`, `summHash` formatting functions

### Testing Strategy
- Embed test cases directly in [SSHKeys.hs](../SSHKeys.hs) alongside parser definitions
- Use `~?=` operator for HUnit assertions
- Reload test data string immediately when parser grammar changes

## Project-Specific Conventions
- **Haskell pragmas**: ApplicativeDo, LambdaCase, RecordWildCards are in use - check file headers
- **Error handling**: Use `Either` type directly, no exceptions or custom error types
- **Column-based output**: printf formatting with fixed widths - maintain alignment when modifying formatters
- **Platform consistency**: Parsec version constraint is wide (≥2.0, <4.0) to support multiple GHC versions

## External Dependencies
- **parsec**: Parsing library (version 2.0-3.x) - core parser combinator framework
- **optparse-applicative**: CLI argument parsing with help/defaults
- **terminal-size**: Terminal width detection for responsive formatting
- **HUnit**: Testing framework
- **base**: GHC 4.5+

## File Reference Guide
| File | Purpose |
|------|---------|
| [SummarizeSSHKeys.hs](../SummarizeSSHKeys.hs) | Main executable entry point |
| [SSHKeys.hs](../SSHKeys.hs) | Parser module + embedded tests |
| [QuietTesting.hs](../QuietTesting.hs) | HUnit test output suppression helper |
| [TestMain.hs](../TestMain.hs) | Test aggregation and exit code handling |
| [SummarizeSSHKeys.cabal](../SummarizeSSHKeys.cabal) | Package metadata & dependencies |
