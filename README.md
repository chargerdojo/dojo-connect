<h1 align="center">dojo-connect</h1>

<p align="center">
  Test an OCPI endpoint running on your own machine, without exposing it to the internet.
</p>

<p align="center">
  <a href="https://github.com/chargerdojo/dojo-connect/releases/latest"><img alt="Latest release" src="https://img.shields.io/github/v/release/chargerdojo/dojo-connect?label=release&color=2e7d32"></a>
  <img alt="Platforms" src="https://img.shields.io/badge/platforms-macOS%20%7C%20Linux%20%7C%20Windows-2e7d32">
  <a href="https://docs.chargerdojo.com/guide/connect-a-local-endpoint"><img alt="Docs" src="https://img.shields.io/badge/docs-connect%20a%20local%20endpoint-0288d1"></a>
  <a href="https://chargerdojo.com"><img alt="ChargerDojo" src="https://img.shields.io/badge/chargerdojo.com-informational?color=555"></a>
</p>

---

`dojo-connect` opens an **outbound** WebSocket to ChargerDojo and forwards the requests we send it
to a local address you name. No inbound firewall rule, no public hostname, no TLS certificate, and
your service is never reachable from the internet at any point.

This repository holds the released binaries. The source is not here.

## ⬇️ Download

Grab the build for your machine from the [latest release](https://github.com/chargerdojo/dojo-connect/releases/latest).

| Platform | File |
| --- | --- |
| 🍎 macOS, Apple silicon | `dojo-connect-darwin-arm64` |
| 🍎 macOS, Intel | `dojo-connect-darwin-amd64` |
| 🐧 Linux, x86-64 | `dojo-connect-linux-amd64` |
| 🐧 Linux, arm64 | `dojo-connect-linux-arm64` |
| 🪟 Windows, x86-64 | `dojo-connect-windows-amd64.exe` |

Every release carries `checksums.txt`. Check your download against it first:

```sh
sha256sum -c checksums.txt --ignore-missing
```

## 🔓 First run

The binaries are not code signed yet, so the first run needs one extra step.

| | |
| --- | --- |
| 🍎 **macOS** | Blocks an unsigned download. Right-click the file and choose Open, or run `xattr -d com.apple.quarantine ./dojo-connect` |
| 🪟 **Windows** | SmartScreen shows a warning. Choose "More info", then "Run anyway" |
| 🐧 **Linux** | `chmod +x ./dojo-connect` |

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
