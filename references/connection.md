# Connection

DotPulse is a phone app for working with the agents of a Hermes that runs on the user's own
machine or server. Each agent profile appears in the app as a **Dot**: an animated character whose
state reflects what the agent is doing. This file is how a phone and a Hermes get connected, and
what a connection is.

## The parts

| Part | Where it lives | What it does |
|---|---|---|
| DotPulse app | The user's phone | Generates the Pairing ID, shows the request, asks for confirmation, lists and revokes connections |
| DotPulse Link | Run by DotPulse, at `link.dotpulse.app` | Decides: whether a Pairing ID is good, who is connected, what a connection may carry. Relays between the two ends without being able to read what it relays |
| DotPulseConnector | A plugin in the user's Hermes | Presents the Pairing ID, stores the connection's credential, keeps the connection up |
| This skill | This repository | Tells the agent when to use the connector's tools, and when not to |

The skill is deliberately the part with no power. It holds no key and no code, and it never
supplies an address: the app and the connector each carry DotPulse Link's address from the
factory, so no pasted text can send a Hermes anywhere else.

```
DotPulse app  ⇄ HTTPS/WSS ⇄  DotPulse Link  ⇄ HTTPS/WSS ⇄  DotPulseConnector  ⇄  Hermes
```

Both ends dial **out** to DotPulse Link. Nothing listens on the phone or on the Hermes machine, and
no port is opened on either.

## First connection, and every one after

The **first** time a Hermes is used with DotPulse, its owner installs DotPulseConnector once, with
Hermes' own plugin command, and restarts Hermes. Hermes does not install a plugin because a
message asks for it, and the agent has no tool to do so. A Pairing ID pasted before that is not
spent: it simply expires, and the user copies a new one afterwards.

**Every connection after that**, from the same phone or another, is the five steps below and
nothing else. The connector keeps each connection up by itself and brings it back after a network
loss or a restart, with the same credential. A connection ends only when its owner revokes it.

## What the user does

1. Opens DotPulse and taps **Copiar conexión**.
2. Pastes what was copied, unchanged, into a private conversation with their Hermes.
3. Reads the verification code Hermes shows.
4. Goes back to DotPulse, checks that the request there shows the same code, and taps **Autorizar**.
5. Confirms in Hermes that the phone is theirs: the **Es mi teléfono** button in Telegram, or
   `/dotpulse confirmar` anywhere else.

No server address, no password, no key, no terminal.

## What the app copies

```
DotPulse · conexión
Skill: https://github.com/1122RealEstate/SkillDotPulse
Pairing ID: DPP1-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
Hermes: conecta DotPulse con este Pairing ID.
```

Nothing else: no address, no key, no token. The agent passes the whole message to `dotpulse_pair`
unchanged; only the Pairing ID in it matters.

## Pairing ID format

| Property | Value |
|---|---|
| Prefix | `DPP1` |
| Body | 5 groups of 5 characters, hyphen-separated |
| Alphabet | Crockford Base32: `0123456789ABCDEFGHJKMNPQRSTVWXYZ` (no `I`, `L`, `O`, `U`) |
| Entropy | 125 bits, from the phone's cryptographic random generator |
| Pattern | `^DPP1-[0-9A-HJKMNP-TV-Z]{5}(-[0-9A-HJKMNP-TV-Z]{5}){4}$` |
| Meaning | Opaque. It encodes no address, user, device or permission |

`DPP1-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX` is used in this repository as a placeholder. It matches the
pattern on purpose: matching the pattern means nothing. DotPulse never issues it.

## What DotPulse enforces

Enforced by the Link service, not by this skill, and covered by its tests:

- **Five minutes.** After that a Pairing ID is `expired`.
- **One use.** The first Hermes to present it claims it for good. Any later attempt, by anyone,
  gets `rejected`.
- **Cancellable.** Copying a new connection cancels the previous unused one.
- **Confirmation on the phone.** Nothing is authorised until *Autorizar* is tapped in the app. The
  request shows which Hermes is asking, where the text was pasted (Hermes or Telegram), when, and
  the code.
- **Confirmation in Hermes.** Enforced by the connector: it does not collect the connection's
  credential until the owner of that Hermes confirms there too.
- **Two minutes for each.** A request missing either answer expires.
- **No guessing.** Repeated wrong attempts from one address are refused for a while.
- **Never stored.** The service never receives the Pairing ID, only a one-way hash of it. The app
  keeps it in memory until the owner answers. The connector uses it once and keeps nothing.

## The verification code

Six digits, shown by Hermes when it presents the Pairing ID and by the app on the request. Both
ends compute it from the Pairing ID and from each other's public keys, so:

- it only matches when that phone and that Hermes are looking at each other;
- the service in the middle cannot produce a matching code for a different Hermes.

If the app cannot verify that the Hermes asking holds the Pairing ID, it shows the request as not
verified and does not offer *Autorizar*.

## Lifecycle

```
pending ──presented──► pending (request on the phone) ──Autorizar──► authorized ──confirmed in Hermes──► connected
   │                         │                                           │              │
   ├─ 5 min ─► expired       ├─ Rechazar ─► rejected                     └─ 2 min ─►    └─ revoke ─► revoked
   └─ cancelled ─► revoked   └─ 2 min ─► expired                            expired
```

| State | Entered when | Leaves to |
|---|---|---|
| `pending` | The app issued the Pairing ID | `authorized`, `rejected`, `expired`, `revoked` |
| `authorized` | The owner tapped *Autorizar* | `connected`, `expired`, `revoked` |
| `connected` | The owner confirmed in Hermes, the connector collected its credential and its connection came up | `revoked` |
| `expired`, `rejected`, `revoked` | See above | nowhere |

`connected` is reached only from `authorized`. The three closed states are never left: a new
attempt always starts with a new Pairing ID from the app.

## Vocabulary

- **Pairing ID**: the temporary, single-use code the app generates. See "Pairing ID format" above.
- **Request**: what appears on the phone when a Hermes presents a Pairing ID.
- **Verification code**: six digits shown by Hermes and by the app. They must match.
- **Connection** (or link): what exists after the owner authorises. It has its own credential,
  unrelated to the Pairing ID.
- **Capability**: one thing a connection may carry. A connection carries only what it was granted.
