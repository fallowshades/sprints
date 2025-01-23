This GitHub project provides a comprehensive guide to integrating TypeScript with React. It covers initial setup using vite and TypeScript, demonstrates various React & TypeScript concepts through clear examples, and delves into more advanced topics. Key aspects include:

- Component Structure and TypeScript Integration: Explains how to correctly type React components, manage return types, and handle potential TypeScript errors.
- Prop Handling and Typing: Offers insights on inline typing, using interfaces, and managing children props with TypeScript.
- State Management: Teaches TypeScript type inference in state management, showcasing various useState examples.
- Event Handling: Guides on typing events in React, such as form submissions and input changes.
- Complex Component Structures: Discusses complex use cases like conditional prop rendering based on type values.
- Context API with TypeScript: Provides a deep dive into using React's Context API in a TypeScript environment.
- Reducers and Global State Management: Includes examples of setting up reducers with TypeScript and using them in React components.
- Data Fetching: Demonstrates fetching data with TypeScript validation using tools like Zod, Axios, and React Query.
- Redux Toolkit (RTK) Integration: Shows how to integrate Redux Toolkit in a TypeScript-React setup, including creating slices and using hooks.
- Practical Application with Task Management: Concludes with a practical task management application, emphasizing localStorage use and handling task state.

Each section is presented with relevant code snippets and explanations, making it an ideal resource for developers looking to deepen their understanding of TypeScript in React applications.

#

## Setup

```sh
npm create vite@latest react-typescript -- --template react-ts
```

## Remove Boilerplate and Get Assets

# React & Typescript

- .tsx - file extension

## 01 - Component Return

- TypeScript infers JSX.Element, helps if no return

```tsx
// TypeScript infers JSX.Element
// this will trigger error
function Component() {}
export default Component
```

- set function return type

```tsx
function Component(): JSX.Element | null | string {
  return null
  return 'hello'
  return <h2>hello from typescript</h2>
}
export default Component
```

## 02- Props

```tsx
function App() {
  return (
    <main>
      <Component name="peter" id={123} />
    </main>
  )
}

export default App
```

- inline types

```tsx
function Component({ name, id }: { name: string; id: number }) {
  return (
    <div>
      <h1>Name : {name}</h1>
      <h1>ID : {id}</h1>
    </div>
  )
}
export default Component
```

- type or interface
- props object or {}

```tsx
type ComponentProps = {
  name: string
  id: number
}

function Component({ name, id }: ComponentProps) {
  return (
    <div>
      <h1>Name : {name}</h1>
      <h1>ID : {id}</h1>
    </div>
  )
}
export default Component
```

- children prop

```tsx
function App() {
  return (
    <main>
      <Component name="peter" id={123}>
        <h2>hello world</h2>
      </Component>
    </main>
  )
}

export default App
```

- React.ReactNode
- PropsWithChildren

```tsx
import { type PropsWithChildren } from 'react'

type ComponentProps = {
  name: string
  id: number
  children: React.ReactNode
}

// type ComponentProps = PropsWithChildren<{
//   name: string;
//   id: number;
// }>;

function Component({ name, id, children }: ComponentProps) {
  return (
    <div>
      <h2>Name : {name}</h2>
      <h2>ID : {id}</h2>
      {children}
    </div>
  )
}
export default Component
```

## 03 - State

- typescript infers primitive types
- by default [] is type never

```tsx
import { useState } from 'react'

function Component() {
  const [text, setText] = useState('shakeAndBake')
  const [number, setNumber] = useState(1)
  const [list, setList] = useState<string[]>([])

  return (
    <div>
      <h2 className="mb-1">hello from typescript</h2>
      <button
        className="btn btn-center"
        onClick={() => {
          // setText(1);
          // setNumber('hello');
          // setList([1, 3]);
          setList(['hello', 'world'])
        }}
      >
        click me
      </button>
    </div>
  )
}
export default Component
```

```tsx
import { useState } from 'react'

type Link = {
  id: number
  url: string
  text: string
}

const navLinks: Link[] = [
  {
    id: 1,
    url: 'https://reactjs.org',
    text: 'react docs',
  },
  {
    id: 2,
    url: 'https://reactrouter.com',
    text: 'react router docs',
  },
  {
    id: 3,
    url: 'https://reacttraining.com',
    // remove text property to see the error
    text: 'react training',
  },
]

function Component() {
  const [text, setText] = useState('shakeAndBake')
  const [number, setNumber] = useState(1)
  const [list, setList] = useState<string[]>([])
  const [links, setLinks] = useState<Link[]>(navLinks)
  return (
    <div>
      <h2 className="mb-1">hello from typescript</h2>
      <button
        className="btn btn-center"
        onClick={() => {
          // setText(1);
          // setNumber('hello');
          // setList([1, 3]);
          // setList(['hello', 'world']);
          // setLinks([...links, { id: 4, url: 'hello', someValue: 'hello' }])
          setLinks([...links, { id: 4, url: 'hello', text: 'hello' }])
        }}
      >
        click me
      </button>
    </div>
  )
}
export default Component
```

```tsx

```

## 04 - Events

- inline function infers object type

When you provide the exact HTML element type in the angle brackets (<>), like HTMLInputElement in your case, you're telling TypeScript exactly what kind of element the event is coming from. This helps TypeScript provide accurate suggestions and error checking based on the properties and methods that are specific to that kind of element. For example, an HTMLInputElement has properties like value and checked that other elements don't have. By specifying the exact element type, TypeScript can help you avoid mistakes and write safer code.

