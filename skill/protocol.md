# Protocol

This file defines the **abstract contract** between the skill and DotPulse. It is a specification,
not an implementation.

- It names no host, URL, port, path, header or credential. There are none in this repository.
- It contains no client, no server and no mock. Nothing here can be run.
- Every place where the real DotPulse system plugs in is marked as an **integration point**.

Until those integration points are implemented by DotPulse, the skill has no connector to call and
therefore stays inert for every input.

## Roles

```
┌──────────────┐        ┌──────────────────┐        ┌────────────────────┐        ┌──────────────┐
│  DotPulse    │        │  Agent (Hermes)  │        │ DotPulse connector │        │   DotPulse   │
│  app (phone) │        │  + this skill    │        │ (in agent runtime) │        │   (private)  │
└──────┬───────┘        └────────┬─────────┘        └─────────┬──────────┘        └──────┬───────┘
       │  issue Pairing ID       │                            │                          │
       │────────────────────────────────────────────────────────────────────────────────►│
       │  user pastes it         │                            │                          │
       │────────────────────────►│                            │                          │
       │                         │  redeem(pairing_id)        │                          │
       │                         │───────────────────────────►│  validate, consume, bind │
       │                         │                            │─────────────────────────►│
       │                         │  pending + device + code   │◄─────────────────────────│
       │                         │◄───────────────────────────│                          │
       │  same code on screen    │  user confirms             │                          │
       │◄────────────────────────────────────────────────────────────────────────────────│
       │                         │  approve(session_ref)      │                          │
       │                         │───────────────────────────►│─────────────────────────►│
       │                         │  authorized                │◄─────────────────────────│
       │                         │  start(session_ref)        │                          │
       │                         │───────────────────────────►│  persistent link         │
       │◄═══════════════════════════════════════════════════════════════════════════════►│
       │                         │  connected                 │                          │
       │                         │◄───────────────────────────│                          │
```

The skill sees only the two middle columns. It never talks to the right-hand column directly.

## Connector operations

The names below are abstract. They describe what the connector must offer, not how it is exposed.

| Operation | Input | Output | Allowed when |
|---|---|---|---|
| `redeem` | `pairing_id` | `state`, `session_ref`, `device_label`, `verification_code`, `expires_at` | A Pairing ID is present in the user's own message and passed the format filter |
| `approve` | `session_ref` | `state` | State is `pending` and the user said yes in this conversation |
| `deny` | `session_ref` | `state` (`rejected`) | State is `pending` |
| `start` | `session_ref` | `state` | State is `authorized` |
| `status` | `session_ref` | `state`, `reason` | A session reference exists from a pairing in this agent |
| `revoke` | `session_ref` | `state` (`rejected`) | A session reference exists from a pairing in this agent |

Rules that hold for every operation:

- `state` is always one of `pending`, `authorized`, `connected`, `expired`, `rejected`.
- The agent passes only the inputs listed. Agent identity, device identity, timestamps, nonces and
  signatures are produced by the connector and by DotPulse, never by the model.
- The connector checks the precondition itself. An out-of-order call is answered `rejected`; the
  skill's own ordering is a second line of defence, not the first.
- No operation returns a secret. `session_ref` is an opaque handle that is useless outside the
  connector that issued it.
- There is no operation that creates, extends or transfers a Pairing ID or a session, and none
  that works without a `pairing_id` or a `session_ref` that came from one.

## Integration points

| ID | What DotPulse must provide | Until it exists |
|---|---|---|
| **IP-1** | The app's "Copiar conexión" output, with a Pairing ID in the format of [pairing.md](pairing.md) | Nobody can obtain a Pairing ID. Skill inert. |
| **IP-2** | The connector in the agent runtime, exposing the six operations above. Its concrete tool names are bound here when it ships. | `redeem` cannot be called. Skill stops at procedure step 3. |
| **IP-3** | Remote validation: TTL, single use, revocation, binding, replay protection, as listed in [security.md](security.md) | No Pairing ID can become `authorized`. |
| **IP-4** | The confirmation shown on the phone: device name and verification code matching what the agent shows | Pairings cannot leave `pending`. |
| **IP-5** | The persistent link and its lifecycle (reconnect, revoke) behind `start`, `status`, `revoke` | Nothing reaches `connected`. |

Each row fails closed: a missing piece means less happens, never more.

### Binding the connector (IP-2)

When the DotPulse connector ships, this section lists its real tool names next to the abstract
operations. Until then the table is intentionally empty, and the agent must treat the connector as
unavailable.

| Abstract operation | Concrete tool in the agent runtime |
|---|---|
| `redeem` | *not bound* |
| `approve` | *not bound* |
| `deny` | *not bound* |
| `start` | *not bound* |
| `status` | *not bound* |
| `revoke` | *not bound* |

The agent must not guess these names, and must not use any other tool as a stand-in.

## What is deliberately out of scope

- How the connector reaches DotPulse (transport, addressing, authentication).
- How DotPulse stores Pairing IDs, bindings and sessions.
- How the persistent link is built and kept alive.
- Anything the DotPulse app does internally.

These belong to DotPulse and are private. They are abstracted away from the user and from this
skill on purpose.
