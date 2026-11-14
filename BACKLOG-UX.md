# BACKLOG-UX — segunda área sobre el mismo banco

> **Mismo repositorio que `BACKLOG.md`, otro inspector.** El inspector de UX y UX writing de
> Kaizora corrió sobre el mismo commit `c5cb68a` de OWASP NodeGoat, el 2026-11-14, de forma
> headless (`claude -p`, sin terminal interactiva). Evalúa con las heurísticas de Nielsen Norman
> Group y la escala de severidad 0-4 de NN/g; el método de cada hallazgo se cita en su campo
> `Método`.
>
> **Por qué esta lista no tiene una cifra de precisión como `BACKLOG.md`.** La muestra de
> seguridad pudo decir «4 de 5 defectos de la verdad de referencia» porque NodeGoat documenta su
> OWASP Top 10 a propósito, con comentarios que marcan cada defecto. NodeGoat no tiene una lista
> de defectos de UX etiquetados de antemano: no hay una verdad de referencia contra la que medir
> una tasa de acierto. Lo que sí se puede mostrar, y se muestra abajo, es cómo se verificó cada
> hallazgo.
>
> **Tres niveles de verificación, no uno solo:**
> - **Independiente (2 de 13):** alguien distinto del inspector que escribió el hallazgo volvió a
>   abrir el archivo citado, por separado, después de que el hallazgo ya estaba escrito.
>   BL-0024 y BL-0027 (abajo).
> - **El propio inspector, al escribirlo (7 de 13):** el inspector abrió el archivo y comprobó la
>   cita en la misma pasada en que redactó el hallazgo (marcado «verificado» en su descripción).
>   BL-0025, BL-0026, BL-0028, BL-0030, BL-0033, BL-0034 y BL-0035.
> - **Verificación estándar del inspector (4 de 13):** la cita de archivo:línea existe, pero sin
>   una segunda pasada explícita sobre esa premisa. BL-0029, BL-0031, BL-0032 y BL-0036.
>
> Ningún nivel es una garantía de que el arreglo propuesto sea el correcto: los tres dicen que el
> código citado existe y hace lo que el hallazgo describe, nada más.

**Próximo ID libre tras esta lista:** BL-0037

## Items

### BL-0024 · [ux] Contributions: los campos del formulario vienen en 0 y enviar sin editar pone todo en 0 %
- **Prioridad:** P2
- **Tamaño:** XS
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** independiente
- **Archivos:** `app/views/contributions.html:67, :75, :83`
- **Método:** Heurísticas H5 y H6 (Nielsen) · severidad NN/g 2
- **Descripción:** los tres `<input>` (`preTax`, `roth`, `afterTax`) traen `value="0"` fijo. Quien cambia un solo campo y envía deja los otros dos en 0; no hay confirmación y el valor actual se sobrescribe. El texto de instrucciones no lo avisa.
- **Hermanos revisados:** `benefits.html:57` y `profile.html` sí precargan el valor actual; contributions es el único formulario con un valor fijo.
- **Criterios de aceptación:** los campos se precargan con el valor actual (`{{preTax}}`, `{{roth}}`, `{{afterTax}}`); los specs de Cypress de contributions siguen en verde.
- **Riesgo si se toca:** bajo
- **Auto-resoluble:** no (cambia el comportamiento de un formulario que escribe datos)