```tsx
import { useState } from 'react'

function Component() {
  const [text, setText] = useState('')
  const [email, setEmail] = useState('')

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value)
    setEmail(e.target.value)
  }

  return (
    <section>
      <h2>events example</h2>
      <form className="form">
        <input
          className="form-input mb-1"
          type="text"
          name="text"
          value={text}
          onChange={(e) => setText(e.target.value)}
        />

        <input
          type="email"
          className="form-input mb-1"
          value={email}
          onChange={handleChange}
        />
        <button type="submit" className="btn btn-block">
          submit
        </button>
      </form>
    </section>
  )
}
export default Component
```

```tsx
import { useState } from 'react'

type Person = {
  name: string
}

function Component() {
  const [text, setText] = useState('')
  const [email, setEmail] = useState('')

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value)
    setEmail(e.target.value)
  }

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    // const formData = new FormData(e.currentTarget);
    const formData = new FormData(e.target as HTMLFormElement)
    // const data = Object.fromEntries(formData);
    const text = formData.get('text') as string
    const person: Person = { name: text }
  }

  return (
    <section>
      <h2>events example</h2>
      <form onSubmit={handleSubmit} className="form">
        <input
          className="form-input mb-1"
          type="text"
          name="text"
          value={text}
          onChange={(e) => setText(e.target.value)}
        />

        <input
          type="email"
          className="form-input mb-1"
          value={email}
          onChange={handleChange}
          name="email"
        />
        <button type="submit" className="btn btn-block">
          submit
        </button>
      </form>
    </section>
  )
}
export default Component
```

The FormData API is a web technology that allows developers to easily construct and manage sets of key/value pairs representing form fields and their values. It is commonly used to send form data, including files, from a client (such as a web browser) to a server in a format that can be easily processed. The FormData API provides a way to programmatically create and manipulate form data, making it useful for AJAX requests and handling file uploads in web applications.

## 05 - Challenge - Profile Card

- initial approach (won't work as expected)

```tsx
type ProfileCardProps = {
  type: 'basic' | 'advanced'
  name: string
  email?: string
}

function Component(props: ProfileCardProps) {
  const { type, name, email } = props

  const alertType = type === 'basic' ? 'success' : 'danger'
  const className = `alert alert-${alertType}`
  return (
    <article className={className}>
      <h2>user : {name}</h2>
      {email && <h2>email : {email}</h2>}
    </article>
  )
}
export default Component
```

- another approach (won't work as expected)

```tsx
type ProfileCardProps = {
  type: 'basic' | 'advanced'
  name: string
  email?: string
}

function Component(props: ProfileCardProps) {
  const { type, name, email } = props

  const alertType = type === 'basic' ? 'success' : 'danger'
  const className = `alert alert-${alertType}`
  return (
    <article className={className}>
      <h2>user : {name}</h2>
      {type === advanced ? <h2>email : {email}</h2> : null}
    </article>
  )
}
export default Component
```

- final approach

```tsx
type BasicProfileCardProps = {
  type: 'basic'
  name: string
}

type AdvancedProfileCardProps = {
  type: 'advanced'
  name: string
  email: string
}
type ProfileCardProps = BasicProfileCardProps | AdvancedProfileCardProps
function Component(props: ProfileCardProps) {
  const { type, name } = props
  if (type === 'basic')
    return (
      <article className="alert alert-success">
        <h2>user : {name}</h2>
      </article>
    )

  return (
    <article className="alert alert-danger">
      <h2>user : {name}</h2>
      <h2>email : {props.email}</h2>
    </article>
  )
}
export default Component
```

## 06 - Context

- basic context

```tsx
import { createContext, useContext } from 'react'

const ThemeProviderContext = createContext<{ name: string } | undefined>(
  undefined
)

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProviderContext.Provider value={{ name: 'susan' }}>
      {children}
    </ThemeProviderContext.Provider>
  )
}

export const useTheme = () => {
  const context = useContext(ThemeProviderContext)

  if (context === undefined)
    throw new Error('useTheme must be used within a ThemeProvider')

  return context
}
```

basic-index.tsx

```tsx
import { useTheme, ThemeProvider } from './basic-context'

function ParentComponent() {
  return (
    <ThemeProvider>
      <Component />
    </ThemeProvider>
  )
  return <Component />
}

function Component() {
  const context = useTheme()
  console.log(context)

  return (
    <div>
      <h2>random component</h2>
    </div>
  )
}
export default ParentComponent
```

context.tsx

```tsx
import { createContext, useState, useContext } from 'react'

type Theme = 'light' | 'dark' | 'system'

type ThemeProviderState = {
  theme: Theme
  setTheme: (theme: Theme) => void
}

const ThemeProviderContext = createContext<ThemeProviderState | undefined>(
  undefined
)

type ThemeProviderProps = {
  children: React.ReactNode
  defaultTheme?: Theme
}

export function ThemeProvider({
  children,
  defaultTheme = 'system',
}: ThemeProviderProps) {
  const [theme, setTheme] = useState<Theme>(defaultTheme)
  return (
    <ThemeProviderContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeProviderContext.Provider>
  )
}

export const useTheme = () => {
  const context = useContext(ThemeProviderContext)

  if (context === undefined)
    throw new Error('useTheme must be used within a ThemeProvider')

  return context
}
```

Component.tsx

```tsx
import { useTheme, ThemeProvider } from './context'

function ParentComponent() {
  return (
    <ThemeProvider>
      <Component />
    </ThemeProvider>
  )
  return <Component />
}

function Component() {
  const context = useTheme()
  console.log(context)

  return (
    <div>
      <h2>random component</h2>
      <button
        onClick={() => {
          if (context.theme === 'dark') {
            context.setTheme('system')
            return
          }
          context.setTheme('dark')
        }}
        className="btn btn-center"
      >
        toggle theme
      </button>
    </div>
  )
}
export default ParentComponent
```
