# Política de privacidad — Sextant

**Versión 5 · Última actualización: 2026-09-28** · [English version](PRIVACY.md)

> **Traducción.** Esta es una traducción de la política en inglés, que es la que prevalece en caso de
> discrepancia. Ambas se actualizan a la vez.
>
> **⚠️ TODAVÍA NO SE PUEDE PUBLICAR** — quedan los mismos tres puntos abiertos que en la versión en
> inglés (revisión legal, quién publica Sextant y el canal de contacto), y además esta traducción
> necesita la misma revisión legal. `npm run publication:check` falla mientras sigan abiertos.

---

## 0. A quién se refiere esta política

Sextant es una extensión para Chrome y Edge que muestra qué metadato de Salesforce es un texto, con sus
traducciones, y permite editar esas traducciones. La publica `[ EDITOR — DECISIÓN PENDIENTE ]`
("el desarrollador", "nosotros").

Sextant es un producto independiente. No lo fabrica, respalda ni mantiene Salesforce, Inc. Salesforce y
Lightning son marcas de Salesforce, Inc.

## Resumen

Sextant no tiene servidor ni cuentas. Funciona dentro de tu navegador, con la sesión de Salesforce que ya
tienes abierta, y se comunica con tu org de Salesforce y con nada más.

Estas seis afirmaciones son las mismas que Sextant muestra en Ajustes → Privacidad y seguridad. Cada una
está respaldada por la propia protección del navegador, por pruebas automáticas o por una comprobación
en cada build de publicación; [`DATA_HANDLING.md`](DATA_HANDLING.md) (en inglés) indica cuál, qué no
significa cada una y dónde comprobarla.

- Sextant solo envía solicitudes a Salesforce, y el navegador impide que sus páginas contacten con cualquier otro sitio.
- Sin analítica, telemetría, informes de errores ni publicidad.
- No hay cuenta ni servidor de Sextant. Nada de lo que haces en Sextant se envía a su desarrollador.
- Todo lo que ejecuta Sextant va dentro de la extensión instalada. No descarga código.
- Tu sesión de Salesforce se lee cuando una solicitud la necesita, y Sextant nunca la guarda, la registra ni la exporta.
- Lo que Sextant guarda se queda en este perfil del navegador. Puedes verlo, exportarlo y borrarlo en Ajustes.

## 1. Qué lee Sextant de Salesforce

Para hacer su trabajo, Sextant lee de la org de Salesforce en la que has iniciado sesión, a través de las
propias API de Salesforce y **en tu nombre**, así que solo puede leer lo que tu usuario de Salesforce
tiene permitido leer:

- **Metadatos traducibles y sus traducciones**: etiquetas personalizadas, etiquetas de objetos y campos,
  valores de listas de selección, tipos de registro, botones y vínculos, acciones rápidas, pestañas,
  aplicaciones, flujos y secciones de formatos de página; sus nombres de API e IDs; y los idiomas
  habilitados en la org.
- **Las listas de componentes de la org**, incluida la fecha de la última modificación de cada
  componente y el **nombre del usuario de Salesforce** que lo modificó por última vez, tal como lo indica
  Salesforce.
- **La identidad de la org y del usuario**: el ID, el nombre y la instancia de la org, y si es un sandbox
  o producción; tu ID de usuario, tu nombre de usuario y tu nombre. Sextant los usa para mantener
  separados los datos de cada org y para indicarte en qué org estás trabajando.
- **Los registros de despliegues recientes** (sus IDs, estado y fechas, y los componentes que contenían),
  para avisarte cuando un despliegue ha terminado.
- **El Setup Audit Trail de tu org, solo si lo importas** (Actividad › ⋯ › *Importar historial de Salesforce…*): para
  el periodo que elijas, quién hizo cada cambio en Setup, cuándo, el usuario con el que había iniciado sesión si era
  otro, y la descripción del propio Salesforce. No se lee nada de él hasta que lo pides, y Sextant te indica antes
  cuántas entradas tiene ese periodo.

