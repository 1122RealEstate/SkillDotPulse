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

> Nothing is connected unless DotPulse's service accepted a Pairing ID that the owner pasted, and
> the owner then approved the request on their own phone.

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
| Someone sends the owner of a Hermes a Pairing ID from *their* phone ("paste this into your Hermes") | **Not blocked by the service.** The request appears on the sender's phone, the sender approves it, and the sender's phone is connected to that Hermes. The only protections today are the owner not pasting a Pairing ID they did not generate, and the warning Hermes shows with the verification code. See "A known limit" below |
| The service is compromised | It cannot read the traffic it relays, and cannot make a wrong Hermes show the right verification code |
| A stranger writes to the bot | Only the owner's private chat is answered |
| Leaked chat history | The IDs in it are spent or expired |
| Lost phone | The owner revokes the connection from Hermes (`/dotpulse`, or asking the agent) |
| Lost Hermes machine | The owner revokes the connection from the app |

## A known limit

Pasting a Pairing ID into a Hermes is the act that lets a phone in. The confirmation on the phone
protects the phone's owner; it does not protect the owner of a Hermes who is talked into pasting
someone else's Pairing ID. In that case the verification code the Hermes shows will not appear on
the victim's own phone, which is the sign that something is wrong, but nothing stops the
connection: the other person approves it on their phone.

Treat a Pairing ID like a door key handed to whoever generated it. Only paste one you copied
yourself, from your own DotPulse, a moment ago. If you pasted one you should not have, disconnect
it at once: `/dotpulse` in Hermes lists the connections and `/dotpulse desconectar <id>` revokes one.

## Reporting a vulnerability

Report privately through this repository's GitHub security advisories ("Report a vulnerability" in
the Security tab). Do not open a public issue, and never include a real Pairing ID.
