## GetExistingTour and CreateNewTour

utils/db.ts

```js
import { PrismaClient } from '@prisma/client';

const prismaClientSingleton = () => {
  return new PrismaClient();
};

type PrismaClientSingleton = ReturnType<typeof prismaClientSingleton>;

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClientSingleton | undefined;
};

const prisma = globalForPrisma.prisma ?? prismaClientSingleton();

export default prisma;

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

actions.js

```js
export const getExistingTour = async ({ city, country }) => {
  return prisma.tour.findUnique({
    where: {
      city_country: {
        city,
        country,
      },
    },
  })
}

export const createNewTour = async (tour) => {
  return prisma.tour.create({
    data: tour,
  })
}
```

NewTour.jsx

```js
  const queryClient = useQueryClient();


 {
  mutationFn: async (destination) => {
      const existingTour = await getExistingTour(destination);
      if (existingTour) return existingTour;
      const newTour = await generateTourResponse(destination);
      if (newTour) {
        await createNewTour(newTour);
        queryClient.invalidateQueries({ queryKey: ['tours'] });
        return newTour;
      }
      toast.error('No matching city found...');
      return null;
    },
}
```

## GetAllTours

actions.js

```js
export const getAllTours = async (searchTerm) => {
  if (!searchTerm) {
    const tours = await prisma.tour.findMany({
      orderBy: {
        city: 'asc',
      },
    })

    return tours
  }

  const tours = await prisma.tour.findMany({
    where: {
      OR: [
        {
          city: {
            contains: searchTerm,
          },
        },
        {
          country: {
            contains: searchTerm,
          },
        },
      ],
    },
    orderBy: {
      city: 'asc',
    },
  })
  return tours
}
```

## All Tours Page

- create ToursPage ToursList and TourCard components
- create loading.js in app/tours

```js
import ToursPage from '@/components/ToursPage'
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query'
import { getAllTours } from '@/utils/actions'
export default async function AllToursPage() {
  const queryClient = new QueryClient()
  await queryClient.prefetchQuery({
    queryKey: ['tours'],
    queryFn: () => getAllTours(),
  })
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <ToursPage />
    </HydrationBoundary>
  )
}
```

## ToursPage

```js
'use client'
import { getAllTours } from '@/utils/actions'
import { useQuery } from '@tanstack/react-query'
import ToursList from './ToursList'

const ToursPage = () => {
  const { data, isPending } = useQuery({
    queryKey: ['tours'],
    queryFn: () => getAllTours(),
  })

  return (
    <>
      {isPending ? (
        <span className=' loading'></span>
      ) : (
        <ToursList data={data} />
      )}
    </>
  )
}
export default ToursPage
```

## ToursList

```js
import TourCard from './TourCard'
const ToursList = ({ data }) => {
  if (data.length === 0) return <h4 className='text-lg '>No tours found...</h4>

  return (
    <div className='grid sm:grid-cols-2  lg:grid-cols-4 gap-8'>
      {data.map((tour) => {
        return <TourCard key={tour.id} tour={tour} />
      })}
    </div>
  )
}
export default ToursList
```

## TourCard

```js
import Link from 'next/link'
const TourCard = ({ tour }) => {
  const { city, title, id, country } = tour

  return (
    <Link
      href={`/tours/${id}`}
      className="card card-compact rounded-xl bg-base-100"
    >
      <div className="card-body items-center text-center">
        <h2 className="card-title text-center">
          {city}, {country}
        </h2>
      </div>
    </Link>
  )
}
export default TourCard

################################################### 3.3

```

## Search Functionality

```js
'use client'
import { getAllTours } from '@/utils/actions'
import { useQuery } from '@tanstack/react-query'
import { useState } from 'react'
import ToursList from './ToursList'

