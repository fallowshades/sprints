## 07 - Reducers

- starter code

```tsx
function Component() {
  return (
    <div>
      <h2>Count: 0</h2>
      <h2>Status: Active</h2>

      <div className="btn-container">
        <button onClick={() => console.log('increment')} className="btn">
          Increment
        </button>
        <button onClick={() => console.log('decrement')} className="btn">
          Decrement
        </button>
        <button onClick={() => console.log('reset')} className="btn">
          Reset
        </button>
      </div>
      <div className="btn-container">
        <button
          onClick={() => console.log('set status to active')}
          className="btn"
        >
          Set Status to Active
        </button>
        <button
          className="btn"
          onClick={() => console.log('set status to inactive')}
        >
          Set Status to Inactive
        </button>
      </div>
    </div>
  )
}
export default Component
```

- reducer setup

reducer.ts

```ts
export type CounterState = {
  count: number
  status: string
}

export const initialState: CounterState = {
  count: 0,
  status: 'Pending...',
}

export const counterReducer = (
  state: CounterState,
  action: any
): CounterState => {
  return state
}
```

index.tsx

```tsx
import { useReducer } from 'react'
import { counterReducer, initialState } from './reducer'

function Component() {
  const [state, dispatch] = useReducer(counterReducer, initialState)
  return (
    <div>
      <h2>Count: {state.count}</h2>
      <h2>Status: {state.status}</h2>
    </div>
  )
}
```

- setup count action

reducer

```ts
type UpdateCountAction = {
  type: 'increment' | 'decrement' | 'reset'
}

// Extend the union type for all possible actions
type CounterAction = UpdateCountAction

export const counterReducer = (
  state: CounterState,
  action: CounterAction
): CounterState => {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 }
    case 'decrement':
      return { ...state, count: state.count - 1 }
    case 'reset':
      return { ...state, count: 0 }
    default:
      return state
  }
}
```

index.tsx

```tsx
<div className="btn-container">
  <button onClick={() => dispatch({ type: 'increment' })} className="btn">
    Increment
  </button>
  <button onClick={() => dispatch({ type: 'decrement' })} className="btn">
    Decrement
  </button>
  <button onClick={() => dispatch({ type: 'reset' })} className="btn">
    Reset
  </button>
</div>
```

- setup active action

reducer.ts

```ts
export type CounterState = {
  count: number
  status: string
}

export const initialState: CounterState = {
  count: 0,
  status: 'Pending...',
}

type UpdateCountAction = {
  type: 'increment' | 'decrement' | 'reset'
}
type SetStatusAction = {
  type: 'setStatus'
  payload: 'active' | 'inactive'
}

// Extend the union type for all possible actions
type CounterAction = UpdateCountAction | SetStatusAction

export const counterReducer = (
  state: CounterState,
  action: CounterAction
): CounterState => {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 }
    case 'decrement':
      return { ...state, count: state.count - 1 }
    case 'reset':
      return { ...state, count: 0 }
    case 'setStatus':
      return { ...state, status: action.payload }
    default:
      const unhandledActionType: never = action
      throw new Error(
        `Unexpected action type: ${unhandledActionType}. Please double check the counter reducer.`
      )
  }
}
```

```tsx
import { useReducer } from 'react'
import { counterReducer, initialState } from './reducer'

function Component() {
  const [state, dispatch] = useReducer(counterReducer, initialState)
  return (
    <div>
      <h2>Count: {state.count}</h2>
      <h2>Status: {state.status}</h2>

      <div className="btn-container">
        <button onClick={() => dispatch({ type: 'increment' })} className="btn">
          Increment
        </button>
        <button onClick={() => dispatch({ type: 'decrement' })} className="btn">
          Decrement
        </button>
        <button onClick={() => dispatch({ type: 'reset' })} className="btn">
          Reset
        </button>
      </div>
      <div className="btn-container">
        <button
          onClick={() => dispatch({ type: 'setStatus', payload: 'active' })}
          className="btn"
        >
          Set Status to Active
        </button>
        <button
          className="btn"
          onClick={() => dispatch({ type: 'setStatus', payload: 'inactive' })}
        >
          Set Status to Inactive
        </button>
      </div>
    </div>
  )
}
export default Component
```

## 08 - Fetch Data

- reference data fetching in typescript-tutorial

