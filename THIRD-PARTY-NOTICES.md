# Third-party notices for the packaged CLI

`Crap4CSharp.Tool` bundles the following runtime assemblies. This file maps each assembly to the notices shipped with its source NuGet package. The source packages declare the MIT license; the project's own license is CC0-1.0 in `LICENSE`.

| Bundled assembly | Source NuGet package | Notices included here |
| --- | --- | --- |
| `Microsoft.CodeAnalysis.dll` and its localized resources | `Microsoft.CodeAnalysis.Common` 4.14.0 | `licenses/MIT.txt`, `licenses/Roslyn-4.14.0-ThirdPartyNotices.rtf` |
| `Microsoft.CodeAnalysis.CSharp.dll` and its localized resources | `Microsoft.CodeAnalysis.CSharp` 4.14.0 | `licenses/MIT.txt`, `licenses/Roslyn-4.14.0-ThirdPartyNotices.rtf` |
| `System.Collections.Immutable.dll` | `System.Collections.Immutable` 9.0.0 | `licenses/MIT.txt`, `licenses/DotNetRuntime-9.0.0-ThirdPartyNotices.txt` |
| `System.Reflection.Metadata.dll` | `System.Reflection.Metadata` 9.0.0 | `licenses/MIT.txt`, `licenses/DotNetRuntime-9.0.0-ThirdPartyNotices.txt` |

The Roslyn notices are byte-for-byte identical in the two Roslyn source packages. The .NET Runtime notices and MIT license are byte-for-byte identical in the two `System.*` source packages. Each shared notice is therefore included once. The original notice files are kept without edits.
