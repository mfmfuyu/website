# RPC

The RPC feature allows sharing the API specifications between the server and the client.

You can export the types of input type specified by the Validator and the output type emitted by `jsonT()`. And Hono Client will able to import it.

## Server

All you need to do on the server side is to write a validator, change it from `json()` to `jsonT()`, and prepare a variable `route`.

```ts
const route = app.post(
  '/posts',
  zValidator(
    'form',
    z.object({
      title: z.string(),
      body: z.string(),
    })
  ),
  (c) => {
    // ...
    return c.jsonT(
      {
        ok: true,
        message: 'Created!',
      },
      201
    )
  }
)
```

Then, export the type to share the API spec with the Client.

```ts
export type AppType = typeof route
```

## Client

On the Client side, import `hc` and `AppType` first.

```ts
import { AppType } from '.'
import { hc } from 'hono/client'
```

`hc` is a function to create a client. Pass `AppType` as Generics and specify the server URL as an argument.

```ts
const client = hc<AppType>('http://localhost:8787/')
```

Call `client.{path}.{method}` and pass the data you wish to send to the server as an argument.

```ts
const res = await client.posts.$post({
  form: {
    title: 'Hello',
    body: 'Hono is a cool project',
  },
})
```

The `res` is compatible with "fetch" Response. You can retrieve data from the server with `res.json()`.

```ts
if (res.ok) {
  const data = await res.json()
  console.log(data.message)
}
```

## Path parameters

You can also handle routes that include path parameters.

```ts
const route = app.get(
  '/posts/:id',
  zValidator(
    'query',
    z.object({
      page: z.string().optional(),
    })
  ),
  (c) => {
    // ...
    return c.jsonT({
      title: 'Night',
      body: 'Time to sleep',
    })
  }
)
```

Specify the string you want to include in the path with `param`.

```ts
const res = await client.posts[':id'].$get({
  param: {
    id: '123',
  },
  query: {},
})
```