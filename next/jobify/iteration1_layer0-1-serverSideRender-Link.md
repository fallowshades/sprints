# Jobify

[dependencies]
- [Favicon](https://favicon.io/)
- [Undraw](https://undraw.co/)
- [Logo - Figma File](https://www.figma.com/community/file/1319010578601364983/jobify-logo-public)

- home link to dashboard

## Create New Next.js Project

```sh
npx create-next-app@latest projectName
```

- choose typescript and eslint

## Assets

- project repo
  - 03-jobify/assets

## Libraries

```sh
   npm install @clerk/nextjs@^4.27.7 @prisma/client@^5.7.0 @tanstack/react-query@^5.14.0 @tanstack/react-query-devtools@^5.14.0 dayjs@^1.11.10 next-themes@^0.2.1 recharts@^2.10.3
   npm install prisma@^5.7.0 -D
```

## shadcn/ui

[Docs](https://ui.shadcn.com/)

- follow Next.js install steps (starting with 2)
- open another terminal window (optional)

```sh
npx shadcn-ui@latest init
```

- setup Button

```sh
npx shadcn-ui@latest add button
```

[Icons](https://lucide.dev/guide/packages/lucide-react)

page.tsx

```tsx
import { Button } from '@/components/ui/button'
import { Camera } from 'lucide-react'

export default function Home() {
  return (
    <div className="h-screen flex items-center justify-center">
      <Button>default button</Button>
      <Button variant="outline" size="icon">
        <Camera />
      </Button>
    </div>
  )
}
```

## Challenge - Build the Home Page (app/page.tsx)

1. **Import necessary modules and components:**

   - Import the `Image` component from 'next/image' for displaying images.
   - Import the `Logo` and `LandingImg` SVG files from the assets directory.
   - Import the `Button` component from the UI components directory.
   - Import the `Link` component from 'next/link' for navigation.

2. **Define the `Home` component:**

   - This component doesn't receive any props.

3. **Inside the `Home` component, return the JSX:**

   - The main wrapper is a `main` HTML element.
   - Inside `main`, there are two main sections: `header` and `section`.
   - The `header` contains the `Image` component that displays the `Logo`.
   - The `section` contains a `div` and an `Image` component.
   - The `div` contains a `h1` heading, a `p` paragraph, and a `Button` component.
   - The `Button` component wraps a `Link` component that navigates to the '/add-job' route when clicked.
   - The `Image` component displays the `LandingImg`.

4. **Apply CSS classes for styling:**

   - CSS classes are applied to the elements for styling. These classes are from Tailwind CSS, a utility-first CSS framework.

5. **Export the `Home` component as the default export of the module.**

## Layout and Home Page

- setup title and description
- add favicon
- setup home page

layout.tsx

```tsx
export const metadata: Metadata = {
  title: 'Jobify Dev',
  description: 'Job application tracking system for job hunters',
}
```

page.tsx

```tsx
import Image from 'next/image'
import Logo from '../assets/logo.svg'
import LandingImg from '../assets/main.svg'
import { Button } from '@/components/ui/button'
import Link from 'next/link'
export default function Home() {
  return (
    <main>
      <header className="max-w-6xl mx-auto px-4 sm:px-8 py-6 ">
        <Image src={Logo} alt="logo" />
      </header>
      <section className="max-w-6xl mx-auto px-4 sm:px-8 h-screen -mt-20 grid lg:grid-cols-[1fr,400px] items-center">
        <div>
          <h1 className="capitalize text-4xl md:text-7xl font-bold">
            job <span className="text-primary">tracking</span> app
          </h1>
          <p className="leading-loose max-w-md mt-4 ">
            I am baby wayfarers hoodie next level taiyaki brooklyn cliche blue
            bottle single-origin coffee chia. Aesthetic post-ironic venmo,
            quinoa lo-fi tote bag adaptogen everyday carry meggings +1 brunch
            narwhal.
          </p>
          <Button asChild className="mt-4">
            <Link href="/add-job">Get Started</Link>
          </Button>
        </div>
        <Image src={LandingImg} alt="landing" className="hidden lg:block " />
      </section>
    </main>
  )
}
```

## Favicon and Logo (optional)

- [Favicon](https://favicon.io/)
- [Undraw](https://undraw.co/)
- [Logo - Figma File](https://www.figma.com/community/file/1319010578601364983/jobify-logo-public)

## Challenge - Setup Dashboard Pages

- create add-job, jobs and stats pages
- group them in (dashboard)
- setup a layout file (for now just pass children)

## Dashboard Pages

- create add-job, jobs and stats pages
- group them in (dashboard)
- setup a layout file (just pass children)

(dashboard)/layout.tsx

```tsx
function layout({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>
}
export default layout
```