const ToursPage = () => {
  const [searchValue, setSearchValue] = useState('')
  const { data, isPending } = useQuery({
    queryKey: ['tours', searchValue],
    queryFn: () => getAllTours(searchValue),
  })

  return (
    <>
      <form className='max-w-lg mb-12'>
        <div className='join w-full'>
          <input
            type='text'
            placeholder='enter city or country here..'
            className='input input-bordered join-item w-full'
            name='search'
            value={searchValue}
            onChange={(e) => setSearchValue(e.target.value)}
            required
          />
          <button
            className='btn btn-primary join-item'
            type='button'
            disabled={isPending}
            onClick={() => setSearchValue('')}
          >
            {isPending ? 'please wait' : 'reset'}
          </button>
        </div>
      </form>
      {isPending ? (
        <span className=' loading'></span>
      ) : (
        <ToursList data={data} />
      )}
    </>
  )
}
export default ToursPage
```

## Challenge - Single Tour Page

- setup page and get info on specific tour

## Solution - Single Tour Page

- create app/tours/[id]/page.js

actions.js

```js
export const getSingleTour = async (id) => {
  return prisma.tour.findUnique({
    where: {
      id,
    },
  })
}
```

```js
import TourInfo from '@/components/TourInfo'
import { getSingleTour } from '@/utils/actions'
import Link from 'next/link'
import { redirect } from 'next/navigation'
const SingleTourPage = async ({ params }) => {
  const tour = await getSingleTour(params.id)
  if (!tour) {
    redirect('/tours')
  }
  return (
    <div>
      <Link href='/tours' className='btn btn-secondary mb-12'>
        back to tours
      </Link>
      <TourInfo tour={tour} />
    </div>
  )
}
export default SingleTourPage
```

## Images

- url are valid for 2 hours
- way more expensive than chat

actions.js

```js
export const generateTourImage = async ({ city, country }) => {
  try {
    const tourImage = await openai.images.generate({
      prompt: `a panoramic view of the ${city} ${country}`,
      n: 1,
      size: '512x512',
    })
    return tourImage?.data[0]?.url
  } catch (error) {
    return null
  }
}
```

app/tours/[id]/page.js

```js
import TourInfo from '@/components/TourInfo'
import { generateTourImage } from '@/utils/actions'
import prisma from '@/utils/prisma'
import Link from 'next/link'
import Image from 'next/image'

const SingleTourPage = async ({ params }) => {
  const tour = await prisma.tour.findUnique({
    where: {
      id: params.id,
    },
  })

  const tourImage = await generateTourImage({
    city: tour.city,
    country: tour.country,
  })
  return (
    <div>
      <Link href='/tours' className='btn btn-secondary mb-12'>
        back to tours
      </Link>

      {tourImage ? (
        <div>
          <Image
            src={tourImage}
            width={300}
            height={300}
            className='rounded-xl shadow-xl mb-16 h-96 w-96 object-cover'
            alt={tour.title}
            priority
          />
        </div>
      ) : null}

      <TourInfo tour={tour} />
    </div>
  )
}
export default SingleTourPage
```

next.config.js

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'oaidalleapiprodscus.blob.core.windows.net',
        port: '',
        pathname: '/private/**',
      },
      {
        protocol: 'https',
        hostname: 'images.unsplash.com',
        port: '',
        pathname: '/**',
      },
    ],
  },
}

module.exports = nextConfig
```

## Alternative

```sh
npm i axios
```

.env.local

```
UNSPLASH_API_KEY=7pmB29Xi9rOWHhYpvtuc4edchzh1w0eawUjJwNAqngA
```

```js
import TourInfo from '@/components/TourInfo'
import { generateTourImage } from '@/utils/actions'
import prisma from '@/utils/prisma'
import Link from 'next/link'
import Image from 'next/image'
import axios from 'axios'
const url = `https://api.unsplash.com/search/photos?client_id=${process.env.UNSPLASH_API_KEY}&query=`

const SingleTourPage = async ({ params }) => {
  const tour = await prisma.tour.findUnique({
    where: {
      id: params.id,
    },
  })

  const { data } = await axios(`${url}${tour.city}`)
  const tourImage = data?.results[0]?.urls?.raw

  // const tourImage = await generateTourImage({
  //   city: tour.city,
  //   country: tour.country,
  // });
  return (
    <div>
      <Link href='/tours' className='btn btn-secondary mb-12'>
        back to tours
      </Link>

      {tourImage ? (
        <div>
          <Image
            src={tourImage}
            width={300}
            height={300}
            className='rounded-xl shadow-xl mb-16 h-96 w-96 object-cover'
            alt={tour.title}
            priority
          />
        </div>
      ) : null}

      <TourInfo tour={tour} />
    </div>
  )
}
export default SingleTourPage
```

## Remove SingIn and SignUp Pages

- delete folders
- remove env variables

## Challenge - Token Logic

OPTIONAL !!!

INVOLVES REFACTORING !!!!

- set max_tokens in chat
- create Token model
- assign some token amount to user
- check token amount before request
- subtract after successful request

## Solution - Token Logic
