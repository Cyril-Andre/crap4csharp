# Feature: versioned CLI distribution

**Feature baseline:** `feature/machine-readable-output`

**Status:** Preview package verified; public name decision pending

## Requirements

Ship the existing JSON-capable CLI as an installable, versioned .NET tool for CRAP IDE users. Preserve the CLI behavior and keep publication as a separate decision. Mr. Das approved the .NET tool format and the `crap4csharp` command on 2026-10-28.

## Design options

| Option | Result | Tradeoff |
| --- | --- | --- |
| .NET tool | One NuGet package, one command on all supported hosts | Requires a compatible .NET runtime and a separate NuGet publication |
| Platform archives | Native/self-contained downloads per OS and architecture | Larger release matrix and more installation paths to document |

**Decision:** package `Crap4CSharp.Tool` as a framework-dependent .NET tool targeting .NET 8, with command `crap4csharp`. The package ID distinguishes the maintained fork from a Microsoft-owned package name. Publish only after separate authorization; the local package and CI artifact are reviewable first.

The package declares the repository's CC0-1.0 license and contributor attribution. Its NuGet metadata must not imply that Microsoft publishes or endorses this fork.

## Slices

| Slice | Outcome |
| --- | --- |
| S1 | Package metadata and installation instructions |
| S2 | Build, install, and JSON smoke test from the packaged tool |
| S3 | CI artifact and cross-host validation |

## Risks and limits

- The upstream JSON contribution is still under review. This package pins the fork's tested contract; an upstream release can replace it after a compatibility check.
- `crap4csharp` invokes `dotnet test` on target projects, so users need a suitable .NET SDK and the target's test dependencies even though the tool itself targets .NET 8.
- On 2026-10-30, the official NuGet flat-container endpoint returned 404 for `Crap4CSharp.Tool`, and a NuGet search for `crap4csharp` found only `crap4csharp` by another publisher. This establishes that the proposed package ID has no published versions at the time of the check; it does not reserve the ID. NuGet assigns package ownership to the publishing account, not the `.nuspec` author. Recheck availability and choose the publisher account immediately before any public publication.
- The existing `crap4csharp` package is also a .NET tool. Its published `0.5.1` archive declares `<Command Name="crap4csharp">` in `tools/net10.0/any/DotnetToolSettings.xml`, confirming an exact command-name collision. Mr. Das needs to decide whether to keep this command name or choose a distinct one before public publication.
- The four bundled runtime dependencies declare MIT. Their original MIT text and third-party notices are included in the package, with the mapping in `THIRD-PARTY-NOTICES.md`. Recheck the mapping whenever dependency versions or packaged assemblies change.

## Local verification

- With .NET SDK 8.0.416 in a network-isolated container, locked restore, Release pack, local tool installation, `--help`, and empty JSON output passed.
- The packaged command analyzed this checkout, ran 259 passing tests, emitted a populated JSON report and returned the expected threshold exit code `2`.
- The Release development suite passed 259 tests. The package metadata contains the CC0-1.0 license and README, and the archive contains no `.cs` source files.
- On 2026-10-30, the rebuilt package included the project license plus all mapped third-party notice files, byte-for-byte matching the repository copies. The package install, help, JSON smoke test, and notice archive check passed.
