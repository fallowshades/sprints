#

##

[client]

- hum state managment for this

```js
const TaskForm = () => {
  const [state, formAction] = useFormState(createTaskCustom, initialState)

  useEffect(() => {
    if (state.message === 'error') {
      toast.error('there was an error')
      return
    }
    if (state.message) {
      toast.success('task created....')
    }
  }, [state])

  return (
    <form action={formAction}>
```

[server-action]

- zod and revalidate

## Server Actions - Loading State, Response, Errors

- clone TaskForm - copy and rename TaskFormCustom
- make it client component
- import and replace in Tasks page
- clone createTask in actions - copy and rename createTaskCustom
- import and replace in TaskFormCustom component

## Server Actions - Loading State

TaskFormCustom

```js
'use client'

import { createTaskCustom } from '@/utils/actions'
import { useFormStatus } from 'react-dom'
// The useFormStatus Hook provides status information of the last form submission.
// useFormState is a Hook that allows you to update state based on the result of a form action.

const SubmitButton = () => {
  const { pending } = useFormStatus()

  return (
    <button
      type="submit"
      className="btn join-item btn-primary"
      disabled={pending}
    >
      {pending ? 'please wait... ' : 'create task'}
    </button>
  )
}

const TaskForm = () => {
  return (
    <form action={createTaskCustom}>
      <div className="join w-full">
        <input
          className="input input-bordered join-item w-full"
          placeholder="Type Here"
          type="text"
          name="content"
          required
        />
        <SubmitButton />
      </div>
    </form>
  )
}
export default TaskForm
```

```js
export const createTaskCustom = async (formData) => {
  await new Promise((resolve) => setTimeout(resolve, 2000))
  const content = formData.get('content')
  // some validation here

  await prisma.task.create({
    data: {
      content,
    },
  })
  // revalidate path
  revalidatePath('/tasks')
}
```

## Error Handling and Response To User

TaskFormCustom.jsx

```js
'use client'

import { createTaskCustom } from '@/utils/actions'
import { useFormStatus, useFormState } from 'react-dom'
// The useFormStatus Hook provides status information of the last form submission.
// useFormState is a Hook that allows you to update state based on the result of a form action.

const SubmitButton = () => {
  const { pending } = useFormStatus()

  return (
    <button
      type="submit"
      className="btn join-item btn-primary"
      disabled={pending}
    >
      {pending ? 'please wait... ' : 'create task'}
    </button>
  )
}

const initialState = {
  message: null,
}

const TaskForm = () => {
  const [state, formAction] = useFormState(createTaskCustom, initialState)

  return (
    <form action={formAction}>
      {state.message ? <p className="mb-2">{state.message}</p> : null}
      <div className="join w-full">
        <input
          className="input input-bordered join-item w-full"
          placeholder="Type Here"
          type="text"
          name="content"
          required
        />
        <SubmitButton />
      </div>
    </form>
  )
}
export default TaskForm
```

actions.js

```js
// fix params
export const createTaskCustom = async (prevState, formData) => {
  await new Promise((resolve) => setTimeout(resolve, 2000))
  const content = formData.get('content')
  // some validation here
  try {
    await prisma.task.create({
      data: {
        content,
      },
    })
    // revalidate path
    revalidatePath('/tasks')
    return { message: 'success!!!' }
  } catch (error) {
    // can't return error
    return { message: 'error...' }
  }
}
```

## Extra - More User Input Validation Options

- required attribute a great start
- zod library

The Zod library is a TypeScript-first schema declaration and validation library that allows developers to create complex type checks with simple syntax.

[Zod](https://zod.dev/)

```sh
npm install zod
```

actions.js

```js
import { z } from 'zod'

export const createTaskCustom = async (prevState, formData) => {
  await new Promise((resolve) => setTimeout(resolve, 2000))
  const content = formData.get('content')

  const Task = z.object({
    content: z.string().min(5),
  })

  // some validation here
  try {
    Task.parse({
      content,
    })
    await prisma.task.create({
      data: {
        content,
      },
    })
    // revalidate path
    revalidatePath('/tasks')
    return { message: 'success!!!' }
  } catch (error) {
    console.log(error)
    // can't return error
    return { message: 'error...' }
  }
}
```

## Providers

[Beautiful Toasts](https://react-hot-toast.com/)

```sh
npm install react-hot-toast
```

- create providers.js file in app

```js
'use client'
import { Toaster } from 'react-hot-toast'

const Providers = ({ children }) => {
  return (
    <>
      <Toaster />
      {children}
    </>
  )
}
export default Providers
```

layout.js

```js
import Providers from './providers'

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Navbar />
        <main className="px-8 py-20 max-w-6xl mx-auto">
          <Providers>{children}</Providers>
        </main>
      </body>
    </html>
  )
}
```

- simplify createTaskCustom action

```js
try {
  return { message: 'success' }
} catch (error) {
  console.log(error)
  return { message: 'error' }
}
```

- add toast

TaskFormCustom.jsx

```js
'use client'
import { createTaskCustom } from '@/utils/actions'
import { useEffect } from 'react'

import { useFormStatus, useFormState } from 'react-dom'
import toast from 'react-hot-toast'
const SubmitBtn = () => {
  const { pending } = useFormStatus()
  return (
    <button
      type="submit"
      className="btn btn-primary join-item"
      disabled={pending}
    >
      {pending ? 'please wait...' : 'create task'}
    </button>
  )
}

const initialState = {
  message: null,
}

const TaskForm = () => {
  const [state, formAction] = useFormState(createTaskCustom, initialState)

  useEffect(() => {
    if (state.message === 'error') {
      toast.error('there was an error')
      return
    }
    if (state.message) {
      toast.success('task created....')
    }
  }, [state])

  return (
    <form action={formAction}>
      <div className="join w-full">
        <input
          type="text "
          className="input input-bordered join-item w-full"
          placeholder="type here"
          name="content"
          required
        />
        <SubmitBtn />
      </div>
    </form>
  )
}
export default TaskForm
```

## Challenge - Add Functionality

DeleteForm.jsx

```js
'use client'
import { useFormStatus } from 'react-dom'
import { deleteTask } from '@/utils/actions'

const SubmitBtn = () => {
  const { pending } = useFormStatus()
  return (
    <button type="submit" className="btn btn-xs btn-error" disabled={pending}>
      {pending ? 'pending...' : 'delete'}
    </button>
  )
}

const DeleteForm = ({ id }) => {
  return (
    <form action={deleteTask}>
      <input type="hidden" name="id" value={id} />
      <SubmitBtn />
    </form>
  )
}
export default DeleteForm
```
