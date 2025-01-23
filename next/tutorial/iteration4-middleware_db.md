#

## summary dev -> production

[role-based-app]

thunder client -roles

- route handler HTTP msg + response
  extend the response w helpers

middleware export config (w matchers)

[Docs](https://nextjs.org/docs/app/building-your-application/routing/middleware)

- dedicated auth services || route handler
- build remote db
  [Host DB](https://planetscale.com/)

---

[deploy]

- route segment config
  'force-dynamic': Force dynamic rendering, which will result in routes being rendered for each user at request time. This option is equivalent to getServerSideProps() in the pages directory.

[Vercel](https://vercel.com/)

## Route Handlers

- install Thunder Client

Route Handlers allow you to create custom request handlers for a given route using the Web Request and Response APIs.

- in app create folder "api"
- in there create folder "tasks" with route.js file

The following HTTP methods are supported: GET, POST, PUT, PATCH, DELETE, HEAD, and OPTIONS. If an unsupported method is called, Next.js will return a 405 Method Not Allowed response.

In addition to supporting native Request and Response. Next.js extends them with NextRequest and NextResponse to provide convenient helpers for advanced use cases.

app/api/tasks/route.js

```js
// the following HTTP methods are supported: GET, POST, PUT, PATCH, DELETE, HEAD, and OPTIONS. If an unsupported method is called, Next.js will return a 405 Method Not Allowed response.

import { NextResponse } from 'next/server'
import db from '@/utils/db'

export const GET = async (request) => {
  const tasks = await db.task.findMany()
  return Response.json({ data: tasks })
  // return NextResponse.json({ data: tasks });
}

export const POST = async (request) => {
  const data = await request.json()
  const task = await db.task.create({
    data: {
      content: data.content,
    },
  })
  return NextResponse.json({ data: task })
}
```

## Middleware

[Docs](https://nextjs.org/docs/app/building-your-application/routing/middleware)

Middleware in Next.js is a piece of code that allows you to perform actions before a request is completed and modify the response accordingly.

- create middleware.js in the root
- by default will be invoked for every route in your project

```js
export function middleware(request) {
  return Response.json({ msg: 'hello there' })
}

export const config = {
  matcher: '/about',
}
```

```js
import { NextResponse } from 'next/server'

// This function can be marked `async` if using `await` inside
export function middleware(request) {
  return NextResponse.redirect(new URL('/', request.url))
}

// See "Matching Paths" below to learn more
export const config = {
  matcher: ['/about/:path*', '/tasks/:path*'],
}
```

## PlanetScale

[Host DB](https://planetscale.com/)

- set DATABASE_URL in .env

```prisma
generator client {
  provider = "prisma-client-js"
}
datasource db {
  provider = "mysql"
  url = env("DATABASE_URL")
  relationMode = "prisma"
}
```

```sh
npx prisma db push
```

### Local Build

### Setup App

package.json

```json
 "build": "npx prisma generate && next build",
```

- prisma-example

```js
import prisma from '@/utils/db'

const prismaHandlers = async () => {
  console.log('prisma example')
  // await prisma.task.create({
  //   data: {
  //     content: 'wake up',
  //   },
  // });
  return prisma.task.findMany()
}

const PrismaExample = async () => {
  const tasks = await prismaHandlers()
  if (tasks.length === 0) {
    return <h2 className='mt-8 font-medium text-lg'>No tasks to show...</h2>
  }

  return (
    <div>
      <h1 className='text-7xl'>PrismaExample</h1>
      {tasks.map((task) => {
        return (
          <h2 key={task.id} className='text-xl py-2'>
            😬 {task.content}
          </h2>
        )
      })}
    </div>
  )
}
export default PrismaExample
```

- clean out the DB

```sh
npm run build
```

```sh
npm start
```

## Force Dynamic

- add loading.js to tasks

tasks.js

```js
export const dynamic = 'force-dynamic'
```

## Deploy

[Vercel](https://vercel.com/)

- sign up for account
- create github repo