Sextant no lee tus registros de negocio —cuentas, contactos, oportunidades, casos, etc.— a través de las
API de Salesforce. Como se ejecuta dentro de las páginas Lightning, puede ver lo que esas páginas
muestran; lee allí el texto de las etiquetas para identificarlas, dentro de tu navegador, y no envía nada
de la página a ningún sitio.

## 2. Tu sesión de Salesforce

El proceso en segundo plano de Sextant lee de tu navegador la cookie de sesión `sid` de tu org cuando
necesita llamar a la API de tu org en tu nombre.

- Solo se lee para la dirección de la API de Salesforce de tu org.
- Sextant nunca la guarda, la registra ni la exporta, y solo se envía a la org a la que pertenece.
- Sextant nunca te pide una contraseña de Salesforce, y nunca debería hacerlo.

Cerrar la sesión en Salesforce termina la sesión; Sextant muestra entonces la org como desconectada.

## 3. Qué guarda Sextant y dónde

Sextant guarda sus datos en el almacenamiento del navegador para esta extensión (`chrome.storage.local`),
en tu perfil del navegador, en tu equipo. No se sincronizan con tu cuenta de Google ni se envían a
ningún sitio.

Guarda: tus ajustes; las orgs que has usado; una caché de los metadatos que ha leído; el historial de
cambios que ha observado en tus orgs y las ediciones de traducciones que hiciste, durante el tiempo que elijas en
Ajustes (7, 30 o 180 días, o hasta que lo borres, dentro de un límite de tamaño); el historial de auditoría de
Salesforce que decidiste importar, hasta que lo elimines; instantáneas de traducciones para compararlas; los cambios
de traducción que pusiste en cola o desplegaste; tu Workspace; y diagnósticos para desarrolladores, solo si los
activas. Parte de esto incluye nombres de usuarios de
Salesforce de tus orgs (por ejemplo, quién modificó por última vez un componente).

Cada elemento guardado, por qué se guarda y durante cuánto tiempo aparece en
[`DATA_HANDLING.md`](DATA_HANDLING.md) §3, que se genera a partir del código. Sextant no cifra estos datos
más allá de lo que ya hacen tu sistema operativo y tu navegador; cualquiera que pueda usar tu perfil del
navegador puede leerlos.

## 4. Qué envía Sextant y a quién

**A tu org de Salesforce:** las solicitudes descritas en el §1 y, solo cuando lo pides, los cambios de
traducción que guardas o despliegas (§5). Van a la dirección de la API de tu propia org.

**Al desarrollador:** nada. No hay servidor, cuenta, analítica, telemetría ni informes de errores de
Sextant. No recibimos, ni vemos, ni conservamos los datos que Sextant maneja en tu navegador.

**A cualquier otro:** nada. Sextant no carga fuentes, scripts, estilos ni imágenes de otros sitios, no
tiene publicidad y no vende, alquila ni comparte datos.

**Chrome Web Store y tu navegador.** Google distribuye Sextant y sus actualizaciones a través de Chrome Web
Store, con sus propias condiciones, y —como cualquier editor— el desarrollador puede ver las estadísticas
agregadas que ofrece la tienda, como el número de usuarios. Tu navegador se comunica con su fabricante
para sus propios fines, como buscar actualizaciones. Sextant no envía nada a ninguno de los dos.

## 5. Qué escribe Sextant en tu org

Cuando guardas o despliegas una traducción, Sextant la escribe en tu org de Salesforce a través de las API
de Salesforce: el mismo cambio que podrías hacer en Setup.

- Solo escribe cuando se lo pides.
- Antes vuelve a leer el valor actual y se niega a sobrescribir un cambio que alguien hizo después de que
  empezaras a editar.
- Guardar un cambio directamente en una org de producción, o en una org que Sextant no puede identificar,
  te pide confirmación antes.
