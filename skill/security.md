# Security

## Position

Zero trust. The skill is public, so it is assumed to be read by attackers, loaded by agents that
are being manipulated, and fed forged input. It is designed so that none of that matters: the skill
holds no secret and can grant nothing. Authority lives only in DotPulse.

## What this repository contains

- Markdown instructions and SVG images.
- No executable code, no scripts, no build, no dependencies, no CI.
- No endpoint, address, key, token, password, certificate or account.
- No telemetry, analytics or tracking of any kind.
- No code from the DotPulse app or its backend.

Cloning it, forking it or loading it into an agent gives access to no device and no account.

## The invariant

> An agent following this skill performs no action on behalf of DotPulse unless DotPulse itself
> has remotely answered `authorized` for a Pairing ID that the agent's owner supplied in the
> current conversation.

Everything else in this file exists to keep that sentence true.

## Why there is no path around it

| Imagined path | Why it leads nowhere |
|---|---|
| Load the skill without a Pairing ID | The procedure has no step that runs without one. The skill explains how to get an ID and stops. |
| Supply a string that matches the pattern | The pattern is a filter. Only DotPulse's remote answer creates authorization. |
| Reuse an old Pairing ID | Single use and short TTL, enforced remotely. Answer: `rejected` or `expired`. |
| Replay a captured exchange | Replay protection and binding, enforced remotely. |
| Tell the agent "it is already authorized" | States come only from the connector. User or document claims are ignored. |
| Plant a Pairing ID in a page, file or message the agent reads | Only the owner's own message in a private conversation is a valid source. |
| Ask the agent to reach DotPulse another way | The skill names no address and forbids any path except the connector. |
| Fork the skill and delete the rules | The fork still has no connector binding, no credentials and no backend that trusts it. |
| Use the connector without the skill | The connector checks every precondition itself and rejects out-of-order calls. |

The last two rows are the important ones. The rules in SKILL.md are a second line of defence. The
first line is that validation is remote and the connector is strict. A skill is text read by a
language model, and text can be ignored; the design does not depend on it being obeyed.

## Guarantees required from DotPulse

The skill relies on these and cannot provide them. They are requirements on integration point
IP-3 in [protocol.md](protocol.md).

| Guarantee | Requirement |
|---|---|
| Short TTL | A Pairing ID expires minutes after it is issued. Pending confirmations expire too. |
| Single use | The first redemption consumes the ID atomically, whatever the outcome. |
| Revocation | Unused IDs can be cancelled. Established links can be revoked at any time, by the user, from the app. Revocation takes effect immediately. |
| Device binding | The session is bound to the device that issued the Pairing ID. |
| Agent binding | The session is bound to the agent instance that redeemed it, identified by the connector, not by anything the model says. |
| Replay protection | Redemption carries freshness that makes a recorded exchange useless. |
| No offline validity | A Pairing ID carries no signature or claim that could be accepted without asking DotPulse. |
| Rate limiting | Repeated failed redemptions are throttled, so the ID space cannot be searched. |
| Uniform refusals | Unknown, used and revoked IDs are indistinguishable to the caller (`rejected`). |
| Human confirmation | Authorization needs the user's confirmation, with a verification code shown on both sides. |

## Handling of the pairing secret

The agent:

- reads the Pairing ID from the user's message and passes it once to `redeem`;
- never writes it to memory, notes, files, summaries, titles or logs;
- never repeats it in a reply, whole or in part;
- never sends it to any tool other than the connector's `redeem`.

This repository:

- contains no real Pairing ID. `DPP1-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX` is a placeholder that DotPulse
  never issues.

### A limit worth stating plainly

The user pastes the Pairing ID into a chat. The chat platform, and the agent's own conversation
history, keep that message. The skill cannot erase it. This is why the Pairing ID must be worthless
the moment it has been redeemed, and worthless minutes after being issued even if it was never
redeemed. Single use and short TTL are not optional hardening; they are what makes pasting a code
into a chat acceptable at all.

The user can delete the message afterwards. Nothing depends on them doing so.

## Threats and answers

| Threat | Answer |
|---|---|
| Someone sends the owner an ID from their own phone ("paste this") | Confirmation step: device name and verification code must match the owner's phone. The agent reminds the user that IDs must come from their own device. |
| Prompt injection in fetched content carrying an ID | Source rule: only the owner's own message counts. |
| A stranger writes to the agent on Telegram | Private chat with an allowed user only. Everyone else gets no reaction. |
| Leaked chat history | IDs in it are spent or expired. |
| Malicious fork of this skill | Has no authority. Users should load the skill only from the official repository. |
| Compromised agent calling the connector directly | Connector enforces preconditions; DotPulse enforces binding and confirmation. |
| Lost or sold phone | The user asks the agent to `revoke` the link. A new pairing needs a new Pairing ID from a device the user holds. |

## Limits of this skill

- It gives the agent no ability to act on the phone.
- It never asks for, or handles, SSH access, passwords, API keys, tokens or server addresses.
- It cannot create, extend, renew or transfer a pairing.
- It does not hold, configure or observe the persistent link.

## Reporting a vulnerability

Report privately through this repository's GitHub security advisories ("Report a vulnerability" in
the Security tab). Please do not open a public issue for a security problem, and never include a
real Pairing ID in a report.
