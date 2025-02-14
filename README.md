## StrapIO

module for working with socket.io with predefined rules. StrapIO will look at Role permission on each action.
StrapIO is looking for all roles which have access to the given contenttype and action type.

UPDATE v2:
You need to subscribe first before you receive any data. `socket.emit('subscribe', 'article')` article is the content-type.

## Installation

```bash
npm i strapio
```

`config/functions/bootstrap.js`

```js
process.nextTick(() => {
  strapi.StrapIO = (require("strapio"))(strapi);
});
```

### Configuration socket.io

```js
process.nextTick(() => {
  strapi.StrapIO = (require("strapio"))(strapi, {
    path: "/other/path/",
    cors: { origin: "*", methods: ["GET", "POST"] },
  });
});
```

## Usage

### server

`api/<content-type>/controllers/<content-type>.js`

```js
module.exports = {
  async create(ctx) {
    let entity;
    if (ctx.is("multipart")) {
      const { data, files } = parseMultipartData(ctx);
      entity = await strapi.services.CONTENTTYPE.create(data, { files });
    } else {
      entity = await strapi.services.CONTENTTYPE.create(ctx.request.body);
    }
    strapi.StrapIO.emit(this, "create", entity);

    // or send custom event
    strapi.StrapIO.emitRaw("myroom", "myevent", entity);

    return sanitizeEntity(entity, { model: strapi.models.CONTENTTYPE });
  },

  async update(ctx) {
    const { id } = ctx.params;

    let entity;
    if (ctx.is("multipart")) {
      const { data, files } = parseMultipartData(ctx);
      entity = await strapi.services.CONTENTTYPE.update({ id }, data, {
        files,
      });
    } else {
      entity = await strapi.services.CONTENTTYPE.update(
        { id },
        ctx.request.body
      );
    }

    strapi.StrapIO.emit(this, "update", entity);

    return sanitizeEntity(entity, { model: strapi.models.CONTENTTYPE });
  },
};
```

### Client

```js
const io = require("socket.io-client");

// Handshake required, token will be verified against strapi
const socket = io.connect(API_URL, {
  query: { token },
});

socket.emit("subscribe", "article"); // article is the room which the client joins

socket.on("find", (data) => {
  console.log("article:", data);
  //do something
});
socket.on("update", (data) => {
  // do something
});
```

### Client, Web

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.2.0/socket.io.js"></script>
<script>
  // Handshake required, token will be verified against strapi
  (function () {
    const socket = io("http://localhost:1337/", {
      path: "/sockettest/",
      query: {
        token,
      },
    });

    socket.emit("subscribe", "article"); // article is the room which the client joins
    socket.emit("subscribe", "myroom"); // custom room

    socket.on("find", (data) => {
      //do something
    });
    socket.on("update", (data) => {
      // do something
    });

    socket.on("myevent", (data) => {});
  })();
</script>
```
## Full example project

- https://github.com/larsonnn/strapio-example-project

## debugging

- `DEBUG=strapio npm run develop`
- `DEBUG=* npm run develop`

## Test

Currently tested with:

```js
{
    "strapi": "3.6.3",
    "strapi-plugin-users-permissions": "3.6.3"
}
```

## Plugin for strapi

You can install strapio with a plugin `npm i strapi-plugin-socket-io`.

## Contribute

just do it over github or chat with me [@Discord](https://discord.gg/8gCdxzs)
