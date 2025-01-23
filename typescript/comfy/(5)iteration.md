## UserSlice

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'
import { toast } from '@/components/ui/use-toast'

export type User = {
  username: string
  jwt: string
}

type UserState = {
  user: User | null
}

const getUserFromLocalStorage = (): User | null => {
  const user = localStorage.getItem('user')
  if (!user) return null
  return JSON.parse(user)
}

const initialState: UserState = {
  user: getUserFromLocalStorage(),
}

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    loginUser: (state, action: PayloadAction<User>) => {
      const user = action.payload
      state.user = user
      localStorage.setItem('user', JSON.stringify(user))

      if (user.username === 'demo user') {
        toast({ description: 'Welcome Guest User' })
        return
      }
      toast({ description: 'Login successful' })
    },
    logoutUser: (state) => {
      state.user = null
      // localStorage.clear()
      localStorage.removeItem('user')
    },
  },
})

export const { loginUser, logoutUser } = userSlice.actions

export default userSlice.reducer
```

## Register and Login Requests

- [API DOCS](https://documenter.getpostman.com/view/18152321/2s9Xy5KpTi)

## Register Page - Setup

- create components/SubmitBtn.tsx

```tsx
import { ActionFunction, Form, Link, redirect } from 'react-router-dom'
import { Card, CardHeader, CardContent, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { SubmitBtn, FormInput } from '@/components'
import { customFetch } from '@/utils'
import { toast } from '@/components/ui/use-toast'
import { AxiosError } from 'axios'

export const action: ActionFunction = async ({ request }): Promise<null> => {
  return null
}

function Register() {
  return (
    <section className='h-screen grid place-items-center'>
      <Card className='w-96 bg-muted'>
        <CardHeader>
          <CardTitle className='text-center'>Register</CardTitle>
        </CardHeader>
        <CardContent>
          <Form>
            <FormInput type='text' name='username' defaultValue='test' />
            <FormInput type='email' name='email' defaultValue='test@test.com' />
            <FormInput type='password' name='password' defaultValue='secret' />

            <Button type='submit' variant='default' className='w-full mt-4'>
              Submit
            </Button>

            <p className='text-center mt-4'>
              Already a member?
              <Button type='button' asChild variant='link'>
                <Link to='/login'>Login</Link>
              </Button>
            </p>
          </Form>
        </CardContent>
      </Card>
    </section>
  )
}
export default Register
```

## Actions

Route actions are the "writes" to route loader "reads". They provide a way for apps to perform data mutations with simple HTML and HTTP semantics while React Router abstracts away the complexity of asynchronous UI and revalidation. This gives you the simple mental model of HTML + HTTP (where the browser handles the asynchrony and revalidation) with the behavior and UX capabilities of modern SPAs.

```tsx
<Form method='post' action=''>
```

App.tsx

```tsx
import { action as registerAction } from './pages/Register';

{
    path: '/register',
    element: <Register />,
    errorElement: <Error />,
    action: registerAction,
  },
```

```tsx
export const action: ActionFunction = async ({ request }): Promise<null> => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)
  console.log(data)
  return null
}
```

## Register User

```tsx
export const action: ActionFunction = async ({
  request,
}): Promise<Response | null> => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)
  try {
    await customFetch.post('/auth/local/register', data)

    toast({ description: 'Registered' })
    return redirect('/login')
  } catch (error) {
    console.log(error)

    const errorMsg =
      error instanceof AxiosError
        ? error.response?.data.error.message
        : 'Registration Failed'
    toast({ description: errorMsg })
    return null
  }
}
```

## SubmitBtn

```sh
npm i @radix-ui/react-icons
```

```tsx
import { useNavigation } from 'react-router-dom'
import { Button } from '@/components/ui/button'
import { ReloadIcon } from '@radix-ui/react-icons'

const SubmitBtn = ({
  text,
  className,
}: {
  text: string
  className?: string
}) => {
  const navigation = useNavigation()
  const isSubmitting = navigation.state === 'submitting'
  return (
    <Button type='submit' className={className} disabled={isSubmitting}>
      {isSubmitting ? (
        <span className='flex '>
          <ReloadIcon className='mr-2 h-4 w-4 animate-spin' />
          Submitting...
        </span>
      ) : (
        text
      )}
    </Button>
  )
}
export default SubmitBtn
```

Register.tsx

```tsx
<SubmitBtn text='Register' className='w-full mt-4' />
```

temp

```ts
// optional
await new Promise((resolve) => setTimeout(resolve, 2000))
```

## Login User - Setup

```tsx
import {
  Form,
  Link,
  redirect,
  type ActionFunction,
  useNavigate,
} from 'react-router-dom'
import { Card, CardHeader, CardContent, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { SubmitBtn, FormInput } from '@/components'
import { customFetch } from '@/utils'
import { toast } from '@/components/ui/use-toast'
import { type ReduxStore } from '@/store'
import { loginUser } from '@/features/user/userSlice'
import { useAppDispatch } from '@/hooks'
import { AxiosResponse } from 'axios'

function Login() {
  return (
    <section className='h-screen grid place-items-center'>
      <Card className='w-96 bg-muted'>
        <CardHeader>
          <CardTitle className='text-center'>Login</CardTitle>
        </CardHeader>
        <CardContent>
          <Form method='POST'>
            <FormInput type='email' label='email' name='identifier' />
            <FormInput type='password' name='password' />

            <SubmitBtn text='Login' className='w-full mt-4' />
            <Button
              type='button'
              variant='outline'
              onClick={loginAsGuestUser}
              className='w-full mt-4'
            >
              Guest User
            </Button>
            <p className='text-center mt-4'>
              Not a member yet?
              <Button type='button' asChild variant='link'>
                <Link to='/register'>Register</Link>
              </Button>
            </p>
          </Form>
        </CardContent>
      </Card>
    </section>
  )
}
export default Login
```

## Guest User

```tsx
const dispatch = useAppDispatch()
const navigate = useNavigate()
const loginAsGuestUser = async (): Promise<void> => {
  try {
    const response = await customFetch.post('/auth/local', {
      identifier: 'test@test.com',
      password: 'secret',
    })
    const username = response.data.user.username
    const jwt = response.data.jwt
    dispatch(loginUser({ username, jwt }))
    navigate('/')
  } catch (error) {
    console.log(error)
    toast({ description: 'Login Failed' })
  }
}
```

## Login Request

App.tsx

```tsx
import { action as loginAction } from './pages/Login';
import { store } from './store';

{
    path: '/login',
    element: <Login />,
    errorElement: <Error />,
    action: loginAction(store),
  },

```

Login.tsx

```tsx
export const action =
  (store: ReduxStore): ActionFunction =>
  async ({ request }): Promise<Response | null> => {
    const formData = await request.formData()
    const data = Object.fromEntries(formData)
    try {
      const response: AxiosResponse = await customFetch.post(
        '/auth/local',
        data
      )
      const username = response.data.user.username
      const jwt = response.data.jwt
      store.dispatch(loginUser({ username, jwt }))
      return redirect('/')
    } catch (error) {
      // console.log(error);
      toast({ description: 'Login Failed' })
      return null
    }
  }
```
