# DotPulse

DotPulse is a phone app for working with the agents of a Hermes that runs on the user's own
machine or server. Each agent profile appears in the app as a **Dot**: an animated character whose
state reflects what the agent is doing.

## What this skill is for

One job: let the owner of a Hermes agent link it with their DotPulse app by pasting a short text
into a conversation with that agent, in Hermes or in Telegram.

```
DotPulse app  ──►  Pairing ID  ──►  Hermes / Telegram  ──►  DotPulse connected
 (generates)       (temporary,       (this skill reads        (link kept by
                    single use)       and redeems it)          DotPulse)
```

## What this skill is not

- It is not the DotPulse app, and it contains none of its code.
- It is not the DotPulse backend, and it does not know where it is.
- It is not a remote-control tool. It does not let an agent act on a phone.
- It is not a credential. Installing, reading or forking this repository gives access to nothing.

## The three parts

| Part | Where it lives | What it does |
|---|---|---|
| DotPulse app | The user's phone | Generates the Pairing ID and shows the confirmation. |
| DotPulse connector | The agent runtime, provided by DotPulse | Redeems the Pairing ID with DotPulse and keeps the link. |
| This skill | This repository | Tells the agent when to use the connector, and when not to. |

The skill is the only public part. It is deliberately the part with no power.

## Vocabulary

- **Pairing ID**: the temporary, single-use code the app generates. See [pairing.md](pairing.md).
- **Connector**: the DotPulse-provided capability the agent calls. See [protocol.md](protocol.md).
- **Binding**: the pair (this agent, that device) that DotPulse fixes when it authorizes.
- **Link**: the persistent connection DotPulse keeps after a successful pairing.
- **Session reference**: an opaque handle the connector returns for `status` and `revoke`. It is
  not a secret and it cannot be used outside the connector.
