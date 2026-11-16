<h1 align="center">dojo-connect</h1>

<p align="center">
  Test your private OCPI endpoint before your first partner does.
</p>

<p align="center">
  <a href="https://github.com/chargerdojo/dojo-connect/releases/latest"><img alt="Latest release" src="https://img.shields.io/github/v/release/chargerdojo/dojo-connect?label=release&color=2e7d32"></a>
  <img alt="Platforms" src="https://img.shields.io/badge/platforms-macOS%20%7C%20Linux%20%7C%20Windows-2e7d32">
  <a href="https://docs.chargerdojo.com/guide/connect-a-local-endpoint"><img alt="Docs" src="https://img.shields.io/badge/docs-connect%20a%20local%20endpoint-0288d1"></a>
  <a href="https://github.com/chargerdojo/dojo-connect/blob/main/LICENSE"><img alt="Licence" src="https://img.shields.io/badge/licence-proprietary-555"></a>
  <a href="https://chargerdojo.com"><img alt="ChargerDojo" src="https://img.shields.io/badge/chargerdojo.com-informational?color=555"></a>
</p>

---

[ChargerDojo](https://chargerdojo.com) acts as your roaming partner against your own OCPI endpoint.
Use `dojo-connect` to run conformance suites against a service on your laptop or private network
without exposing it to the internet. It requires Integrate or above.

The connector opens an **outbound** WebSocket to ChargerDojo and forwards our requests to the
local origins you name. Charging journeys and load runs need a public endpoint or tunnel;
they do not run through this connector.

This repository holds the released binaries. The source is not here.

## Install

Pick your platform. The macOS and Linux blocks download the binary, check its published checksum,
and install it on your `PATH`. On Windows, download and check it in a folder you control.

**macOS, Apple silicon** (Intel: swap `darwin-arm64` for `darwin-amd64`):

```sh
curl -fsSLO https://github.com/chargerdojo/dojo-connect/releases/latest/download/dojo-connect-darwin-arm64
curl -fsSLO https://github.com/chargerdojo/dojo-connect/releases/latest/download/checksums.txt
shasum -a 256 --ignore-missing -c checksums.txt
sudo install -m 755 dojo-connect-darwin-arm64 /usr/local/bin/dojo-connect
```

**Linux, x86-64** (arm64: swap `linux-amd64` for `linux-arm64`):

```sh
curl -fsSLO https://github.com/chargerdojo/dojo-connect/releases/latest/download/dojo-connect-linux-amd64
curl -fsSLO https://github.com/chargerdojo/dojo-connect/releases/latest/download/checksums.txt
sha256sum --ignore-missing -c checksums.txt
sudo install -m 755 dojo-connect-linux-amd64 /usr/local/bin/dojo-connect
```

**Windows, x86-64**, in PowerShell:

```powershell
Invoke-WebRequest https://github.com/chargerdojo/dojo-connect/releases/latest/download/dojo-connect-windows-amd64.exe -OutFile dojo-connect.exe
Invoke-WebRequest https://github.com/chargerdojo/dojo-connect/releases/latest/download/checksums.txt -OutFile checksums.txt
Get-FileHash .\dojo-connect.exe -Algorithm SHA256
```

Compare the printed hash with the `dojo-connect-windows-amd64.exe` entry in `checksums.txt`.

On macOS or Linux, check it worked:

```sh
dojo-connect --version
```

On Windows, run `.\dojo-connect.exe --version` from the download folder.

Every build is on the [latest release](https://github.com/chargerdojo/dojo-connect/releases/latest)
page if you would rather download by hand.

> [!IMPORTANT]
> **On macOS, download with `curl` rather than a browser.** We do not sign the binaries yet, so a
> browser marks the file as quarantined and macOS kills the first run without printing anything:
> no dialog, no error, just an exit code. If that has already happened, clear the mark and run it
> again:
>
> ```sh
> xattr -d com.apple.quarantine <file>
> ```
>
> On Windows, SmartScreen warns instead: choose **More info**, then **Run anyway**. Verify the
> checksum before you run either.

## Run it

As an organisation admin, open **Testing**, then **Endpoints** in ChargerDojo. Choose **Add an
endpoint**, then **Behind a local connector**, then **Pair a connector**. Copy the pairing token
when it appears; it is shown once. Run the binary next to your service:

```sh
dojo-connect --target http://localhost:8000 --token <pairing token>
```

In Bash or zsh, read the token without echoing it or saving it in shell history:

```sh
printf 'Pairing token: '
read -rs DOJO_CONNECTOR_TOKEN
printf '\n'
export DOJO_CONNECTOR_TOKEN
dojo-connect --target http://localhost:8000
```

In CI, inject `DOJO_CONNECTOR_TOKEN` from your secret store.

Confirm the connector shows as connected in **Endpoints**, select it in the endpoint form, then
[save and register your endpoint](https://docs.chargerdojo.com/guide/connect-a-partner#add-the-endpoint-in-the-browser).
Keep the connector running until the suite finishes. Press Ctrl+C to stop it and revoke it in
**Local connectors** when it is no longer needed.

### Flags

| Flag | Meaning |
| --- | --- |
| `--target` | Where to forward, for example `http://localhost:8000`. Required. Repeat it for a service that answers on more than one port. |
| `--token` | The pairing token. Or set `DOJO_CONNECTOR_TOKEN`. |
| `--version` | Print the version and exit. |

### Exit codes

A CI job reads these.

| Code | Meaning |
| --- | --- |
| `0` | Asked to stop, stopped cleanly. |
| `1` | The flags are wrong. |
| `2` | The server refused the pairing token, or the token was revoked while connected. |
| `3` | Stopped and not coming back: another connector took over this token, the server could not read a frame we sent or did not receive the initial hello, or reconnecting was given up on. |

## What it does, and what it will not do

It is a pipe. It holds no test suites, no checks and no verdicts: everything that decides whether
your implementation is correct stays on our servers.

- It forwards **only** to the origins you name with `--target`. A request for any other host or port is
  refused.
- It needs no inbound access to your network.
- One token, one machine. Start a connector elsewhere with the same token and the first one stops.
  Pair a second connector for a second machine.

## Help

The full setup and troubleshooting guide is at
**[docs.chargerdojo.com/guide/connect-a-local-endpoint](https://docs.chargerdojo.com/guide/connect-a-local-endpoint)**.

Trouble with a release, a checksum that does not match, or a binary that will not start:
[support@chargerdojo.com](mailto:support@chargerdojo.com).
