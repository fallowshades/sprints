#

## summary: setup dehydra and message format

[client-handle-single-state]
- import provided page and toast
- onChange/submit wrapped by query provider --> hook get extra reducer return and network-awarness

[metate-extra-reducer-logic]
- key assigned to thunk and extra reducers
- mutate and stale time --> is hydrated
- submit mutate w async open ai req


## client-handle-single-state + network

### Challenge - Profile Page

1. **Import Dependencies:**

   - Import the `UserProfile` component from `'@clerk/nextjs'`.

2. **Render the UserProfile Component:**

   - Within the component's render function, return the `UserProfile` component.

This component serves as a page for displaying the user's profile information. It utilizes the `UserProfile` component from the `'@clerk/nextjs'` package to render the user's profile details. This is a common pattern in Next.js applications for handling user authentication and profile management.

### Solution - Profile Page

```js
import { UserProfile } from '@clerk/nextjs'
const UserProfilePage = () => {
  return <UserProfile />
}
export default UserProfilePage
```

### Challenge - Add React-Hot-Toast Library

- setup app/providers.js
- import/add Toaster component
- wrap {children} in layout.js

### Solution - Add React-Hot-Toast Library

app/providers.jsx

```js
'use client'
import { Toaster } from 'react-hot-toast'
export default function Providers({ children }) {
  return (
    <>
      <Toaster position="top-center" />
      {children}
    </>
  )
}
```

app/layout.js

```js
<Providers>{children}</Providers>
```

### Challenge - Chat Structure

1. **Import Dependencies:**

   - Import the necessary dependencies, including `useState` from 'react' and `toast` from 'react-hot-toast'.

2. **State Management:**

   - Initialize state variables using the `useState` hook:
     - `text`: to manage the text input for composing messages.
     - `messages`: to manage the list of messages.

3. **Handle Form Submission:**

   - Implement a `handleSubmit` function to handle form submissions when sending messages. It should prevent the default form behavior.

4. **Render UI Elements:**

   - Render the chat interface with the following components and elements:
     - A 'messages' header using an `<h2>` element.
     - A `<form>` element for composing and sending messages.
     - Inside the form:
       - An `<input>` element for entering messages, with event handling to update the `text` state.
       - A 'Send' button to submit messages.

This component represents a chat interface where users can send messages. It uses React state to manage the input text and a list of messages. When a message is submitted, it prevents the default form behavior (form submission) and handles message composition.

### Solution - Chat Structure

- setup components/Chat.jsx
- import in app/(dashboard)/chat/page.js

```js
'use client'
import { useState } from 'react'
import toast from 'react-hot-toast'

const Chat = () => {
  const [text, setText] = useState('')
  const [messages, setMessages] = useState([])

  const handleSubmit = (e) => {
    e.preventDefault()
  }

  return (
    <div className="min-h-[calc(100vh-6rem)] grid grid-rows-[1fr,auto]">
      <div>
        <h2 className="text-5xl">messages</h2>
      </div>
      <form onSubmit={handleSubmit} className="max-w-4xl pt-12">
        <div className="join w-full">
          <input
            type="text"
            placeholder="Message GeniusGPT"
            className="input input-bordered join-item w-full"
            value={text}
            required
            onChange={(e) => setText(e.target.value)}
          />
          <button className="btn btn-primary join-item" type="submit">
            ask question
          </button>
        </div>
      </form>
    </div>
  )
}
export default Chat
```

### React Query

#### Install

```sh
npm i @tanstack/react-query @tanstack/react-query-devtools

```

#### Setup

```js
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // the data will be considered fresh for 1 minute
      staleTime: 60 * 1000,
    },
  },
})

ReactDOM.createRoot(document.getElementById('root')).render(
  <QueryClientProvider client={queryClient}>
    <App />
    <ReactQueryDevtools initialIsOpen={false} />
  </QueryClientProvider>
)
```

#### UseQuery

```js
const Items = () => {
  const { isPending, isError, data } = useQuery({
    queryKey: ['tasks'],
    // A query function can be literally any function that returns a promise.
    queryFn: () => axios.get('/someUrl'),
  })

  if (isPending) {
    return <p>Loading...</p>
  }

  if (isError) {
    return <p>Error...</p>
  }
  return (
    <div className="items">
      {data.taskList.map((item) => {
        return <SingleItem key={item.id} item={item} />
      })}
    </div>
  )
}
export default Items
```

#### UseMutation

```js
const { mutate, isPending, data } = useMutation({
  mutationFn: (taskTitle) => axios.post('/', { title: taskTitle }),
  onSuccess: () => {
    // do something
  },
  onError: () => {
    // do something
  },
})

const handleSubmit = (e) => {
  e.preventDefault()
  mutate(newItemName)
}
```

-----------------------------------------------

## React Query and Next.js

- WE CAN USE SERVER ACTIONS 🚀🚀🚀🚀🚀🚀

  app/providers.jsx

```js
// In Next.js, this file would be called: app/providers.jsx
'use client'

// We can not useState or useRef in a server component, which is why we are
// extracting this part out into it's own file with 'use client' on top
import { useState } from 'react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { Toaster } from 'react-hot-toast'
export default function Providers({ children }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            // With SSR, we usually want to set some default staleTime
            // above 0 to avoid refetching immediately on the client
            staleTime: 60 * 1000,
          },
        },
      })
  )

  return (
    <QueryClientProvider client={queryClient}>
      <Toaster position="top-center" />
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

- WRAP EACH PAGE

chat/page.js

```js
import Chat from '@/components/Chat'
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query'
export default async function ChatPage() {
  const queryClient = new QueryClient()
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <Chat />
    </HydrationBoundary>
  )
}
```

utils/actions.js

```js
'use server'

export const generateChatResponse = async (chatMessage) => {
  console.log(chatMessage)
  return 'awesome'
}
```

components/Chat.jsx

```js
'use client'
import { useState } from 'react'
import toast from 'react-hot-toast'
import { useMutation } from '@tanstack/react-query'
import { generateChatResponse } from '@/utils/actions'
const Chat = () => {
  const [text, setText] = useState('')
  const [messages, setMessages] = useState([])

  const { mutate } = useMutation({
    mutationFn: (message) => generateChatResponse(message),
  })

  const handleSubmit = (e) => {
    e.preventDefault()
    mutate(text)
  }
}
```

## OPENAI API

[Pricing](https://openai.com/pricing)

```sh
npm i openai
```

- create API KEY
- save in .env.local

```js
OPENAI_API_KEY=....
```

utils/actions.js

```js
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export const generateChatResponse = async (message) => {
  const response = await openai.chat.completions.create({
    messages: [
      { role: 'system', content: 'you are a helpful assistant' },
      { role: 'user', content: message };
    ],
    model: 'gpt-3.5-turbo',
    temperature: 0,
  });
  console.log(response.choices[0].message)
  console.log(response);
  return 'awesome';
};
```