[Zod](https://zod.dev/)
[React Query](https://tanstack.com/query/latest/docs/framework/react/overview)
[Axios](https://axios-http.com/docs/intro)

```sh
npm i zod axios @tanstack/react-query

```

```tsx
import { useState, useEffect } from 'react'
const url = 'https://www.course-api.com/react-tours-project'

function Component() {
  // tours
  const [isLoading, setIsLoading] = useState<boolean>(false)
  const [isError, setIsError] = useState<string | null>(null)

  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true)
      try {
        const response = await fetch(url)
        if (!response.ok) {
          throw new Error(`Failed to fetch tours...`)
        }

        const rawData = await response.json()
        console.log(rawData)
      } catch (error) {
        const message =
          error instanceof Error ? error.message : 'there was an error...'
        setIsError(message)
      } finally {
        setIsLoading(false)
      }
    }

    fetchData()
  }, [])

  if (isLoading) {
    return <h3>Loading...</h3>
  }

  if (isError) {
    return <h3>Error: {isError}</h3>
  }

  return (
    <div>
      <h2 className="mb-1">Tours</h2>
    </div>
  )
}
export default Component
```

types.ts

```ts
import { z } from 'zod'

export const tourSchema = z.object({
  id: z.string(),
  name: z.string(),
  image: z.string(),
  info: z.string(),
  price: z.string(),
  // someValue: z.string(),
})

export type Tour = z.infer<typeof tourSchema>
```

index-fetch.tsx

```tsx
import { useState, useEffect } from 'react'
const url = 'https://www.course-api.com/react-tours-project'
import { type Tour, tourSchema } from './types'
function Component() {
  // tours
  const [tours, setTours] = useState<Tour[]>([])
  const [isLoading, setIsLoading] = useState(false)
  const [isError, setIsError] = useState<string | null>(null)

  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true)
      try {
        const response = await fetch(url)
        if (!response.ok) {
          throw new Error(`Failed to fetch tours...`)
        }
        const rawData: Tour[] = await response.json()
        const result = tourSchema.array().safeParse(rawData)

        if (!result.success) {
          console.log(result.error.message)
          throw new Error(`Failed to parse tours`)
        }
        setTours(result.data)
      } catch (error) {
        const message =
          error instanceof Error ? error.message : 'there was an error...'
        setIsError(message)
      } finally {
        setIsLoading(false)
      }
    }
    fetchData()
  }, [])

  if (isLoading) {
    return <h3>Loading...</h3>
  }
  if (isError) {
    return <h3>Error {isError}</h3>
  }

  return (
    <div>
      <h2 className="mb-1">Tours</h2>
      {tours.map((tour) => {
        return (
          <p key={tour.id} className="mb-1">
            {tour.name}
          </p>
        )
      })}
    </div>
  )
}
export default Component
```

- React Query

types.ts

```ts
import { z } from 'zod'
import axios from 'axios'
const url = 'https://course-api.com/react-tours-project'

export const tourSchema = z.object({
  id: z.string(),
  name: z.string(),
  image: z.string(),
  info: z.string(),
  price: z.string(),
  // someValue: z.string(),
})

export type Tour = z.infer<typeof tourSchema>

export const fetchTours = async (): Promise<Tour[]> => {
  const response = await axios.get<Tour[]>(url)
  const result = tourSchema.array().safeParse(response.data)
  if (!result.success) {
    throw new Error('Parsing failed')
  }
  return result.data
}
```

main.tsx

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import './index.css'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient()
ReactDOM.createRoot(document.getElementById('root')!).render(
  <QueryClientProvider client={queryClient}>
    <App />
  </QueryClientProvider>
)
```

index.tsx

```tsx
import { fetchTours } from './types'
import { useQuery } from '@tanstack/react-query'

function Component() {
  const {
    isPending,
    isError,
    error,
    data: tours,
  } = useQuery({
    queryKey: ['tours'],
    queryFn: fetchTours,
  })

  if (isPending) return <h2>Loading...</h2>
  if (isError) return <h2>Error : {error.message} </h2>
  return (
    <div>
      <h2 className="mb-1">Tours </h2>
      {tours.map((tour) => {
        return (
          <p className="mb-1" key={tour.id}>
            {tour.name}
          </p>
        )
      })}
    </div>
  )
}

export default Component
```
