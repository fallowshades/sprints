# itteration 3 layer 1 Register

[send-key:VALUE-pair_to-nonBlocking/noWaterFall]

- the default value will later be for search

-name,type wo value

## Vertical scalling

### Action handled by parent router

#### 1. Typical Form Submission

```js
import { useState } from 'react'
import axios from 'axios'
const MyForm = () => {
  const [value, setValue] = useState('')

  const handleSubmit = async (event) => {
    event.preventDefault()
    const data = await axios.post('url', { value })
  }

  return <form onSubmit={handleSubmit}>.....</form>
}

export default MyForm
```

#### 2. React Router - Action

Route actions are the "writes" to route loader "reads". They provide a way for apps to perform data mutations with simple HTML and HTTP semantics while React Router abstracts away the complexity of asynchronous UI and revalidation. This gives you the simple mental model of HTML + HTTP (where the browser handles the asynchrony and revalidation) with the behavior and UX capabilities of modern SPAs.

Register.jsx

```js
import { Form, redirect, useNavigation, Link } from 'react-router-dom'
import Wrapper from '../assets/wrappers/RegisterAndLoginPage'
import { FormRow, Logo } from '../components'

const Register = () => {
  return (
    <Wrapper>
      <Form method='post' className='form'>
        ...
      </Form>
    </Wrapper>
  )
}
export default Register
```

App.jsx

```jsx
{
  path: 'register',
  element: <Register />,
  action: () => {
   console.log('hello there');
   return null;
    },
},
```

App.jsx

```jsx
import{action as registerAction} from './pages/Register'
{
  path: 'register',
  element: <Register />,
  action: registerAction,
},
```

Register.jsx

```js
export const action = async (data) => {
  return null
}
```

## Horizontall scalling

### session monitored

### error encapsulated

#### 4. React-Toastify

Import and set up the react-toastify library.

[React Toastify](https://fkhadra.github.io/react-toastify/introduction)

```sh
npm i react-toastify@9.1.2
```

main.jsx

```js
import 'react-toastify/dist/ReactToastify.css'
import { ToastContainer } from 'react-toastify'
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
    <ToastContainer position='top-center' />
  </React.StrictMode>
)
```

Register.jsx

```js
import customFetch from '../utils/customFetch'
import { toast } from 'react-toastify'

export const action = async ({ request }) => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)
  try {
    await customFetch.post('/auth/register', data)
    toast.success('Registration successful')
    return redirect('/login')
  } catch (error) {
    toast.error(error?.response?.data?.msg)
    return error
  }
}
```

#### 3. useNavigation() and navigation.state (bad idea to do first. get hoosk error. can only be called inside the body of a f())

This hook tells you everything you need to know about a page navigation to build pending navigation indicators and optimistic UI on data mutations. Things like:

- Global loading indicators
- Adding busy indicators to submit buttons

Navigation State

idle - There is no navigation pending.
submitting - A route action is being called due to a form submission using POST, PUT, PATCH, or DELETE
loading - The loaders for the next routes are being called to render the next page

Register.jsx

```js
const Register = () => {
  const navigation = useNavigation()
  const isSubmitting = navigation.state === 'submitting'
  return (
    <Wrapper>
      <Form method='post' className='form'>
        ....
        <button type='submit' className='btn btn-block' disabled={isSubmitting}>
          {isSubmitting ? 'submitting...' : 'submit'}
        </button>
        ...
      </Form>
    </Wrapper>
  )
}
export default Register
```

#### 5. Login User

```js
import { Link, Form, redirect, useNavigation } from 'react-router-dom'
import Wrapper from '../assets/wrappers/RegisterAndLoginPage'
import { FormRow, Logo } from '../components'
import customFetch from '../utils/customFetch'
import { toast } from 'react-toastify'

export const action = async ({ request }) => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)
  try {
    await customFetch.post('/auth/login', data)
    toast.success('Login successful')
    return redirect('/dashboard')
  } catch (error) {
    toast.error(error?.response?.data?.msg)
    return error
  }
}

const Login = () => {
  const navigation = useNavigation()
  const isSubmitting = navigation.state === 'submitting'
  return (
    <Wrapper>
      <Form method='post' className='form'>
        <Logo />
        <h4>login</h4>

        <FormRow type='email' name='email' defaultValue='john@gmail.com' />
        <FormRow
          type='password'
          name='password'
          defaultValue='secretDuckTape'
        />
        <button type='submit' className='btn btn-block' disabled={isSubmitting}>
          {isSubmitting ? 'submitting...' : 'submit'}
        </button>
        <button type='button' className='btn btn-block'>
          explore the app
        </button>
        <p>
          Not a member yet?
          <Link to='/register' className='member-btn'>
            Register
          </Link>
        </p>
      </Form>
    </Wrapper>
  )
}
export default Login
```

App.jsx

```jsx
import{action as loginAction} from './pages/Login'
{
  path: 'login',
  element: <Login />,
  action: loginAction,
},


```

#### 6. Access Action Data (optional)

```js
import { useActionData } from 'react-router-dom'

export const action = async ({ request }) => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)
  const errors = { msg: '' }
  if (data.password.length < 3) {
    errors.msg = 'password too short'
    return errors
  }
  try {
    await customFetch.post('/auth/login', data)
    toast.success('Login successful')
    return redirect('/dashboard')
  } catch (error) {
    // toast.error(error?.response?.data?.msg);
    errors.msg = error.response.data.msg
    return errors
  }
}
const Login = () => {
  const errors = useActionData()

  return (
    <Wrapper>
      <Form method="post" className="form">
        ...
        {</p>errors && <p style={{ color: 'red' }}>{errors.msg}</p>}
        ...
      </Form>
    </Wrapper>
  )
}
export default Login
```
