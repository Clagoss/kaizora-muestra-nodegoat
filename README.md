# Una muestra real: el inspector de seguridad de Kaizora sobre OWASP NodeGoat

Este repositorio publica **solo los resultados** de una corrida del inspector de seguridad de
[Kaizora](https://carloslagos.cl/kaizora/) sobre [OWASP NodeGoat](https://github.com/OWASP/NodeGoat), una
aplicación pública hecha para ser vulnerable. No contiene el producto Kaizora ni el código de NodeGoat.

> **In English:** raw results of one run of Kaizora's security inspector (a set of Claude Code agents) on OWASP
> NodeGoat: the HTML report, the 23-item backlog and the ground truth written before the run. 4 of 5 reference
> defects found, 31 findings, 25 checked by hand. One run on an easy, well-known repo; not an accuracy rate.
> The report and the agents are written in Spanish.

- `informe/informe.html`: el informe de mejora continua tal como lo genera Kaizora (HTML autocontenido; las fuentes están en `informe/fuentes/`).
- `BACKLOG.md`: los 23 ítems que salieron, cada uno con descripción, evidencia, criterios de aceptación y riesgo.
- `evaluacion/`: el caso (repositorio, commit exacto y la verdad de referencia escrita **antes** de ver el resultado) y los hallazgos crudos del inspector.

## Las cifras, crudas

Una corrida, un inspector, sobre el commit `c5cb68a7084e4ae7dcc60e6a98768720a81841e8` de NodeGoat (Apache-2.0).

| Medida | Resultado |
|---|---|
| Defectos de la verdad de referencia encontrados | 4 de 5 (el quinto es la misma consulta `$where` en otra línea) |
| Hallazgos totales | 31 |
| Fuera de la verdad y verificados a mano (la línea existe y contiene lo descrito) | 21 |
| Sin confirmar | 6 (no entraron al backlog) |
| Precisión mínima (con lo verificado) | 81 % |

## Qué NO demuestra

- **No es una tasa de acierto.** Es una sola corrida sobre un solo repositorio.
- NodeGoat es famoso y probablemente está en el entrenamiento del modelo; además trae comentarios que marcan los defectos. Es el caso fácil.
- «Verificado» significa que la línea existe y contiene lo que el inspector describe. No se probó cada explotación.
- Un backlog de 23 ítems sale de 31 hallazgos: solo se anotaron los 25 verificados, agrupando los que apuntan al mismo defecto.
- El informe, el backlog y los mensajes están en español.

## Otra muestra y el producto

- **Segunda muestra, sobre un repositorio menos conocido:** el inspector de código sobre Fliks, una app Python de 480 líneas. Calzó 2 de 7 defectos anotados antes (5 de 7 con su archivo citado), 0 de 8 hallazgos falsos (falso = la línea no contiene lo que el inspector describe) y 3 defectos que no estaban anotados (uno reproducido ejecutando código; otro depende de configuración fuera del repo). Los 8 hallazgos, tal cual: <https://carloslagos.cl/kaizora/muestra/fliks-hallazgos.md>
- **Kaizora** es un kit de agentes para Claude Code que inspecciona tu repositorio con regularidad, anota lo que encuentra en un backlog priorizado y te lo reporta; por defecto solo lee e informa, y no cambia tu código sin tu OK; el modo autónomo corre en tu CI y solo lo activas tú. Está en beta, en español, por US$ 20: [ver la ficha en Gumroad](https://carloslagos.cl/ir/kaizora/?lugar=repo-muestra&desde=github).

## Atribución y licencia

NodeGoat es de OWASP, bajo licencia Apache-2.0 (<https://github.com/OWASP/NodeGoat>). Aquí solo hay referencias a archivos y líneas de ese proyecto, sin copiar su código. Kaizora no tiene afiliación con OWASP.
El contenido de este repositorio se publica bajo la licencia Apache-2.0 (ver `LICENSE`).

© 2026 Carlos Lagos.