### BL-0025 · [ux] Pantallas de error y de éxito con contexto incompleto: valores en 0, enlace roto, campo vacío y menú sin nombre
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** el propio inspector, al escribirlo
- **Archivos:** `app/routes/contributions.js:50-55, :58-63`; `app/routes/profile.js:65-75`; `app/views/layout.html:56, :75`; `app/routes/memos.js:27-31`; `app/routes/allocations.js:25-29`; `app/routes/research.js:31-33`; `app/routes/benefits.js:19-25`
- **Método:** Heurísticas H1 y H9 (Nielsen) · severidad NN/g 2
- **Descripción:** una sola causa (un `res.render` sin las variables que esperan `layout.html` y la vista) con cuatro síntomas: (1) el error de contributions no pasa los tres campos y la columna actual muestra «0 %» aunque haya otro valor guardado; (2) el error de profile no pasa `userId` (el menú queda apuntando a `/allocations/`, que da 404) ni `website` (el campo sale vacío); (3) `layout.html:75` pinta `{{firstName}} {{lastName}}`, pero `/memos`, `/allocations`, `/research` y `/benefits` no los pasan y el menú de usuario sale sin nombre; (4) «Invalid contribution percentages» no dice qué campo falló ni qué se espera.
- **Hermanos revisados:** el éxito de profile sí pasa el documento completo; el punto 3 se resolvería en un solo lugar con `res.locals` o un middleware.
- **Criterios de aceptación:** todo render con layout recibe `userId`, `firstName` y `lastName` desde un solo lugar; el error de contributions conserva los valores actuales y dice qué campo es inválido; el error de profile conserva `website` y el enlace de Allocations funciona.
- **Riesgo si se toca:** bajo
- **Auto-resoluble:** no (toca varios manejadores)

### BL-0026 · [ux] Allocations: un filtro sin resultados muestra la página de error 500 en vez de un estado vacío
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** el propio inspector, al escribirlo
- **Archivos:** `app/data/allocations-dao.js:88`; `app/routes/allocations.js:24`; `app/views/allocations.html:32-49`
- **Método:** Heurísticas H9 y H1 (Nielsen) · severidad NN/g 2
- **Descripción:** `if (!allocations.length) return callback("ERROR: No allocations found for the user", null)` llega a `next(err)` y la persona ve la página sin estilo «Oops.. ERROR: No allocations found for the user» con status 500. Con los datos sembrados basta un threshold alto. La vista tiene el bucle pero ninguna rama «sin resultados».
- **Hermanos revisados:** `memos.html` y `benefits.html` tampoco tienen estado vacío (ver BL-0029).
- **Criterios de aceptación:** un filtro sin coincidencias devuelve 200 con un texto como «No allocations above X%. Try a lower threshold»; los demás errores siguen yendo a `next(err)`.
- **Riesgo si se toca:** medio (DAO vecino a código intencional de seguridad)
- **Auto-resoluble:** no

### BL-0027 · [ux] Profile: «Website» nunca se guarda y ningún campo se puede vaciar, pero se muestra «Profile updated successfully»
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** independiente
- **Archivos:** `app/routes/profile.js:42-50, :82-90`; `app/data/profile-dao.js:42-76`; `app/views/profile.html:68-72`
- **Método:** Heurísticas H1 y H5 (Nielsen) · severidad NN/g 2 (daña la confianza: el sistema dice haber guardado algo que no guardó)
- **Descripción:** el formulario ofrece «Website», pero el manejador no lo lee del body y el DAO no lo escribe. Además el DAO guarda cada campo solo `if (valor)`, así que vaciar un campo opcional no lo borra. En ambos casos aparece el éxito en verde con el dato viejo guardado.
- **Hermanos revisados:** el mismo `if (campo)` aplica a los demás campos del perfil.
- **Criterios de aceptación:** `website` se guarda o se quita del formulario; vaciar un campo opcional se refleja, o el mensaje de éxito aclara qué se guardó.
- **Riesgo si se toca:** alto (escritura de datos personales)
- **Auto-resoluble:** no (decisión del dueño)

