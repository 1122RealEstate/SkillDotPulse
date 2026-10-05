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

> [!WARNING]
> **Todavía no se puede conectar siguiendo este README.** El flujo completo funciona y está probado
> en un entorno de pruebas, pero dos piezas que no están en este repositorio aún no son públicas:
> el servicio de conexión de DotPulse y el conector para Hermes. Mira
> [Estado del proyecto](#estado-del-proyecto) antes de intentarlo.

## Qué es

DotPulse es la app para trabajar desde el teléfono con los agentes de tu Hermes. Cada agente es un
**Dot**.

Esta Skill le dice al agente de Hermes cuándo usar las herramientas del **conector de DotPulse**
(un plugin de Hermes) y cuándo no. La Skill no contiene código, direcciones ni claves: sola, no
conecta nada.

<p align="center">
  <img src="assets/dotpulse-flow.svg" alt="DotPulse, Pairing ID, Hermes o Telegram, autorizas en DotPulse, Connected" width="100%">
</p>

## Estado del proyecto

Fecha de este estado: 5 de octubre de 2026.

### Qué funciona y cómo se probó

| Qué | Cómo se probó |
|---|---|
| Emparejar: copiar, pegar en Hermes, confirmar, conectado | El código de la app (Swift) contra un servicio de conexión real, el conector real y un `hermes serve` real, todo en una misma máquina |
| Usar Hermes a través de la conexión | En esa misma prueba: la API de Hermes y su canal en vivo respondieron a través del enlace cifrado |
| Revocar, y que no se pueda volver | En esa misma prueba y en las pruebas del servicio |
| Pairing ID caducado, reutilizado, incorrecto; rechazo; saltarse la confirmación; repetición de mensajes; pérdida de red; reinicio; dos Hermes con el mismo ID; ID filtrado después de usarse | Pruebas automáticas del servicio y del conector por sockets reales en la misma máquina |
| El conector dentro de Hermes | Cargado por el sistema de plugins de Hermes: registra sus herramientas, la orden `/dotpulse` y esta Skill |

Versiones: Hermes Agent 0.21.2 (Python 3.11) en macOS; la app compilada para iOS 26.6 e instalada
en un iPhone 17 Pro Max.

### Qué no se ha probado

- **La pantalla de la app paso a paso.** La app compila y se instaló en un iPhone, pero el
  recorrido en pantalla (pulsar «Copiar conexión», ver la solicitud, pulsar Autorizar) no se ha
  hecho todavía con una persona: lo que se probó es el código que hay detrás.
- **Telegram de verdad.** El conector atiende un Pairing ID pegado en Telegram, y eso está probado
  sin red contra la librería de Telegram. No se ha hecho la prueba con un bot real.
- **Un modelo decidiendo.** Las herramientas responden a través del registro de herramientas de
  Hermes; no se ha ejecutado una conversación real en la que un modelo lea el mensaje pegado y
  decida llamarlas.
- **Internet.** Todo se probó en una sola máquina. Nada se ha probado entre un teléfono en una red
  y un Hermes en otra.

### Qué no está disponible todavía

- **El servicio de conexión de DotPulse no tiene dirección pública.** Sin él, la app no muestra
  «Copiar conexión» y el conector responde que no está configurado.
- **El conector para Hermes no se distribuye todavía.** No está en este repositorio y no hay aún
  un sitio público desde el que instalarlo.
- **Aviso en el teléfono.** La solicitud solo aparece con DotPulse abierto; no llega una
  notificación.

## Cómo conectar DotPulse con Hermes

Este es el flujo tal como está construido. Los pasos 1 a 6 y 9 son lo que hace una persona; lo que
ocurre en Hermes (7) y lo que muestran las listas (10, 11) está comprobado en las pruebas; lo que
aparece en la pantalla de la app (4, 8) está descrito como está programado, sin comprobar aún en
pantalla.

**Antes de empezar** hacen falta las dos piezas que aún no son públicas: una app DotPulse compilada
con el servicio de conexión, y el conector de DotPulse instalado en tu Hermes.

1. **Abre DotPulse** en tu teléfono.
2. **Ve a la pantalla de conexión.** La primera vez es lo que ves al abrir la app («Conecta tu
   Hermes»). Más adelante: Ajustes → Servidores → Añadir servidor. Arriba, deja elegido *Emparejar*.
3. **Pulsa «Copiar conexión».** Puedes cambiar antes el nombre con el que se presentará el teléfono.
4. **Qué se copia.** Cuatro líneas, y nada más:

   ```
   DotPulse · conexión
   Skill: https://github.com/1122RealEstate/SkillDotPulse
   Pairing ID: DPP1-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
   Hermes: conecta DotPulse con este Pairing ID.
   ```

   El Pairing ID es distinto cada vez, vale 5 minutos y sirve una sola vez. No hay direcciones,
   claves ni contraseñas. La app muestra hasta qué hora vale y se queda esperando.
5. **Abre Hermes**: tu conversación con él en el ordenador.
6. **Pega lo copiado, sin cambiar nada**, y envíalo.
7. **Qué ocurre en Hermes.** El conector presenta el Pairing ID a DotPulse. Si es bueno, Hermes
   contesta:

   > DotPulse quiere conectar «iPhone de Ana» con este Hermes.
   >
   > Código de verificación: 123 456
   >
   > Abre DotPulse: verás esta solicitud con el mismo código. Si coincide, pulsa Autorizar allí.
   > Si no coincide o no la esperabas, pulsa Rechazar. Caduca en 2 minutos.

   El nombre es el de tu teléfono y el código cambia cada vez.
8. **Qué aparece en DotPulse.** Vuelve a la app. Se abre una pantalla «Autorizar conexión» con el
   nombre del Hermes que lo pide, dónde se pegó (Hermes o Telegram), la hora, el **código de
   verificación** en grande, lo que la conexión permitirá y dos botones: **Autorizar** y **Rechazar**.
9. **Confirma.** Comprueba que el código es el mismo que te mostró Hermes y pulsa **Autorizar**.
   Tienes 2 minutos.
10. **Cómo saber que está conectado.** En DotPulse, Ajustes → Conexiones: ese Hermes aparece con el
    estado *Conectado*. En Hermes, escribe `/dotpulse`: lista el teléfono como
    `conectado, en línea`.
11. **Qué ocurre después.** La app empieza a usar ese Hermes: aparecen tus Dots y puedes conversar
    con ellos. Hermes mantiene la conexión por su cuenta, también después de reiniciarse. Para ver
    qué se autorizó, `/dotpulse` muestra las capacidades de cada conexión. Hoy existe una:
    `hermes.api`, que deja a la app usar ese Hermes (conversaciones, Dots, tareas y voz).

No hace falta SSH, terminal, claves públicas ni abrir puertos.

## Instalación de la Skill

Hermes no instala una Skill por recibir un enlace en un mensaje, y su agente no puede instalarlas
por sí mismo. Hay dos formas reales de que Hermes tenga esta Skill:

**Con el conector (la normal).** El conector de DotPulse trae esta misma Skill dentro y la registra
al cargarse, con el nombre `dotpulse:dotpulse`. Quien instala el conector no tiene que instalar
nada más.

**Desde este repositorio.** Con la orden de Hermes para instalar una Skill desde una dirección:

```bash
hermes skills install https://raw.githubusercontent.com/1122RealEstate/SkillDotPulse/main/SKILL.md
```

La forma corta `1122RealEstate/SkillDotPulse` no sirve: Hermes solo acepta así las Skills que están
en una subcarpeta de un repositorio. Hermes trata esta Skill como «de la comunidad»: la analiza
antes de instalarla y pide confirmación.

Instalada así, la Skill no aparece en la lista del agente hasta que el conector está instalado,
porque declara que necesita la herramienta `dotpulse_pair`. Es lo esperado: sin conector no hay
nada que pueda hacer.

## Conectar usando Telegram

> [!NOTE]
> **Sin verificar con Telegram real.** Esta sección describe lo que el conector hace y lo que se
> probó sin red. No la tomes como comprobada hasta que este aviso desaparezca.

El conector atiende el mismo texto pegado en tu chat privado de Telegram con el bot de tu Hermes.
No hay un segundo protocolo: es el mismo Pairing ID y la misma confirmación en DotPulse.

- Envía exactamente lo que copió DotPulse, sin cambiar nada. No hay ninguna orden que escribir.
- Solo se atiende en un **chat privado** y solo a un usuario que tu bot ya tenga permitido. En un
  grupo o canal el mensaje se descarta, y a un desconocido no se le contesta.
- En Telegram el mensaje lo contesta el conector directamente: el Pairing ID no llega al modelo.
- El bot responde con el nombre del teléfono y el código de verificación, y un segundo mensaje
  cuando contestas en DotPulse: «DotPulse conectado…» o el motivo por el que no.

## Seguridad, en lenguaje sencillo

- **El Pairing ID caduca a los 5 minutos.** Si no lo usas, deja de valer solo.
- **Funciona una sola vez.** En cuanto un Hermes lo presenta, no vuelve a servir, ni a ti ni a nadie.
- **No lo compartas antes de usarlo.** Quien lo pegue en *su* Hermes antes que tú hará que te llegue
  una solicitud de un Hermes que no es el tuyo. Recházala.
- **No pegues un Pairing ID que no hayas copiado tú.** Pegar en tu Hermes uno que te envió otra
  persona conectaría *el teléfono de esa persona* con tu Hermes, y sería ella quien lo confirmara.
  Si te ha pasado, desconéctalo ya (más abajo).
- **DotPulse siempre pide confirmación.** Nada se conecta sin que pulses Autorizar en tu teléfono.
- **Cómo saber quién pide acceso.** La solicitud muestra el nombre de ese Hermes, si el texto se
  pegó en Hermes o en Telegram, la hora y un código. El código tiene que ser el mismo que te acaba
  de mostrar tu Hermes. Si no lo es, o no estabas conectando nada, no es tu solicitud.
- **Cómo rechazarla.** Pulsa Rechazar. No se conecta nada y ese Pairing ID queda inservible. Si no
  haces nada, la solicitud caduca sola a los 2 minutos.
- **Cómo desconectar más adelante.** En DotPulse: Ajustes → Conexiones → Desconectar y revocar.
  O en Hermes: `/dotpulse` para ver las conexiones y `/dotpulse desconectar <identificador>`. La
  conexión se corta en el momento y solo se puede volver con un Pairing ID nuevo.
- **Lo que hay en este repositorio no da acceso a nada.** Solo son instrucciones e imágenes.

El modelo completo, incluido un límite conocido, está en [references/security.md](references/security.md).

## Si algo no funciona

Los mensajes de Hermes son los que devuelve el conector, tal como salen en sus pruebas. Los de la
app son los que tiene programados.

| Situación | Qué verás | Qué hacer |
|---|---|---|
| Pairing ID caducado | Hermes: «Ese Pairing ID ha caducado. En DotPulse pulsa «Copiar conexión» para generar otro.» | Copia una conexión nueva y pégala antes de 5 minutos |
| Pairing ID ya utilizado, o que no existe | Hermes: «Ese Pairing ID no es válido o ya se usó. En DotPulse pulsa «Copiar conexión» para generar otro.» | Copia una conexión nueva. Hermes no distingue entre usado e inexistente: DotPulse no se lo dice |
| Se cortó al copiar | Hermes: «Ese Pairing ID está incompleto o dañado. En DotPulse pulsa «Copiar conexión» otra vez y pégalo entero.» | Vuelve a copiar y pega el texto completo |
| Solicitud rechazada | Hermes (al preguntarle, o en Telegram): «La solicitud se rechazó en DotPulse. No se conectó nada.» | Si fue un error, empieza de nuevo con otra conexión |
| Nadie confirmó a tiempo | Hermes: «Nadie confirmó la solicitud en DotPulse a tiempo. No se conectó nada. Para intentarlo de nuevo, pulsa «Copiar conexión» otra vez.» | Repite, y vuelve a DotPulse en menos de 2 minutos |
| Hermes sin conexión a Internet | Hermes: «No pude contactar con DotPulse. Comprueba la conexión a Internet de este Hermes y genera un Pairing ID nuevo.» | Revisa la red del equipo de Hermes y repite con una conexión nueva |
| Hermes apagado o sin red, ya conectado | En DotPulse, Ajustes → Conexiones: «Sin conexión ahora». La app indica «Tu Hermes no está en línea.» | Nada: se reconecta solo cuando ese equipo vuelve |
| DotPulse sin conexión a Internet | En la app: «No se pudo contactar con DotPulse. Comprueba tu conexión a Internet.» | Revisa la red del teléfono |
| Skill o conector no disponibles | El agente no tiene la herramienta `dotpulse_pair` y `/dotpulse` no existe | Instala el conector de DotPulse en Hermes. Aún no es público: mira el estado del proyecto |
| Servicio de conexión sin configurar | Hermes: «Este Hermes todavía no tiene configurado el servicio DotPulse Link, así que no puede conectarse con DotPulse.» | Es el estado actual de cualquier instalación: el servicio aún no tiene dirección pública |
| Conexión revocada | En la app: «Esta conexión fue revocada.» En Hermes, `/dotpulse` deja de listarla | Para volver, copia y pega una conexión nueva |
| Demasiados intentos | Hermes: «Demasiados intentos seguidos. Espera unos minutos y genera un Pairing ID nuevo en DotPulse.» | Espera unos minutos |

## Demostración

Una conexión real de principio a fin, el 5 de octubre de 2026, en una sola máquina: el código Swift
de la app hizo de teléfono, y el conector se ejecutó con el intérprete de Hermes y arrancó un
`hermes serve` de verdad. Esto es lo que dejaron escrito el servicio y el conector:

```
servicio  pairing 72426789… issued                      ← la app copió una conexión
servicio  pairing 72426789… claimed (hermes)            ← se pegó en Hermes
servicio  pairing 72426789… approved, link b9d7f183…    ← se autorizó desde el teléfono
servicio  credential for link b9d7f183… collected       ← Hermes recogió su credencial
conector  a pairing ended: connected
conector  link b9d7f183… attached                       ← conexión saliente de Hermes, en pie
servicio  device attached to link b9d7f183…             ← la app usa Hermes a través del enlace
servicio  link b9d7f183… revoked by its device          ← se revocó desde el teléfono
conector  link b9d7f183… was revoked; forgotten         ← Hermes destruyó su credencial
```

Entre «device attached» y «revoked», la prueba pidió datos a la API de Hermes con el token que
Hermes le entregó cifrado, y abrió su canal en vivo. Después de revocar, la conexión no se pudo
volver a abrir, y el mismo Pairing ID pegado otra vez fue rechazado.

Los identificadores están recortados porque así los escriben los registros: nunca contienen un
Pairing ID ni una credencial.

## Estados de una conexión

<p align="center">
  <img src="assets/dotpulse-dots.svg" alt="Seis Dots, uno por estado: pending, authorized, connected, expired, rejected y revoked" width="100%">
</p>

| Estado | Qué significa |
|---|---|
| `pending` | Hay un Pairing ID copiado, o una solicitud esperando tu confirmación |
| `authorized` | Pulsaste Autorizar; Hermes está recogiendo su credencial |
| `connected` | La conexión existe y Hermes la mantiene |
| `expired` | Se agotó el tiempo. Hace falta un Pairing ID nuevo |
| `rejected` | No se aceptó: no válido, ya usado o rechazado por ti |
| `revoked` | Se desconectó. Solo se vuelve con un Pairing ID nuevo |

## Qué hay en el repositorio

```
SKILL.md                      la Skill: cuándo usar las herramientas del conector y cuándo no
references/
  dotpulse.md                 qué es DotPulse y qué piezas hay
  pairing.md                  el Pairing ID y el ciclo de vida de una conexión
  protocol.md                 qué hacen las herramientas por debajo
  security.md                 qué se impone y quién lo impone
examples/
  pairing-example.md          conversaciones de ejemplo
assets/                       logo, Dots y diagrama (SVG, sin JavaScript ni recursos externos)
```

Los Dots de las imágenes son los de la app, dibujados por su propio motor. Los documentos de la
Skill están en inglés porque los lee el agente; el agente responde en el idioma del usuario.

## Avisar de un problema de seguridad

Usa los avisos de seguridad privados de este repositorio (pestaña *Security* → *Report a
vulnerability*) y no incluyas nunca un Pairing ID real.
