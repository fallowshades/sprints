## Landing Components

- in src/components create :
  - Hero
  - HeroCarousel
  - FeaturedProducts
  - SectionTitle
  - ProductsGrid
- setup export

Landing.tsx

```tsx
import { Hero, FeaturedProducts } from '@/components'

function Landing() {
  return (
    <>
      <Hero />
      <FeaturedProducts />
    </>
  )
}
export default Landing
```

## CustomFetch

- create utils/customFetch.ts and setup export

- [API DOCS](https://documenter.getpostman.com/view/18152321/2s9Xy5KpTi)

```ts
import axios from 'axios'

const productionUrl = 'https://strapi-store-server.onrender.com/api'

export const customFetch = axios.create({
  baseURL: productionUrl,
})
```

## Products Types

- create utils/types.ts and setup export

```ts
export type ProductsResponse = {
  data: Product[]
  meta: ProductsMeta
}

export type Product = {
  id: number
  attributes: {
    category: string
    company: string
    createdAt: string
    description: string
    featured: boolean
    image: string
    price: string
    publishedAt: string
    shipping: boolean
    title: string
    updatedAt: string
    colors: string[]
  }
}

export type ProductsMeta = {
  categories: string[]
  companies: string[]
  pagination: Pagination
}

export type Pagination = {
  page: number
  pageCount: number
  pageSize: number
  total: number
}
```

## Loaders

Loaders in React Router are functions that handle the loading of data or components asynchronously before a route is rendered.

```tsx
{
  index: true,
  element: <Landing />,
  loader: () => {
  console.log('landing page');
  // need to return something (at least null)
  return null;
  },
},
```

## Landing Loader

```tsx
import { FeaturedProducts, Hero } from '@/components'
import { customFetch, type ProductsResponse } from '@/utils'
import { useLoaderData, type LoaderFunction } from 'react-router-dom'

const url = '/products?featured=true'

export const loader: LoaderFunction = async (): Promise<ProductsResponse> => {
  const response = await customFetch<ProductsResponse>(url)
  return { ...response.data }
}

function Landing() {
  const result = useLoaderData() as ProductsResponse
  console.log(result)

  return (
    <>
      <Hero />
      <FeaturedProducts />
    </>
  )
}
export default Landing
```

App.tsx

```tsx
{
  index: true,
  element: <Landing />,
  loader: landingLoader,
  errorElement: <ErrorElement />,
},
```

## Separator Component - Shadcn

[Separator](https://ui.shadcn.com/docs/components/separator)

```sh
npx shadcn-ui@latest add separator
```

## SectionTitle

```tsx
import { Separator } from '@/components/ui/separator'

const SectionTitle = ({ text }: { text: string }) => {
  return (
    <div>
      <h2 className="text-3xl font-medium tracking-wider capitalize mb-8">
        {text}
      </h2>
      <Separator />
    </div>
  )
}
export default SectionTitle
```

## FeaturedProducts

```tsx
import ProductsGrid from './ProductsGrid'
import SectionTitle from './SectionTitle'
const FeaturedProducts = () => {
  return (
    <section className="pt-24 ">
      <SectionTitle text="featured products" />
      <ProductsGrid />
    </section>
  )
}
export default FeaturedProducts
```

## Format Price

- create utils/formatAsDollars and setup export

```ts
export const formatAsDollars = (price: string | number): string => {
  const dollarsAmount = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(Number(price) / 100)
  return dollarsAmount
}
```

## Card Component - Shadcn

[Card ](https://ui.shadcn.com/docs/components/card)

```sh
npx shadcn-ui@latest add card

```

## ProductsGrid

```tsx
import { Link, useLoaderData } from 'react-router-dom'
import { Card, CardContent } from '@/components/ui/card'
import { formatAsDollars, ProductsResponse } from '@/utils'
const ProductsGrid = () => {
  const { data: products } = useLoaderData() as ProductsResponse

  return (
    <div className="pt-12 grid gap-4 md:grid-cols-2 lg:grid-cols-3 ">
      {products.map((product) => {
        const { title, price, image } = product.attributes
        const dollarsAmount = formatAsDollars(price)

        return (
          <Link to={`/products/${product.id}`} key={product.id}>
            <Card>
              <CardContent className="p-4">
                <img
                  src={image}
                  alt={title}
                  className="rounded-md h-64 md:h-48 w-full object-cover"
                />
                <div className="mt-4 text-center">
                  <h2 className="text-xl font-semibold capitalize">{title}</h2>
                  <p className="text-primary font-light mt-2">
                    {dollarsAmount}
                  </p>
                </div>
              </CardContent>
            </Card>
          </Link>
        )
      })}
    </div>
  )
}
export default ProductsGrid
```

## Hero

```tsx
import { Link } from 'react-router-dom'
import { Button } from './ui/button'
import HeroCarousel from './HeroCarousel'
const Hero = () => {
  return (
    <section className=" grid grid-cols-1 lg:grid-cols-2 gap-24 items-center">
      <div>
        <h1 className="max-w-2xl text-4xl font-bold tracking-tight  sm:text-6xl ">
          We’re changing the way people shop.
        </h1>

        <p className="mt-8 max-w-xl text-lg leading-8">
          Anim aute id magna aliqua ad ad non deserunt sunt. Qui irure qui lorem
          cupidatat commodo. Elit sunt amet fugiat veniam occaecat fugiat
          aliqua. Anim aute id magna aliqua ad ad non deserunt sunt. Qui irure
          qui lorem cupidatat commodo.
        </p>

        <Button asChild size="lg" className="mt-10">
          <Link to="/products">Our Products</Link>
        </Button>
      </div>
      {/* hero carousel */}
      <HeroCarousel />
    </section>
  )
}
export default Hero
```

## Carousel Component - Shadcn

[Carousel](https://ui.shadcn.com/docs/components/carousel)

```sh
npx shadcn-ui@latest add carousel

```

## HeroCarousel

- get assets from final

```tsx
import {
  Carousel,
  CarouselContent,
  CarouselItem,
  CarouselNext,
  CarouselPrevious,
} from '@/components/ui/carousel'
import { Card, CardContent } from '@/components/ui/card'

import hero1 from '../assets/hero1.webp'
import hero2 from '../assets/hero2.webp'
import hero3 from '../assets/hero3.webp'
import hero4 from '../assets/hero4.webp'

const carouselImages = [hero1, hero2, hero3, hero4]

function HeroCarousel() {
  return (
    <div className="hidden lg:block">
      <Carousel>
        <CarouselContent>
          {carouselImages.map((image, index) => (
            <CarouselItem key={index}>
              <Card>
                <CardContent className="p-2">
                  <img
                    src={image}
                    alt="hero"
                    className="w-full h-[24rem]  rounded-md object-cover"
                  />
                </CardContent>
              </Card>
            </CarouselItem>
          ))}
        </CarouselContent>
        <CarouselPrevious />
        <CarouselNext />
      </Carousel>
    </div>
  )
}
export default HeroCarousel
```

## Products Page - Setup

- create following components and setup export
  - Filters
  - ProductsContainer
  - PaginationContainer
  - ProductsList

## Products Page

- don't forget to import and setup loader in the App.tsx

```tsx
import { Filters, ProductsContainer, PaginationContainer } from '@/components'
import { customFetch, type ProductsResponse } from '../utils'
import { type LoaderFunction } from 'react-router-dom'

const url = '/products'

export const loader: LoaderFunction = async (): Promise<ProductsResponse> => {
  const response = await customFetch<ProductsResponse>(url)

  return { ...response.data }
}

const Products = () => {
  return (
    <>
      <Filters />
      <ProductsContainer />
      <PaginationContainer />
    </>
  )
}
export default Products
```

## ProductsList

```tsx
import { formatAsDollars, type ProductsResponse } from '@/utils'
import { Link, useLoaderData } from 'react-router-dom'
import { Card, CardContent } from './ui/card'
const ProductList = () => {
  const { data: products } = useLoaderData() as ProductsResponse
  return (
    <div className="mt-12 grid gap-y-8">
      {products.map((product) => {
        const { title, price, image, company } = product.attributes
        const dollarsAmount = formatAsDollars(price)

        return (
          <Link key={product.id} to={`/products/${product.id}`}>
            <Card>
              <CardContent className="p-8 gap-y-4 grid md:grid-cols-3 ">
                <img
                  src={image}
                  alt={title}
                  className="h-64 w-full md:h-48  md:w-48  rounded-md object-cover"
                />
                <div>
                  <h2 className="text-xl font-semibold capitalize">{title}</h2>
                  <h4>{company}</h4>
                </div>
                <p className="text-primary md:ml-auto">{dollarsAmount}</p>
              </CardContent>
            </Card>
          </Link>
        )
      })}
    </div>
  )
}

export default ProductList
```

## ProductsContainer

```tsx
import { useLoaderData } from 'react-router-dom'
import ProductsGrid from './ProductsGrid'
import ProductsList from './ProductsList'
import { useState } from 'react'
import { LayoutGrid, List } from 'lucide-react'
import { ProductsResponse } from '@/utils'
import { Button } from './ui/button'
import { Separator } from './ui/separator'

const ProductsContainer = () => {
  const { meta } = useLoaderData() as ProductsResponse
  const totalProducts = meta.pagination.total
  const [layout, setLayout] = useState<'grid' | 'list'>('grid')

  return (
    <>
      {/* HEADER */}
      <section>
        <div className="flex justify-between items-center mt-8 ">
          <h4 className="font-medium text-md">
            {totalProducts} product{totalProducts > 1 && 's'}
          </h4>
          <div className="flex gap-x-4">
            <Button
              onClick={() => setLayout('grid')}
              variant={layout === 'grid' ? 'default' : 'ghost'}
              size="icon"
            >
              <LayoutGrid />
            </Button>
            <Button
              onClick={() => setLayout('list')}
              size="icon"
              variant={layout === 'list' ? 'default' : 'ghost'}
            >
              <List />
            </Button>
          </div>
        </div>
        <Separator className="mt-4" />
      </section>

      {/* PRODUCTS */}
      <div>
        {totalProducts === 0 ? (
          <h5 className="text-2xl mt-16">
            Sorry, no products matched your search...
          </h5>
        ) : layout === 'grid' ? (
          <ProductsGrid />
        ) : (
          <ProductsList />
        )}
      </div>
    </>
  )
}

export default ProductsContainer
```

## Skeleton Component - Shadcn

[Skeleton](https://ui.shadcn.com/docs/components/skeleton)

```sh
npx shadcn-ui@latest add skeleton
```

## Loading Component

- create components/Loading.tsx

```tsx
import { Skeleton } from './ui/skeleton'

function Loading() {
  return (
    <div className="pt-12 grid gap-4 md:grid-cols-2 lg:grid-cols-3 ">
      {Array.from({ length: 3 }).map((_, index) => {
        return (
          <div key={index} className="flex flex-col space-y-3">
            <Skeleton className="h-[125px] w-full rounded-xl" />
            <div className="space-y-2">
              <Skeleton className="h-4 mx-auto w-[250px]" />
              <Skeleton className="h-4 mx-auto w-[200px]" />
            </div>
          </div>
        )
      })}
    </div>
  )
}
export default Loading
```

## Global Loading

HomeLayout.tsx

```tsx
import { Outlet, useNavigation } from 'react-router-dom'
import { Header, Navbar, Loading } from '@/components'

const HomeLayout = () => {
  const navigation = useNavigation()
  const isPageLoading = navigation.state === 'loading'
  return (
    <>
      <Header />
      <Navbar />

      <div className="align-element py-20">
        {isPageLoading ? <Loading /> : <Outlet />}
      </div>
    </>
  )
}
export default HomeLayout
```
