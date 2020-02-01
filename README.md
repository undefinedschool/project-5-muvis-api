# Project 5: Muvis API

Vamos a utilizar la [API de The Movie DB](https://www.themoviedb.org/faq/api) para obtener la información de las primeras 100 [películas mejor puntuadas](https://developers.themoviedb.org/3/movies/get-top-rated-movies).

**Nota:** vamos a necesitar obtener una _API KEY_ para poder realizar los requests, como se indica en el [FAQ](https://www.themoviedb.org/faq/api). Esta _API KEY_ va a estar definida en el `.env`, no en el código.

## Contenido

- [Info sobre las películas](https://github.com/undefinedschool/project-5-muvis-api#info-sobre-las-pel%C3%ADculas)
- [Detalles de implementación](https://github.com/undefinedschool/project-5-muvis-api#detalles-de-implementaci%C3%B3n)
- [Base de Datos](https://github.com/undefinedschool/project-5-muvis-api#base-de-datos)
- [API Endpoints](https://github.com/undefinedschool/project-5-muvis-api/blob/master/README.md#api-endpoints)
  - [Query Strings](https://github.com/undefinedschool/project-5-muvis-api/blob/master/README.md#query-strings)
  - [Validaciones](https://github.com/undefinedschool/project-5-muvis-api/blob/master/README.md#validaciones)
- [Middleware a utilizar](https://github.com/undefinedschool/project-5-muvis-api/blob/master/README.md#middleware-a-utilizar)
- [`development` vs `production` mode](https://github.com/undefinedschool/project-5-muvis-api/blob/master/README.md#development-vs-production-mode)
  - [Uso de `morgan`](https://github.com/undefinedschool/project-5-muvis-api/blob/master/README.md#uso-de-morgan)
- [Hosting](https://github.com/undefinedschool/project-5-muvis-api/blob/master/README.md#hosting)
- [Extra: buenas prácticas de performance y seguridad para aplicaciones Express](https://github.com/undefinedschool/project-5-muvis-api/blob/master/README.md#extra-buenas-pr%C3%A1cticas-de-performance-y-seguridad-para-aplicaciones-express)

---

## Info sobre las películas

El formato con el que guardaremos las películas en la base de datos será el siguiente:

```JSON
{
  "id": 105,
  "title": "Back to the Future",
  "overview": "Eighties teenager Marty McFly is accidentally sent back in time to 1955, inadvertently disrupting his parents' first meeting and attracting his mother's romantic interest. Marty must repair the damage to history by rekindling his parents' romance and - with the help of his eccentric inventor friend Doc Brown - return to 1985.",
  "date": "July 3, 1985",
  "genres": ["Adventure", "Comedy", "Science Fiction", "Family"],
  "poster": "https://image.tmdb.org/t/p/w1280/pTpxQB1N0waaSc3OSn0e9oc8kx9.jpg",
  "rate": 8.32
}
```

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

## Detalles de implementación

- Utilizar `express-generator` para generar el scaffolding del proyecto, ejemplo `npx express-generator --git --no-view muvis-api`
- Usar [`got`](https://github.com/sindresorhus/got) para realizar los requests.
- Utilizar variables de entorno definidas en un archivo `.env` para valores como el puerto y hostname del server o _API KEYs_ necesarias
- Para manipular las fechas, se recomienda utilizar [`date-fns`](https://date-fns.org/) (o hacerlo a mano).
- Utilizar el `Router` de `Express` para definir la lógica de routing en un módulo aparte, y setear `/api/muvis` como prefijo de todas las rutas. Utilizar el método [`route()`](http://expressjs.com/en/4x/api.html#router.route), para definir las rutas de una forma más declarativa. 
- Utilizar `nodemon` para desarrollar (**sólo en modo desarrollo**).
- En el caso de que una película no cuente con la imagen correspondiente (poster), utilizar un [placeholder](https://placeholder.com/)
- En caso de necesitar _debuggear_ la aplicación, utilizar [esta guía](https://itnext.io/the-absolute-easiest-way-to-debug-node-js-with-vscode-2e02ef5b1bad).

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

## Base de Datos

- Para guardar las películas obtenidas, vamos a utilizar [`low-db`](https://github.com/typicode/lowdb) como base de datos. Utilizarlo un módulo aparte (importarlo donde sea necesario) y **definirle una API para interactuar con el mismo a través de diferentes métodos**.
- Al iniciar la aplicación, en el caso de que la base de datos (ver ítem anterior) se encuentre vacía, debemos realizar el request correspondiente para obtener las primeras 100 películas de las mejor puntuadas para realizar el _seeding_ inicial.
- Antes de guardar una nueva película en la db, vamos a validarla con un _schema_, utilizando [`ajv`](https://github.com/epoberezkin/ajv). Por ejemplo, si tenemos el método `add` para agregar películas, podríamos hacer algo como

```js
add(movie) {
  if (valid(movie)) {
    db.get('muvis').push(movie);
  } else {
    console.error(errors);
  }
}
```

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

## API Endpoints

- `GET /api/muvis`: retorna la lista de películas, en formato `JSON`.
- `GET /api/muvis/:id`: retorna la película con el `id` correspondiente, en formato `JSON`. En el caso de que no exista, generar el error `"404 - The movie with the id {ID} was not found"` (donde ID es el parámetro utilizado) con `status code` 404 y pasarle el objeto `err` a `next`.
- `POST /api/muvis`: agrega una nueva película, con la [info especificada más arriba](https://github.com/undefinedschool/project-5-muvis-api#info-sobre-las-pel%C3%ADculas). Validar el `body` del request como se indica más abajo. Si es inválido, generar el error `"400 - Bad Request"` y pasarle el objeto `err` a `next`, sino, agregar la película correspondiente y retornar la info de la misma en el `response`. El `id` con el que se guarde esta película debe ser mayor al máximo actual.
- `PUT /api/muvis/:id`: actualiza la info de una película (el `id` no puede editarse). Para esto, primero se debe buscarla por el `id` y si no existe, debe agregar la nueva película, con el `id` correspondiente (mayor al máximo actual).
- `DELETE /api/muvis/:id`: elimina la película con el `id` correspondiente. Para esto, primero debe buscarla por el `id`, si no existe, generar el error `"404 - The movie with the id {ID} was not found"` (donde ID es el parámetro utilizado), y un `status code` 404 y pasarle el objeto `err` a `next`.
- `GET /api/muvis/genres`: retorna la lista de géneros (sin repetir) ordenados de forma ascendente, correspondientes a las películas que tengamos en la base de datos
- `GET /api/muvis/years`: retorna la lista de años (sin repetir) ordenados de forma ascendente, correspondientes a las películas que tengamos en la base de datos
- `GET /api/muvis/rates`: retorna la lista de puntajes (sin repetir) ordenados de forma ascendente, correspondientes a las películas que tengamos en la base de datos

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

### Query strings

- Si se utiliza el query string `?year=` con `GET /api/muvis`, debe retornarse la lista de películas correspondientes a ese año, en formato `JSON`.
- Si se utiliza el query string `?genre=` con `GET /api/muvis`, debe retornarse la lista de películas correspondientes a ese género, en formato `JSON`.
- Si se utilizan los query strings `?sortBy=title`, `?sortBy=year` ó `?sortBy=rate` con `GET /api/muvis`, debe retornarse la lista de películas ordenada por año ó nombre de forma ascendente, respectivamente.

En todos los casos, si no hay películas para mostrar, debe retornarse el array vacío `[]` (siempre como `JSON`).

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

## Validaciones

Utilizar el middleware [`express-validator`](https://express-validator.github.io/) para realizar las siguientes validaciones sobre el input (`body` del request)
  
- `title`: debe existir y tener al menos 2 caracteres, sino generar el error `movie title is required and should have minimum 2 characters.`, con `status code` 400 y pasarle el objeto `err` a `next`.
- `date`: debe ser una fecha con el formato `yyyy-mm-dd`, sino generar el error `movie date is required and should be in yyyy-mm-dd format.`, con `status code` 400 y pasarle el objeto `err` a `next`.
- `overview`: debe existir y tener al menos 50 caracteres, sino generar el error `movie description is required and should have minimum 50 characters.`, con `status code` 400 y pasarle el objeto `err` a `next`.
- `rate`: debe existir y ser un número entre 1 y 10, sino generar el error `movie rate is required and should be a number between 1 and 10.`, con `status code` 400 y pasarle el objeto `err` a `next`.
- `genres`: debe existir y ser un array de strings, sino generar el error `movie genre is required.`, con `status code` 400 y pasarle el objeto `err` a `next`.

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

## Middleware a utilizar

- `Router` de `Express`
- Custom Error-handling middleware (ver detalles en [helpful-express-middleware](https://www.rithmschool.com/courses/node-express-fundamentals/helpful-express-middleware))
- [`express-validator`](https://express-validator.github.io/), para realizar las validaciones correspondientes
- [`morgan`](https://www.npmjs.com/package/morgan), para loguear en la terminal todos los _requests_ y _responses_ generados (ver detalles en [Uso de `morgan`](https://github.com/undefinedschool/project-5-muvis-api#uso-de-morgan))
- [`helmet.js`](https://helmetjs.github.io/)
  - Ver [Use Helmet](http://expressjs.com/en/advanced/best-practice-security.html#use-helmet)

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

## `development` vs `production` mode

Definir los scripts `dev` y `start` en el `package.json` para poder correr nuestra aplicación en diferentes modos

- **modo desarrollo**: 
  - `npm run dev` 
  - corre `nodemon` y [`morgan`](https://www.npmjs.com/package/morgan) en modo `dev` (ver detalles abajo)
- **modo producción**: 
  - [Setear `NODE_ENV` en modo producción](http://expressjs.com/en/advanced/best-practice-performance.html#set-node_env-to-production), usando [`cross-env`](https://www.npmjs.com/package/cross-env)
  - `npm run start` (corre el script `pm2 start app.js`, usando [`pm2`](https://pm2.keymetrics.io/))
    - Ver [Ensure your app automatically restarts](http://expressjs.com/en/advanced/best-practice-performance.html#ensure-your-app-automatically-restarts)
  - Habilitar la [compresión `GZIP`](https://alligator.io/nodejs/compression/) para todos los requests
  - [`morgan`](https://www.npmjs.com/package/morgan) en modo `common` (ver detalles abajo)

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

### Uso de `morgan`

```js
if (process.env.NODE_ENV === 'production') {
  app.use(morgan('common', {
    // log 400s and 500s only
    skip: (req, res) => res.statusCode < 400, 
    stream: `${__dirname}/../morgan.log`
  }));
} else {
  app.use(morgan('dev'));
}
```

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

## Hosting

Hostear la API con [`now`](http://now.sh/)

Para setear las variables de entorno, ver 
  - [Using Environment Variables and Secrets](https://zeit.co/docs/v2/build-step#using-environment-variables-and-secrets)   
  - [Secrets and Environment Variables in Next.js and Now](https://www.youtube.com/watch?v=pRbQcy9f5ew)

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)

## Extra: buenas prácticas de performance y seguridad para aplicaciones Express

- Ver [Production best practices: performance and reliability](http://expressjs.com/en/advanced/best-practice-performance.html)
- Ver [Production Best Practices: Security](http://expressjs.com/en/advanced/best-practice-security.html)

[↑ Ir al inicio](https://github.com/undefinedschool/project-5-muvis-api#project-5-muvis-api)
