## Setup

```sh
npm create vite@latest comfy-store -- --template react-ts
```

```sh
npm install && npm run dev
```

## Boilerplate Code

- change title (optional)
- remove boilerplate and add assets
- remove App.css
- delete everything in index.css
- remove everything from App.tsx
- snippets extension (rfce)
- get README.md

```tsx
function App() {
  return <div>App</div>
}
export default App
```

## Install Libraries

```sh
npm install @reduxjs/toolkit@^2.1.0 axios@^1.6.7 react-redux@^9.1.0 react-router-dom@^6.21.3

```

## Tailwind CSS

[Tailwind Docs](https://tailwindcss.com/docs/guides/vite)

```sh
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

index.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

- restart the project

```tsx
function App() {
  return <h1 className="text-7xl font-bold ">App</h1>
}
export default App
```

## Shadcn/ui

[ShadcnUI](https://ui.shadcn.com/)

- setup

[Shadcn and Vite](https://ui.shadcn.com/docs/installation/vite)

tsconfig.json

```json
{
  "compilerOptions": {
    // ...
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
    // ...
  }
}
```

```sh
npm i -D @types/node
```

vite.config.ts

```ts
import path from 'path'
import react from '@vitejs/plugin-react'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

```sh
npx shadcn-ui@latest init
```

```sh
npx shadcn-ui@latest add button

```

- restart server

```tsx
import { Button } from './components/ui/button'

function App() {
  return (
    <div>
      <h1 className="text-7xl font-bold ">App</h1>
      <Button
        variant="destructive"
        size="lg"
        onClick={() => console.log('it worked!!!')}
      >
        Click Me
      </Button>
    </div>
  )
}
export default App
```

## Shadcn in Detail

- choose component
- install component
- component is located in components/ui
- can modify the code and take from project to project
- check the export
- import in other components
- uses Typescript so harder to make mistakes
- variables in index.css
- more options in the docs

## Redux Toolkit

```sh
npm install @reduxjs/toolkit react-redux
```

- create src/features
  - cart/cartSlice.ts
  - theme/themeSlice.ts
  - user/userSlice.ts

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'

const initialState = {
  name: 'user slice',
}
const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {},
})

export default userSlice.reducer
```

- create src/store.ts

```ts
import { configureStore } from '@reduxjs/toolkit'
import themeReducer from './features/theme/themeSlice'
import cartReducer from './features/cart/cartSlice'
import userReducer from './features/user/userSlice'
// ...

export const store = configureStore({
  reducer: {
    themeState: themeReducer,
    cartState: cartReducer,
    userState: userReducer,
  },
})

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch

export type ReduxStore = {
  getState: () => RootState
  dispatch: AppDispatch
}
```

- create hooks.ts

```ts
import { useDispatch, useSelector } from 'react-redux'
import type { TypedUseSelectorHook } from 'react-redux'
import type { RootState, AppDispatch } from './store'

// Use throughout your app instead of plain `useDispatch` and `useSelector`
export const useAppDispatch: () => AppDispatch = useDispatch
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector
```

- setup Provider

main.tsx

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import './index.css'

import { store } from './store'
import { Provider } from 'react-redux'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <Provider store={store}>
    <App />
  </Provider>
)
```

```tsx
import { useAppSelector } from './hooks'
function App() {
  const { name } = useAppSelector((state) => state.userState)
  console.log(name)
}
```

## Create Pages

- create src/pages

  - About
  - Cart
  - Checkout
  - Error
  - HomeLayout
  - Landing
  - Login
  - Orders
  - Products
  - Register
  - SingleProduct
  - index.ts

  ```tsx
  function About() {
    return <h1 className="text-4xl">About Page</h1>
  }
  export default About
  ```

  index.ts

  ```ts
  export { default as HomeLayout } from './HomeLayout'
  export { default as Landing } from './Landing'
  export { default as SingleProduct } from './SingleProduct'
  export { default as Products } from './Products'
  export { default as Cart } from './Cart'
  export { default as Error } from './Error'
  export { default as About } from './About'
  export { default as Login } from './Login'
  export { default as Register } from './Register'
  export { default as Checkout } from './Checkout'
  export { default as Orders } from './Orders'
  ```

  It's taking the default export from the 'About' module and giving it a name of 'About'.

App.tsx