### BL-0028 · [ux] Los POST responden con la vista renderizada, sin redirect: recargar reenvía el formulario
- **Prioridad:** P2
- **Tamaño:** M
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** el propio inspector, al escribirlo
- **Archivos:** `app/routes/memos.js:13-16`; `app/routes/session.js:239`; `app/routes/profile.js:100`; `app/routes/contributions.js:70`; `app/routes/benefits.js:51`
- **Método:** Heurísticas H3 y H5 (Nielsen) · severidad NN/g 2
- **Descripción:** cada escritura termina con `res.render` en la URL del POST. Recargar `/memos` tras publicar duplica el memo; tras registrarse la persona queda viendo el dashboard en `/signup` y, si recarga, recibe «User name already in use» sobre su propia cuenta; recargar tras guardar contributions o profile repite la escritura.
- **Hermanos revisados:** `/login` sí redirige. Los cuatro manejadores de escritura y el signup comparten la forma.
- **Criterios de aceptación:** tras un POST exitoso hay un redirect 303 a la página GET con el mensaje de éxito visible; recargar no repite la escritura.
- **Riesgo si se toca:** medio (el signup es autenticación)
- **Auto-resoluble:** no

### BL-0029 · [ux] Memos: «Send a memo» no dice que es público ni de quién es cada nota; sin estado vacío ni confirmación
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** estándar del inspector
- **Archivos:** `app/views/memos.html:9, :15-23, :28-34`; `app/data/memos-dao.js:26-35`; `app/routes/memos.js:19-33`
- **Método:** Heurísticas H2, H1 y H10 (Nielsen) · severidad NN/g 2
- **Descripción:** el título sugiere un mensaje dirigido, pero `getAllMemos` devuelve los memos de todos los usuarios: quien escribe no sabe quién lo va a ver. Cada memo se guarda con `timestamp`, pero la vista no muestra fecha ni autor. Sin memos la lista queda en blanco. No hay confirmación ni validación de memo vacío.
- **Hermanos revisados:** la lista sin estado vacío se repite en `allocations.html` (BL-0026) y `benefits.html`.
- **Criterios de aceptación:** el título o el texto de ayuda avisan que el memo es visible para todos; hay estado vacío con texto; un memo vacío no se publica.
- **Riesgo si se toca:** bajo (microcopy y estado vacío); mostrar el autor toca el dato
- **Auto-resoluble:** no (decisiones de producto)

### BL-0030 · [ux] Signup: al fallar la validación se pierden nombre y apellido; los mensajes no dicen la regla; un error a la vez
- **Prioridad:** P2
- **Tamaño:** XS
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** el propio inspector, al escribirlo
- **Archivos:** `app/routes/session.js:201-204, :159-185`; `app/views/signup.html:57, :80, :86`
- **Método:** Heurísticas H9 y H5 (Nielsen) · severidad NN/g 2
- **Descripción:** el objeto `errors` que vuelve a la vista trae `userName` y `email`, pero no `firstName` ni `lastName`: cualquier error obliga a reescribirlos. Los mensajes de error no dicen la regla (1 a 20 caracteres el usuario, 1 a 100 los nombres). La validación corta en el primer fallo, así que se corrige un campo por intento.
- **Hermanos revisados:** `login.html` repinta `userName` bien.
- **Criterios de aceptación:** `firstName` y `lastName` sobreviven al error; los mensajes dicen qué se espera.
- **Riesgo si se toca:** bajo
- **Auto-resoluble:** no (el manejador vive en la ruta de autenticación; lo decide el dueño del producto)

### BL-0031 · [ux] Un enlace profundo a una página con sesión pierde el destino tras el login
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** estándar del inspector
- **Archivos:** `app/routes/session.js:36-42, :116-117`; `app/routes/index.js:44-76`
- **Método:** Heurística H3 (Nielsen) · severidad NN/g 2
- **Descripción:** quien abre `/memos` o `/allocations/2` sin sesión termina en `/login` sin que se guarde el destino; tras entrar siempre va a `/dashboard` (o a `/benefits` si es admin). `/login` tampoco dice por qué se llegó ahí.
- **Hermanos revisados:** otros dos middlewares de sesión también redirigen sin destino.
- **Criterios de aceptación:** el destino original se conserva, validado como ruta relativa, y se vuelve a él tras el login; el login explica por qué se llegó ahí.
- **Riesgo si se toca:** alto (autenticación y redirecciones)
- **Auto-resoluble:** no (restricción dura: autenticación)

