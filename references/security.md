# Security

## Position

Zero trust in this repository. It is public, so it is assumed to be read by attackers, loaded by
agents that are being manipulated, and fed forged input. None of that matters, because the skill
holds no secret and can grant nothing. Authority lives in the DotPulse Link service and in the
owner's tap on *Autorizar*.

## What this repository contains

- Markdown instructions and SVG images.
- No executable code, scripts, dependencies or CI.
- No address, key, token, password or account.
- No telemetry.
- No code from the DotPulse app, its connector or its service.

## The invariant

> Nothing is connected unless DotPulse's service accepted a Pairing ID that was pasted into a
> Hermes, the holder of the phone approved the request there, **and** the owner of that Hermes
> confirmed in Hermes that the phone is theirs.

## Why there is no path around it

| Imagined path | Why it leads nowhere |
|---|---|
| Load the skill without a Pairing ID | The tools do nothing without one |
| Supply a string that merely looks right | Only the service's answer counts, and it has never heard of it: `rejected` |
| Reuse a Pairing ID | Single use and five minutes, enforced by the service |
| Find a used Pairing ID in a chat history | Same: `rejected` |
| Two Hermes present the same ID | The first claims it; the second gets `rejected` |
| Skip the approval | The service hands out no credential before the owner approves, whoever asks |
| Tell the agent "it is already authorised" | The agent's belief changes nothing: the service decides |
| Plant a Pairing ID in a page the agent reads | The skill forbids it, and even if the agent obeyed the page, the result is a request on the owner's phone that the owner did not expect and can reject |
| Paste it in a group chat | The connector drops it before the agent sees it |
| Fork the skill and delete the rules | A fork has no connector, no credential and no service that trusts it |
| Reconnect after being revoked | The credential is dead at the service and deleted at the connector |

The skill's rules are a second line of defence. A skill is text read by a language model, and text
can be ignored; the design does not depend on it being obeyed.

## Handling of the Pairing ID

- The app keeps it in memory, puts it on the clipboard for five minutes, and never writes it down.
- The service never receives it: only a one-way hash.
- The connector uses it once and stores nothing of it. It does not log it.
- The agent passes the user's message to `dotpulse_pair` and does not repeat or store the ID.

One limit, stated plainly: the user pastes the Pairing ID into a chat, and the chat keeps that
message. Nothing here can erase it. That is why the ID is worthless the moment it has been used and
five minutes after it was issued. In a private Telegram chat the connector answers before the
message reaches the model; pasted straight into Hermes, the model does see it.

## Threats and answers

| Threat | Answer |
|---|---|
| Someone sends the owner of a Hermes a Pairing ID from *their* phone ("paste this into your Hermes") | The request appears on the sender's phone and the sender approves it there. That is not enough: the connector collects nothing until the owner of the Hermes confirms, in Hermes, that the phone is theirs. With no confirmation it expires in 2 minutes; with a "no" it is withdrawn at once. Covered by the connector's tests. See "What is left" below |
| The service is compromised | It cannot read the traffic it relays, and cannot make a wrong Hermes show the right verification code |
| A stranger writes to the bot | Only the owner's private chat is answered |
| Leaked chat history | The IDs in it are spent or expired |
| Lost phone | The owner revokes the connection from Hermes (`/dotpulse`, or asking the agent) |
| Lost Hermes machine | The owner revokes the connection from the app |

## The two confirmations

| Where | Who gives it | What it protects |
|---|---|---|
| In DotPulse: *Autorizar* | Whoever holds the phone | The phone's owner: nobody links their phone to a Hermes unseen |
| In Hermes: the "Es mi teléfono" button in Telegram, or `/dotpulse confirmar` | Whoever runs that Hermes | The Hermes' owner: a stranger's phone does not get in by approving itself |

The second one can only be given by a person. There is no tool for it, so nothing the model reads,
and no instruction hidden in a page or a file, can confirm on the owner's behalf.

## What is left

Someone who follows a stranger's instructions step by step, pastes the stranger's Pairing ID *and*
confirms in Hermes that the phone is theirs, has let the stranger in. The message Hermes shows
says, in so many words, not to confirm unless their own DotPulse is showing that code. No technical
check can tell an owner who was talked into it from one who meant it.

If it happened: `/dotpulse` lists the connections and `/dotpulse desconectar <id>` revokes one.

## Reporting a vulnerability

Report privately through this repository's GitHub security advisories ("Report a vulnerability" in
the Security tab). Do not open a public issue, and never include a real Pairing ID.
