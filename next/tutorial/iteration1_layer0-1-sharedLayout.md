# Final App

[root]

- home page link folder
- css plugin
- inter meta data

##

[Tutorial](https://nextjs-tutorial-production.vercel.app/)

## folder page

### Node Version

The minimum Node.js version has been bumped from 16.14 to 18.17, since 16.x has reached end-of-life.

### Create Next App

```sh
npx create-next-app@latest appName
```

### Folder Structure

- app folder - where we will spend most of our time
  - setup routes, layouts, loading states, etc
- node_modules - project dependencies
- public - static assets
- .gitignore - sets which will be ignored by source control
- bunch of config files (will discuss as we go)
- in package.json look scripts
- 'npm run dev' to spin up the project on http://localhost:3000

```sh
npm run dev
```

### Home Page

- page.js in the root of app folder
- represents root of our application
  '/' local domain or production domain
- react component (server component)
- bunch of css classes (will discuss Tailwind in few lectures)
- export component as default
- file name "page" has a special meaning
- snippets extension

  app/page.js

```js
const HomePage = () => {
  return (
    <div>
      <h1 className="text-7xl">HomePage</h1>
    </div>
  )
}
export default HomePage
```

### Create Pages in Next.js

- in the app folder create a folder with the page.js file
  - about/page.js
  - contact/page.js
- can have .js .jsx .tsx extension

app/about/page.js

```js
const AboutPage = () => {
  return (
    <div>
      <h1 className="text-7xl">AboutPage</h1>
    </div>
  )
}
export default AboutPage
```

### Link Component

- navigate around the project
- import Link from 'next/link'
  home page

```js
import Link from 'next/link'

const HomePage = () => {
  return (
    <div>
      <h1 className="text-7xl">HomePage</h1>
      <Link href="/about" className="text-2xl">
        about page
      </Link>
    </div>
  )
}
export default HomePage
```

## children outlet (header)

### Nested Routes

- app/about/info/page.js
- if no page.js in a segment will result in 404

```js
const AboutInfoPage = () => {
  return <h1 className="text-7xl">AboutInfoPage</h1>
}
export default AboutInfoPage
```

## Challenge

- create following pages :
  - client, drinks, prisma-example, query and tasks

## Tailwind and DaisyUI

- both videos optional

- remove extra code in globals.css
- install daisyui
- install tailwindcss typography plugin
- configure tailwind

```sh
npm i -D daisyui@latest
npm i @tailwindcss/typography
```

tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  ...
  plugins: [require('@tailwindcss/typography'), require('daisyui')]
};

```

## Layouts and Templates

- layout.js
- template.js

  Layout is a component which wraps other pages and layouts. Allow to share UI. Even when the route changes, layout DOES NOT re-render. Can fetch data but can't pass it down to children. Templates are the same but they re-render.

- the top-most layout is called the Root Layout. This required layout is shared across all pages in an application. Root layouts must contain html and body tags.
- any route segment can optionally define its own Layout. These layouts will be shared across all pages in that segment.
- layouts in a route are nested by default. Each parent layout wraps child layouts below it using the React children prop.

```js
import { Inter } from 'next/font/google'
import './globals.css'

const inter = Inter({ subsets: ['latin'] })

export const metadata = {
  title: 'Next.js Tutorial',
  description: 'Build awesome stuff with Next.js!',
}

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  )
}
```

## Challenge - Navbar

- in the root create components folder
- create Navbar.jsx component
- import Link component
- setup a list of links (to existing pages)
- iterate over the list
- render Navbar in the app/layout.js
- setup the same layout for all pages (hint: wrap children prop)
