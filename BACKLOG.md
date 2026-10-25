# BACKLOG — loop de mejora continua

**Próximo ID:** BL-0024

> **Muestra real, no un caso de éxito.** El inspector de seguridad de Kaizora sobre OWASP NodeGoat (Apache-2.0, commit `c5cb68a`), el 2026-10-25. NodeGoat es una aplicación pública y famosa hecha para ser vulnerable, y el modelo probablemente la conoce de su entrenamiento: es el caso fácil (incluso hay comentarios en el código que marcan los defectos).
>
> **De 31 hallazgos a 23 ítems.** 25 hallazgos se verificaron a mano contra el código (la línea existe y contiene lo descrito) y se agruparon por defecto (por ejemplo, el SSRF y la respuesta sin escapar de `research.js` son un ítem; la contraseña en texto plano guardada y comparada, otro): 23 ítems. Los otros 6 no se pudieron confirmar y **no se anotaron**: un hallazgo sin comprobar no entra al backlog. Detalle en `evaluaciones/`.

## Items


### BL-0001 · [seguridad] Se ejecuta con eval() lo que el usuario escribe en el formulario de contribuciones
- **Prioridad:** P0
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/contributions.js:32`
- **Descripción:** `handleContributionsUpdate` hace `eval(req.body.preTax)` y lo mismo con `afterTax` y `roth` (líneas 32 a 34): cualquier usuario con sesión ejecuta JavaScript en el servidor. El propio código lo comenta como inseguro y deja una alternativa con `parseInt` comentada justo debajo.
- **Hermanos revisados:** Las otras dos `eval` del mismo handler (33 y 34) caen en este item. No hay más `eval(` en `app/`.
- **Criterios de aceptación:** Un valor como `process.exit()` en `preTax` ya no ejecuta nada; los tres campos se leen con `parseInt` y un valor no numérico responde 400; hay una prueba que lo demuestra.
- **Riesgo si se toca:** Bajo: cambia solo el parseo de tres campos numéricos.
- **Auto-resoluble:** no

### BL-0002 · [seguridad] El parámetro threshold se concatena dentro de una consulta $where de MongoDB
- **Prioridad:** P0
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/data/allocations-dao.js:78`
- **Descripción:** `getByUserIdAndThreshold` arma `{$where: \`this.userId == ${parsedUserId} && this.stocks > '${threshold}'\`}` con `threshold` sin validar (línea 78): permite inyectar JavaScript en la consulta (por ejemplo `1'; return 1 == '1`). El código trae comentada una versión con `parseInt` y rango 0 a 99.
- **Hermanos revisados:** La rama de la línea 73 (también `$where`) usa el valor ya convertido a entero: no es explotable con texto, pero conviene dejar una sola forma. Sin otros `$where` en `app/`.
- **Criterios de aceptación:** `threshold` se convierte con `parseInt` y se valida en rango; un texto con comillas responde 400; ninguna consulta usa `$where` con datos del usuario.
- **Riesgo si se toca:** Medio: cambia el filtro de una pantalla; probar con valores límite.
- **Auto-resoluble:** no

### BL-0003 · [seguridad] El servidor descarga la dirección que el usuario le pasa (SSRF) y devuelve el cuerpo como HTML
- **Prioridad:** P0
- **Tamaño:** M
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/research.js:15`
- **Descripción:** `displayResearch` hace `needle.get(req.query.url + req.query.symbol)` (línea 15) y escribe el cuerpo recibido con `Content-Type: text/html` (línea 25): el servidor visita cualquier dirección que indique el usuario, incluida la red interna, y refleja lo que reciba.
- **Hermanos revisados:** Es el único lugar de `app/` que llama a `needle`.
- **Criterios de aceptación:** La dirección base sale de una lista fija en el servidor, no de la URL; el cuerpo se devuelve como texto plano o escapado; una dirección interna responde 400.
- **Riesgo si se toca:** Medio: cambia la pantalla de investigación.
- **Auto-resoluble:** no

### BL-0004 · [seguridad] Las asignaciones se piden por el id de la URL, no por el de la sesión
- **Prioridad:** P0
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/index.js:63`
- **Descripción:** `app.get("/allocations/:userId", isLoggedIn, allocationsHandler.displayAllocations)` solo exige sesión iniciada: cualquier usuario puede pedir `/allocations/<otro id>` y ver las asignaciones de otra persona.
- **Hermanos revisados:** Misma forma en `/benefits` (item siguiente por prioridad) y en `updateBenefits`.
- **Criterios de aceptación:** El handler compara `:userId` con el de la sesión y responde 403 si no coincide (o ignora el parámetro y usa el de la sesión); hay una prueba con dos usuarios.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** no

### BL-0005 · [seguridad] Las contraseñas se guardan y se comparan en texto plano
- **Prioridad:** P0
- **Tamaño:** M
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/data/user-dao.js:25`
- **Descripción:** `addUser` guarda `password` tal como llega (línea 25) y `validateLogin` compara con `fromDB === fromUser` (línea 61). El código trae comentada una versión con hash y sal. Una filtración de la base expone todas las contraseñas.
- **Hermanos revisados:** Sin otros lugares que guarden contraseñas. Las cuentas existentes necesitan migración.
- **Criterios de aceptación:** Las contraseñas nuevas se guardan con hash y sal; el login compara con la función de hash; hay un plan de migración para las cuentas existentes.
- **Riesgo si se toca:** Alto: toca la autenticación y los datos existentes. Requiere decisión humana.
- **Auto-resoluble:** no

### BL-0006 · [seguridad] /learn redirige a cualquier dirección que venga en la URL
- **Prioridad:** P1
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/index.js:72`
- **Descripción:** `return res.redirect(req.query.url)` (línea 72) redirige sin validar el destino: sirve para phishing desde el dominio del sitio.
- **Hermanos revisados:** Las otras llamadas a `res.redirect` en `app/routes/session.js` usan rutas fijas.
- **Criterios de aceptación:** El destino se valida contra una lista de rutas o dominios permitidos; cualquier otro responde 400.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** no

### BL-0007 · [seguridad] El secreto de la cookie de sesión está escrito en el código versionado
- **Prioridad:** P1
- **Tamaño:** XS
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `config/env/all.js:8`
- **Descripción:** `cookieSecret: "session_cookie_secret_key_here"` (línea 8) está en un archivo versionado; con ese secreto se pueden falsificar cookies de sesión.
- **Hermanos revisados:** Mismo problema con `cryptoKey` en la línea 9 de ese archivo.
- **Criterios de aceptación:** El secreto sale de una variable de entorno; el valor del repositorio se rota; el arranque falla si falta.
- **Riesgo si se toca:** Bajo, pero invalida las sesiones activas.
- **Auto-resoluble:** no

### BL-0008 · [seguridad] Una clave de API de la configuración de desarrollo está versionada
- **Prioridad:** P1
- **Tamaño:** XS
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `config/env/development.js:6`
- **Descripción:** `zapApiKey` (valor omitido aquí) está escrita en un archivo versionado (línea 6).
- **Hermanos revisados:** Repetida en `config/env/test.js` (línea 6 según el inspector; no se abrió ese archivo).
- **Criterios de aceptación:** La clave sale del entorno y la versionada se revoca.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** no

### BL-0009 · [seguridad] /benefits solo exige sesión iniciada, no el rol de administración
- **Prioridad:** P1
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/index.js:55`
- **Descripción:** `app.get("/benefits", isLoggedIn, ...)` y el POST de la línea siguiente solo comprueban que haya sesión (líneas 55 y 56): cualquier usuario ve y modifica los beneficios de otros.
- **Hermanos revisados:** Mismo patrón en `/allocations/:userId`.
- **Criterios de aceptación:** Las dos rutas exigen el rol de administración (middleware de rol) y responden 403 a quien no lo tiene.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** no

### BL-0010 · [seguridad] updateBenefits modifica el usuario que venga en el cuerpo sin comprobar quién pide
- **Prioridad:** P1
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/data/benefits-dao.js:25`
- **Descripción:** `updateBenefits(userId, startDate, ...)` actualiza `_id: parseInt(userId)` (línea 25) con el id recibido, sin comprobar que quien pide tenga permiso sobre ese usuario.
- **Hermanos revisados:** Va junto con el item de `/benefits` (control de acceso en la ruta).
- **Criterios de aceptación:** La actualización solo procede si quien pide es administrador; una prueba con un usuario común recibe 403.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** no

### BL-0011 · [seguridad] Las plantillas se renderizan con autoescape desactivado
- **Prioridad:** P1
- **Tamaño:** M
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `server.js:137`
- **Descripción:** `autoescape: false` (línea 137) hace que Swig imprima todas las variables de las plantillas sin escapar: abre XSS reflejado y almacenado en todas las vistas que muestran datos del usuario.
- **Hermanos revisados:** Relacionado con el item de memos. No se revisó vista por vista.
- **Criterios de aceptación:** `autoescape: true`; las vistas que necesiten HTML lo marcan de forma explícita y acotada; una prueba con `<script>` en un campo se ve como texto.
- **Riesgo si se toca:** Medio: puede romper vistas que dependían del HTML sin escapar.
- **Auto-resoluble:** no

### BL-0012 · [seguridad] Los memos se muestran sin escapar
- **Prioridad:** P1
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/views/memos.html:31`
- **Descripción:** `{{ marked(doc.memo) }}` (línea 31) imprime el memo convertido de Markdown a HTML; con el autoescape desactivado, un memo con HTML activo llega al navegador de quien lo lea.
- **Hermanos revisados:** Depende del item de autoescape.
- **Criterios de aceptación:** El resultado de `marked` se sanea con una librería de sanitización actual antes de mostrarse.
- **Riesgo si se toca:** Medio.
- **Auto-resoluble:** no

### BL-0013 · [seguridad] La protección CSRF está comentada en el servidor
- **Prioridad:** P1
- **Tamaño:** M
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `server.js:7`
- **Descripción:** `// const csrf = require('csurf');` (línea 7): la protección está desactivada y los formularios de cambio no validan token.
- **Hermanos revisados:** Falta revisar cada formulario de las vistas.
- **Criterios de aceptación:** Los POST exigen un token por sesión; un POST sin token responde 403.
- **Riesgo si se toca:** Medio: toca todos los formularios.
- **Auto-resoluble:** no

### BL-0014 · [seguridad] SSN y datos bancarios se guardan sin cifrar
- **Prioridad:** P1
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/data/profile-dao.js:62`
- **Descripción:** `user.ssn = ssn;` (línea 62) guarda el dato en la base sin cifrar; lo mismo ocurre con los datos bancarios según el inspector. Una filtración de la base expone datos sensibles.
- **Hermanos revisados:** Sin migración de lo ya guardado.
- **Criterios de aceptación:** Esos campos se cifran antes de guardarse con una clave que no esté en el repositorio; hay un plan para los registros existentes.
- **Riesgo si se toca:** Alto: datos personales y migración. Requiere decisión humana.
- **Auto-resoluble:** no

### BL-0015 · [seguridad] La cookie de sesión no declara secure ni caducidad
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `server.js:78`
- **Descripción:** `app.use(session({ ... }))` (línea 78) configura `httpOnly` pero no `secure` ni `maxAge`, y usa el nombre por defecto de la cookie.
- **Hermanos revisados:** El servidor escucha por HTTP; `secure` solo sirve detrás de TLS.
- **Criterios de aceptación:** `secure: true` en producción, `maxAge` definido y nombre de cookie propio.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** no

### BL-0016 · [seguridad] Los encabezados de seguridad (helmet) están desactivados
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `server.js:10`
- **Descripción:** `// const helmet = require("helmet");` (línea 10): sin CSP, HSTS, frameguard ni nosniff.
- **Hermanos revisados:** Depende del despliegue detrás de TLS.
- **Criterios de aceptación:** Helmet activo con una política mínima; los encabezados aparecen en la respuesta.
- **Riesgo si se toca:** Bajo; la CSP puede romper scripts en línea.
- **Auto-resoluble:** no

### BL-0017 · [seguridad] El login responde distinto si el usuario no existe o si la contraseña es incorrecta
- **Prioridad:** P2
- **Tamaño:** XS
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/session.js:85`
- **Descripción:** El login devuelve mensajes distintos en las líneas 85 y 94 (`invalidUserNameErrorMessage` y `invalidPasswordErrorMessage`): permite saber qué nombres de usuario existen.
- **Hermanos revisados:** Sin otros mensajes de login.
- **Criterios de aceptación:** Ambos casos devuelven el mismo mensaje y el mismo código.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** sí

### BL-0018 · [seguridad] La política de contraseñas acepta de 1 a 20 caracteres cualesquiera
- **Prioridad:** P2
- **Tamaño:** XS
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/session.js:144`
- **Descripción:** `const PASS_RE = /^.{1,20}$/;` (línea 144) acepta casi cualquier cosa.
- **Hermanos revisados:** Se verificó solo esta línea.
- **Criterios de aceptación:** Mínimo de 8 caracteres y una prueba con contraseñas débiles.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** sí

### BL-0019 · [seguridad] Una expresión regular con retroceso catastrófico valida la ruta bancaria
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/profile.js:59`
- **Descripción:** `const regexPattern = /([0-9]+)+\#/;` (línea 59) tiene cuantificadores anidados: con una entrada larga de dígitos sin `#` el servidor se queda calculando.
- **Hermanos revisados:** Sin otras expresiones con cuantificadores anidados revisadas.
- **Criterios de aceptación:** La expresión se reescribe sin anidar (`/[0-9]+#/`) y una entrada de 50 dígitos responde en milisegundos.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** sí

### BL-0020 · [seguridad] El nombre de usuario se escribe sin limpiar en el log
- **Prioridad:** P2
- **Tamaño:** XS
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/session.js:64`
- **Descripción:** `console.log("Error: attempt to login with invalid user: ", userName)` (línea 64) escribe un dato del usuario tal cual: permite falsificar líneas del log con saltos de línea.
- **Hermanos revisados:** Sin otros logs revisados.
- **Criterios de aceptación:** El valor se escapa o se serializa (JSON) antes de escribirse.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** sí

### BL-0021 · [seguridad] La configuración completa, con secretos, se imprime al arrancar
- **Prioridad:** P2
- **Tamaño:** XS
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `config/config.js:13`
- **Descripción:** `console.log(util.inspect(config, false, null))` (línea 13) imprime la configuración completa en cada arranque, secretos incluidos.
- **Hermanos revisados:** Relacionado con los items de secretos versionados.
- **Criterios de aceptación:** El arranque no imprime la configuración; si hace falta, imprime solo claves no sensibles.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** sí

### BL-0022 · [seguridad] El cierre de sesión se hace con GET
- **Prioridad:** P2
- **Tamaño:** XS
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/routes/index.js:41`
- **Descripción:** `app.get("/logout", ...)` (línea 41): cualquier página puede cerrar la sesión de quien la visite con una imagen o un enlace.
- **Hermanos revisados:** Sin otras acciones con GET que cambien estado revisadas.
- **Criterios de aceptación:** El cierre de sesión es un POST con token.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** sí

### BL-0023 · [seguridad] La plantilla de error imprime el objeto de error completo
- **Prioridad:** P3
- **Tamaño:** XS
- **Área:** seguridad
- **Fuente:** inspector-seguridad · 2026-10-25
- **Estado:** nuevo
- **Archivos:** `app/views/error-template.html:11`
- **Descripción:** `{{error}}` (línea 11) imprime el error en la página, sin escapar si el autoescape sigue desactivado.
- **Hermanos revisados:** Depende del item de autoescape.
- **Criterios de aceptación:** La página de error muestra un mensaje genérico; el detalle va al log del servidor.
- **Riesgo si se toca:** Bajo.
- **Auto-resoluble:** sí