- La escritura se hace **como tu usuario de Salesforce**, con tus permisos. Salesforce registra los
  despliegues de metadatos a nombre de tu usuario. El *Setup Audit Trail* de Salesforce **no** registra los
  cambios de traducción —ni los hechos con Sextant ni los hechos en el propio Translation Workbench de
  Setup—, y por eso Sextant conserva su propio registro local de los cambios que haces con él.

## 6. Inteligencia artificial

**Sextant no contiene ninguna función de IA y no envía nada a ningún proveedor de IA.**

La política de seguridad de la extensión bloquearía en cualquier caso una solicitud así. Si eso cambia
alguna vez, esta política dirá —antes de que la función se publique— exactamente qué se enviaría, a qué
servicio y con la clave de quién.

## 7. Archivos que creas

Sextant puede exportar archivos cuando lo pides: una exportación de sus datos locales, el historial de
Activity, el Workspace, informes y archivos `package.xml`. Se guardan donde tú eliges y son tuyos. Pueden
contener metadatos de la org, textos de traducción y nombres de personas de tus orgs. Una exportación de
los datos locales de Sextant nunca contiene tu sesión de Salesforce ni ninguna otra credencial.

Sextant también puede importar un archivo de Workspace o de Activity que elijas. Se lee localmente y se
comprueba antes de conservar nada de él.

## 8. Si te pones en contacto con nosotros

Si nos escribes —para pedir ayuda o para informar de un problema de seguridad— recibimos lo que envías,
como tu dirección de correo y tu mensaje, y lo usamos solo para responderte y para resolver el problema
que nos comunicas. No envíes nunca un ID de sesión de Salesforce, una contraseña ni datos reales de una
org (consulta `SECURITY.md` §2).

> **⚠️ REQUIERE REVISIÓN LEGAL** — cuánto tiempo se conserva la correspondencia, y con qué base legal,
> depende del editor y del canal que se elijan (§0, §14).

## 9. Tus opciones y derechos

> **⚠️ REQUIERE REVISIÓN LEGAL** — esta sección describe derechos previstos en la normativa de protección
> de datos (como el RGPD de la UE y del Reino Unido) y debe revisarse antes de publicarse.

- **Ver, exportar y borrar** lo que guarda Sextant: Ajustes → Privacidad y seguridad. Borrar ahí elimina
  los datos de Sextant de tu navegador y nunca cambia nada en Salesforce.
- **Eliminarlo todo**: desinstala Sextant; el navegador borra su almacenamiento.
- **Impedir que Sextant lea una org**: cierra la sesión en esa org o retira el acceso de la extensión a los
  sitios de Salesforce en los ajustes de extensiones de tu navegador.

Como los datos que maneja Sextant se quedan en tu navegador y no se nos envían, no conservamos ninguna
copia y no podemos consultarlos, corregirlos ni borrarlos por ti: puedes hacer cada una de esas cosas
directamente, como se indica arriba. Para lo que tú mismo nos hayas enviado (§8), puedes pedirnos que lo
consultemos, corrijamos o borremos a través del contacto del §14.

Si usas Sextant en la org de Salesforce de una organización, esa organización puede tener sus propias
normas sobre los datos que contiene, incluidos los nombres de usuarios que Sextant muestra y guarda
localmente.

## 10. Menores

Sextant es una herramienta profesional para administradores y desarrolladores de Salesforce. No está
dirigida a menores.

## 11. Seguridad

Cómo protege Sextant lo que maneja, contra qué no puede protegerse y cómo informar de un problema se
explica en [`SECURITY.md`](SECURITY.md) (en inglés). Ningún software está libre de defectos; el enfoque de
Sextant es limitar lo que puede alcanzar, comprobarlo automáticamente y decir con claridad dónde están los
límites.

## 12. El sitio web y la demo interactiva

El sitio web de Sextant (`https://usesextant.dev`) y la demo interactiva que aloja (`/demo/`) son páginas estáticas.

