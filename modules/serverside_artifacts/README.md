# Server-Side Payloads

This module uses the generic REST helpers in `lib/cna-rest-core/cna_rest_core.cna` and the Cobalt Strike REST API helpers in `lib/cs_rest_api_lib.cna`.

## Setup

1. Copy `config/local.example.cna` to `config/local.cna` and configure `$url_base`, `$username`, and `$password`.
2. Navigate to **Cobalt Strike** -> **Script Manager** and load `modules/serverside_artifacts/payload_generation.cna`.
3. The module calls `csCheckConnection()` during load and prints whether the Cobalt Strike REST API is reachable.
4. A new menu option **Server-Side Payloads** will now be available on the top menu bar.

<p align="center">
<img src="https://github.com/user-attachments/assets/767f7646-80b3-4308-8ef0-307acd97652a" alt="Stager Payload Generator"/><br/><span style="font-family:monospace;font-size:0.8rem;">Server-Side Payloads Menu Item</span>
</p>

## Usage

### Stager Payload Generator

Cobalt Strike's Stager Payload Generator outputs source code and artifacts to stage a Cobalt Strike listener onto a host.

Navigate to **Server-Side Payloads** -> **Stager Payload Generator**

<p align="center">
  <img src="https://github.com/user-attachments/assets/1976e7e9-9524-43b4-861f-62411dd2dbfb" alt="Stager Payload Generator" /><br/><span style="font-family:monospace;font-size:0.8rem;">Stager Payload Generator</span>
</p>

<details>

<summary><strong>Parameters</strong></summary>

- **Filename**: Name of the payload.
- **Listener**: Press the ... button to select a Cobalt Strike listener you would like to output a payload for.
- **Output**: Use the drop-down to select one of the following output types (most options give you shellcode formatted as a byte array for that language):
    - **C**: Shellcode formatted as a byte array.
    - **C#**: Shellcode formatted as a byte array.
    - **COM Scriptlet**: A .sct file to run a listener.
    - **Java**: Shellcode formatted as a byte array.
    - **Perl**: Shellcode formatted as a byte array.
    - **PowerShell**: PowerShell script to run shellcode
    - **PowerShell Command**: PowerShell one-liner to run a Beacon stager.
    - **Python**: Shellcode formatted as a byte array.
    - **Raw**: blob of position independent shellcode.
    - **Ruby**: Shellcode formatted as a byte array.
    - **Veil**: Custom shellcode suitable for use with the Veil Evasion Framework.
    - **VBA**: Shellcode formatted as a byte array.
- **x64**: Check the box to generate an x64 stager for the selected listener.

</details>

### Stageless Payload Generator

Cobalt Strike's Stageless Payload Generator outputs source code and artifacts, without a stager, to a Cobalt Strike listener onto a host.

Navigate to **Server-Side Payloads** -> **Stageless Payload Generator**

<p align="center">
  <img src="https://github.com/user-attachments/assets/3721d437-8626-48b4-93f2-f14c9629914b" alt="Stageless Payload Generator"/><br/><span style="font-family:monospace;font-size:0.8rem;">Stageless Payload Generator</span>
</p>

<details>

<summary><strong>Parameters</strong></summary>

- **Filename**: Name of the payload.
- **Listener**: Press the ... button to select a Cobalt Strike listener you would like to output a payload for.
- **Guardrails**: By default, the Listener guardrails will be used. Use this textbox to overwrite the settings for the beacon (The format should be ```Key1=Value1``` and the possible keys are: ```IP```, ```User```, ```Server``` and ```Domain```). Wildcards are supported.
- **Output**: Use the drop-down to select one of the following output types (most options give you shellcode formatted as a byte array for that language):
    - **C**: Shellcode formatted as a byte array.
    - **C#**: Shellcode formatted as a byte array.
    - **Java**: Shellcode formatted as a byte array.
    - **Perl**: Shellcode formatted as a byte array.
    - **Python**: Shellcode formatted as a byte array.
    - **Raw**: blob of position independent shellcode.
    - **Ruby**: Shellcode formatted as a byte array.
    - **VBA**: Shellcode formatted as a byte array.
- **UDRL Aggressor Script**: Aggressor Script to hook your desired server-side UDRL
- **Sleepmask Aggressor Script**: Aggressor Script to hook your desired server-side Sleepmask
- **Options**: User defined options map to pass to the hooks when generating the payload. The format should be ```Key1=Value1``` or ```"Key with spaces"="Value with spaces"```.
- **Exit Function**: This function determines the method/behavior that Beacon uses when the exit command is executed.
    - **Process**: Terminates the whole process.
    - **Thread**: Terminates only the current thread.
- **System Call**: Select one of the following system call methods to use at execution time when generating a stageless beacon payload from the Cobalt Strike UI or a supported aggressor function:
    - **None**: Use the standard Windows API function.
    - **Direct**: Use the Nt* version of the function.
    - **Indirect**: Jump to the appropriate instruction within the Nt* version of the function.
- **HTTP Library**: Select the Microsoft library (WinINet or WinHTTP) for the generated payload.
- **DNS Comm Mode**: This option allows you to use DNS Over HTTPS (DOH) for egressing from the target using a DNS Beacon. The default value is determined by the Malleable C2 `comm_mode` option from the listener definition. You can define more DOH configuration options in Malleable C2.
- **x64**: Check the box to generate an x64 payload for the selected listener.

</details>

### Download Payload

This dialog allows you to download a server-side generated payload on the client machine.

<p align="center">
  <img src="https://github.com/user-attachments/assets/d3ac16d0-2457-4923-849f-507de9f7d044" alt="Stageless Payload Generator"/><br/><span style="font-family:monospace;font-size:0.8rem;">Download Payloads</span>
</p>

<details>

<summary><strong>Parameters</strong></summary>

- **Filename**: Name of the payload to download.

</details>

## Server-Side Assembly Execution

`artifact_execution.cna` registers the Beacon command:

```text
execute-serverside-assembly
```

The command executes a .NET assembly artifact from the server-side artifact store through the Cobalt Strike REST API.

Usage:

```text
execute-serverside-assembly [assembly_name] [arguments]
execute-serverside-assembly "PATCHES: library,function,offset,patch" [assembly_name] [arguments]
```

Assemblies must exist in the artifact store under `assemblies/`. The command validates the available assembly list and prints the accepted names when the requested assembly is missing.
