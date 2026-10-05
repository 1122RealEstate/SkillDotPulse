<p align="center">
  <img src="assets/dotpulse-logo.svg" alt="DotPulse Skill" width="340">
</p>

<p align="center">
  <img src="assets/dotpulse-dots-animated.svg" alt="DotPulse Skill: conecta Hermes con DotPulse con un Pairing ID temporal" width="100%">
</p>

<p align="center">
  <b>La Skill oficial para conectar Hermes con DotPulse.</b><br>
  Copias una conexión en el teléfono, la pegas en Hermes y confirmas en el teléfono.
</p>

---

> [!NOTE]
> **DotPulse Link y DotPulseConnector ya están en línea.** Lo que falta para usarlo es la app:
> DotPulse todavía no está en el App Store. Mira [Estado actual](#estado-actual).

## ¿Qué es DotPulse?

DotPulse es la app de iPhone para controlar y gestionar los agentes de tu Hermes. Cada agente es un
**Dot**: un personaje cuyo estado refleja lo que el agente está haciendo.

Tus agentes siguen corriendo en tu Hermes, en tu equipo o servidor. DotPulse no pone modelos ni
ejecuta agentes: es el cliente.

```
DotPulse (iPhone)  ⇄ HTTPS/WSS ⇄  DotPulse Link  ⇄ HTTPS/WSS ⇄  DotPulseConnector  ⇄  Hermes
```

| Pieza | Qué hace | Dónde está |
|---|---|---|
| **Skill** (este repositorio) | Entiende la intención y el orden de los pasos. Solo instrucciones | Aquí |
| **DotPulseConnector** | Mantiene la conexión real con tu Hermes | Un plugin dentro de tu Hermes |
| **DotPulse Link** | El puente seguro público entre tu teléfono y tu Hermes | `link.dotpulse.app`, de DotPulse |
| **DotPulse** | El cliente | Tu iPhone |

No configuras direcciones IP, puertos, SSH, certificados ni la dirección de Link: la app y el
Connector la traen de fábrica. No hay cuentas ni inicio de sesión.

<p align="center">
  <img src="assets/dotpulse-flow.svg" alt="DotPulse, Pairing ID, Hermes o Telegram, autorizas en DotPulse, Connected" width="100%">
</p>

## ¿Qué hace la Skill?

Le dice al agente de Hermes qué hacer cuando pegas una conexión de DotPulse, y qué no hacer nunca.
**La Skill no conecta por sí sola**: no contiene código, claves ni direcciones.

Cuando recibe un texto con un Pairing ID (`DPP1-…`), sigue este orden:

1. Reconoce el Pairing ID. Es una credencial temporal de emparejamiento, no una instrucción ni
   código: nada de lo que venga con él se ejecuta.
2. Comprueba que DotPulseConnector está instalado.
3. Comprueba que su versión es compatible con el servicio.
4. Comprueba que DotPulse Link responde.
5. Solo entonces presenta el Pairing ID.

Si falta el Connector, si es incompatible o si Link no responde, **el Pairing ID no se gasta**: la
Skill te dice qué pasa y nada más. Y nunca dice que conectó si el servicio no lo confirma.

Estas comprobaciones no dependen de que el agente obedezca: el Connector las hace por su cuenta
antes de presentar nada.

La Skill jamás pide IMEI ni identificadores de Apple, claves privadas, contraseñas, acceso SSH,
tokens (de Cloudflare ni de nada) ni direcciones de servidor. Tampoco guarda ni reenvía Pairing IDs.

## ¿Qué es DotPulseConnector?

Un plugin de Hermes. Es lo que de verdad conecta: presenta el Pairing ID a DotPulse Link, guarda
la credencial de la conexión en un archivo privado de tu Hermes y mantiene el enlace abierto,
reconectando solo si se cae la red o se reinicia el equipo.

Solo hace conexiones **salientes** hacia `link.dotpulse.app`. No abre ningún puerto en tu equipo.

Le da a Hermes tres herramientas (`dotpulse_pair`, `dotpulse_status`, `dotpulse_disconnect`) y una
orden para ti, `/dotpulse`.

### Instalarlo

Hace falta una sola vez por cada Hermes, y se puede hacer sin salir de la conversación:

1. Pega la conexión de DotPulse en tu Hermes. Si falta el Connector, o el que hay es antiguo,
   Hermes te lo dice **sin gastar el Pairing ID** y te pregunta si quieres que lo instale.
2. Responde que sí. Hermes ejecuta esta orden, y solo esta (puede pedirte que la apruebes):

   ```bash
   hermes plugins install 1122RealEstate/DotPulseConnector --ref 1ca4876b81d40c43b7f8782d07c6ea2dd02238ba --enable --force
   ```

3. Reinicia Hermes para que lo cargue: envía `/restart` en el chat (Telegram y las demás
   plataformas de mensajería), o cierra y vuelve a abrir Hermes en el escritorio o la terminal.
4. Copia una conexión **nueva** en DotPulse y pégala.

También puedes ejecutar tú esa orden en el equipo de Hermes, o abrir la instalación en Hermes
Desktop con este enlace, que muestra un diálogo de confirmación:

```
hermes://plugin/install?repo=1122RealEstate/DotPulseConnector&enable=1&force=1
```

| | |
|---|---|
| Repositorio oficial | [1122RealEstate/DotPulseConnector](https://github.com/1122RealEstate/DotPulseConnector) |
| Versión | 0.4.2 |
| Commit | `1ca4876b81d40c43b7f8782d07c6ea2dd02238ba` |
| Hermes mínimo | 0.21 (probado con 0.21.2) |

`--ref` fija el commit exacto: es la comprobación de integridad. `--force` sustituye una copia
anterior del Connector sin tocar las conexiones que ya tengas. Hermes analiza el plugin antes de
instalarlo. Después puedes comparar la copia instalada con su manifiesto:

```bash
python3 ~/.hermes/plugins/dotpulse/verify.py      # ok: 9 archivos coinciden con el manifiesto
```

Cualquier otro repositorio u otro commit ofrecido como «conector de DotPulse» no es oficial.

**Por qué no es automático del todo.** Hermes no instala un plugin por recibir un mensaje: no hay
ninguna herramienta del agente para ello, los plugins nuevos quedan desactivados hasta que su dueño
los activa, y solo se cargan al reiniciar. Es una decisión de seguridad de Hermes y DotPulse no la
rodea. Por eso la primera vez hay un «sí» y un reinicio; después, nunca más.

## ¿Qué es Link?

DotPulse Link es el servicio que pone en contacto tu teléfono con tu Hermes. Hace tres cosas:

- **Decide** si un Pairing ID vale, quién está conectado y qué puede llevar cada conexión.
- **Reenvía** el tráfico entre los dos extremos, cifrado de extremo a extremo. No puede leerlo.
- **Revoca**: cuando eliminas una conexión, corta los dos extremos en el momento.

No ejecuta IA ni agentes y no guarda conversaciones, prompts, respuestas ni archivos.

## Primera conexión

La primera vez que usas DotPulse con un Hermes hay un paso más, [instalar el Connector](#instalarlo):
Hermes te lo ofrece al pegar la conexión, tú dices que sí y lo reinicias. Esa conexión pegada no
se gasta; copia una nueva después.

Después:

1. **Abre DotPulse** y ve a Conexiones → Conectar Hermes.
2. **Pulsa «Copiar conexión».** Se copian cuatro líneas, y nada más:

   ```
   DotPulse · conexión
   Skill: https://github.com/1122RealEstate/SkillDotPulse
   Pairing ID: DPP1-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
   Hermes: conecta DotPulse con este Pairing ID.
   ```

   El Pairing ID es distinto cada vez, vale 5 minutos y sirve una sola vez.
3. **Pégalo, sin cambiar nada,** en tu Hermes o en tu chat privado de Telegram con él.
4. **Hermes responde** con el nombre de tu teléfono y un código de verificación de seis cifras.
5. **En DotPulse** aparece la solicitud con el mismo código. Compruébalo y pulsa **Autorizar**.
6. **En Hermes** confirma que ese teléfono es el tuyo: `/dotpulse confirmar`, o el botón
   **✅ Es mi teléfono** en Telegram.
7. **Conectado.** La app lo muestra solo cuando el servicio lo confirma, y `/dotpulse` lo lista:

   ```
   • iPhone de Ana — 64e9ecc2f11a — conectado, en línea — capacidades: hermes.api
   ```

Hacen falta las dos confirmaciones, en cualquier orden y en 2 minutos. Sin las dos no se conecta
nada.

## Conexiones futuras

Con el Connector ya instalado, conectar otro teléfono, o el mismo después de eliminarlo, es solo
los pasos 1 a 7: copiar, pegar, autorizar. No se instala nada más.

Una conexión hecha se mantiene sola. Si tu Hermes se reinicia o tu teléfono cambia de red, vuelve
sin que hagas nada. Solo termina cuando la eliminas:

- **En DotPulse:** Conexiones → la conexión → Eliminar conexión.
- **En Hermes:** `/dotpulse` para verlas y `/dotpulse desconectar <identificador>`.

En los dos casos el servicio la marca como revocada y corta los dos extremos en el momento. La
credencial deja de servir para siempre (cualquier intento recibe `403 revoked`), y el Connector
borra de tu equipo todo lo que pertenecía a esa conexión: su credencial, su identificador, la
clave y el nombre del teléfono, sus permisos y el registro del emparejamiento. No toca nada más:
ni tu Hermes, ni tus agentes, ni tus conversaciones, ni las demás conexiones. Para volver hacen
falta una conexión nueva y una autorización nueva.

## Seguridad

- **El Pairing ID caduca a los 5 minutos y funciona una sola vez.** Uno encontrado después en un
  historial de chat no sirve para nada.
- **Dos confirmaciones tuyas:** Autorizar en tu teléfono y confirmar en tu Hermes. La segunda solo
  la puede dar una persona: no existe herramienta para que el agente confirme por ti.
- **No pegues un Pairing ID que no hayas copiado tú.** Si alguien te envía uno y lo pegas, el
  teléfono que pide entrar es el suyo. Lo notarás porque tu DotPulse no muestra ninguna solicitud
  con ese código: no confirmes, escribe `/dotpulse rechazar`.
- **El código de verificación tiene que coincidir** en Hermes y en la app. Sale del Pairing ID y de
  las claves de los dos extremos, así que el servicio no puede fabricar uno que coincida.
- **Cada conexión es independiente:** su credencial, sus claves de sesión y su revocación.
  Comprometer un Hermes no da nada sobre otro.
- **La seguridad no vive en la Skill.** Caducidad, un solo uso, confirmaciones, permisos y
  revocación los impone el código de Link y del Connector, obedezca o no el agente.
- **Este repositorio no da acceso a nada.** Solo son instrucciones e imágenes.

El modelo completo, con lo que queda sin cubrir: [references/security.md](references/security.md).

## Privacidad

- **No hace falta cuenta** para la conexión básica. Cada instalación de DotPulse tiene su propia
  identidad criptográfica, y la clave privada no sale del iPhone.
- **Lo que hablas con tus agentes no lo puede leer DotPulse.** Va cifrado de extremo a extremo
  entre tu teléfono y tu Hermes; Link solo lo reenvía.
- **Link guarda únicamente los metadatos mínimos** para mantener y proteger las conexiones: un
  identificador aleatorio de tu instalación, el nombre que le pusiste, claves públicas y, de cada
  conexión, los nombres del teléfono y del Hermes, sus permisos, su estado, cuándo se creó y su
  última actividad.
- **No guarda** conversaciones, prompts, respuestas, archivos, tokens ni tu dirección IP. Si algún
  día existiera una función que guardase contenido, sería explícita y con tu consentimiento.
- **No se usa ningún identificador de tu teléfono** (ni IMEI ni identificadores de Apple). El
  Pairing ID ni siquiera llega al servicio: solo un resumen irreversible.
- **Puedes borrarlo todo:** Ajustes → Conexiones → «Borrar mis datos del servicio de conexión»
  revoca tus conexiones y elimina tu instalación del servicio en el momento.

## Troubleshooting

| Situación | Qué hacer |
|---|---|
| Hermes dice que falta el Connector, o que el que hay es antiguo | Dile que sí lo instale y envía `/restart`. Tu Pairing ID no se gastó; copia una conexión nueva después. [Detalles](#instalarlo) |
| «…ya no es compatible con el servicio de DotPulse» | Lo mismo: [actualiza el Connector](#instalarlo), reinicia y copia una conexión nueva |
| «No pude contactar con DotPulse» | Revisa la conexión a Internet del equipo de Hermes y copia una conexión nueva |
| «Ese Pairing ID ha caducado» / «no es válido o ya se usó» | Copia una conexión nueva y pégala antes de 5 minutos |
| «…incompleto o dañado» | Vuelve a copiar y pega el texto entero |
| El código no coincide, o tu teléfono no muestra ninguna solicitud | No confirmes: `/dotpulse rechazar` |
| «Tu Hermes no está en línea» | Nada: se reconecta solo cuando ese equipo vuelve |
| «Esta conexión fue revocada» | Copia y pega una conexión nueva |

Todos los mensajes y casos: [references/troubleshooting.md](references/troubleshooting.md).

## Estado actual

A 5 de octubre de 2026. «Probado» quiere decir ejecutado de verdad, no solo escrito.

| Qué | Estado |
|---|---|
| DotPulse Link en `https://link.dotpulse.app` | **En línea.** HTTPS con certificado válido, WSS, y `/health` y `/ready` respondiendo |
| Conexión nueva, mismo código en los dos lados, autorización, conectado y agentes reales listados, contra el Link de producción | Probado: el Connector 0.4.2 instalado desde GitHub sobre un plugin antiguo, cargado y manejado por Hermes 0.21.2, con su `hermes serve` real. El teléfono fue un sustituto de pruebas que habla el mismo protocolo que la app |
| Reconexión: la app se cierra y vuelve; el proceso del Connector muere y vuelve solo | Probado en producción, con la misma credencial y sin emparejar de nuevo |
| Eliminar la conexión desde el teléfono: Hermes queda desconectado y sin rastro de ella en disco | Probado en producción |
| Volver a conectar tras eliminar | Probado en producción: solo con un Pairing ID nuevo y una autorización nueva |
| La conexión sobrevive a un reinicio del servicio | Probado en producción: tras reiniciar el contenedor la conexión seguía guardada y el Connector se reconectó solo |
| La credencial revocada no vuelve | Probado en producción: `403 revoked` por petición y por WebSocket |
| Instalar el Connector con `hermes plugins install --ref` desde GitHub | Probado con Hermes 0.21.2 |
| El Connector comprueba servicio y versión antes de gastar un Pairing ID | Probado: pruebas automáticas |
| Pruebas hostiles (Pairing ID robado, reutilizado o caducado, Hermes o teléfono falsos, repetición, fuerza bruta, servicio comprometido) | Probado: pruebas automáticas |
| Instalar esta Skill desde GitHub | Probado con Hermes 0.21.2 |
| **La app DotPulse en el App Store** | **Todavía no está publicada** |
| La app DotPulse en un iPhone real, con datos móviles ↔ Hermes en otra red | **Sin probar.** Las pruebas anteriores se hicieron con un teléfono de pruebas y los dos extremos en la misma máquina |
| El agente ofreciendo e instalando el Connector desde el chat | **Sin probar con un modelo.** La orden se ejecutó sin terminal interactiva, como la ejecuta el agente, y sustituyó un plugin antiguo |
| Telegram real | **Sin probar.** Probado sin red contra la librería de Telegram |
| Un modelo decidiendo llamar a las herramientas | **Sin probar.** Las herramientas se llamaron directamente |
| Aviso push de una solicitud de conexión | **No existe.** La solicitud aparece con DotPulse abierto |
| Enviar una tarea a un grupo de agentes | **No existe** |

## Instalar la Skill

El Connector trae esta misma Skill dentro. Para instalarla aparte:

```bash
hermes skills install https://raw.githubusercontent.com/1122RealEstate/SkillDotPulse/main/SKILL.md
```

Sin el Connector, la Skill solo sirve para que Hermes te diga que hace falta instalarlo, sin
gastar el Pairing ID que pegaste.

## Estados de una conexión

<p align="center">
  <img src="assets/dotpulse-dots.svg" alt="Seis Dots, uno por estado: pending, authorized, connected, expired, rejected y revoked" width="100%">
</p>

| Estado | Qué significa |
|---|---|
| `pending` | Hay un Pairing ID copiado, o una solicitud esperando tu confirmación |
| `authorized` | Pulsaste Autorizar en el teléfono; falta confirmar en Hermes, o Hermes está recogiendo su credencial |
| `connected` | La conexión existe y Hermes la mantiene |
| `expired` | Se agotó el tiempo. Hace falta un Pairing ID nuevo |
| `rejected` | No se aceptó: no válido, ya usado o rechazado por ti |
| `revoked` | Se desconectó. Solo se vuelve con un Pairing ID nuevo |

## Qué hay en el repositorio

```
README.md                     esto
SKILL.md                      la Skill: concisa y operativa, es lo que lee el agente
references/
  connection.md               las piezas, el Pairing ID y el ciclo de vida de una conexión
  protocol.md                 qué hacen las herramientas por debajo
  security.md                 qué se impone y quién lo impone
  troubleshooting.md          cada mensaje que puedes ver y qué hacer
examples/
  pairing-example.md          conversaciones de ejemplo
assets/                       logo, Dots y diagrama (SVG, sin JavaScript ni recursos externos)
```

Los Dots de las imágenes son los de la app. Los documentos de la Skill están en inglés porque los
lee el agente; el agente responde en tu idioma.

## Avisar de un problema de seguridad

Usa los avisos de seguridad privados de este repositorio (pestaña *Security* → *Report a
vulnerability*) y no incluyas nunca un Pairing ID real.