### BL-0032 · [ux] La página de error es un HTML sin estilo («Oops..»), no hay 404 propio y no ofrece salida
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** estándar del inspector
- **Archivos:** `app/views/error-template.html:5, :10-11`; `app/routes/error.js:9-12`; `app/routes/index.js:79-82`
- **Método:** Heurísticas H9, H3 y H4 (Nielsen) · severidad NN/g 2
- **Descripción:** el único texto es «Oops..» y el error en bruto, sin layout, sin estilos y sin enlace para volver. Las URL que no existen caen en el «Cannot GET» por defecto de Express: no hay manejador 404. Nada explica qué pasó ni qué hacer.
- **Hermanos revisados:** el detalle interno del error es una vulnerabilidad intencional de seguridad (ver `BACKLOG.md`); este item cubre lo que esa no: el copy, el estilo, la salida y el 404.
- **Criterios de aceptación:** vista de error con layout, mensaje en lenguaje simple, enlace de vuelta al dashboard y un 404 propio.
- **Riesgo si se toca:** medio
- **Auto-resoluble:** no (la plantilla es parte de una vulnerabilidad intencional)

### BL-0033 · [ux] Research: anuncia «the stock information you requested» aunque la consulta falle, y la respuesta no tiene navegación
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** el propio inspector, al escribirlo
- **Archivos:** `app/routes/research.js:16-28`; `app/views/research.html:17-19`
- **Método:** Heurísticas H1, H9 y H3 (Nielsen) · severidad NN/g 2
- **Descripción:** el encabezado «The following is the stock information you requested.» se escribe siempre (solo el `Content-Type` depende de si la respuesta fue 200), incluso si la petición falla o el destino responde 404 o 500. Un símbolo inexistente da ese encabezado seguido de nada o de un HTML ajeno. La respuesta no usa el layout: no hay menú ni forma de volver.
- **Hermanos revisados:** `/learn` redirige sin pantalla intermedia.
- **Criterios de aceptación:** si la consulta falla se muestra un mensaje de error dentro del layout; el éxito conserva su encabezado.
- **Riesgo si se toca:** medio
- **Auto-resoluble:** no (el manejador es la SSRF intencional de `BACKLOG.md`)

### BL-0034 · [ux] Login: ninguna pantalla dice con qué cuenta entrar; «Submit» es el botón de todo; «Logout» y «Log Out» para la misma acción
- **Prioridad:** P3
- **Tamaño:** XS
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** el propio inspector, al escribirlo (el detalle «Logout»/«Log Out»)
- **Archivos:** `app/views/login.html:49, :120-124`; `app/views/layout.html:10, :67, :82`
- **Método:** severidad NN/g 1-2
- **Descripción:** el desplegable de ayuda del login invita a «start hacking the application», pero las cuentas sembradas solo están en el README, y el camino real, «New user? Sign Up», es un enlace chico. El botón principal de siete pantallas distintas se llama «Submit», sin el verbo de la acción. Cerrar sesión se llama «Logout» en un lugar del layout y «Log Out» en otro.
- **Hermanos revisados:** un spec de Cypress del repo verifica el texto «Log Out»: unificarlo pide ajustar ese spec.
- **Criterios de aceptación:** el login enlaza a las instrucciones o le da más peso al signup; un solo nombre para cerrar sesión; botones con el verbo de la acción.
- **Riesgo si se toca:** bajo
- **Auto-resoluble:** no (mostrar credenciales es decisión del dueño; renombrar botones afecta specs existentes)

