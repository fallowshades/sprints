# iteration 1 layer 3

[layout]

- map link and the global header

 [components]
 - split rendering work (FETCH ON SERVER, useState client)
  - data fetching
  - security
  - caching
  - bundle size

 - - [Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [Client Components](https://nextjs.org/docs/app/building-your-application/rendering/client-components)

[segment-all-X-id]
-  remote image 

## root

### Challenge - Navbar

- in the root create components folder
- create Navbar.jsx component
- import Link component
- setup a list of links (to existing pages)
- iterate over the list
- render Navbar in the app/layout.js
- setup the same layout for all pages (hint: wrap children prop)

### Setup Navbar

components/Navbar

```js
import Link from 'next/link'

const links = [
  { href: '/client', label: 'client' },
  { href: '/drinks', label: 'drinks' },
  { href: '/tasks', label: 'tasks' },
  { href: '/query', label: 'react-query' },
]

const Navbar = () => {
  return (
    <nav className="bg-base-300 py-4">
      <div className="navbar px-8 max-w-6xl mx-auto flex-col sm:flex-row">
        <li>
          <Link href="/" className="btn btn-primary">
            Next.js
          </Link>
        </li>
        <ul className="menu menu-horizontal md:ml-8">
          {links.map((link) => {
            return (
              <li key={link.href}>
                <Link href={link.href} className=" capitalize">
                  {link.label}
                </Link>
              </li>
            )
          })}
        </ul>
      </div>
    </nav>
  )
}

export default Navbar
```

app/layout

```js
import { Inter } from 'next/font/google'
import './globals.css'

// alias
import Navbar from '@/components/Navbar'

const inter = Inter({ subsets: ['latin'] })

export const metadata = {
  title: 'Next.js Tutorial',
  description: 'Build awesome stuff with Next.js!',
}

export default function RootLayout({ children }) {
  return (
    <html lang="en" className="bg-base-200">
      <body className={inter.className}>
        <Navbar />
        <main className="px-8 py-20 max-w-6xl mx-auto ">{children}</main>
      </body>
    </html>
  )
}
```

## components managment split

### Server Components VS Client Components

- [Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [Client Components](https://nextjs.org/docs/app/building-your-application/rendering/client-components)

- BY DEFAULT, NEXT.JS USES SERVER COMPONENTS !!!!
- To use Client Components, you can add the React "use client" directive

#### Server Components

Benefits :

- data fetching
- security
- caching
- bundle size

Data Fetching: Server Components allow you to move data fetching to the server, closer to your data source. This can improve performance by reducing time it takes to fetch data needed for rendering, and the amount of requests the client needs to make.
Security: Server Components allow you to keep sensitive data and logic on the server, such as tokens and API keys, without the risk of exposing them to the client.
Caching: By rendering on the server, the result can be cached and reused on subsequent requests and across users. This can improve performance and reduce cost by reducing the amount of rendering and data fetching done on each request.
Bundle Sizes: Server Components allow you to keep large dependencies that previously would impact the client JavaScript bundle size on the server. This is beneficial for users with slower internet or less powerful devices, as the client does not have to download, parse and execute any JavaScript for Server Components.
Initial Page Load and First Contentful Paint (FCP): On the server, we can generate HTML to allow users to view the page immediately, without waiting for the client to download, parse and execute the JavaScript needed to render the page.
Search Engine Optimization and Social Network Shareability: The rendered HTML can be used by search engine bots to index your pages and social network bots to generate social card previews for your pages.
Streaming: Server Components allow you to split the rendering work into chunks and stream them to the client as they become ready. This allows the user to see parts of the page earlier without having to wait for the entire page to be rendered on the server.

#### Client Components

Benefits :

- Interactivity: Client Components can use state, effects, and event listeners, meaning they can provide immediate feedback to the user and update the UI.
- Browser APIs: Client Components have access to browser APIs, like geolocation or localStorage, allowing you to build UI for specific use cases.

### Challenge - Setup Counter

- setup our home page (no restrictions)
- setup a simple counter in app/client/page.js

#### Home Page

```js
import Link from 'next/link'

export default function Home() {
  return (
    <div>
      <h1 className="text-5xl mb-8 font-bold">Next.js Tutorial</h1>
      <Link href="/client" className="btn btn-accent">
        get started
      </Link>
    </div>
  )
}
```

#### Counter ( Client Component)

```js
// try without
'use client'
import { useState } from 'react'
const Client = () => {
  const [count, setCount] = useState(0)
  return (
    <div>
      <h1 className="text-7xl font-bold mb-4 ">{count}</h1>
      <button className="btn btn-primary" onClick={() => setCount(count + 1)}>
        increase
      </button>
    </div>
  )
}
export default Client
```

### Fetch Data in Server Components

- just add async and start using await 🚀🚀🚀
- the same for db
- Next.js extends the native Web fetch() API to allow each request on the server to set its own persistent caching semantics.

```js
const url = 'someUrl'

const ServerComponent = async () => {
  const response = await fetch(url)
  const data = await response.json()
  console.log(data)
  return (
    <div>
      <h1 className="text-7xl">DrinksPage</h1>
    </div>
  )
}
export default ServerComponent
```

### Challenge - Fetch Data in Drinks Page

- setup logic
- visit the page
- look for log in the terminal 😀

app/drinks/page.js

```js
const url = 'https://www.thecocktaildb.com/api/json/v1/1/search.php?f=a'

const DrinksPage = async () => {
  const response = await fetch(url)
  const data = await response.json()
  console.log(data)
  return (
    <div>
      <h1 className="text-7xl">DrinksPage</h1>
    </div>
  )
}
export default DrinksPage
```

## segment and links in all page

### Loading Component

The special file loading.js helps you create meaningful Loading UI with React Suspense. With this convention, you can show an instant loading state from the server while the content of a route segment loads. The new content is automatically swapped in once rendering is complete.

- drinks/loading.js

```js
const loading = () => {
  return <span className="loading"></span>
}
export default loading
```

- refactor drinks page

```js
const fetchDrinks = async () => {
  // just for demo purposes
  await new Promise((resolve) => setTimeout(resolve, 1000))
  const response = await fetch(url)
  const data = await response.json()
  return data
}

const DrinksPage = async () => {
  const data = await fetchDrinks()
  console.log(data)
  return (
    <div>
      <h1 className="text-7xl">DrinksPage</h1>
    </div>
  )
}
export default DrinksPage
```

### Error Component

The error.js file convention allows you to gracefully handle unexpected runtime errors in nested routes.

- drinks/error.js
- 'use client'

```js
'use client'
const error = () => {
  return <div>there was an error...</div>
}
export default error
```

- refactor drinks (optional)

```js
// modify the url
const url = 'https://www.thecocktaildb.com/api/json/v1/1/search.phps?f=a'

const fetchDrinks = async () => {
  // just for demo purposes
  await new Promise((resolve) => setTimeout(resolve, 1000))
  const response = await fetch(url)
  // throw error
  if (!response.ok) {
    throw new Error('Failed to fetch drinks...')
  }
  const data = await response.json()
  return data
}
```

- DON'T FORGET TO FIX THE URL 😜😜😜

### Nested Layouts

- create app/drinks/layout.js
- UI will be applied to app/drinks - segment
- don't forget about the "children"
- we can fetch data in the layout but...
  at the moment can't pass data down to children (pages) 😞

  layout.js

```js
export default function DrinksLayout({ children }) {
  return (
    <div className="max-w-xl ">
      <div className="mockup-code mb-8">
        <pre data-prefix="$">
          <code>npx create-next-app@latest nextjs-tutorial</code>
        </pre>
      </div>
      {children}
    </div>
  )
}
```

### Dynamic Routes

- app/drinks/[id]/page.js

```js
const page = ({ params }) => {
  console.log(params)

  return (
    <div>
      <h1 className="text-7xl">{params.id}</h1>
    </div>
  )
}
export default page
```

### Challenge - Drinks List

- render a list of drinks
- each item is a link to a drink[id] page

### Drinks List

- create components/DrinksList.jsx
- render in drinks/page and pass the drinks down
- iterate over the drinks and return links

DrinksList.jsx

```js
import Link from 'next/link'

const DrinksList = ({ drinks }) => {
  return (
    <ul className="menu menu-vertical pl-0">
      {drinks.map((drink) => (
        <li key={drink.idDrink}>
          <Link
            href={`/drinks/${drink.idDrink}`}
            className="text-xl font-medium"
          >
            {drink.strDrink}
          </Link>
        </li>
      ))}
    </ul>
  )
}
export default DrinksList
```

app/drinks/page.js

```js
const url = 'https://www.thecocktaildb.com/api/json/v1/1/search.php?f=a'
import DrinksList from '@/components/DrinksList'
const fetchDrinks = async () => {
  // just for demo purposes
  await new Promise((resolve) => setTimeout(resolve, 1000))
  const response = await fetch(url)

  if (!response.ok) {
    throw new Error('Failed to fetch drinks...')
  }
  const data = await response.json()
  return data
}

const DrinksPage = async () => {
  const data = await fetchDrinks()

  return (
    <div>
      <DrinksList drinks={data.drinks} />
    </div>
  )
}
export default DrinksPage
```

## slug parameter prop

### Challenge - Render Single Drink

- in drinks/[id], fetch and render drink title
- hint:params object

```js
const url = 'https://www.thecocktaildb.com/api/json/v1/1/lookup.php?i='
```

### Render Single Drink

drinks/[id]/page.js

```js
import Link from 'next/link'
const url = 'https://www.thecocktaildb.com/api/json/v1/1/lookup.php?i='

const getSingleDrink = async (id) => {
  const res = await fetch(`${url}${id}`)

  if (!res.ok) {
    throw new Error('Failed to fetch drink...')
  }
  return res.json()
}

const SingleDrink = async ({ params }) => {
  const data = await getSingleDrink(params.id)
  const title = data?.drinks[0]?.strDrink
  const imgSrc = data?.drinks[0]?.strDrinkThumb
  return (
    <div>
      <Link href="/drinks" className="btn btn-primary mt-8 mb-12">
        back to drinks
      </Link>
      <h1 className="text-4xl mb-8">{title}</h1>
    </div>
  )
}
export default SingleDrink
```

### Next Image Component

- get random image from pexels site
  [Random Drink](https://www.pexels.com/photo/red-glass-with-garnished-beverage-2842876/)

The Next.js Image component extends the HTML <img> element with features for automatic image optimization:

- Size Optimization: Automatically serve correctly sized images for each device, using modern image formats like WebP and AVIF.
- Visual Stability: Prevent layout shift automatically when images are loading.
- Faster Page Loads: Images are only loaded when they enter the viewport using native browser lazy loading, with optional blur-up placeholders.
- Asset Flexibility: On-demand image resizing, even for images stored on remote servers

drinks[id]/page.js

```js
import Link from 'next/link'
const url = 'https://www.thecocktaildb.com/api/json/v1/1/lookup.php?i='

import drinkImg from './drink.jpg'
import Image from 'next/image'

const SingleDrink = async ({ params }) => {
  const data = await getSingleDrink(params.id)
  const title = data?.drinks[0]?.strDrink
  const imgSrc = data?.drinks[0]?.strDrinkThumb
  return (
    <div>
      <Link href="/drinks" className="btn btn-primary mt-8 mb-12">
        back to drinks
      </Link>
      {/* <img src={imgSrc} alt='' /> */}
      <Image src={drinkImg} className="w-48 h-48 rounded" alt="" />
      <h1 className="text-4xl mb-8">{title}</h1>
    </div>
  )
}
export default SingleDrink
```

### Remote Images

- To use a remote image, the src property should be a URL string.

- Since Next.js does not have access to remote files during the build process, you'll need to provide the width, height and optional blurDataURL props manually.

- The width and height attributes are used to infer the correct aspect ratio of image and avoid layout shift from the image loading in. The width and height do not determine the rendered size of the image file.

- To safely allow optimizing images, define a list of supported URL patterns in next.config.js. Be as specific as possible to prevent malicious usage.

- restart dev server

- priority property to prioritize the image for loading

```js
const SingleDrink = async ({ params }) => {
  const data = await getSingleDrink(params.id)
  const title = data?.drinks[0]?.strDrink
  const imgSrc = data?.drinks[0]?.strDrinkThumb
  return (
    <div>
      <Link href="/drinks" className="btn btn-primary mt-8 mb-12">
        back to drinks
      </Link>
      {/* <img src={imgSrc} alt='' /> */}
      {/* <Image src={drinkImg} className='w-48 h-48 rounded' alt='' /> */}
      <Image
        src={imgSrc}
        width={300}
        height={300}
        className="w-48 h-48 rounded shadow-lg mb-4"
        priority
        alt=""
      />
      <h1 className="text-4xl mb-8">{title}</h1>
    </div>
  )
}
export default SingleDrink
```

```js
** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'www.thecocktaildb.com',
        port: '',
        pathname: '/images/**',
      },
    ],
  },
};

module.exports = nextConfig;
```

### Remote Images - Responsive

- The fill prop allows your image to be sized by its parent element
- sizes property helps the browser select the most appropriate image size to load based on the user's device and screen size, improving website performance and user experience.

  DrinksList.jsx

```js
import Image from 'next/image'
import Link from 'next/link'

const DrinksList = ({ drinks }) => {
  return (
    <ul className="grid sm:grid-cols-2 gap-6 mt-6">
      {drinks.map((drink) => (
        <li key={drink.idDrink}>
          <Link
            href={`/drinks/${drink.idDrink}`}
            className="text-xl font-medium"
          >
            <div className="relative h-48 mb-4">
              <Image
                src={drink.strDrinkThumb}
                fill
                sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw"
                alt={drink.strDrink}
                className="rounded-md object-cover"
              />
            </div>
            {drink.strDrink}
          </Link>
        </li>
      ))}
    </ul>
  )
}
export default DrinksList
```
