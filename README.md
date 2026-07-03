# Cobalt Strike REST API Aggressor Script Library

Aggressor Script REST client library and module collection for working with arbitrary REST APIs from the Cobalt Strike client.

The `lib/cna-rest-core/cna_rest_core.cna` submodule is API-agnostic: it provides HTTP/HTTPS transport, request headers and bodies, JSON conversion between Sleep and JSON, response helpers, and file-download handling. The Cobalt Strike REST wrapper in `lib/cs_rest_api_lib.cna` builds on that generic layer for JWT authentication and authenticated Cobalt Strike API calls. Feature modules build on these pieces where needed for server-side payload workflows.

<p align="center">
<img width="60%" alt="sleep2rest" src="https://github.com/user-attachments/assets/e0b0b5dc-543a-4b8b-a540-90b630752d2d" />
</p>

> [!NOTE]
> This project is still in early development and can introduce breaking changes.

## Current Functionality

### Core Libraries

- `lib/cna-rest-core/cna_rest_core.cna`
  - Generic HTTP/HTTPS request helper for arbitrary REST APIs.
  - `GET`, `POST`, `PUT`, and `DELETE` transport support through caller-supplied request properties.
  - Optional TLS certificate verification bypass for local/test infrastructure.
  - Response maps containing status, headers, content, and file-download detection.
  - JSON parsing and serialization helpers: `json_to_sleep()`, `sleep_to_json()`, `Json2Sleep()`, and `Sleep2Json()`.
  - Response status and error classification helpers.
- `lib/cs_rest_api_lib.cna`
  - Cobalt Strike REST API wrapper built on the generic REST transport.
  - Cobalt Strike REST API authentication.
  - JWT token caching.
  - Authenticated wrappers: `apiGET()`, `apiPOST()`, `apiPUT()`, and `apiDELETE()`.
  - `csCheckConnection()` health check.
- `lib/config_loader.cna`
  - `load_local_config()` helper for loading untracked operator configuration from `config/local.cna`.

### Feature Modules

