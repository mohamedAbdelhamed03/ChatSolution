# AGENTS.md

## Cursor Cloud specific instructions

This repo is a single **ASP.NET Core (`net10.0`) Web API** project, `ChatSolution`. The `docs/`
directory describes an ambitious E2EE messaging platform, but the committed code is currently only
the default Web API scaffold (a `GET /weatherforecast` minimal-API endpoint + OpenAPI).

### Toolchain
- The **.NET 10 SDK** is installed at `~/.dotnet` (user-local, not a system/apt install) and added to
  `PATH` via `~/.bashrc`. It is invoked as `dotnet` in login shells; scripts that don't source
  `~/.bashrc` can call it via `~/.dotnet/dotnet`.
- There is no `global.json`, so the SDK version is not pinned.

### Build / run / test
- Build (also serves as the compile + analyzer check): `dotnet build`.
- Run in dev mode: `dotnet run --launch-profile http` → serves `http://localhost:5042`
  (`ASPNETCORE_ENVIRONMENT=Development`). The `https` profile also exposes `https://localhost:7281`.
- Endpoints: `GET /weatherforecast` (JSON), and `GET /openapi/v1.json` (OpenAPI, **Development only**).
- There is **no test project** in this repo, so there are no automated tests to run; `dotnet build`
  is the primary correctness gate.

### Non-obvious caveats
- Running under the `http` profile logs a benign warning `Failed to determine the https port for
  redirect.` (from `UseHttpsRedirection` with no HTTPS port). It is safe to ignore for local dev.
- `dotnet restore`/`build` emit `NU1903` (transitive `Microsoft.OpenApi` advisory). This is a
  warning only and does not block build/run.
- The scaffold shipped with `Program.cs` referencing a `WeatherForecast` type that was not defined
  anywhere, so a clean checkout did not compile. `WeatherForecast.cs` was added to restore the
  standard template type so the app builds and runs.
