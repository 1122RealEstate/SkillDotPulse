# DotPulse

DotPulse is a phone app for working with the agents of a Hermes that runs on the user's own
machine or server. Each agent profile appears in the app as a **Dot**: an animated character whose
state reflects what the agent is doing.

## What this skill is for

One job: let the owner of a Hermes connect it to their DotPulse app by pasting a short text into a
conversation with that Hermes.

```
DotPulse app ──► Pairing ID ──► pasted into Hermes ──► confirmed in the app ──► connected
 (generates)     (5 minutes,     (the connector          (the owner presses      (kept up by
                  one use)        presents it)             Autorizar)              DotPulse)
```

## The parts

| Part | Where it lives | What it does |
|---|---|---|
| DotPulse app | The user's phone | Generates the Pairing ID, shows the request, asks for confirmation, lists and revokes connections |
| DotPulse Link service | Run by DotPulse | Decides: whether a Pairing ID is good, who is connected, what a connection may carry. Relays between the two ends without being able to read what it relays |
| DotPulse connector | A plugin in the user's Hermes | Presents the Pairing ID, stores the connection's credential, keeps the connection up |
| This skill | This repository | Tells the agent when to use the connector's tools, and when not to |

The skill is the only public part, and deliberately the part with no power. It holds no address,
no key and no code.

## What this skill is not

- Not the app, the service or the connector, and it contains none of their code.
- Not a remote control: it gives the agent no ability to act on a phone.
- Not a credential: installing, reading or forking this repository gives access to nothing.

## Vocabulary

- **Pairing ID**: the temporary, single-use code the app generates. See [pairing.md](pairing.md).
- **Request**: what appears on the phone when a Hermes presents a Pairing ID.
- **Verification code**: six digits shown by Hermes and by the app. They must match.
- **Connection** (or link): what exists after the owner authorises. It has its own credential,
  unrelated to the Pairing ID.
- **Capability**: one thing a connection may carry. A connection carries only what it was granted.
