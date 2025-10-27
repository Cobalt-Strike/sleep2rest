# Cobalt Strike REST API Aggressor Script Library

This library provides Aggressor Script functions for interacting with the [Cobalt Strike REST API](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/userguide/content/api/index.html) from the client. It supports authentication, ```GET```/```POST```/```PUT```/```DELETE``` requests, and automatic parsing of JSON responses into native Sleep hashes and arrays.

<p align="center">
<img width="60%" alt="sleep2rest" src="https://github.com/user-attachments/assets/e0b0b5dc-543a-4b8b-a540-90b630752d2d" />
</p>

> [!NOTE]
> This tool is still in early development stage and subject to breaking changes.

## Features
- Authenticate and obtain JWT tokens
- Make ```GET```, ```POST```, ```PUT``` and ```DELETE``` requests
- Parse JSON responses to Sleep data structures
- Utility functions for easy API integration

## Requirements
- The Cobalt Strike API Server should be running.
- Cobalt Strike should be installed and configured.
- Cobalt Strike should be properly licensed
- Add the following flags to your Java command line when launching Cobalt Strike:
    ```bash
    --add-exports java.base/sun.net.www.protocol.https=ALL-UNNAMED
    --add-exports java.base/sun.net.www.http=ALL-UNNAMED
    --add-opens   java.base/sun.net.www.protocol.https=ALL-UNNAMED
    --add-opens   java.base/sun.net.www.http=ALL-UNNAMED
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

## Configuration

### Global Variables

- `$url_base`: Base URL for the REST API
- `$token`: JWT token for authentication
- `$username`: Username for API authentication
- `$password`: Password for API authentication

## Usage

### Functions Overview
- **apiGET(endpoint)**: GET request
- **apiPOST(endpoint, body)**: POST request
- **apiPUT(endpoint, body)**: PUT request
- **apiDELETE(endpoint)**: DELETE request

### Setup

1. Clone the repository:

   ```bash
      git clone https://github.com/Cobalt-Strike/sleep2rest.git
   ```
2. **Edit the Global Variables in [cs_rest_api_lib.cna](https://github.com/Cobalt-Strike/sleep2rest/blob/be8729dbafd271b6baedfdfb86aa6cdda1a4b613/cs_rest_api_lib.cna#L23)**:
   ```perl
   # Configure these variables for your environment
    $url_base = "https://<APISERVER>:50443";
    $username = "<username>";
    $password = "<password>";
    # End of configuration
   ```
3. Include the ```cs_rest_api_lib.cna``` script into your script following [this example](https://github.com/Cobalt-Strike/sleep2rest/blob/61da42afa8ad66951e388590dad0714f0ab7373f/examples/serverside_artifact_execution.cna#L2).
4. Load your ```.cna``` script into the Cobaltstrike client through **Cobalt Strike > Script Manager > Load**
5. Enjoy!

### Example

#### ```.cna``` samples

| Name | Description |
|------|-------------|
|[example_usage.cna](https://github.com/Cobalt-Strike/sleep2rest/blob/main/examples/example_usage.cna) | Simple GET and POST requests to the Cobalt Strike REST API. |
|[serverside_payload_generation.cna](https://github.com/Cobalt-Strike/sleep2rest/blob/main/examples/serverside_payload_generation.cna) | Script that provides an alternative menu to generate payloads server-side. [README](https://github.com/Cobalt-Strike/sleep2rest/blob/main/examples/Readme_serverside_payload_generation.md) |
|[serverside_artifact_execution.cna](https://github.com/Cobalt-Strike/sleep2rest/blob/main/examples/serverside_artifact_execution.cna)| Script that provides an example to run server-side stored .NET assemblies. It can be easily extended to run BOFs.|

#### Code Snippet
```perl
include(script_resource("cs_rest_api_lib.cna"));

# Authenticate and get beacons
$response = apiGET("/api/v1/beacons");
if ($response["status"] == 200) {
    $beacons = $response["content"];
    println("Beacons: " . $beacons);
} else {
    println("API error: " . $response["status"]);
}

# POST example
$beacon_id = "123456789";
$body = '{"command": "ps"}';
$response = apiPOST("/api/v1/beacons/".$beacon_id."/consoleCommand", $body);
if ($response["status"] == 200) {
    println("Response: " . $response);
    $statusUrl = $response["content"]["statusUrl"];
    
    # Get task result
    $task_response = apiGET($statusUrl);
    $status = $task_response["content"]["taskStatus"];
    println("Final GET Response: " . $task_response);
} else {
    println("API error: " . $response["status"]);
}
```

## Support

For issues and questions:
- Review [Cobalt Strike documentation](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/welcome_main.htm) for API requirements
- Consult [Sleep documentation](https://sleep.dashnine.org/) for Sleep issues.

---

> [!WARNING]
> This tool provides direct access to Cobalt Strike capabilities, which include powerful adversary simulation capabilities. Use responsibly and only in environments where you have explicit permission to perform security testing.
