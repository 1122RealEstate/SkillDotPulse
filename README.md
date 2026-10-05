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
> **Todavía no se puede conectar siguiendo este README.** Todo el recorrido funciona y está probado
> en una sola máquina, pero dos piezas que no están en este repositorio aún no son públicas: el
> servicio de conexión de DotPulse y el conector para Hermes. Mira
> [Funciones actualmente soportadas](#funciones-actualmente-soportadas) antes de intentarlo.

## Qué es

DotPulse es la app para trabajar desde el teléfono con los agentes de tu Hermes. Cada agente es un
**Dot**.

| Pieza | Qué hace | Dónde está |
|---|---|---|
| **Skill** (este repositorio) | Le dice al agente de Hermes cuándo usar las herramientas del conector y cuándo no. Solo instrucciones | Aquí |
| **Conector** | Un plugin de Hermes: presenta el Pairing ID, guarda la credencial y mantiene la conexión | Aparte; aún no es público |
| **Servicio de conexión** | Decide si un Pairing ID vale y reenvía entre el teléfono y Hermes sin poder leer lo que pasa | De DotPulse; aún sin dirección pública |
| **DotPulse** | Genera la conexión, pide tu confirmación y administra conexiones, agentes y grupos | Tu teléfono |

La Skill no contiene código, direcciones ni claves: sola, no conecta nada.

<p align="center">
  <img src="assets/dotpulse-flow.svg" alt="DotPulse, Pairing ID, Hermes o Telegram, autorizas en DotPulse, Connected" width="100%">
</p>

## Funciones actualmente soportadas

Estado a 5 de octubre de 2026. «Probado» quiere decir ejecutado de verdad, no solo escrito.

| Función | Estado | Cómo se probó |
|---|---|---|
| Emparejar: copiar, pegar en Hermes, confirmar en el teléfono y en Hermes, conectado | Probado en una sola máquina | El código Swift de la app contra un servicio real, el conector real y un `hermes serve` 0.21.2 real |
| El conector instalado con la orden de Hermes y manejado por Hermes | Probado desde un repositorio local | `hermes plugins install --ref`, y después las herramientas y `/dotpulse` a través del registro y las órdenes de Hermes |
| Usar Hermes a través de la conexión cifrada | Probado en una sola máquina | La API de Hermes y su canal en vivo respondieron a través del enlace |
| Reconexión tras perder la red o reiniciar | Probado en una sola máquina | Pruebas automáticas del servicio y el conector |
| Revocar y eliminar una conexión | Probado en una sola máquina | La credencial muere, se cortan los dos extremos y no se puede volver |
| Pairing ID ajeno: un teléfono que se aprueba a sí mismo | Probado: no conecta | Pruebas automáticas |
| Instalar esta Skill desde GitHub | Probado | `hermes skills install` en una instalación limpia de Hermes 0.21.2 |
| Listar, crear, editar y eliminar agentes de Hermes | Probado contra un Hermes real | Un agente temporal creado, cambiado, renombrado y eliminado, releyendo cada cambio |
| Pantallas de la app (Conexiones, Agentes, Grupos, Actividad, autorización) | **Sin probar en pantalla** | La app compila y se instaló en un iPhone 17 Pro Max; nadie ha recorrido aún las pantallas |
| Conectar entre redes distintas (móvil ↔ Wi-Fi) | **Sin probar** | Necesita el servicio de conexión público |
| Telegram real | **Sin probar** | El conector lo atiende y está probado sin red contra la librería de Telegram |
| Un modelo decidiendo llamar a las herramientas | **Sin probar** | Las herramientas se llamaron directamente |
| Enviar una tarea a un grupo de agentes | **No existe** | Hermes no comunica agentes de instalaciones distintas; DotPulse tendría que repartirla y esa parte no está hecha |
| Aviso push de una solicitud de conexión | **No existe** | La solicitud solo aparece con DotPulse abierto |
| OpenClaw | **No existe** | No hay integración ni se ha probado nada con OpenClaw |

**Qué falta para que un usuario nuevo pueda conectar:**

1. Alojar el servicio de conexión con un dominio y TLS.
2. Publicar el conector en un repositorio desde el que Hermes pueda instalarlo.
3. Compilar la app con la dirección de ese servicio.

## Conectar DotPulse

Los pasos de persona son 1 a 3, 5, 6, 8 y 9. Lo que ocurre en Hermes (7) y lo que dicen las
listas (10) está comprobado en las pruebas. Lo que aparece en la pantalla de la app está descrito
como está programado, sin recorrer aún en pantalla.

**Antes de empezar** hacen falta las dos piezas que aún no son públicas: una app DotPulse compilada
con el servicio de conexión, y el [conector instalado](#instalar-el-conector) en tu Hermes.

1. **Abre DotPulse** en tu teléfono.
2. **Ve a la pantalla de conexión.** La primera vez es lo que ves al abrir la app («Conecta tu
   Hermes»). Más adelante: pestaña Conexiones → Añadir conexión. Arriba, deja elegido *Emparejar*.
3. **Pulsa «Copiar conexión».** Puedes cambiar antes el nombre con el que se presentará el teléfono.
4. **Qué se copia.** Cuatro líneas, y nada más:

   ```
   DotPulse · conexión
   Skill: https://github.com/1122RealEstate/SkillDotPulse
   Pairing ID: DPP1-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
   Hermes: conecta DotPulse con este Pairing ID.
   ```

   El Pairing ID es distinto cada vez, vale 5 minutos y sirve una sola vez. No hay direcciones,
   claves ni contraseñas.
5. **Abre Hermes** o tu chat privado de Telegram con él.
6. **Pega lo copiado, sin cambiar nada**, y envíalo.

### Conectar con Hermes

7. **Qué ocurre en Hermes.** El conector presenta el Pairing ID a DotPulse. Si es bueno, Hermes
   contesta:

   > DotPulse quiere conectar «iPhone de Ana» con este Hermes.
   >
   > Código de verificación: 123 456
   >
   > Hacen falta dos confirmaciones, y tienes 2 minutos:
   > 1. En DotPulse, en tu teléfono: verás esta solicitud con el mismo código. Pulsa Autorizar.
   > 2. Aquí: confirma que ese teléfono es el tuyo escribiendo /dotpulse confirmar (o /dotpulse
   >    rechazar si no lo es).
   >
   > Si tu propio DotPulse no está mostrando este código ahora mismo, no confirmes: alguien está
   > intentando conectar su teléfono a tu Hermes.

8. **En DotPulse.** Al copiar se abre una pantalla con tu Dot que te acompaña todo el camino:
   «Conexión copiada», «Esperando a Hermes…» con el tiempo que queda, y, cuando Hermes presenta
   la conexión, «Hermes quiere conectarse» con su nombre, dónde se pegó (Hermes o Telegram), la
   hora y el **código de verificación**. Comprueba que el código es el mismo y pulsa
   **Autorizar** (o **Rechazar**). La pantalla pasa a «Conectando con Hermes…»: todavía no está
   conectado.
9. **En Hermes.** Escribe `/dotpulse confirmar`. Hermes responde «Confirmado aquí…». Da igual cuál
   de las dos confirmaciones des primero; sin las dos no se conecta nada.
10. **Cómo saber que está conectado.** La pantalla de DotPulse muestra «Hermes conectado» solo
    cuando el servicio lo confirma, y ofrece **Ver agentes**. En la pestaña Conexiones ese Hermes
    aparece *En línea*. En Hermes, `/dotpulse` lo lista así:

    ```
    • iPhone de Ana — 64e9ecc2f11a — conectado, en línea — capacidades: hermes.api
    ```
11. **Qué ocurre después.** La app empieza a usar ese Hermes: aparecen tus Dots y sus agentes.
    Hermes mantiene la conexión por su cuenta y la recupera si se cae la red o se reinicia. La
    capacidad concedida es la que ves: `hermes.api`, que deja a la app usar ese Hermes
    (conversaciones, Dots, tareas y voz). Hoy no existe ninguna otra.

No hace falta SSH, terminal, claves públicas ni abrir puertos.

### Conectar mediante Telegram

> [!NOTE]
> **Sin verificar con Telegram real.** Lo que sigue es lo que hace el conector y lo que se probó
> sin red contra la librería de Telegram. No lo tomes como comprobado hasta que este aviso
> desaparezca.

- Envía exactamente lo que copió DotPulse a tu **chat privado** con el bot de tu Hermes. No hay
  ninguna orden que escribir.
- Solo se atiende a un usuario que tu bot ya tenga permitido. En un grupo o canal el mensaje se
  descarta, y a un desconocido no se le contesta.
- El mensaje lo contesta el conector directamente: el Pairing ID no llega al modelo.
- El bot responde con el nombre del teléfono, el código y dos botones: **✅ Es mi teléfono** y
  **✖️ No es mío**. Ese botón es la confirmación en Hermes; la otra es Autorizar en DotPulse.
- Cuando están las dos, un segundo mensaje dice «DotPulse conectado…», o el motivo por el que no.

## Instalar el conector

Hermes instala un plugin con su propia orden. No instala plugins por recibir un mensaje, y su
agente no tiene ninguna herramienta para hacerlo: hace falta **una orden y un reinicio, una vez
por Hermes**. No hay forma soportada de evitarlo.

```bash
hermes plugins install <repositorio del conector> --ref <commit> --enable
hermes gateway restart
```

> [!WARNING]
> El repositorio del conector **todavía no es público**, así que hoy no hay nada que poner en
> `<repositorio del conector>`. Cuando lo sea, aquí aparecerán su nombre y el commit de cada versión.

Lo que sí está comprobado, con Hermes 0.21.2 y el conector 0.2.0 desde un repositorio local:

- Hermes **analiza el plugin antes de instalarlo** y lo instala. Avisa de que declara dos
  dependencias, `aiohttp` y `cryptography`, que no instala él; Hermes ya las trae.
- `--ref` fija el commit exacto. Es la comprobación de integridad: un commit nombra un único
  conjunto de archivos.
- Después de instalar, `python3 ~/.hermes/plugins/dotpulse/verify.py` compara cada archivo con el
  manifiesto de la versión y responde `ok: 9 archivos coinciden con el manifiesto`.
- Tras reiniciar, Hermes tiene las herramientas `dotpulse_pair`, `dotpulse_status` y
  `dotpulse_disconnect`, la orden `/dotpulse` y esta Skill.

El conector no abre ningún puerto en tu equipo: solo hace conexiones salientes.

## Instalar la Skill

Con el conector no hace falta: trae esta misma Skill dentro y la registra con el nombre
`dotpulse:dotpulse`. Para instalarla aparte, desde este repositorio:

```bash
hermes skills install https://raw.githubusercontent.com/1122RealEstate/SkillDotPulse/main/SKILL.md
```

Probado con Hermes 0.21.2 en una instalación limpia. Hermes la analiza como Skill «de la
comunidad», la da por segura (`Verdict: SAFE`) y la instala con el nombre `dotpulse`, junto con
`references/` y `examples/`. Sin `--yes` pide confirmación. La forma corta
`1122RealEstate/SkillDotPulse` no sirve: Hermes solo acepta así las Skills que están en una
subcarpeta.

Instalada así, la Skill no aparece en la lista del agente hasta que el conector está instalado,
porque declara que necesita la herramienta `dotpulse_pair`. Sin conector no hay nada que pueda hacer.

## Administrar conexiones

En DotPulse, pestaña **Conexiones**. Puedes tener varios Hermes conectados; uno es el que está *en
uso* (el que alimenta Inicio y las conversaciones).

Cada conexión muestra su nombre, que es Hermes, su estado, cuándo se conectó, su última actividad,
sus capacidades y sus agentes. Desde ahí puedes cambiarle el nombre, pasar a usarla, reconectar y
eliminarla.

En Hermes, `/dotpulse` lista los teléfonos conectados a ese Hermes.

## Administrar agentes

Pestaña **Agentes**: los agentes que cada Hermes conectado tiene de verdad, Hermes por Hermes.

| Con un agente de Hermes puedes | |
|---|---|
| Verlo, crearlo, cambiarle el nombre | Sí |
| Editar su descripción y sus instrucciones | Sí |
| Activar y desactivar sus herramientas | Sí |
| Ver su modelo | Sí; se cambia desde la conversación con ese Dot |
| Eliminarlo | Sí, salvo el agente principal de cada Hermes |
| Editar sus skills o su memoria desde DotPulse | No está hecho |
| Activarlo o desactivarlo sin eliminarlo | Hermes no tiene esa operación |

Cada cambio se hace en Hermes y se vuelve a leer de Hermes: si no quedó guardado, DotPulse lo dice.
Eliminar un agente pide confirmación, indica a qué Hermes pertenece y de qué grupos saldrá, y no se
puede deshacer.

## Crear grupos

Pestaña **Grupos** → Nuevo grupo: nombre, objetivo y los agentes que quieras, de una o de varias
conexiones. Solo se ofrecen los agentes que se pueden leer en ese momento. Puedes elegir un
coordinador.

Un grupo es una lista de referencias: no copia agentes y solo existe en tu teléfono. Si eliminas un
agente o una conexión, sale de sus grupos.

**Enviar una tarea a un grupo todavía no existe.** La pantalla lo indica.

## Revocar o eliminar una conexión

- **En DotPulse:** Conexiones → la conexión → **Eliminar conexión**. Pide confirmación, revoca la
  conexión en el servicio, corta el enlace, borra sus credenciales del teléfono y saca a sus
  agentes de los grupos. Si no se puede revocar, no se elimina: no desaparece mientras siga
  autorizada.
- **En Hermes:** `/dotpulse` para ver las conexiones y `/dotpulse desconectar <identificador>`.
  Hermes responde «Conexión revocada. «…» ya no puede usar este Hermes; para volver a conectar
  hará falta un Pairing ID nuevo.»

En los dos casos se corta en el momento, Hermes destruye su credencial y no reintenta. Solo se
vuelve con una conexión nueva.

## Seguridad

- **El Pairing ID caduca a los 5 minutos** y **funciona una sola vez**.
- **Hacen falta dos confirmaciones tuyas:** Autorizar en tu teléfono y confirmar en tu Hermes.
- **No pegues un Pairing ID que no hayas copiado tú.** Si alguien te envía uno y lo pegas en tu
  Hermes, el teléfono que pide entrar es el suyo. Lo notarás porque tu DotPulse no muestra ninguna
  solicitud con ese código: **no confirmes en Hermes**, escribe `/dotpulse rechazar` o pulsa «No es
  mío». Sin tu confirmación ese teléfono no entra, aunque su dueño pulse Autorizar.
- **No compartas el tuyo antes de usarlo.** Quien lo pegue en su Hermes antes que tú hará que te
  llegue una solicitud de un Hermes que no es el tuyo. Recházala.
- **Cómo saber quién pide acceso.** La solicitud muestra el nombre de ese Hermes, si el texto se
  pegó en Hermes o en Telegram, la hora y un código que tiene que ser el mismo en los dos sitios.
- **Cómo rechazar.** Rechazar en DotPulse, o `/dotpulse rechazar` en Hermes. Si no haces nada,
  caduca sola a los 2 minutos.
- **Lo que hay en este repositorio no da acceso a nada.** Solo son instrucciones e imágenes.

El modelo completo, con lo que queda sin cubrir, está en [references/security.md](references/security.md).

## Troubleshooting

Los mensajes de Hermes son los que devuelve el conector, tal como salen en sus pruebas. Los de la
app son los que tiene programados.

| Situación | Qué verás | Qué hacer |
|---|---|---|
| Pairing ID caducado | Hermes: «Ese Pairing ID ha caducado. En DotPulse pulsa «Copiar conexión» para generar otro.» | Copia una conexión nueva y pégala antes de 5 minutos |
| Pairing ID ya utilizado, o que no existe | Hermes: «Ese Pairing ID no es válido o ya se usó. En DotPulse pulsa «Copiar conexión» para generar otro.» | Copia una conexión nueva. Hermes no distingue entre usado e inexistente |
| Se cortó al copiar | Hermes: «Ese Pairing ID está incompleto o dañado. En DotPulse pulsa «Copiar conexión» otra vez y pégalo entero.» | Vuelve a copiar y pega el texto completo |
| Rechazaste en el teléfono | Hermes: «La solicitud se rechazó en DotPulse. No se conectó nada.» | Si fue un error, empieza de nuevo |
| Rechazaste en Hermes | Hermes: «Solicitud rechazada desde Hermes. No se conectó nada.» | Lo mismo |
| Faltó una confirmación | Hermes: «Nadie confirmó la solicitud en DotPulse a tiempo. No se conectó nada. Para intentarlo de nuevo, pulsa «Copiar conexión» otra vez.» | Repite, y da las dos confirmaciones en 2 minutos |
| `/dotpulse confirmar` sin nada pendiente | Hermes: «No hay ninguna solicitud de DotPulse esperando confirmación aquí.» | La solicitud caducó o ya se respondió |
| Hermes sin Internet al emparejar | Hermes: «No pude contactar con DotPulse. Comprueba la conexión a Internet de este Hermes y genera un Pairing ID nuevo.» | Revisa la red del equipo de Hermes |
| Hermes apagado o sin red, ya conectado | En DotPulse, Conexiones: «Sin conexión ahora»; la app indica «Tu Hermes no está en línea.» | Nada: se reconecta solo cuando ese equipo vuelve |
| DotPulse sin Internet | En la app: «No se pudo contactar con DotPulse. Comprueba tu conexión a Internet.» | Revisa la red del teléfono |
| Conector no instalado | El agente no tiene la herramienta `dotpulse_pair` y `/dotpulse` no existe | [Instala el conector](#instalar-el-conector). Aún no es público |
| Servicio sin configurar | Hermes: «Este Hermes todavía no tiene configurado el servicio DotPulse Link, así que no puede conectarse con DotPulse.» | Es el estado actual de cualquier instalación: el servicio aún no tiene dirección pública |
| Conexión revocada | En la app: «Esta conexión fue revocada.» En Hermes, `/dotpulse` deja de listarla | Para volver, copia y pega una conexión nueva |
| Demasiados intentos | Hermes: «Demasiados intentos seguidos. Espera unos minutos y genera un Pairing ID nuevo en DotPulse.» | Espera unos minutos |

## Demostración

Una conexión real de principio a fin, el 5 de octubre de 2026, en una sola máquina. El conector se
instaló con `hermes plugins install --ref` y lo cargó Hermes; las herramientas y la orden se
llamaron a través de Hermes; el conector arrancó un `hermes serve` de verdad. El teléfono fue un
sustituto de pruebas que habla el mismo protocolo que la app. Esto fue lo que salió (los rótulos
de la izquierda están traducidos; lo de la derecha es literal, salvo el nombre del equipo):

```
Hermes cargó el conector                          /dotpulse registrada
la app copió una conexión                         (el Pairing ID no se imprime)
dotpulse_pair, por el registro de Hermes          pending · iPhone de pruebas · código 428 425
el teléfono muestra el mismo código               Hermes · <nombre del equipo> · 428 425
Autorizar solo en el teléfono                     todavía nada conectado
/dotpulse confirmar                               Confirmado aquí. En cuanto pulses Autorizar en DotPulse, «iPhone de pruebas» quedará conectado.
el proceso del conector se conectó                connected
dotpulse_status                                   connected · en línea · ['hermes.api']
el teléfono llega a `hermes serve` por el enlace  health 0.21.2 · profiles 200
/dotpulse                                         • iPhone de pruebas — 64e9ecc2f11a — conectado, en línea — capacidades: hermes.api
dotpulse_disconnect                               Conexión revocada. «iPhone de pruebas» ya no puede usar este Hermes; para volver a conectar hará falta un Pairing ID nuevo.
el teléfono la ve revocada                        ok
```

En otra prueba el teléfono fue el propio código Swift de la app, con el mismo resultado.

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
