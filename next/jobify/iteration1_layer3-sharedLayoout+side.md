#

##

[ux-active-page]
- links and auth matcher
usePathname to highlight
- sidebar and nav

[component-lib]
- span sr-only dropdown link
- useTheme context --> set theme

[Theming](https://ui.shadcn.com/docs/theming)
[Themes](https://ui.shadcn.com/themes)

  [Dark Mode](https://ui.shadcn.com/docs/dark-mode/next)
[Dark Mode](https://ui.shadcn.com/docs/dark-mode/next)


## sidebar active page

### Challenge - Add Clerk Auth

- setup new app, configure fields - (or use existing)
- add ENV Vars
- wrap layout in Clerk Provider
- add middleware
- set only home page public
- restart dev server

### Clerk Auth

- setup new app, configure fields - (or use existing)
- add ENV Vars
- wrap layout
- add middleware
- make '/' public
- restart dev server

layout.tsx

```tsx
import { ClerkProvider } from '@clerk/nextjs'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body className={inter.className}>{children}</body>
      </html>
    </ClerkProvider>
  )
}
```

middleware.tsx

```tsx
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

### Challenge - Build the links.tsx Component

1. **Create File and Import necessary modules and components:**

   - create utils/links.tsx
   - Import the `AreaChart`, `Layers`, and `AppWindow` components from 'lucide-react' for displaying icons.

2. **Define the `NavLink` type:**

   - This type has three properties: `href` (a string), `label` (a string), and `icon` (a React Node).

3. **Define the `links` constant:**

   - This constant is an array of `NavLink` objects.
   - Each object represents a navigation link with a `href`, `label`, and `icon`.

4. **Define the navigation links:**

   - The first link has a `href` of '/add-job', a `label` of 'add job', and an `icon` of `<Layers />`.
   - The second link has a `href` of '/jobs', a `label` of 'all jobs', and an `icon` of `<AppWindow />`.
   - The third link has a `href` of '/stats', a `label` of 'stats', and an `icon` is not defined yet.

5. **Export the `links` constant:**
   - This constant can be imported in other components to create navigation menus.

### Links Data

- create utils/links.tsx

utils/links.tsx

```tsx
import { AreaChart, Layers, AppWindow } from 'lucide-react'

type NavLink = {
  href: string
  label: string
  icon: React.ReactNode
}

const links: NavLink[] = [
  {
    href: '/add-job',
    label: 'add job',
    icon: <Layers />,
  },
  {
    href: '/jobs',
    label: 'all jobs',
    icon: <AppWindow />,
  },
  {
    href: '/stats',
    label: 'stats',
    icon: <AreaChart />,
  },
]

export default links
```

### Challenge - Dashboard Layout

- create following components :

  - Sidebar
  - Navbar
  - LinksDropdown
  - ThemeToggle

- setup (dashboard/layout.tsx)

1. **Import necessary modules and components:**

   - Import `Navbar` and `Sidebar` components.
   - Import `PropsWithChildren` from 'react'.

2. **Define the `layout` component:**

   - This component receives `children` as props.

3. **Return the JSX:**

   - The main wrapper is a `main` element with a grid layout.
   - The first `div` contains the `Sidebar` component and is hidden on small screens.
   - The second `div` spans 4 columns on large screens and contains the `Navbar` component and the `children`.

4. **Export the `layout` component.**
   dashboard/layout.tsx

### Dashboard Layout

- create following components :

  - Sidebar
  - Navbar
  - LinksDropdown
  - ThemeToggle

(dashboard/layout.tsx)

```tsx
import Navbar from '@/components/Navbar'
import Sidebar from '@/components/Sidebar'

import { PropsWithChildren } from 'react'

function layout({ children }: PropsWithChildren) {
  return (
    <main className="grid lg:grid-cols-5">
      {/* first-col hide on small screen */}
      <div className="hidden lg:block lg:col-span-1 lg:min-h-screen">
        <Sidebar />
      </div>
      {/* second-col hide dropdown on big screen */}

      <div className="lg:col-span-4">
        <Navbar />
        <div className="py-16 px-4 sm:px-8 lg:px-16">{children}</div>
      </div>
    </main>
  )
}
export default layout
```

### Challenge - Build Sidebar Component

1. **Import necessary modules and components:**

   - Import `Logo`, `links`, `Image`, `Link`, `Button`, and `usePathname`.

2. **Define the `Sidebar` component:**

   - Use `usePathname` to get the current route.

3. **Return the JSX:**

   - The main wrapper is an `aside` element.
   - Inside `aside`, display the `Logo` using `Image`.
   - Map over `links` to create `Button` components for each link.
   - Each `Button` wraps a `Link` that navigates to the link's `href`.

4. **Export the `Sidebar` component.**

### Sidebar

- render links and logo
- check the path, if active use different variant
  Sidebar.tsx

```tsx
'use client'
import Logo from '@/assets/logo.svg'
import links from '@/utils/links'
import Image from 'next/image'
import Link from 'next/link'
import { Button } from './ui/button'
import { usePathname } from 'next/navigation'
function Sidebar() {
  const pathname = usePathname()

  return (
    <aside className="py-4 px-8 bg-muted h-full">
      <Image src={Logo} alt="logo" className="mx-auto" />
      <div className="flex flex-col mt-20 gap-y-4">
        {links.map((link) => {
          return (
            <Button
              asChild
              key={link.href}
              variant={pathname === link.href ? 'default' : 'link'}
            >
              <Link href={link.href} className="flex items-center gap-x-2 ">
                {link.icon} <span className="capitalize">{link.label}</span>
              </Link>
            </Button>
          )
        })}
      </div>
    </aside>
  )
}
export default Sidebar
```

### Challenge - Build Navbar Component

1. **Import necessary modules and components:**

   - Import `LinksDropdown`, `UserButton` from '@clerk/nextjs', and `ThemeToggle`.

2. **Define the `Navbar` component:**

   - This component doesn't receive any props.

3. **Return the JSX:**

   - The main wrapper is a `nav` element with Tailwind CSS classes for styling.
   - Inside `nav`, there are two `div` elements.
   - The first `div` contains the `LinksDropdown` component.
   - The second `div` contains the `ThemeToggle` and `UserButton` components.

4. **Export the `Navbar` component.**

### Navbar

Navbar.tsx

```tsx
import LinksDropdown from './LinksDropdown'
import { UserButton } from '@clerk/nextjs'
import ThemeToggle from './ThemeToggle'

function Navbar() {
  return (
    <nav className="bg-muted py-4 sm:px-16 lg:px-24 px-4 flex items-center justify-between">
      <div>
        <LinksDropdown />
      </div>
      <div className="flex items-center gap-x-4">
        <ThemeToggle />
        <UserButton afterSignOutUrl="/" />
      </div>
    </nav>
  )
}
export default Navbar
```

##

## Challenge - Build LinksDropdown Component

1. **Explore the Dropdown-Menu Component:**

   - Explore the dropdown-menu component in the shadcn library.

2. **Install the Dropdown-Menu Component:**

   - Install it using `npx shadcn-ui@latest add dropdown-menu`

3. **Import necessary modules and components:**

   - Import `DropdownMenu`, `DropdownMenuContent`, `DropdownMenuItem`, `DropdownMenuTrigger` from the dropdown-menu component.
   - Import `AlignLeft` from 'lucide-react' for the menu icon.
   - Import `Button` from the local UI components.
   - Import `links` from the local utilities.
   - Import `Link` from 'next/link' for navigation.

4. **Define the `DropdownLinks` function component:**

   - This component doesn't receive any props.

5. **Inside the `DropdownLinks` component, return the JSX:**

   - The main wrapper is the `DropdownMenu` component.
   - Inside `DropdownMenu`, there is a `DropdownMenuTrigger` component that triggers the dropdown menu. It has a `Button` component with an `AlignLeft` icon. This button is hidden on large screens.
   - The `DropdownMenuContent` component contains the dropdown menu items. Each item is a `DropdownMenuItem` component that wraps a `Link` component. The `Link` component navigates to the link's `href` when clicked.

6. **Export the `DropdownLinks` component:**
   - The `DropdownLinks` component is exported as the default export of the module. This allows it to be imported in other files using the file path.

## LinksDropdown

- [docs](https://ui.shadcn.com/docs/components/dropdown-menu)

```sh
npx shadcn-ui@latest add dropdown-menu
```

LinksDropdown.tsx

```tsx
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'
import { AlignLeft } from 'lucide-react'
import { Button } from './ui/button'
import links from '@/utils/links'
import Link from 'next/link'
function DropdownLinks() {
  return (*/=
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
            <DropdownMenuItem key={link.href}>
              <Link href={link.href} className="flex items-center gap-x-2 ">
                {link.icon} <span className="capitalize">{link.label}</span>
              </Link>
            </DropdownMenuItem>
          )
        })}
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
export default DropdownLinks
```

## Challenge - Add New Theme

- reference shadcn docs

[Theming](https://ui.shadcn.com/docs/theming)
[Themes](https://ui.shadcn.com/themes)

- setup theme in globals.css

## Challenge - Setup providers.tsx

- create providers.tsx
- wrap children in layout

## Providers

- create providers.tsx
- wrap children in layout
- add suppressHydrationWarning prop

app/providers.tsx

```tsx
'use client'

const Providers = ({ children }: { children: React.ReactNode }) => {
  return <>{children}</>
}
export default Providers
```

app/layout

```tsx
<html lang="en" suppressHydrationWarning>
  <body className={inter.className}>
    <Providers>{children}</Providers>
  </body>
</html>
```

## Challenge - Add Dark Mode

- reference shadcn docs and setup dark theme
  [Dark Mode](https://ui.shadcn.com/docs/dark-mode/next)

## Dark Mode

[Dark Mode](https://ui.shadcn.com/docs/dark-mode/next)

```sh
npm install next-themes

```

components/theme-provider.tsx

```tsx
'use client'

import * as React from 'react'
import { ThemeProvider as NextThemesProvider } from 'next-themes'
import { type ThemeProviderProps } from 'next-themes/dist/types'

export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>
}
```

app/providers.tsx

```tsx
'use client'
import { ThemeProvider } from '@/components/theme-provider'
const Providers = ({ children }: { children: React.ReactNode }) => {
  return (
    <>
      <ThemeProvider
        attribute="class"
        defaultTheme="system"
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

ThemeToggle.tsx

```tsx
'use client'

import * as React from 'react'
import { Moon, Sun } from 'lucide-react'
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
        <Button variant="outline" size="icon">
          <Sun className="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
          <Moon className="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
          <span className="sr-only">Toggle theme</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
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
