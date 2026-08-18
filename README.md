<h1 align="center">dojo-connect</h1>

<p align="center">
  Test an OCPI endpoint running on your own machine, without exposing it to the internet.
</p>

<p align="center">
  <a href="https://github.com/chargerdojo/dojo-connect/releases/latest"><img alt="Latest release" src="https://img.shields.io/github/v/release/chargerdojo/dojo-connect?label=release&color=2e7d32"></a>
  <img alt="Platforms" src="https://img.shields.io/badge/platforms-macOS%20%7C%20Linux%20%7C%20Windows-2e7d32">
  <a href="https://docs.chargerdojo.com/guide/connect-a-local-endpoint"><img alt="Docs" src="https://img.shields.io/badge/docs-connect%20a%20local%20endpoint-0288d1"></a>
  <a href="https://github.com/chargerdojo/dojo-connect/blob/main/LICENSE"><img alt="Licence" src="https://img.shields.io/badge/licence-proprietary-555"></a>
  <a href="https://chargerdojo.com"><img alt="ChargerDojo" src="https://img.shields.io/badge/chargerdojo.com-informational?color=555"></a>
</p>

---

`dojo-connect` opens an **outbound** WebSocket to ChargerDojo and forwards the requests we send it
to a local address you name. No inbound firewall rule, no public hostname, no TLS certificate, and
your service is never reachable from the internet at any point.

This repository holds the released binaries. The source is not here.

## ⬇️ Install

Pick your platform. Each block downloads the binary, checks it against the published checksum, and
puts it on your `PATH` so every later command is just `dojo-connect`.

**🍎 macOS, Apple silicon** (Intel: swap `darwin-arm64` for `darwin-amd64`):

```sh
curl -fsSLO https://github.com/chargerdojo/dojo-connect/releases/latest/download/dojo-connect-darwin-arm64
curl -fsSLO https://github.com/chargerdojo/dojo-connect/releases/latest/download/checksums.txt
shasum -a 256 --ignore-missing -c checksums.txt
sudo install -m 755 dojo-connect-darwin-arm64 /usr/local/bin/dojo-connect
```

**🐧 Linux, x86-64** (arm64: swap `linux-amd64` for `linux-arm64`):

```sh
curl -fsSLO https://github.com/chargerdojo/dojo-connect/releases/latest/download/dojo-connect-linux-amd64
curl -fsSLO https://github.com/chargerdojo/dojo-connect/releases/latest/download/checksums.txt
sha256sum --ignore-missing -c checksums.txt
sudo install -m 755 dojo-connect-linux-amd64 /usr/local/bin/dojo-connect
```

**🪟 Windows, x86-64**, in PowerShell:

```powershell
Invoke-WebRequest https://github.com/chargerdojo/dojo-connect/releases/latest/download/dojo-connect-windows-amd64.exe -OutFile dojo-connect.exe
```

Check it worked:

```sh
dojo-connect --version
```

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
> On Windows, SmartScreen warns instead: choose **More info**, then **Run anyway**.

## ▶️ Run it

Pair a connector in ChargerDojo under **Endpoints**, then run the binary next to your service with
the token it gives you:

```sh
dojo-connect --target http://localhost:8000 --token <pairing token>
```

On a machine other people can log into, and in CI, pass the token as an environment variable so it
stays out of your shell history:

```sh
export DOJO_CONNECTOR_TOKEN=cdpair_...
dojo-connect --target http://localhost:8000
```

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
| `3` | Stopped and not coming back: another connector took over this token, the server could not read a frame we sent, or reconnecting was given up on. |

## 🔒 What it does, and what it will not do

It is a pipe. It holds no test suites, no checks and no verdicts: everything that decides whether
your implementation is correct stays on our servers.

- It forwards **only** to the origins you name with `--target`. A request for any other host is
  refused.
- It needs no inbound access to your network.
- One token, one machine. Start a connector elsewhere with the same token and the first one stops.
  Pair a second connector for a second machine.

## 💬 Help

Pairing a connector needs a paid plan. The full guide, including what to do when a connector will
not connect, is at
**[docs.chargerdojo.com/guide/connect-a-local-endpoint](https://docs.chargerdojo.com/guide/connect-a-local-endpoint)**.

Trouble with a release, a checksum that does not match, or a binary that will not start:
[ahoy@synergyboat.com](mailto:ahoy@synergyboat.com).
