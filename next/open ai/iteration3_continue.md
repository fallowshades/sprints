#

##

[spread-notation]

- state managment mutated reducer --> state

[prompt]

- create new obj --> spread messages useState data
- clean up new input
- simplify opject from key value - pair Object.fromEntries(formData.entries())

- formData create a sequence of 3 requests
- generate response with stops (template literal insert rules)
- can map stops for FormRowSelect
- prisma push model with @text and @unique

## stat managment within useState context (should mb rdk)

## handle submit obj.fromentries(formData) + msg +query arr[]

### Context !!!

utils/actions

```js
export const generateChatResponse = async (chatMessages) => {
  try {
    const response = await openai.chat.completions.create({
      messages: [
        { role: 'system', content: 'you are a helpful assistant' },
        ...chatMessages,
      ],
      model: 'gpt-3.5-turbo',
      temperature: 0,
    })
    return response.choices[0].message
  } catch (error) {
    return null
  }
}
```

Chat.jsx

```js
const Chat = () => {
  const [text, setText] = useState('')
  const [messages, setMessages] = useState([])
  const { mutate, isPending, data } = useMutation({
    mutationFn: (query) => generateChatResponse([...messages, query]),

    onSuccess: (data) => {
      if (!data) {
        toast.error('Something went wrong...')
        return
      }
      setMessages((prev) => [...prev, data])
    },
    onError: (error) => {
      toast.error('Something went wrong...')
    },
  })
  const handleSubmit = (e) => {
    e.preventDefault()
    const query = { role: 'user', content: text }
    mutate(query)
    setMessages((prev) => [...prev, query])
    setText('')
  }
}
```

### Messages

Chat.jsx

```js
return (
  <div>
    {messages.map(({ role, content }, index) => {
      const avatar = role == 'user' ? '👤' : '🤖'
      const bcg = role == 'user' ? 'bg-base-200' : 'bg-base-100'
      return (
        <div
          key={index}
          className={` ${bcg} flex py-6 -mx-8 px-8
               text-xl leading-loose border-b border-base-300`}
        >
          <span className='mr-4 '>{avatar}</span>
          <p className='max-w-3xl'>{content}</p>
        </div>
      )
    })}
    {isPending && <span className='loading'></span>}
  </div>
)

return (
  <button
    className='btn btn-primary join-item'
    type='submit'
    disabled={isPending}
  >
    {isPending ? 'please wait' : 'ask question'}
  </button>
)
```

---

### Challenge - New Tour Page

- create NewTour and TourInfo components
- create New Tour page : app/(dashboard)/tours/new-tour/page.js
- add react query boilerplate
- render NewTour component
- setup form with two inputs city and country

### Solution - New Tour Page

tours/new-tour/page.js

```js
import NewTour from '@/components/NewTour'
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query'
export default async function ChatPage() {
  const queryClient = new QueryClient()
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <NewTour />
    </HydrationBoundary>
  )
}
```

```js
'use client'

import toast from 'react-hot-toast'
import TourInfo from '@/components/TourInfo'

const NewTour = () => {
  const handleSubmit = (e) => {
    e.preventDefault()

    const formData = new FormData(e.currentTarget)
    const destination = Object.fromEntries(formData.entries())
  }

  return (
    <>
      <form onSubmit={handleSubmit} className='max-w-2xl'>
        <h2 className=' mb-4'>Select your dream destination</h2>
        <div className='join w-full'>
          <input
            type='text'
            className='input input-bordered join-item w-full'
            placeholder='city'
            name='city'
            required
          />
          <input
            type='text'
            className='input input-bordered join-item w-full'
            placeholder='country'
            name='country'
            required
          />
          <button className='btn btn-primary join-item' type='submit'>
            generate tour
          </button>
        </div>
      </form>
      <div className='mt-16'>
        <TourInfo />
      </div>
    </>
  )
}
export default NewTour
```

### GenerateTourResponse Setup

actions.js

```js
export const getExistingTour = async ({ city, country }) => {
  return null
}

export const generateTourResponse = async ({ city, country }) => {
  return null
}

export const createNewTour = async (tour) => {
  return null
}
```

NewTour.jsx

