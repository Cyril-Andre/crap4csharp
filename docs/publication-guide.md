# Publishing the CLI and IDE adapters

This guide describes release preparation; it does not authorize a publication. Status checked on
2026-10-30. The current deliverables are the `Crap4CSharp.IDE.Cli` preview package in this repository
and the `crapide-vscode` preview VSIX in [Cyril-Andre/crapide](https://github.com/Cyril-Andre/crapide).
Neither is publicly published by these projects yet. The VSIX does **not** contain the CLI.
The current [notice/package PR #2](https://github.com/Cyril-Andre/crap4csharp/pull/2) is a draft
targeting `feature/machine-readable-output`; merge the reviewed release changes into that branch and
then into the fork's release branch before packaging a public build.

## Provenance and product boundary

The CLI here extends [microsoft/crap4csharp](https://github.com/microsoft/crap4csharp) with JSON and
SARIF output, source locations, and stable machine-readable behavior for IDE integration. The C#
project is a port of Robert C. Martin's [crap4java](https://github.com/unclebob/crap4java), in the
lineage of his [crap4clj](https://github.com/unclebob/crap4clj). Alberto Savoia and Bob Evans
[created the CRAP metric](https://testing.googleblog.com/2011/02/this-code-is-crap.html) itself.
[CRAP IDE](https://github.com/Cyril-Andre/crapide) calls this CLI as an external process. This CLI is
the required analysis backend: the IDE adapters consume its results instead of calculating CRAP scores.
JSON and SARIF carry the data needed by adapters for VS Code, Cursor, Rider, and Visual Studio. The CLI
must be installed on the host that runs each extension and must support the documented output contract.

The upstream [JSON/SARIF PR #11](https://github.com/microsoft/crap4csharp/pull/11) was still open at
this check. Until it is merged and released upstream, install a version built from this fork when using
CRAP IDE. An upstream merge alone does not supply a published tool or promise an identical contract.
This fork's package metadata must identify Cyril André as publisher and must not suggest Microsoft or
Uncle Bob publishes or endorses it.

## Release identity

Another publisher already distributes a NuGet tool under the ID
[`crap4csharp`](https://www.nuget.org/packages/crap4csharp/0.5.1), with the executable command
`crap4csharp`. That package is unrelated to this fork. Its existence matters only for naming and
installation: `dotnet tool install -g crap4csharp` would install the other tool, and a second global
tool with the same command can collide. CRAP IDE must document its own exact package ID and command.

Mr. Das chose the package ID `Crap4CSharp.IDE.Cli` and executable command `crap4csharp-ide` on
2026-10-30. The names distinguish the required CRAP IDE backend from the other NuGet tool. Recheck
the package ID's availability and publisher ownership immediately before release; a source checkout
does not reserve a NuGet ID. The VS Code/Cursor extension's prepared release branch now looks for
`crap4csharp-ide` on `PATH` and names this package in its setup guide. Merge and release that change
with the CLI package; an explicit `crapide.cliPath` remains available without changing the JSON contract.

## Release sequence

1. **Set release version and publishers.** Use the chosen CLI package ID and command; choose the NuGet
   publisher account, VS Code Marketplace publisher, and Open VSX namespace. The VS Code manifest says
   `publisher: crapide`, `name: crapide-vscode`, `version: 0.1.0`, and `private: true`. Confirm registry
   namespace ownership; remove `private` only when the public release is approved. Use the same
   `publisher.extension` identity on both extension registries. Reserve a new version for every public
   release; do not assume a previously uploaded version can be replaced.
2. **Freeze the CLI contract.** Pin the exact fork commit and document `--format json`, stdout/stderr,
   source paths, `coveragePercent`, null values, and exit codes `0`/`1`/`2`. Compare the package against
   [CRAP IDE's JSON contract](https://github.com/Cyril-Andre/crapide/blob/main/contracts/json-contract.md)
   and its real-CLI test. An upstream release can replace the fork only after the same compatibility
   check.
3. **Build and verify the CLI.** Perform locked restore, Release build/test, `dotnet pack`, and a fresh
   local tool installation on the supported OS matrix. Verify `--help`, valid empty and populated JSON,
   exit `2` with valid JSON, fatal exit `1`, packaged license/notices, and package metadata. CI already
   packs and smokes the preview on Ubuntu, Windows, and macOS. Test an installation alongside any
   existing `crap4csharp` tool to verify the distinct command. The target projects still need a
   compatible .NET SDK, `coverlet.collector`, and a supported test-project layout.
4. **Publish the CLI first.** In the chosen nuget.org account, configure a scoped API key or trusted
   publishing, then push the reviewed `.nupkg` to nuget.org. Keep credentials out of the repository.
   Verify the public listing, owner, repository URL, license, version, downloaded package contents, and
   a clean `dotnet tool install` from nuget.org. Announce the exact package ID and command in CRAP IDE's
   install guide as the **required backend for CRAP IDE**. A local tool manifest or `--tool-path`
   installation is safer than a global install when commands collide.
5. **Release the VS Code/Cursor VSIX.** In `crapide/extensions/vscode`, run `npm ci`, `npm run check`,
   `npm test`, `npm run package`, and its VSIX allowlist check. Run the optional real-CLI contract and
   normal-window checks against the *published* CLI on VS Code and Cursor, including a remote workspace
   host. Update README, changelog, installation/troubleshooting, minimum editor version, screenshots,
   and the external CLI prerequisite. Confirm that the package contains compiled JS, metadata, and MIT
   license but no CLI binary or secrets.
6. **Publish the same extension version to both registries.** Register and verify the `crapide`
   publisher on the VS Code Marketplace; publish the VSIX there for VS Code. Create/claim the `crapide`
   namespace on Open VSX, accept its publisher agreement, and publish the same reviewed VSIX there for
   Cursor. Cursor uses Open VSX through its own marketplace proxy and review, so appearance in Cursor
   search may follow Open VSX publication rather than being immediate. Check both listings, publisher
   identity, version, installability, command activation, Problems, and CodeLens in clean profiles.
7. **Maintain the release.** Tag the CLI and extension source independently, record package/VSIX
   digests and tested versions, monitor installation reports, and publish fixes with new versions.

The publication commands, **only after approval and with the chosen identities**, are conceptually:

```sh
# In this repository: package and publish the approved CLI version.
dotnet pack src/Crap4CSharp/Crap4CSharp.csproj -c Release -o artifacts/cli
dotnet nuget push artifacts/cli/<chosen-id>.<version>.nupkg \
  --source https://api.nuget.org/v3/index.json --api-key <scoped-key>

# In crapide/extensions/vscode: publish one verified VSIX to both registries.
npm ci
npm run package
npx vsce publish --packagePath crapide-vscode-<version>.vsix
npx ovsx publish crapide-vscode-<version>.vsix -p <open-vsx-token>
```

Use secret storage or trusted publishing rather than putting credentials into a shell history or
repository file. The VS Code Marketplace and Open VSX have separate publisher accounts and release
records. Publishing the CLI does not install it for users: the extension listing must explain that
users install it separately on the workspace host, including inside Dev Containers or remote hosts.
The extension listing must also disclose that analysis invokes the project's tests and replaces
`coverage/` in the invocation folder.

Official release references: [NuGet publication](https://learn.microsoft.com/en-us/nuget/nuget-org/publish-a-package),
[VS Code publishing](https://code.visualstudio.com/api/working-with-extensions/publishing-extension),
[Open VSX publishing](https://github.com/eclipse-openvsx/openvsx/wiki/Publishing-Extensions),
[Cursor extension registry](https://prod.cursor.com/help/customization/extensions).
