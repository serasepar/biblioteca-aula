# Reto: despliega la Biblioteca del aula

Un equipo de desarrollo te entrega esta aplicación web. Funciona en su ordenador con `python app.py`. Tu trabajo es **dockerizarla y desplegarla en AWS** con los datos en PostgreSQL.

Esta guía no tiene soluciones: tiene **preguntas**. Si sabes responderlas, sabrás qué hacer. Cuando no sepas, busca en la documentación oficial (Docker Hub, docs.docker.com, docs de Flask y de AWS) o vuelve a lo que hiciste con el Muro de notas.

---

## Antes de empezar

**Dónde trabajas:** en los PC del aula no hay Docker, así que todo lo de Docker se ejecuta en **tu EC2 del laboratorio**, desde la fase 2.
- El código y los ficheros Docker los editas en el IDE de tu PC y los llevas a la EC2 con `git push` (en el PC) y `git pull` (en la EC2), como en el Muro.
- Para ver la app en el navegador, usa un túnel SSH hacia el puerto que publiques en la EC2.

- Trabaja en tu repositorio (creado desde la plantilla) y haz un commit al terminar cada fase.
- El fichero `.env` **nunca** se sube a GitHub. Compruébalo con `git status` antes de cada commit.
- No hace falta cambiar la lógica de la aplicación. Si crees que necesitas tocar `app.py`, piénsalo dos veces: casi siempre hay otra forma.

**Cuando algo falle**, antes de preguntar, recorre estas cuatro preguntas en orden:
1. ¿La imagen **se construye**? (mira la salida de `build`)
2. ¿El contenedor **sigue en marcha**? (mira `ps`: ¿`Up` o `Exited`?)
3. ¿Qué dicen los **logs**? (`logs` del servicio que falla)
4. Si es la base de datos: ¿**no llega** (timeout, red) o **llega y la rechaza** (credenciales, base de datos que no existe)?

---

## Fase 1 · Conoce la aplicación

Lee el `README.md` y `app.py` antes de escribir nada de Docker. No puedes desplegar algo que no sabes cómo funciona.

Responde a estas preguntas:

| Pregunta | Respuesta |
|---|---|
| ¿Qué versión de Python necesita? | |
| ¿Dónde están sus dependencias? | |
| ¿Qué variables de entorno lee? ¿Cuáles son obligatorias? | |
| ¿En qué puerto escucha? ¿En qué **dirección**? | |
| ¿Qué base de datos usa? ¿Quién crea la tabla, y cuándo? | |
| ¿Qué ficheros de la carpeta **no** hacen falta para ejecutarla? | |

<details><summary>Pista</summary>

La dirección en la que escucha no aparece en el README. Busca la línea `app.run(...)` y consulta en la documentación de Flask qué valor usa si no se indica `host`.
</details>

**Hecho cuando** sabes responder a todo y explicar qué pasa si falta `DATABASE_URL`.

---

## Fase 2 · Una base de datos para probar

La app no funciona sin PostgreSQL. Tu primer contenedor será la base de datos.

1. ¿Qué imagen oficial vas a usar? ¿Qué versión (etiqueta) y por qué no `latest`?
2. La imagen oficial necesita algunas variables de entorno para crear el usuario y la base de datos. ¿Cuáles? (búscalo en su página de Docker Hub)
3. ¿Dónde guarda PostgreSQL sus datos dentro del contenedor? ¿Qué pasa con ellos si borras el contenedor?
4. ¿Necesitas publicar el puerto 5432? ¿Para quién?
5. ¿Qué `DATABASE_URL` usaría la app para conectarse a este PostgreSQL desde fuera del contenedor?

**Hecho cuando:**
- El contenedor de PostgreSQL está en marcha.
- Puedes entrar en él y listar las bases de datos (pista: `psql` está **dentro** del contenedor).
- Lo borras, lo vuelves a crear y los datos siguen ahí.

---

## Fase 3 · La imagen de la aplicación

Ahora escribe el `Dockerfile` de la app.

1. ¿Qué imagen base eliges? Compara `python:3.12`, `python:3.12-slim` y `python:3.12-alpine`: ¿qué cambia? ¿Cuánto ocupa tu imagen final?
2. ¿En qué orden copias los ficheros para que, si solo cambias `app.py`, Docker **no** vuelva a instalar las dependencias?
3. ¿Qué ficheros y carpetas de tu proyecto **no** deben entrar en la imagen? ¿Por qué es especialmente grave que entre `.env`?
4. ¿Qué comando arranca la aplicación cuando se inicia el contenedor?

<details><summary>Pista</summary>

Recuerda las capas del Dockerfile del Muro: cada instrucción es una capa, y Docker reutiliza las capas hasta la primera que cambia.
</details>

