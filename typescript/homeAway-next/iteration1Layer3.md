# take home away

##

### routing for role based applicat

## batch user owned messages

```sh
npx create-next-app@latest home-away
```

```sh
npm run dev
```

### Remove Boilerplate

- in globals.css remove all code after directives
- page.tsx

```tsx
function HomePage() {
  return <h1 className='text-3xl'>HomePage</h1>
}
export default HomePage
```

- layout.tsx

```tsx
export const metadata: Metadata = {
  title: 'HomeAway',
  description: 'Feel at home, away from home.',
}
```

- get a hold of the README.MD

### Create Pages

- bookings
- checkout
- favorites
- profile
- properties
- rentals
- reviews
- new file - pageName/page.tsx

```tsx
function BookingsPage() {
  return <h1 className='text-3xl'>BookingsPage</h1>
}
export default BookingsPage
```

### Shadcn/ui

[﻿Docs](https://ui.shadcn.com/)

[﻿Next Install](https://ui.shadcn.com/docs/installation/next)

```sh
npx shadcn-ui@latest init
```

- New York
- Zinc

```sh
npx shadcn-ui@latest add button
```

```tsx
import { Button } from '@/components/ui/button'

function HomePage() {
  return (
    <div>
      <h1 className='text-3xl'>HomePage</h1>
      <Button variant='outline' size='lg' className='capitalize m-8'>
        Click me
      </Button>
    </div>
  )
}
export default HomePage
```

```sh
npx shadcn-ui@latest add breadcrumb calendar card checkbox dropdown-menu input label popover scroll-area select separator table textarea toast skeleton
```

- components
  - ui
  - card
  - form
  - home
  - navbar
  - properties

## for pages representing all filtered and user imput pages

### Navbar - Setup

- create
- navbar
  - DarkMode.tsx
  - LinksDropdown.tsx
  - Logo.tsx
  - Navbar.tsx
  - NavSearch.tsx
  - SignOutLink.tsx
  - UserIcon.tsx

### Tailwind Custom Class

globals.css

```css
@layer components {
  .container {
    @apply mx-auto max-w-6xl xl:max-w-7xl px-8;
  }
}
```

### Navbar - Structure

```tsx
import NavSearch from './NavSearch'
import LinksDropdown from './LinksDropdown'
import DarkMode from './DarkMode'
function Navbar() {
  return (
    <nav className='border-b'>
      <div className='container flex flex-col sm:flex-row  sm:justify-between sm:items-center flex-wrap gap-4 py-8'>
        <Logo />
        <NavSearch />
        <div className='flex gap-4 items-center '>
          <DarkMode />
          <LinksDropdown />
        </div>
      </div>
    </nav>
  )
}
export default Navbar
```

```tsx
import Navbar from '@/components/navbar/Navbar'

return (
  <html lang='en' suppressHydrationWarning>
    <body className={inter.className}>
      <Navbar />
      <main className='container py-10'>{children}</main>
    </body>
  </html>
)
```

### Logo

```sh
npm install react-icons
```

[﻿Lucide Icons](https://react-icons.github.io/react-icons/)

```tsx
import Link from 'next/link'
import { LuTent } from 'react-icons/lu'
import { Button } from '../ui/button'

function Logo() {
  return (
    <Button size='icon' asChild>
      <Link href='/'>
        <LuTent className='w-6 h-6' />
      </Link>
    </Button>
  )
}
```

### NavSearch

```tsx
import { Input } from '../ui/input'

function NavSearch() {
  return (
    <Input
      type='search'
      placeholder='find a property...'
      className='max-w-xs dark:bg-muted '
    />
  )
}
export default NavSearch
```

## style provide managed state managment

### Theme

[﻿Theming Options](https://ui.shadcn.com/docs/theming)
[﻿Themes](https://ui.shadcn.com/themes)

- replace css variables in in globals.css

### Providers

- create app/providers.tsx

```tsx
'use client'

function Providers({ children }: { children: React.ReactNode }) {
  return <>{children}</>
}
export default Providers
```

layout.tsx

```tsx
import Providers from './providers'

return (
  <html lang='en' suppressHydrationWarning>
    <body className={inter.className}>
      <Providers>
        <Navbar />
        <main className='container py-10'>{children}</main>
      </Providers>
    </body>
  </html>
)
```

### DarkMode

[﻿Next.js Dark Mode](https://ui.shadcn.com/docs/dark-mode/next)

```sh
npm install next-themes
```

- create app/theme-provider.tsx

```tsx
'use client'

import * as React from 'react'
import { ThemeProvider as NextThemesProvider } from 'next-themes'
import { type ThemeProviderProps } from 'next-themes/dist/types'

export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>
}
```

providers.tsx

```tsx
'use client'
import { ThemeProvider } from './theme-provider'

function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider
      attribute='class'
      defaultTheme='system'
      enableSystem
      disableTransitionOnChange
    >
      {children}
    </ThemeProvider>
  )
}
export default Providers
```

### DarkMode

- make sure you export as default !!!

```tsx
'use client'

import * as React from 'react'
import { MoonIcon, SunIcon } from '@radix-ui/react-icons'
import { useTheme } from 'next-themes'

import { Button } from '@/components/ui/button'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'

export default function ModeToggle() {
  const { setTheme } = useTheme()

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant='outline' size='icon'>
          <SunIcon className='h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0' />
          <MoonIcon className='absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100' />
          <span className='sr-only'>Toggle theme</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align='end'>
        <DropdownMenuItem onClick={() => setTheme('light')}>
          Light
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme('dark')}>
          Dark
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme('system')}>
          System
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
```

### UserIcon

```tsx
import { LuUser2 } from 'react-icons/lu'

function UserIcon() {
  return <LuUser2 className='w-6 h-6 bg-primary rounded-full text-white' />
}
export default UserIcon
```

## maintainable routing

### Links Data

- create utils/links.ts

```ts
type NavLink = {
  href: string
  label: string
}

export const links: NavLink[] = [
  { href: '/', label: 'home' },
  { href: '/favorites ', label: 'favorites' },
  { href: '/bookings ', label: 'bookings' },
  { href: '/reviews ', label: 'reviews' },
  { href: '/rentals/create ', label: 'create rental' },
  { href: '/rentals', label: 'my rentals' },
  { href: '/profile ', label: 'profile' },
]
```

### LinksDropdown

```tsx
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
  DropdownMenuSeparator,
} from '@/components/ui/dropdown-menu'
import { LuAlignLeft } from 'react-icons/lu'
import Link from 'next/link'
import { Button } from '../ui/button'
import UserIcon from './UserIcon'
import { links } from '@/utils/links'
import SignOutLink from './SignOutLink'

function LinksDropdown() {
  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant='outline' className='flex gap-4 max-w-[100px]'>
          <LuAlignLeft className='w-6 h-6' />
          <UserIcon />
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent className='w-52' align='start' sideOffset={10}>
        {links.map((link) => {
          return (
            <DropdownMenuItem key={link.href}>
              <Link href={link.href} className='capitalize w-full'>
                {link.label}
              </Link>
            </DropdownMenuItem>
          )
        })}
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
export default LinksDropdown
```

### Clerk

[﻿Clerk Docs](https://clerk.com/)
[﻿Clerk + Next.js Setup](https://clerk.com/docs/quickstarts/nextjs)

- create new application

```sh
npm install @clerk/nextjs
```

- create .env.local

```bash
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
```

In Next.js, environment variables that start with NEXT*PUBLIC* are exposed to the browser. This means they can be accessed in your front-end code.

For example, NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY can be used in both server-side and client-side code.

On the other hand, CLERK_SECRET_KEY is a server-side environment variable. It's not exposed to the browser, making it suitable for storing sensitive data like API secrets.

layout.tsx

```tsx
import { ClerkProvider } from '@clerk/nextjs'

return (
  <ClerkProvider>
    <html lang='en' suppressHydrationWarning>
      <body className={inter.className}>
        <Providers>
          <Navbar />
          <main className='container py-10'>{children}</main>
        </Providers>
      </body>
    </html>
  </ClerkProvider>
)
```

- create middleware.ts

```ts
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server'

const isProtectedRoute = createRouteMatcher([
  '/bookings(.*)',
  '/checkout(.*)',
  '/favorites(.*)',
  '/profile(.*)',
  '/rentals(.*)',
  '/reviews(.*)',
])

export default clerkMiddleware((auth, req) => {
  if (isProtectedRoute(req)) auth().protect()
})

export const config = {
  matcher: ['/((?!.*\\..*|_next).*)', '/', '/(api|trpc)(.*)'],
}
```

- restart dev server

### SignUp/SignIn and Customize Avatar (optional)

- customization
  - avatars

### Toast Component

[﻿Toast](https://ui.shadcn.com/docs/components/toast)

providers.tsx

```tsx
'use client'
import { ThemeProvider } from './theme-provider'
import { Toaster } from '@/components/ui/toaster'

function Providers({ children }: { children: React.ReactNode }) {
  return (
    <>
      <Toaster />
      <ThemeProvider
        attribute='class'
        defaultTheme='system'
        enableSystem
        disableTransitionOnChange
      >
        {children}
      </ThemeProvider>
    </>
  )
}
export default Providers
```

### SignOutLink

- redirectUrl

```tsx
'use client'

import { SignOutButton } from '@clerk/nextjs'
import { useToast } from '../ui/use-toast'

function SignOutLink() {
  const { toast } = useToast()
  const handleLogout = () => {
    toast({ description: 'You have been signed out.' })
  }
  return (
    <SignOutButton redirectUrl='/'>
      <button className='w-full text-left' onClick={handleLogout}>
        Logout
      </button>
    </SignOutButton>
  )
}
export default SignOutLink
```

###
