<p align="center">
  <img src="assets/dotpulse-logo.svg" alt="DotPulse Skill" width="340">
</p>

<p align="center">
  <img src="assets/dotpulse-dots-animated.svg" alt="DotPulse Skill: conecta Hermes con DotPulse usando solo un Pairing ID temporal" width="100%">
</p>

<p align="center">
  <b>La Skill oficial para enlazar Hermes con DotPulse.</b><br>
  Un Pairing ID temporal, generado en tu teléfono. Nada más.
</p>

---

## Qué es

DotPulse es la app para trabajar desde el teléfono con los agentes de tu Hermes. Cada agente es un
**Dot**.

Esta Skill enseña a Hermes a hacer una sola cosa: reconocer el **Pairing ID** que copias desde
DotPulse, pedirle a DotPulse que lo valide y, solo si DotPulse dice que sí, iniciar el enlace.

## Cómo funciona

<p align="center">
  <img src="assets/dotpulse-flow.svg" alt="DotPulse App, Pairing ID, Hermes o Telegram, DotPulse conectado" width="100%">
</p>

1. Abres DotPulse en el teléfono.
2. Pulsas **Copiar conexión**. DotPulse genera un Pairing ID temporal.
3. Pegas lo copiado en tu conversación con Hermes, o en tu chat privado de Telegram con él.
4. Hermes carga esta Skill, encuentra el Pairing ID y se lo pasa a DotPulse para validarlo.
5. Compruebas que el nombre del dispositivo y el código que muestra Hermes son los de tu teléfono.
6. DotPulse queda conectado. El Pairing ID ya no sirve para nada más.

No hay direcciones de servidor, contraseñas, claves ni terminal.

## La Skill sola no da acceso a nada

> [!IMPORTANT]
> Este repositorio es público a propósito. Instalarlo, leerlo o copiarlo **no da acceso a ningún
> dispositivo ni a ninguna cuenta**. Sin un Pairing ID válido, la Skill no ejecuta ninguna acción.

| | En este repositorio |
|---|---|
| Instrucciones para el agente (Markdown) | Sí |
| Imágenes de marca (SVG) | Sí |
| Código ejecutable, scripts o dependencias | No |
| Código de la app DotPulse o de su backend | No |
| Direcciones, claves, tokens o credenciales | No |
| Telemetría | No |

La autorización no vive aquí. Vive en DotPulse, que valida cada Pairing ID en remoto:

- **Caduca en minutos.** Un Pairing ID viejo no vale.
- **Un solo uso.** El primer intento lo consume, salga bien o mal.
- **Revocable.** Puedes cancelarlo antes de usarlo y retirar el enlace cuando quieras.
- **Ligado a tu dispositivo y a tu agente.** No se puede mover a otro.
- **Sin repetición.** Un intercambio capturado no se puede volver a usar.
- **No se guarda.** El agente no lo escribe en memoria, archivos ni registros, y no lo repite.

Que un código *parezca* válido no significa nada: la Skill no confía en la forma, solo en la
respuesta de DotPulse.

## Estados de un emparejamiento

<p align="center">
  <img src="assets/dotpulse-dots.svg" alt="Los cinco estados: pending, authorized, connected, expired y rejected" width="100%">
</p>

| Estado | Qué significa |
|---|---|
| `pending` | DotPulse aceptó el Pairing ID y espera tu confirmación. |
| `authorized` | DotPulse confirmó el emparejamiento para este agente y ese dispositivo. |
| `connected` | El enlace está activo y lo mantiene DotPulse. |
| `expired` | Se agotó el tiempo. Hace falta un Pairing ID nuevo. |
| `rejected` | No se aceptó: desconocido, ya usado, revocado o rechazado por ti. |

Cualquier respuesta que no sea una de estas cinco se trata como `rejected`.

## Qué hay en el repositorio

```
SKILL.md                      la Skill: cuándo se activa, qué valida, qué tiene prohibido
skill/
  dotpulse.md                 qué es DotPulse y para qué sirve la Skill
  pairing.md                  el Pairing ID y su ciclo de vida
  protocol.md                 el contrato abstracto con DotPulse y los puntos de integración
  security.md                 modelo de amenazas y garantías exigidas
examples/
  pairing-example.md          conversaciones de ejemplo, caso por caso
assets/                       logo, Dots y animación (SVG, sin JavaScript)
```

Los documentos de la Skill están en inglés porque los lee el agente. El agente responde al usuario
en su idioma.

## Estado de la integración

La Skill define el comportamiento del agente y el contrato con DotPulse. Las piezas privadas de
DotPulse se conectan en los puntos de integración marcados en
[skill/protocol.md](skill/protocol.md) (IP-1 a IP-5). Mientras alguna no esté disponible, la Skill
se queda inerte: una pieza que falta significa que ocurre menos, nunca más.

## Seguridad

El modelo completo está en [skill/security.md](skill/security.md). Para avisar de una
vulnerabilidad, usa los avisos de seguridad privados de este repositorio (pestaña *Security* →
*Report a vulnerability*) y no incluyas nunca un Pairing ID real.

Carga la Skill solo desde este repositorio oficial:
`https://github.com/1122RealEstate/SkillDotPulse`
