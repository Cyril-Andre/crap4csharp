# Feature: versioned CLI distribution

**Feature baseline:** `feature/machine-readable-output`

**Status:** Renamed preview package verified locally; cross-host CI pending

## Requirements

Ship the existing JSON-capable CLI as an installable, versioned .NET tool required by CRAP IDE extensions. Preserve the analysis and report contract and keep publication as a separate decision. Mr. Das approved the .NET tool format on 2026-10-28 and superseded the initial `crap4csharp` command choice on 2026-10-30.

## Design options

| Option | Result | Tradeoff |
| --- | --- | --- |
| .NET tool | One NuGet package, one command on all supported hosts | Requires a compatible .NET runtime and a separate NuGet publication |
| Platform archives | Native/self-contained downloads per OS and architecture | Larger release matrix and more installation paths to document |

**Decision:** package `Crap4CSharp.IDE.Cli` as a framework-dependent .NET tool targeting .NET 8, with command `crap4csharp-ide`. The ID and command identify this fork as the CLI backend required by CRAP IDE extensions. Publish only after separate authorization; the local package and CI artifact are reviewable first.

The package declares the repository's CC0-1.0 license and contributor attribution. Its NuGet metadata must not imply that Microsoft publishes or endorses this fork.

## Slices

| Slice | Outcome |
| --- | --- |
| S1 | Package metadata and installation instructions |
| S2 | Build, install, and JSON smoke test from the packaged tool |
| S3 | CI artifact and cross-host validation |

## Risks and limits

- The upstream JSON contribution is still under review. This package pins the fork's tested contract; an upstream release can replace it after a compatibility check.
- The CLI invokes `dotnet test` on target projects, so users need a suitable .NET SDK and the target's test dependencies even though the tool itself targets .NET 8.
- Recheck `Crap4CSharp.IDE.Cli` ID availability and publisher ownership immediately before public publication. A NuGet package ID is claimed by the publishing account, not by the `.nuspec` author.
- The separate NuGet `crap4csharp` package declares the `crap4csharp` command. The chosen `crap4csharp-ide` command avoids that conflict. Release the matching VS Code/Cursor extension fallback alongside this package.
- The four bundled runtime dependencies declare MIT. Their original MIT text and third-party notices are included in the package, with the mapping in `THIRD-PARTY-NOTICES.md`. Recheck the mapping whenever dependency versions or packaged assemblies change.

## Local verification

- With .NET SDK 8.0.416 in a network-isolated container, locked restore, Release pack, local tool installation, `--help`, and empty JSON output passed.
- The packaged command analyzed this checkout, ran 259 passing tests, emitted a populated JSON report and returned the expected threshold exit code `2`.
- The Release development suite passed 259 tests. The package metadata contains the CC0-1.0 license and README, and the archive contains no `.cs` source files.
- On 2026-10-30, the rebuilt package included the project license plus all mapped third-party notice files, byte-for-byte matching the repository copies. The package install, help, JSON smoke test, and notice archive check passed.
- After the identity change, a network-isolated .NET SDK 8.0.416 container passed locked solution restore and 259 Release tests. It packed `Crap4CSharp.IDE.Cli.0.1.0-preview.1.nupkg`, installed it locally, ran `crap4csharp-ide --help` and empty JSON successfully, and confirmed the tool manifest command plus all required notice entries. Cross-host CI awaits the branch update.
