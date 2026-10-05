# What the tools do underneath

This file describes behaviour, not an API. The routes and message formats between the connector,
the app and DotPulse Link are private to DotPulse and are not in this repository. The service's
address (`link.dotpulse.app`) is public knowledge and still never comes from here: the app and the
connector each carry it, and neither takes it from a message.

## Roles

```
DotPulse app            Link service              DotPulse connector (in Hermes)        Agent + this skill
────────────            ────────────              ──────────────────────────────        ──────────────────
issues a Pairing ID ──► remembers its hash
                                                  ◄── dotpulse_pair(text) ───────────── user pasted it
                        claims it, once      ◄──  presents the hash + its public key
request + code     ◄──                            returns device name + code ─────────► shown to the user
owner taps Autorizar ─► authorized
                        hands over the      ──►   stores the connection's credential
                        connection credential     (private file) and connects out
                        connected            ◄══  outbound connection, kept up
app's traffic      ═══► relays, sealed       ══►  the local Hermes API
```

The agent only ever sees the right-hand column.

## Operations

| What happens | Who does it | How the agent sees it |
|---|---|---|
| Present the Pairing ID (redeem) | Connector, when `dotpulse_pair` is called | `state` and, on `pending`, `device`, `verification_code`, `request` |
| Approve or reject on the phone | Whoever holds the phone, in the app | Later, through `dotpulse_status(request)` |
| Confirm or reject in Hermes | The owner of the Hermes: a button in Telegram, or `/dotpulse confirmar` / `/dotpulse rechazar`. There is no tool for it | Later, through `dotpulse_status(request)` |
| Collect the credential and connect (start) | Connector, by itself, once both answers are yes | Nothing to do |
| Status and capabilities | Connector asks the service | `dotpulse_status` |
| Revoke | The owner in the app, or `dotpulse_disconnect` | The connection becomes `revoked` |

There is no operation that creates, extends or transfers a Pairing ID or a connection, and none
that reaches `connected` without the owner's approval. The service enforces the order; an
out-of-order request is refused whoever sends it.

## Before a Pairing ID is spent

`dotpulse_pair` does not present anything until two things are true, and it checks both itself:

1. DotPulse Link answers.
2. The protocol version this connector speaks is one the service accepts.

If either fails it returns `unreachable`, `not-configured` or `incompatible`, and the Pairing ID
is untouched: the service never heard of the attempt. `dotpulse_status`, with no arguments,
reports the same two facts (`connector.reachable`, `connector.compatible`) without touching any
Pairing ID, which is how the skill checks first.

## The connection

- The connector dials **out** to the Link service and keeps that connection up: heartbeat,
  reconnection with backoff, and the same credential after a restart. Nothing listens on the Hermes
  machine and no port is opened. Telegram is not involved in keeping it up.
- The credential that keeps it up is created when the owner approves. It is not the Pairing ID and
  is not derived from it. It can be rotated and revoked.
- What travels between the app and Hermes is encrypted end to end between those two. The service
  relays it and cannot read it.
- Revoking cuts both ends at once. The connector then deletes its credential and does not retry.

## Capabilities

A connection carries only the capabilities it was granted, and those are the ones the owner saw on
the authorisation screen. `dotpulse_status` lists them per connection.

| Capability | What it allows |
|---|---|
| `hermes.api` | The DotPulse app can use this Hermes: conversations, Dots, tasks and voice |

This is the only capability that exists today. The service checks it every time the app opens a
stream, and the connector checks it again.

## Where the connector comes from

The tools exist only when DotPulseConnector is installed in Hermes as a plugin. It is not in this
repository: it lives at https://github.com/1122RealEstate/DotPulseConnector. Hermes installs a
plugin with its own command, from a git repository pinned to a commit; it does not install one
because a message asks for it, and the agent has no tool to do so.

```bash
hermes plugins install 1122RealEstate/DotPulseConnector --ref 1ca4876b81d40c43b7f8782d07c6ea2dd02238ba --enable --force
hermes gateway restart
```

Version 0.4.2, for Hermes 0.21 or newer. It speaks protocol 1, which is what DotPulse Link accepts
today; a connector outside the accepted range is told `incompatible` before any Pairing ID is spent.