| Module | Load path | Functionality |
|--------|-----------|---------------|
| [Server-side artifacts](modules/serverside_artifacts/README.md) | `modules/serverside_artifacts/payload_generation.cna` | Top-level `Server-Side Payloads` menu for stager generation, stageless generation, and generated payload download through the Cobalt Strike REST API. |
| [Server-side assembly execution](modules/serverside_artifacts/README.md#server-side-assembly-execution) | `modules/serverside_artifacts/artifact_execution.cna` | `execute-serverside-assembly` Beacon command for executing .NET assemblies from the server-side artifact store, including optional patch rules. |

## Requirements

- Cobalt Strike client and team server installed and licensed.
- Cobalt Strike API server running only for the Cobalt Strike REST wrapper, examples, and modules that call the Cobalt Strike REST API.
- Java 17+ for the Cobalt Strike client.
- The Cobalt Strike client must be launched with these Java module flags when using the REST helpers:

```text
--add-exports=java.base/sun.net.www.protocol.https=ALL-UNNAMED
--add-exports=java.base/sun.net.www.http=ALL-UNNAMED
--add-opens=java.base/sun.net.www.protocol.https=ALL-UNNAMED
--add-opens=java.base/sun.net.www.http=ALL-UNNAMED
```

An example of the ```launch-cobaltstrike-client.bat``` file to launch the Windows client would be:

```bash
@echo off
setlocal

set "JAVA_EXE=C:\Program Files\Microsoft\jdk-21.0.8.9-hotspot\bin\javaw.exe"
if not exist "%JAVA_EXE%" (
	for %%J in (javaw.exe java.exe) do (
		where %%~J >nul 2>&1 && set "JAVA_EXE=%%~J" && goto :found_java
	)
	echo ERROR: javaw/java not found. Install JRE/JDK or adjust JAVA_EXE in this script.
	pause
	exit /b 1
)
:found_java

set "JAR=C:\Program Files\cobaltstrike\client\cobaltstrike-client.jar"

rem --- build JVM_OPTS piece by piece (no carets) ---
set "JVM_OPTS=-XX:ParallelGCThreads=4"
set "JVM_OPTS=%JVM_OPTS% -XX:+AggressiveHeap"
set "JVM_OPTS=%JVM_OPTS% -XX:+UseParallelGC"
set "JVM_OPTS=%JVM_OPTS% --add-exports=java.base/sun.net.www.protocol.https=ALL-UNNAMED"
set "JVM_OPTS=%JVM_OPTS% --add-exports=java.base/sun.net.www.http=ALL-UNNAMED"
set "JVM_OPTS=%JVM_OPTS% --add-opens=java.base/sun.net.www.protocol.https=ALL-UNNAMED"
set "JVM_OPTS=%JVM_OPTS% --add-opens=java.base/sun.net.www.http=ALL-UNNAMED"

start "" /min "%JAVA_EXE%" %JVM_OPTS% -jar "%JAR%"

endlocal
exit /b 0
```

Optional module-specific requirement:

- Server-side artifact modules: configured Cobalt Strike REST API credentials and relevant artifacts uploaded to the server-side artifact store.

## Configuration

Copy [config/local.example.cna](config/local.example.cna) to `config/local.cna` and edit local values. `config/local.cna` is ignored by git and should not be committed.

```perl
$url_base = "https://<APISERVER>:50443";
$username = "";
$password = "";
$token_duration_ms = 86400000;
```

## Quick Start

1. Clone the repository:

```bash
git clone https://github.com/Cobalt-Strike/sleep2rest.git
```

2. Initialize submodules:

```bash
git submodule update --init --recursive
```

3. Create local configuration:

```text
copy config\local.example.cna config\local.cna
```

4. Set the values needed by the modules you plan to load.

5. Load scripts through **Cobalt Strike > Script Manager > Load**.

For generic REST usage, include only the transport library:

```perl
include(script_resource("lib/cna-rest-core/cna_rest_core.cna"));

$response = json_to_sleep(__makeAPIRequest(
    "https://api.example.local/v1/status",
    %(
        method => "GET",
        headers => %(Accept => "application/json"),
        body => $null,
        skipVerify => 0
    )
));

println("Status: " . $response["status"]);
println("Content: " . $response["content"]);
```

For Cobalt Strike REST API usage, include the libraries in this order:

```perl
include(script_resource("lib/cna-rest-core/cna_rest_core.cna"));
include(script_resource("lib/cs_rest_api_lib.cna"));
include(script_resource("lib/config_loader.cna"));

load_local_config();

if (csCheckConnection()) {
    $response = apiGET("/api/v1/beacons");
    println("Beacons response: " . $response);
}
```

Feature modules load their shared dependencies themselves.

## Commands And UI

| Entry point | Registered behavior |
|-------------|---------------------|
| `execute-serverside-assembly [assembly_name] [arguments]` | Executes a server-side .NET assembly artifact from `artifacts/assemblies/`. |
| `execute-serverside-assembly "PATCHES: library,function,offset,patch" [assembly_name] [arguments]` | Executes a server-side .NET assembly with patch rules. |
| `Server-Side Payloads` menu | Opens dialogs for stager generation, stageless generation, and payload download. |

## Repository Layout

| Path | Purpose |
|------|---------|
| `lib/` | Shared Cobalt Strike API/config helpers and the `cna-rest-core` submodule. |
| `modules/` | Loadable feature modules. |
| `examples/` | Basic REST API usage examples. |
| `config/` | Example local configuration. |

## Examples

| Example | Description |
|---------|-------------|
| [examples/basic_rest_usage.cna](examples/basic_rest_usage.cna) | Loads local configuration, calls `GET /api/v1/beacons`, demonstrates task polling, executes a server-side assembly, and requests a file download. |

## Notes

- Include `lib/cna-rest-core/cna_rest_core.cna` before `lib/cs_rest_api_lib.cna` when writing custom Cobalt Strike REST scripts.

## Support

- Review the [Cobalt Strike documentation](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/welcome_main.htm) for API server and client requirements.
- Review the [Sleep documentation](https://sleep.dashnine.org/) for Aggressor Script language behavior.
- Check the module-specific README files under `modules/` for detailed setup and usage.

---

> [!WARNING]
> This project provides direct access to Cobalt Strike capabilities. Use it only in environments where you have explicit authorization to perform security testing.
