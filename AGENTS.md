# AGENTS.md

## Commands

- Run repo-wide build tooling from the repository root; `SboxBuild` assumes the current directory contains `engine/` and `game/`.
- Full bootstrap: `./Bootstrap.bat` runs Developer build, shader build, then content build.
- CI build: `dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- build --config Developer`.
- Faster C#-only build: `dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- build --config Developer --skip-native`.
- Verify formatting: `dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- format --verify`.
- Apply formatting: `dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- format`.
- Run all tests through the repo wrapper: `dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- test`.
- After a successful build, skip rebuilding for tests with `dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- test --no-build`.
- Focused MSTest example from PowerShell: `$env:FACEPUNCH_ENGINE="$PWD\game"; dotnet test .\engine\Sandbox.Test.Unit\Sandbox.Test.Unit.csproj -c Release --filter FullyQualifiedName~Vector3`.
- PR checks on GitHub are only `format` and `tests`; triage automation requires both check names to pass.

## Build And Generation

- The repo targets .NET 10 SDK and Visual Studio 2026 per `README.md` and CI setup.
- `SboxBuild build` treats this checkout as a public source distribution when `public/` or `steamworks/` is missing; in that mode it downloads public artifacts and skips native build.
- Managed builds restore `engine/`, build `Tools/CodeGen` and `Tools/CreateGameCache`, clear `game/bin/managed`, then build `engine/Sandbox-Engine.slnx` in Release with warnings as errors.
- `engine/Sandbox.Engine` imports `engine/CodeGen.Targets`; design-time builds skip codegen, normal builds run `Tools/CodeGen` before compile and include `obj/.generated/**/*.cs`.
- `Interop.*.cs` files under `engine/**` are generated and gitignored; regenerate via the build pipeline instead of hand-editing them.
- Shader packing runs from `engine/Definitions/shaders.def`; `build-shaders` requires `game/bin/managed/shadercompiler.exe`, so build managed first.
- `build-content` requires `game/bin/win64/contentbuilder.exe`; it runs content building from the `game/` directory.

## Layout

- Main managed solution: `engine/Sandbox-Engine.slnx`.
- Build tool solution: `engine/Tools/Sandbox-Tools.slnx`; main entrypoint is `engine/Tools/SboxBuild/Program.cs`.
- Runtime/editor binaries and content live under `game/`; launchers such as `game/sbox-dev.exe` are build/runtime artifacts, not source entrypoints.
- Addon and sample projects under `game/` can generate local `.slnx`, `.csproj`, `obj`, `.sbox`, IDE, and transient asset files; these are generally debris unless the task is explicitly about project assets.

## Style

- `.editorconfig` uses tabs with width 4 for C# and Razor, 2 spaces for project/config JSON/XML, CRLF, and final newlines.
- C# style in this repo prefers explicit types over `var`, file-scoped namespaces, braces, and static local functions where possible.
- `.gitattributes` forces CRLF for `game/**` content except C# files; avoid normalizing line endings in game assets.

## Tests

- Test projects use MSTest (`[TestMethod]`) with `Microsoft.NET.Test.Sdk`.
- The test wrapper runs `dotnet test` from `engine/` and sets `FACEPUNCH_ENGINE` to the repo `game/` directory; set that env var when running focused `dotnet test` directly.
- `pullrequest` pipeline does extra Windows-only steps: shader build is allowed to fail, then content build, tests, and addon/menu build run.