**Hecho cuando** `docker build` termina sin errores y has comprobado (entrando en un contenedor de esa imagen) que dentro no están `.env` ni `.venv`.

---

## Fase 4 · Que responda

Arranca un contenedor con tu imagen, publicando el puerto. Lo más probable es que **no funcione a la primera**. Esta es la fase más importante: no la saltes con una solución copiada.

1. El contenedor está en marcha, pero el navegador no conecta. ¿En qué dirección está escuchando la app **dentro** del contenedor? ¿Por qué eso impide que llegue tu navegador?
2. Mira los logs al arrancar. ¿Qué advertencia da Flask? ¿Qué significa?
3. ¿Qué es un servidor WSGI de producción? Elige uno. ¿Dónde lo declaras como dependencia?
4. La app tiene `debug=True`. ¿Por qué es peligroso en producción? ¿Sigue activo con tu solución?
5. Ahora la app arranca, pero no llega a la base de datos con `localhost`. ¿Qué es `localhost` dentro de un contenedor?

<details><summary>Pista para la pregunta 1</summary>

Un servidor que escucha en `127.0.0.1` solo acepta conexiones que nacen en su misma máquina. ¿Cuál es "su misma máquina" cuando está dentro de un contenedor?
</details>

<details><summary>Pista para la pregunta 3</summary>

Un servidor de producción puede indicar él mismo en qué dirección y puerto escuchar. Quizá así no necesites tocar `app.py`.
</details>

**Hecho cuando** ves la lista de libros en el navegador, servida desde el contenedor, y el pie de página dice "Servido por:" seguido de algo que **no** es el nombre de tu PC. ¿Qué es ese nombre?

---

## Fase 5 · Todo junto con Compose

Dos contenedores lanzados a mano con comandos largos no es un despliegue. Escribe un `docker-compose.yml`.

1. ¿Cuántos servicios hay? ¿Cuál publica puertos hacia fuera y cuál no necesita hacerlo?
2. Dentro de la red de Compose, ¿qué nombre de host usa la app para llegar a PostgreSQL?
3. ¿Dónde pones las contraseñas para que no queden escritas en el `docker-compose.yml`?
4. Si la app arranca antes de que PostgreSQL esté listo, ¿qué ocurre? ¿Cómo haces que espere?
5. ¿Por qué no meter la app y PostgreSQL en el **mismo** contenedor?

**Hecho cuando:**
- Desde cero (sin contenedores ni volúmenes), un solo comando levanta todo y la app funciona.
- Creas libros, bajas todo, lo vuelves a levantar, y los libros siguen ahí.
- En el `docker-compose.yml` no aparece ninguna contraseña.

---

## Fase 6 · A AWS

Ahora la app vive en una EC2 y los datos en RDS.

1. ¿Qué recursos de AWS necesitas? Dibuja el camino: tu navegador, la EC2, el contenedor de la app y RDS, con el **puerto** de cada flecha.
2. ¿Qué regla de entrada necesita el grupo de seguridad de RDS? ¿Desde dónde? ¿Por qué no desde `0.0.0.0/0`?
3. En AWS, ¿sigues necesitando el contenedor de PostgreSQL? ¿Usarás el mismo `docker-compose.yml` o uno distinto? ¿Qué cambia?
4. ¿Qué tienes que cambiar en la **imagen** de la app para que use RDS en vez de tu PostgreSQL? (piénsalo bien antes de responder)
5. ¿Cómo llega tu navegador a la app? ¿Abres un puerto en el grupo de seguridad de la EC2 o usas un túnel SSH? Ventajas e inconvenientes.
6. ¿Cómo compruebas, desde la EC2 y **antes** de arrancar la app, que RDS es alcanzable y las credenciales son correctas?

**Hecho cuando:**
- La app funciona en tu navegador servida desde la EC2.
- Los libros se guardan en RDS: reconstruyes el contenedor de la app y siguen ahí.
- En la EC2 no hay ningún contenedor de PostgreSQL.

---

## Fase 7 · Actualizar

Una app desplegada no está terminada: cambia.

Haz un cambio visible en la app (por ejemplo, el título de la página o un texto del pie) y llévalo hasta AWS.

1. ¿Cuál es la secuencia completa, desde que guardas el fichero en el IDE hasta que el cambio se ve en el navegador?
2. ¿Qué comando de Docker usas en la EC2? ¿Qué pasaría si olvidas reconstruir?
3. Después de actualizar, ¿qué contenedores son nuevos y cuáles no? ¿Cómo lo compruebas?
4. ¿Qué cambia en el pie "Servido por:"? ¿Por qué?

**Hecho cuando** el cambio se ve en AWS y los libros siguen ahí.