- **El sitio web** no crea cookies, no ejecuta ningún script, no tiene analítica y no carga fuentes, estilos,
  scripts ni imágenes de ningún otro sitio. Como en cualquier sitio web, el proveedor de alojamiento recibe cada
  solicitud —la página solicitada, tu dirección IP y la identificación de tu navegador— bajo sus propias condiciones.
- **La demo** ejecuta el propio código de Sextant en tu navegador contra una org de Salesforce ficticia. No envía
  nada a ningún sitio: su política de seguridad de contenidos no permite ninguna solicitud salvo la de sus propios
  archivos, y no guarda nada en el almacenamiento de tu navegador, así que al cerrar la pestaña se descarta todo
  lo que hiciste en ella.
- Ni el sitio web ni la demo se conectan a ninguna org de Salesforce, y nada de lo que haces en la demo llega al
  desarrollador.

> **⚠️ REQUIERE REVISIÓN LEGAL** — las condiciones de tratamiento de datos y la retención de registros de acceso
> del proveedor de alojamiento (`docs/HOSTING.md` §8) son del proveedor; si esta política debe nombrarlo es una
> cuestión para el asesor legal.

## 13. Cambios en esta política

Cada versión tiene fecha y número. Un cambio en lo que Sextant recoge, guarda o envía se hará aquí **antes**
de la versión que lo introduzca, y se describirá en las notas de esa versión y en la ficha de la tienda.

| Versión | Fecha | Cambio |
|---|---|---|
| 5 | 2026-09-28 | Se añade el §15: la Política de Datos de Usuario de Chrome Web Store y sus requisitos de Uso Limitado, tal como se aplican a Sextant. Nada cambia en lo que Sextant lee, guarda o envía. |
| 4 | 2026-09-22 | Se añade la importación del Setup Audit Trail: qué lee Sextant (§1) y qué guarda (§3) cuando decides importar a Actividad el propio historial de auditoría de tu org; y las opciones de retención del historial de Actividad (§3). |
| 3 | 2026-09-16 | Se añade el §12: el sitio web y la demo interactiva —sin cookies, sin scripts ni analítica en el sitio; la demo no envía ni guarda nada— y la dirección pública de esta política. Las secciones 12 y 13 pasan a ser la 13 y la 14. |
| 2 | 2026-09-15 | Se corrigen dos afirmaciones inexactas: la protección del navegador sobre el límite de red de Sextant proviene de su política de seguridad de contenido, no de sus permisos de host; y los cambios de traducción no aparecen en el Setup Audit Trail de Salesforce. Se añaden los nombres de usuarios de Salesforce, los registros de despliegues, las importaciones, las estadísticas de Chrome Web Store y tus derechos. |
| 1 | 2026-09-01 | Primera versión (solo en inglés). |

## 14. Contacto

<!-- DECISIÓN PENDIENTE: el mismo canal que SECURITY.md §1. -->
`[ CONTACTO DE PRIVACIDAD — SIN CONFIGURAR ]`

## 15. La Política de Datos de Usuario de Chrome Web Store

El uso que Sextant hace de la información que maneja cumple la Política de Datos de Usuario de Chrome Web Store,
incluidos los requisitos de Uso Limitado. En la práctica:

- **Su única finalidad, y nada más.** Lo que Sextant lee se usa solo para mostrarte qué es un elemento de metadatos de
  Salesforce y qué dice en cada idioma, y para cambiar traducciones cuando lo pides (§1, §5).
- **Sin cesión.** No se cede nada a nadie salvo a tu propia org de Salesforce, como parte de esa finalidad (§4).
- **Nadie lo lee.** Nada de ello llega al desarrollador (§4), así que nadie allí puede leerlo. Lo que tú nos envíes está
  cubierto por el §8.
- **Sin publicidad, sin intermediarios, sin decisiones de crédito.** Nunca se usa, vende ni cede para publicidad, a
  intermediarios de datos u otros revendedores, ni para decidir la solvencia de nadie o su acceso a un préstamo.
