#

##

[ssg]

- all public route
- meta data and link to page
- role based with clerk middleware

(Clerk Docs)[https://clerk.com/]

[navigation]

- head and sidebar layout
- navlinks

(React Icons )[https://react-icons.github.io/react-icons/]

[theme-context]

- extract user id from currentUser context
- document element set attribute

  [DaisyUI](https://daisyui.com/components/drawer/)

## Create Next App

```sh
npx create-next-app@latest appName
```

## Libraries

```sh
npm install @clerk/nextjs@4.26.1 @prisma/client@5.5.2 @tanstack/react-query@5.8.1 @tanstack/react-query-devtools@5.8.1 axios@1.6.1  openai@4.14.2   react-hot-toast@2.4.1 react-icons@4.11.0
```

```sh
npm install -D @tailwindcss/typography@0.5.10  daisyui@3.9.4 prisma@5.5.2
```

## DaisyUI

- remove default code from globals.css
  tailwind.config.js

```js
{
plugins: [require('@tailwindcss/typography'), require('daisyui')],
}
```

## Challenge - Create Pages

- create following pages - chat, profile, tours
- group them together with route group (dashboard)
- setup one layout file for all three pages

## Challenge - Home Page

- setup title and description in the layout
- code home page (DaisyUI Hero Component)

## Solution - Home Page

app/layout.js

```js
export const metadata = {
  title: 'GPTGenius',
  description:
    'GPTGenius: Your AI language companion. Powered by OpenAI, it enhances your conversations, content creation, and more!',
}
```

app/page.js

```js
import Link from 'next/link'
const HomePage = () => {
  return (
    <div className='hero min-h-screen bg-base-200'>
      <div className='hero-content text-center'>
        <div className='max-w-md'>
          <h1 className='text-6xl font-bold text-primary'>GPTGenius</h1>
          <p className='py-6 text-lg leading-loose'>
            GPTGenius: Your AI language companion. Powered by OpenAI, it
            enhances your conversations, content creation, and more!
          </p>
          <Link href='/chat' className='btn btn-secondary '>
            Get Started
          </Link>
        </div>
      </div>
    </div>
  )
}

export default HomePage
```

## Clerk

(Clerk Docs)[https://clerk.com/]

- create account
- create new application
- complete Next.js setup

```sh
npm install @clerk/nextjs
```

```js
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = your_publishable_key
CLERK_SECRET_KEY = your_secret_key
```

Environment variables with this `NEXT_PUBLIC_` prefix are exposed to client-side JavaScript code, while those without the prefix are only accessible on the server-side and are not exposed to the client-side code.

```sh
NEXT_PUBLIC_
```

```js
const apiKey = process.env.NEXT_PUBLIC_API_KEY
```

layout.js

```js
import { ClerkProvider } from '@clerk/nextjs'

export default function RootLayout({ children }) {
  return (
    <ClerkProvider>
      <html lang='en'>
        <body>{children}</body>
      </html>
    </ClerkProvider>
  )
}
```

middleware.ts

```js
import { authMiddleware } from '@clerk/nextjs'

// This example protects all routes including api/trpc routes
// Please edit this to allow other routes to be public as needed.
// See https://clerk.com/docs/references/nextjs/auth-middleware for more information about configuring your Middleware
export default authMiddleware({
  publicRoutes: ['/'],
})

export const config = {
  matcher: ['/((?!.+\\.[\\w]+$|_next).*)', '/', '/(api|trpc)(.*)'],
}
```

## Challenge - Custom SignUp and SignIn Pages

- follow the docs and setup custom pages
- use clerk's component

app/sign-up/[[...sign-up]]/page.js

```js
import { SignUp } from '@clerk/nextjs'

const SignUpPage = () => {
  return (
    <div className='min-h-screen flex justify-center items-center'>
      <SignUp />
    </div>
  )
}
export default SignUpPage
```

app/sign-in/[[...sign-in]]/page.js

```js
import { SignIn } from '@clerk/nextjs'

const SignInPage = () => {
  return (
    <div className='min-h-screen flex justify-center items-center'>
      <SignIn />
    </div>
  )
}
export default SignInPage
```

.env.local

```js
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/chat
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/chat
```

## React Icons

(React Icons )[https://react-icons.github.io/react-icons/]

```sh
npm install react-icons --save
```

```js
import { FaBeer } from 'react-icons/fa';

<FaBeer>

```

## Challenge - Setup Dashboard Layout

- setup layout for for all pages
- DaisyUI Drawer Component
  [DaisyUI](https://daisyui.com/components/drawer/)

## Solution

- create components/sidebar

layout.js

```js
import { FaBarsStaggered } from 'react-icons/fa6'
import Sidebar from '@/components/Sidebar'
const layout = ({ children }) => {
  return (
    <div className='drawer lg:drawer-open'>
      <input id='my-drawer-2' type='checkbox' className='drawer-toggle' />
      <div className='drawer-content'>
        {/* Page content here */}
        <label
          htmlFor='my-drawer-2'
          className='drawer-button lg:hidden fixed top-6 right-6'
        >
          <FaBarsStaggered className='w-8 h-8 text-primary' />
        </label>
        <div className='bg-base-200 px-8 py-12 min-h-screen'>{children}</div>
      </div>
      <div className='drawer-side'>
        <label
          htmlFor='my-drawer-2'
          aria-label='close sidebar'
          className='drawer-overlay'
        ></label>
        <Sidebar />
      </div>
    </div>
  )
}
export default layout
```

## Sidebar

- create SidebarHeader, NavLinks, MemberProfile

Sidebar.jsx

```js
import SidebarHeader from './SidebarHeader'
import NavLinks from './NavLinks'
import MemberProfile from './MemberProfile'

const Sidebar = () => {
  return (
    <div className='px-4 w-80 min-h-full bg-base-300 py-12 grid grid-rows-[auto,1fr,auto] '>
      {/* first row */}
      <SidebarHeader />
      {/* second row */}
      <NavLinks />
      {/* third row */}
      <MemberProfile />
    </div>
  )
}
export default Sidebar
```

## SidebarHeader

- create ThemeToggle

```js
import ThemeToggle from './ThemeToggle'
import { SiOpenaigym } from 'react-icons/si'

const SidebarHeader = () => {
  return (
    <div className='flex items-center mb-4 gap-4 px-4'>
      <SiOpenaigym className='w-10 h-10 text-primary' />
      <h2 className='text-xl font-extrabold text-primary mr-auto'>GPTGenius</h2>
      <ThemeToggle />
    </div>
  )
}
export default SidebarHeader
```

## Challenge - NavLinks

1. **Import Dependencies:**

   - Import the `Link` component from `next/link`.

2. **Define Navigation Links:**

   - Create an array named `links` that contains objects representing navigation links. Each object should have a `href` property specifying the link's destination and a `label` property for the link's text label.

3. **Create the NavLinks Component:**

   - Define a functional component named `NavLinks`.

4. **Render Navigation Links:**

   - Within the component, render an unordered list (`<ul>`) with a class of `'menu text-base-content'`.

   - Use the `map` function to iterate through the `links` array and generate list items (`<li>`) for each link.

   - For each link object in the `links` array, create a `Link` component with the `href` attribute set to the link's `href` property.

   - Display the link's label (`link.label`) as the text content of the `Link` component.

This component is responsible for rendering navigation links based on the `links` array. It uses the `next/link` package to create client-side navigation links in a Next.js application. The navigation links are generated dynamically based on the `links` array.

## Solution - NavLinks

```js
import Link from 'next/link'
const links = [
  { href: '/chat', label: 'chat' },
  { href: '/tours', label: 'tours' },
  { href: '/tours/new-tour', label: 'new tour' },
  { href: '/profile', label: 'profile' },
]

const NavLinks = () => {
  return (
    <ul className='menu  text-base-content'>
      {links.map((link) => {
        return (
          <li key={link.href}>
            <Link href={link.href} className='capitalize'>
              {link.label}
            </Link>
          </li>
        )
      })}
    </ul>
  )
}
export default NavLinks
```

## Challenge - MemberProfile

1. **Import Dependencies:**

   - Import the necessary dependencies at the top of the file:
     - `UserButton`, `currentUser`, and `auth` from `@clerk/nextjs`.

2. **Create the MemberProfile Component:**

   - Define an asynchronous functional component named `MemberProfile`.

3. **Fetch Current User:**

   - Inside the component, use the `currentUser()` function to asynchronously fetch the currently authenticated user and store it in the `user` variable.

4. **Get User ID:**

   - Use the `auth()` function to extract the `userId` from the authentication context.

5. **Render User Profile:**

   - Render a `div` element containing the user's profile information.
   - Include a `UserButton` component, which provides a button for signing out and redirects to the specified URL (`'/'` in this case) after sign-out.
   - Display the user's email address using `user.emailAddresses[0].emailAddress`.

This component fetches the currently authenticated user and displays their email address along with a sign-out button. It uses the `@clerk/nextjs` library for authentication and user management in a Next.js application.

## Solution - MemberProfile

```js
import { UserButton, currentUser, auth } from '@clerk/nextjs'
const MemberProfile = async () => {
  const user = await currentUser()
  const { userId } = auth()
  return (
    <div className='px-4 flex items-center gap-2'>
      <UserButton afterSignOutUrl='/' />
      <p>{user.emailAddresses[0].emailAddress}</p>
    </div>
  )
}
export default MemberProfile
```

## Challenge - ThemeToggle

- setup themes in tailwind.config.js

1. **Import Dependencies:**

   - Import the necessary dependencies at the top of the file:
     - `BsMoonFill` and `BsSunFill` from 'react-icons/bs'.
     - `useState` from 'react'.

2. **Define Theme Options:**

   - Create an object named `themes` to hold theme options. In this example, there are two themes: 'winter' and 'dracula'.

3. **Initialize Theme State:**

   - Use the `useState` hook to initialize the `theme` state variable with the default theme, such as `themes.winter`.

4. **Toggle Theme Function:**

   - Define a function named `toggleTheme` to handle theme toggling.
   - Inside the function, check the current theme (`theme`) and switch it to the opposite theme (`themes.dracula` if it's 'winter', or `themes.winter` if it's 'dracula').
   - Update the document's root element (`document.documentElement`) with the new theme by setting the 'data-theme' attribute.

5. **Button Rendering:**

   - Render a button element with an `onClick` event handler that triggers the `toggleTheme` function.

6. **Conditional Icon Rendering:**

   - Inside the button, conditionally render icons based on the current theme.
   - If the theme is 'winter', render a moon icon (e.g., `<BsMoonFill />`).
   - If the theme is 'dracula', render a sun icon (e.g., `<BsSunFill />`).

This component allows users to toggle between two themes (e.g., light and dark) by clicking the button, which updates the `data-theme` attribute on the document's root element and changes the displayed icon accordingly.

## Solution - ThemeToggle

tailwind.config.js

```js
{
  daisyui: {
    themes: ['winter', 'dracula'],
  },
}

```

```js
'use client'

import { BsMoonFill, BsSunFill } from 'react-icons/bs'
import { useState } from 'react'

const themes = {
  winter: 'winter',
  dracula: 'dracula',
}

const ThemeToggle = () => {
  const [theme, setTheme] = useState(themes.winter)

  const toggleTheme = () => {
    const newTheme = theme === themes.winter ? themes.dracula : themes.winter
    document.documentElement.setAttribute('data-theme', newTheme)
    setTheme(newTheme)
  }

  return (
    <button onClick={toggleTheme} className='btn btn-sm btn-outline'>
      {theme === 'winter' ? (
        <BsMoonFill className='h-4 w-4 ' />
      ) : (
        <BsSunFill className='h-4 w-4' />
      )}
    </button>
  )
}
export default ThemeToggle
```
