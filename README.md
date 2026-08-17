# dojo-connect

The ChargerDojo local connector. It lets us test an OCPI endpoint running on your machine or
inside your network, without exposing it to the internet.

This repository holds the released binaries and nothing else. The source is not here.

## Download

Pick the build for your machine from the [latest release](https://github.com/chargerdojo/dojo-connect/releases/latest):

| Platform             | File                             |
| -------------------- | -------------------------------- |
| macOS, Apple silicon | `dojo-connect-darwin-arm64`      |
| macOS, Intel         | `dojo-connect-darwin-amd64`      |
| Linux, x86-64        | `dojo-connect-linux-amd64`       |
| Linux, arm64         | `dojo-connect-linux-arm64`       |
| Windows, x86-64      | `dojo-connect-windows-amd64.exe` |

Every release also carries `checksums.txt`. Check your download against it before you run it:

```sh
sha256sum -c checksums.txt --ignore-missing
```

## First run

The binaries are not code signed yet, so the first run needs one extra step.

- **macOS** blocks an unsigned download. Right-click the file and choose Open, or run
  `xattr -d com.apple.quarantine ./dojo-connect`.
- **Windows** SmartScreen shows a warning. Choose "More info", then "Run anyway".
- **Linux** needs `chmod +x ./dojo-connect`.

## Run it

Pair a connector in ChargerDojo under **Endpoints**, then run the binary next to your service with
the token it gave you:

```sh
dojo-connect --target http://localhost:8000 --token <pairing token>
```

On a machine other people can log into, and in CI, pass the token as an environment variable so it
stays out of your shell history:

```sh
export DOJO_CONNECTOR_TOKEN=cdpair_...
dojo-connect --target http://localhost:8000
```

## Flags

| Flag        | Meaning                                                                                     |
| ----------- | ------------------------------------------------------------------------------------------- |
| `--target`  | Where to forward, for example `http://localhost:8000`. Required. Repeat it for a service that answers on more than one port. |
| `--token`   | The pairing token. Or set `DOJO_CONNECTOR_TOKEN`.                                            |
| `--version` | Print the version and exit.                                                                  |

## Exit codes

A CI job reads these.

| Code | Meaning                                                                                                                                         |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| 0    | Asked to stop, stopped cleanly.                                                                                                                 |
| 1    | The flags are wrong.                                                                                                                            |
| 2    | The server refused the pairing token, or the token was revoked while connected.                                                                 |
| 3    | Stopped and not coming back: another connector took over this token, the server could not read a frame we sent, or reconnecting was given up on. |

## What it does and does not do

It opens an outbound WebSocket to us and forwards the requests we send it to the addresses you
named with `--target`. It needs no inbound firewall rule, no public hostname and no TLS
certificate, and your service is never reachable from the internet.

It is a pipe. It holds no test suites, no checks and no verdicts: everything that decides whether
your implementation is correct stays on our servers. A request for any host you did not name with
`--target` is refused.

One token, one machine. Start a connector elsewhere with the same token and the first one stops.
Pair a second connector for a second machine.

## Getting help

Pairing a connector needs a paid plan. The full guide, including what to do when a connector will
not connect, is at
[docs.chargerdojo.com/guide/connect-a-local-endpoint](https://docs.chargerdojo.com/guide/connect-a-local-endpoint).

Problems with a release, a checksum that does not match, or a binary that will not start:
[ahoy@synergyboat.com](mailto:ahoy@synergyboat.com).
