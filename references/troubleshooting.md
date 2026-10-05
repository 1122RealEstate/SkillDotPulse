# Troubleshooting

What the user can see, and what to do about it. Messages from Hermes are the connector's own
wording, as it returns them; those from the app are the ones it is programmed to show. Quote them
as they are: they are in the user's language already.

## Before anything is connected

| Situation | What appears | What to do |
|---|---|---|
| The connector is not installed | The agent has no `dotpulse_pair` tool and `/dotpulse` does not exist | Install DotPulseConnector once (the command is in SKILL.md, "Official connector"). The pasted Pairing ID was not spent; copy a new connection afterwards |
| The connector is too old or too new | Hermes: «Este conector de DotPulse (versión …) ya no es compatible con el servicio de DotPulse. Actualízalo y genera una conexión nueva. El Pairing ID no se ha usado.» | Update the connector, restart Hermes, copy a new connection |
| DotPulse Link cannot be reached from Hermes | Hermes: «No pude contactar con DotPulse. Comprueba la conexión a Internet de este Hermes y genera un Pairing ID nuevo.» | Check that the Hermes machine has Internet access. The Pairing ID was not spent |
| The connector has no service to talk to | Hermes: «Este Hermes todavía no tiene configurado el servicio DotPulse Link, así que no puede conectarse con DotPulse.» | Only a connector older than 0.4.0, or one pointed elsewhere on purpose. Update it |
| The phone has no Internet | App: «No se pudo contactar con DotPulse. Comprueba tu conexión a Internet.» | Check the phone's network |

## While pairing

| Situation | What appears | What to do |
|---|---|---|
| The Pairing ID expired | Hermes: «Ese Pairing ID ha caducado. En DotPulse pulsa «Copiar conexión» para generar otro.» | Copy a new connection and paste it within 5 minutes |
| Already used, or never existed | Hermes: «Ese Pairing ID no es válido o ya se usó. En DotPulse pulsa «Copiar conexión» para generar otro.» | Copy a new connection. Hermes cannot tell used from unknown, on purpose |
| Cut off while copying | Hermes: «Ese Pairing ID está incompleto o dañado. En DotPulse pulsa «Copiar conexión» otra vez y pégalo entero.» | Copy again and paste the whole text |
| More than one Pairing ID in the message | Hermes asks for a single fresh one | Copy a new connection and paste only that |
| Rejected on the phone | Hermes: «La solicitud se rechazó en DotPulse. No se conectó nada.» | Start again if it was a mistake |
| Rejected in Hermes | Hermes: «Solicitud rechazada desde Hermes. No se conectó nada.» | The same |
| One confirmation was missing | Hermes: «Nadie confirmó la solicitud en DotPulse a tiempo. No se conectó nada. Para intentarlo de nuevo, pulsa «Copiar conexión» otra vez.» | Repeat, and give both confirmations within 2 minutes |
| `/dotpulse confirmar` with nothing pending | Hermes: «No hay ninguna solicitud de DotPulse esperando confirmación aquí.» | The request expired or was already answered |
| The code on the phone is different, or the phone shows no request | Nothing matches | Do **not** confirm. Type `/dotpulse rechazar`: another phone is asking |
| Too many attempts | Hermes: «Demasiados intentos seguidos. Espera unos minutos y genera un Pairing ID nuevo en DotPulse.» | Wait a few minutes |
| Pasted in a group or a channel | No answer | Paste it in the private chat with the bot |

## Once connected

| Situation | What appears | What to do |
|---|---|---|
| Hermes is off or has no network | App, Conexiones: «Sin conexión ahora»; «Tu Hermes no está en línea.» | Nothing: it reconnects by itself when that machine is back |
| The phone changed network | A short pause | Nothing: the app reconnects by itself |
| The connection was revoked | App: «Esta conexión fue revocada.» In Hermes, `/dotpulse` no longer lists it | Copy and paste a new connection to come back |
| A connection the owner does not recognise | `/dotpulse` lists it | `/dotpulse desconectar <id>`, or remove it in the app |

## What never fixes anything

Asking the user for a server address, a port, SSH access, a password, a key, a token or an
identifier of their phone. None of these is part of DotPulse, and a request for one is not from
DotPulse.