```tsx
import {
  HomeLayout,
  Landing,
  Error,
  Products,
  SingleProduct,
  Cart,
  About,
  Register,
  Login,
  Checkout,
  Orders,
} from './pages'

import { Button } from './components/ui/button'
import { useAppSelector } from './hooks'
function App() {
  const { name } = useAppSelector((state) => state.userState)
  console.log(name)

  return (
    <div>
      <h1 className="text-7xl font-bold ">App</h1>
      <Button
        variant="destructive"
        size="lg"
        onClick={() => console.log('it worked!!!')}
      >
        Click Me
      </Button>
      <Cart />
    </div>
  )
}
export default App
```

- START CODING ALONG 😄😄😄😄😄

## React Router

```sh
npm i react-router-dom
```

App.tsx

```tsx
import { RouterProvider, createBrowserRouter } from 'react-router-dom'

const router = createBrowserRouter([
  {
    path: '/',
    element: <Landing />,
  },
  {
    path: '/cart',
    element: <Cart />,
  },
])

const App = () => {
  return <RouterProvider router={router} />
}
export default App
```

## Link Component

Cart.tsx

```tsx
import { Button } from '@/components/ui/button'
import { Link } from 'react-router-dom'
function Cart() {
  return (
    <div>
      <h1 className="text-4xl">Cart Page</h1>
      <Link to="/" className="text-7xl text-red-900">
        back home
      </Link>
      <Button asChild size="lg">
        <Link to="/">home button</Link>
      </Button>
    </div>
  )
}
export default Cart
```

## Outlet

App.tsx

```tsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <HomeLayout />,

    children: [
      {
        index: true,
        element: <Landing />,
      },
      {
        path: 'products',
        element: <Products />,
      },
      {
        path: 'products/:id',
        element: <SingleProduct />,
      },
      {
        path: 'cart',
        element: <Cart />,
      },
      { path: 'about', element: <About /> },
      {
        path: 'checkout',
        element: <Checkout />,
      },
      {
        path: 'orders',
        element: <Orders />,
      },
    ],
  },
  {
    path: '/login',
    element: <Login />,
  },
  {
    path: '/register',
    element: <Register />,
  },
])
```

HomeLayout.tsx

```tsx
import { Outlet } from 'react-router-dom'

const HomeLayout = () => {
  return (
    <>
      <header>header</header>
      <nav>navbar</nav>
      <Outlet />
    </>
  )
}
export default HomeLayout
```

## Header

- create components/Header.tsx (basic return)
- create components/index.ts(optional)

```ts
export { default as Header } from './Header'
```

- render in HomeLayout

```tsx
import { Link, useNavigate } from 'react-router-dom'
import { Button } from './ui/button'
import { useState } from 'react'
const Header = () => {
  const navigate = useNavigate()

  // temp
  const [user, setUser] = useState<{ username: string } | null>({
    username: 'demo user',
  })

  const handleLogout = () => {
    navigate('/')
    setUser(null)
  }

  return (
    <header>
      <div className="align-element flex justify-center sm:justify-end py-2">
        {/* USER */}
        {user ? (
          <div className="flex gap-x-2 sm:gap-x-8 items-center">
            <p className="text-xs sm:text-sm">Hello, {user.username}</p>
            <Button variant="link" size="sm" onClick={handleLogout}>
              Logout
            </Button>
          </div>
        ) : (
          <div className="flex gap-x-6 justify-center items-center -mr-4">
            <Button asChild variant="link" size="sm">
              <Link to="/login">Sign in / Guest</Link>
            </Button>
            <Button asChild variant="link" size="sm">
              <Link to="/register">Register</Link>
            </Button>
          </div>
        )}
      </div>
    </header>
  )
}
export default Header
```

## Tailwind Custom Class

index.css

```css
@layer components {
  .align-element {
    @apply mx-auto max-w-6xl px-8;
  }
}
```

HomeLayout.tsx

```tsx
<div className="align-element py-20">
  <Outlet />
</div>
```

Header.tsx

```tsx
<div className="align-element flex ......">...</div>
```

## Navbar

- create following components :
  - Navbar, Logo, LinksDropdown, NavLinks, ModeToggle, CartButton
- render navbar in the HomeLayout