```js
import { useMutation, useQueryClient } from '@tanstack/react-query'
import {
  createNewTour,
  generateTourResponse,
  getExistingTour,
} from '@/utils/actions'
import toast from 'react-hot-toast'
import TourInfo from '@/components/TourInfo'

const NewTour = () => {
  const {
    mutate,
    isPending,
    data: tour,
  } = useMutation({
    mutationFn: async (destination) => {
      const newTour = await generateTourResponse(destination)
      if (newTour) {
        return newTour
      }
      toast.error('No matching city found...')
      return null
    },
  })

  const handleSubmit = (e) => {
    e.preventDefault()

    const formData = new FormData(e.currentTarget)
    const destination = Object.fromEntries(formData.entries())
    mutate(destination)
  }

  if (isPending) {
    return <span className='loading loading-lg'></span>
  }
  return (
    <>
      <form onSubmit={handleSubmit} className='max-w-2xl'>
        <h2 className=' mb-4'>Select your dream destination</h2>
        <div className='join w-full'>
          <input
            type='text'
            className='input input-bordered join-item w-full'
            placeholder='city'
            name='city'
            required
          />
          <input
            type='text'
            className='input input-bordered join-item w-full'
            placeholder='country'
            name='country'
            required
          />
          <button
            className='btn btn-primary join-item'
            type='submit'
            disabled={isPending}
          >
            {isPending ? 'please wait...' : 'generate tour'}
          </button>
        </div>
      </form>
      <div className='mt-16'>
        <div className='mt-16'>{tour ? <TourInfo tour={tour} /> : null}</div>
      </div>
    </>
  )
}
export default NewTour
```

---

## Prompt

Later we will use shorter prompt

```js
{
  "tour": {
    ...
   "stops": ["stop name ", "stop name","stop name"]
  }
}
```

```js
const query = `Find a exact ${city} in this exact ${country}.
If ${city} and ${country} exist, create a list of things families can do in this ${city},${country}. 
Once you have a list, create a one-day tour. Response should be  in the following JSON format: 
{
  "tour": {
    "city": "${city}",
    "country": "${country}",
    "title": "title of the tour",
    "description": "short description of the city and tour",
    "stops": ["short paragraph on the stop 1 ", "short paragraph on the stop 2","short paragraph on the stop 3"]
  }
}
"stops" property should include only three stops.
If you can't find info on exact ${city}, or ${city} does not exist, or it's population is less than 1, or it is not located in the following ${country},   return { "tour": null }, with no additional characters.`
```

## GenerateTourResponse

```js
export const generateTourResponse = async ({ city, country }) => {
  const query = `Find a exact ${city} in this exact ${country}.
If ${city} and ${country} exist, create a list of things families can do in this ${city},${country}. 
Once you have a list, create a one-day tour. Response should be  in the following JSON format: 
{
  "tour": {
    "city": "${city}",
    "country": "${country}",
    "title": "title of the tour",
    "description": "short description of the city and tour",
    "stops": ["short paragraph on the stop 1 ", "short paragraph on the stop 2","short paragraph on the stop 3"]
  }
}
"stops" property should include only three stops.
If you can't find info on exact ${city}, or ${city} does not exist, or it's population is less than 1, or it is not located in the following ${country},   return { "tour": null }, with no additional characters.`

  try {
    const response = await openai.chat.completions.create({
      messages: [
        { role: 'system', content: 'you are a tour guide' },
        { role: 'user', content: query },
      ],
      model: 'gpt-3.5-turbo',
      temperature: 0,
    })
    // potentially returns a text with error message
    const tourData = JSON.parse(response.choices[0].message.content)

    if (!tourData.tour) {
      return null
    }

    return tourData.tour
  } catch (error) {
    console.log(error)
    return null
  }
}
```

## Shorter Prompt

```js
{
  "tour": {
    ...
   "stops": ["stop name ", "stop name","stop name"]
  }
}
```

## TourInfo

TourInfo.jsx

```js
const TourInfo = ({ tour }) => {
  const { title, description, stops } = tour
  return (
    <div className='max-w-2xl'>
      <h1 className='text-4xl font-semibold mb-4'>{title}</h1>
      <p className='leading-loose mb-6'>{description}</p>
      <ul>
        {stops.map((stop) => {
          return (
            <li key={stop} className='mb-4 bg-base-100 p-4 rounded-xl'>
              <p className='text'>{stop}</p>
            </li>
          )
        })}
      </ul>
    </div>
  )
}
export default TourInfo
```

## Add Prisma

```sh
npm install prisma --save-dev
npm install @prisma/client
```

```sh
npx prisma init
```

- ADD .ENV TO .GITIGNORE !!!!

## PlanetScale

## Model

```prisma
datasource db {
  provider     = "mysql"
  url          = env("DATABASE_URL")
  relationMode = "prisma"
}

generator client {
  provider = "prisma-client-js"
}

model Tour {
  id String @id @default(uuid())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  city String
  country String
  title String
  description String @db.Text
  image String? @db.Text
  stops Json
  @@unique([city, country])
}
```

```sh
npx prisma db push
```

```sh
npx prisma studio
```

@db.Text: This attribute is used to specify the type of the column in the underlying database. When you use @db.Text, you're telling Prisma that the particular field should be stored as a text column in the database. Text columns can store large amounts of string data, typically used for long-form text that exceeds the length limits of standard string columns. This is often used for descriptions, comments, JSON-formatted strings, etc.

@@unique: This attribute is used at the model level to enforce the uniqueness of a specific combination of fields within the database. In this case, @@unique([city, country]) ensures that no two rows in the table have the same combination of city and country. This means you can have multiple tours in the same city or country, but not multiple tours with the same city and country combination. It essentially acts as a composite unique constraint on the two fields.