### BL-0035 · [ux] Dashboard: cifras fijas iguales para todos los usuarios y tres tarjetas con enlace vacío
- **Prioridad:** P3
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** el propio inspector, al escribirlo
- **Archivos:** `app/views/dashboard.html:12, :41-51, :45, :72, :92`
- **Método:** Heurísticas H2 y H8 (Nielsen) · severidad NN/g 1
- **Descripción:** varias cifras en dólares y porcentajes son texto fijo de la vista, presentadas como personales: cambiar los valores en `/contributions` nunca las mueve. Tres tarjetas tienen un enlace vacío (`href="#"`) con pie sin texto, que parece clicable.
- **Hermanos revisados:** el dashboard es la única vista con datos inventados; el resto lee datos reales.
- **Criterios de aceptación:** las cifras salen de datos reales o se marcan como «Sample data»; se quitan los enlaces vacíos.
- **Riesgo si se toca:** bajo
- **Auto-resoluble:** no (decisión de producto)

### BL-0036 · [ux] Tutorial: sin enlace desde la aplicación ni de vuelta a ella, «Exit» ambiguo, sin siguiente/anterior
- **Prioridad:** P2
- **Tamaño:** S
- **Área:** ux
- **Fuente:** kz-inspector-ux · 2026-11-14
- **Verificación:** estándar del inspector
- **Archivos:** `app/views/tutorial/layout.html:35, :67-70`; `app/routes/tutorial.js:8-13`; `app/views/layout.html:47-68`; `app/views/login.html:79`
- **Método:** Heurísticas H3, H4 y H6 (Nielsen) · severidad NN/g 2
- **Descripción:** el tutorial solo se enlaza desde el login, en una pestaña nueva; dentro de la aplicación no hay enlace de vuelta a él. La única salida del tutorial es «Exit» (un ícono de apagado), que lleva al login: ni cierra sesión ni vuelve al dashboard. Las páginas no tienen «Next/Previous».
- **Hermanos revisados:** un archivo de recorrido guiado (`redirects-steps.js`) tiene textos de relleno y se activa con un selector que no existe en ninguna vista: parece código muerto, fuera del área de este inspector.
- **Criterios de aceptación:** decisión del dueño sobre agregar un enlace al tutorial desde la aplicación, renombrar «Exit» y agregar navegación entre páginas.
- **Riesgo si se toca:** bajo
- **Auto-resoluble:** no (el tutorial es contenido de OWASP; depende de si se puede tocar)

## Qué NO demuestra esta lista

- **No es una tasa de acierto.** A diferencia de `BACKLOG.md` (seguridad), aquí no hay una verdad
  de referencia escrita antes de correr el inspector: NodeGoat no etiqueta sus problemas de UX.
- **Es una sola corrida, sobre un solo repositorio**, igual que las otras dos muestras.
- De los 13 hallazgos, **solo 2 pasaron una verificación hecha por un proceso distinto del que
  los escribió** (ver la tabla de niveles arriba). Los otros 11 son tan buenos como la disciplina
  del propio inspector al citar su fuente.
- Todo lo marcado `Auto-resoluble: no` implica una decisión de producto (qué dice un mensaje, qué
  flujo cambia): eso lo decide el dueño del proyecto, no el inspector.

## La corrida, en cifras

Una corrida, un inspector, headless (sin terminal interactiva), sobre el mismo commit
`c5cb68a` de NodeGoat que usa `BACKLOG.md`.

| Medida | Resultado |
|---|---|
| Hallazgos que entraron al backlog | 13 (0 duplicados: el backlog de UX partía vacío) |
| Verificación independiente | 2 de 13 |
| Verificación del propio inspector al escribirlos | 7 de 13 |
| Verificación estándar (sin segunda pasada) | 4 de 13 |
| Hipótesis descartadas por el inspector al abrir el archivo (comportamiento intencional del banco) | 4 |
| Costo medido (`cost-state` de la sesión) | US$ 2,77 |
| Duración de reloj | 5 min 25 s (sin espera humana: corrida headless) |
| Modelos | orquestador en Claude Fable 5.2 (US$ 2,36) · el inspector mismo en Claude Sonnet 5.5 (US$ 0,41, como declara su ficha) |