```tsx
import CartButton from './CartButton'
import LinksDropdown from './LinksDropdown'
import Logo from './Logo'
import ModeToggle from './ModeToggle'
import NavLinks from './NavLinks'

const Navbar = () => {
  return (
    <nav className="bg-muted py-4">
      <div className="align-element flex justify-between items-center">
        <Logo />
        <LinksDropdown />
        <NavLinks />
        <div className="flex justify-center items-center gap-x-4">
          <ModeToggle />
          <CartButton />
        </div>
      </div>
    </nav>
  )
}
export default Navbar
```

## Logo

- already installed with shadcn
  [lucide-react](https://lucide.dev/icons/)

Logo.tsx

```tsx
import { Link } from 'react-router-dom'
import { Armchair } from 'lucide-react'

function Logo() {
  return (
    <Link
      to="/"
      className="hidden lg:flex justify-center items-center bg-primary p-2 rounded-lg text-white "
    >
      <Armchair className="w-8 h-8" />
    </Link>
  )
}

export default Logo
```

## Links

- create src/utils/links.ts
- create src/utils/index.ts

links.ts

```ts
type Link = {
  href: string
  label: string
}

export const links: Link[] = [
  { href: '/', label: 'home' },
  { href: 'about', label: 'about' },
  { href: 'products', label: 'products' },
  { href: 'cart', label: 'cart' },
  { href: 'checkout', label: 'checkout' },
  { href: 'orders', label: 'orders' },
]
```

index.ts

```ts
export * from './links'
```

## DropdownMenu Component - Shadcn

```sh
npx shadcn-ui@latest add dropdown-menu
```

[DropdownMenu](https://ui.shadcn.com/docs/components/dropdown-menu)

## LinksDropdown

```tsx
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'
import { AlignLeft } from 'lucide-react'
import { Button } from './ui/button'
import { links } from '@/utils'
import { NavLink } from 'react-router-dom'

function LinksDropdown() {
  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild className="lg:hidden">
        <Button variant="outline" size="icon">
          <AlignLeft />

          <span className="sr-only">Toggle links</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent
        className="w-52 lg:hidden "
        align="start"
        sideOffset={25}
      >
        {links.map((link) => {
          return (
            <DropdownMenuItem key={link.label}>
              <NavLink
                to={link.href}
                className={({ isActive }) => {
                  return `capitalize w-full ${isActive ? 'text-primary' : ''}`
                }}
              >
                {link.label}
              </NavLink>
            </DropdownMenuItem>
          )
        })}
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
export default LinksDropdown
```

## NavLinks

```tsx
import { links } from '@/utils'
import { NavLink } from 'react-router-dom'

function NavLinks() {
  return (
    <div className="hidden lg:flex justify-center items-center gap-x-4">
      {links.map((link) => {
        return (
          <NavLink
            to={link.href}
            className={({ isActive }) => {
              return `capitalize font-light tracking-wide ${
                isActive ? 'text-primary' : ''
              }`
            }}
            key={link.label}
          >
            {link.label}
          </NavLink>
        )
      })}
    </div>
  )
}
export default NavLinks
```

## Theme

[Theming](https://ui.shadcn.com/docs/theming)
[Themes](https://ui.shadcn.com/themes)

- overwrite base in index.css

## Dark Mode

[Dark Mode](https://ui.shadcn.com/docs/dark-mode/vite)

## ApplyTheme

- setup Theme in features/theme/themeSlice.ts

```ts
export type Theme = 'dark' | 'light' | 'system'
```

- create utils/applyTheme
- setup export

```ts
import { type Theme } from '@/features/theme/themeSlice'

export function applyTheme(theme: Theme) {
  const root = window.document.documentElement

  root.classList.remove('light', 'dark')

  if (theme === 'system') {
    const systemTheme = window.matchMedia('(prefers-color-scheme: dark)')
      .matches
      ? 'dark'
      : 'light'

    root.classList.add(systemTheme)
    return
  }

  root.classList.add(theme)
}
```

## ThemeSlice

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'
import { applyTheme } from '@/utils'
export type Theme = 'dark' | 'light' | 'system'

type ThemeState = {
  theme: Theme
}

const initializeTheme = (): Theme => {
  const theme = (localStorage.getItem('theme') as Theme) || 'system'
  applyTheme(theme)
  return theme
}

const initialState: ThemeState = {
  theme: initializeTheme(),
}

const themeSlice = createSlice({
  name: 'theme',
  initialState,
  reducers: {
    setTheme: (state, action: PayloadAction<Theme>) => {
      state.theme = action.payload
      applyTheme(action.payload)
      localStorage.setItem('theme', action.payload)
    },
  },
})

export const { setTheme } = themeSlice.actions

export default themeSlice.reducer
```

## ModeToggle

```tsx
import { Moon, Sun } from 'lucide-react'
import { useAppDispatch } from '@/hooks'
import { Button } from '@/components/ui/button'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'
import { setTheme } from '@/features/theme/themeSlice'

function ModeToggle() {
  const dispatch = useAppDispatch()

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="outline" size="icon">
          <Sun className="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
          <Moon className="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
          <span className="sr-only">Toggle theme</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem onClick={() => dispatch(setTheme('light'))}>
          Light
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => dispatch(setTheme('dark'))}>
          Dark
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => dispatch(setTheme('system'))}>
          System
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}

export default ModeToggle
```

## CartButton

```tsx
import { Button } from './ui/button'
import { ShoppingCart } from 'lucide-react'
import { Link } from 'react-router-dom'

function CartButton() {
  const numItemsInCart = 5

  return (
    <Button
      asChild
      variant="outline"
      size="icon"
      className="flex justify-center items-center relative"
    >
      <Link to="/cart">
        <ShoppingCart />
        <span className="absolute -top-3 -right-3 bg-primary text-white rounded-full h-6 w-6 flex items-center justify-center text-xs">
          {numItemsInCart}
        </span>
      </Link>
    </Button>
  )
}
export default CartButton
```

## Error Page

```tsx
import { useRouteError, Link, isRouteErrorResponse } from 'react-router-dom'
import { Button } from '@/components/ui/button'

const Error = () => {
  // type:unknown
  const error = useRouteError()
  console.log(error)
  if (isRouteErrorResponse(error) && error.status === 404) {
    return (
      <main className="grid min-h-[100vh] place-items-center px-8 ">
        <div className="text-center">
          <p className="text-9xl font-semibold text-primary">404</p>
          <h1 className="mt-4 text-3xl font-bold tracking-tight sm:text-5xl">
            Page not found
          </h1>
          <p className="mt-6 text-lg leading-7">
            Sorry, we couldn’t find the page you’re looking for.
          </p>
          <div className="mt-10 ">
            <Button asChild size="lg" variant="secondary">
              <Link to="/">Go back home</Link>
            </Button>
          </div>
        </div>
      </main>
    )
  }

  return (
    <main className="grid min-h-[100vh] place-items-center px-8 ">
      <h4 className="text-center font-bold text-4xl">there was an error... </h4>
    </main>
  )
}
export default Error
```

App.tsx

```tsx
{
    path: '/',
    element: <HomeLayout />,
    errorElement: <Error />,
}
{
    path: '/login',
    element: <Login />,
    errorElement: <Error />,
  },
  {
    path: '/register',
    element: <Register />,
    errorElement: <Error />,
  },
```

## About Page

```tsx
function About() {
  return (
    <section>
      <h1 className="flex flex-wrap gap-2 sm:gap-x-6 items-center justify-center text-4xl font-bold leading-none tracking-tight sm:text-6xl ">
        We love
        <span className="bg-primary py-2 px-4 rounded-lg  tracking-widest text-white">
          comfy
        </span>
      </h1>

      <p className="mt-6 text-lg tracking-wide leading-8 max-w-2xl mx-auto">
        Lorem ipsum dolor sit amet consectetur adipisicing elit. Hic veniam
        odit, officiis eos mollitia alias, doloremque, aspernatur ratione
        asperiores voluptas labore minus dolores reprehenderit corporis quos.
        Assumenda molestias harum dignissimos?
      </p>
    </section>
  )
}
export default About
```

## Error Element

- create src/components/ErrorElement
- setup export

```tsx
import { useRouteError } from 'react-router-dom'
const ErrorElement = () => {
  const error = useRouteError()
  console.log(error)

  return <h4 className="font-bold text-4xl">there was an error... </h4>
}
export default ErrorElement
```

App.tsx

```tsx
 { path: 'about', element: <About />, errorElement: <ErrorElement /> },
```
