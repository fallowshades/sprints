#### Challenge (40) - Setup User Slice

- setup user slice
- add to store

##### userSlice.js

- create features/user/userSlice.js
- Import Dependencies:

  - Import `createSlice` from `'@reduxjs/toolkit'`.
  - Import `toast` from `'react-toastify'`.

- Define Initial State:

  - Create an `initialState` object with default values for `user` and `theme`.

- Create Redux Slice:

  - Use `createSlice` to define a Redux slice named `'user'`.
  - Set the slice name to `'user'`.
  - Use the `initialState` object as the initial state.

- Define Reducer Functions:

  - Create the `loginUser` reducer function with the signature `(state, action) => {}`.

    - Inside the function, log a message like `'login'`.

  - Create the `logoutUser` reducer function with the signature `(state) => {}`.

    - Inside the function, log a message like `'logout'`.

  - Create the `toggleTheme` reducer function with the signature `(state) => {}`.
    - Inside the function, log a message like `'toggle theme'`.

- Export Actions:
  - Export the action creators:
    - `loginUser`
    - `logoutUser`
    - `toggleTheme`

#### Solution (40) - Setup User Slice

```js
import { createSlice } from '@reduxjs/toolkit'
import { toast } from 'react-toastify'

const initialState = {
  user: { username: 'coding addict' },
  theme: 'dracula',
}

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    loginUser: (state, action) => {
      console.log('login')
    },
    logoutUser: (state) => {
      console.log('logout')
    },
    toggleTheme: (state) => {
      console.log('toggle theme')
    },
  },
})

export const { loginUser, logoutUser, toggleTheme } = userSlice.actions

export default userSlice.reducer
```

store.js

```js
import { configureStore } from '@reduxjs/toolkit'

import cartReducer from './features/cart/cartSlice'
import userReducer from './features/user/userSlice'

export const store = configureStore({
  reducer: {
    cartState: cartReducer,
    userState: userReducer,
  },
})
```

#### Challenge (41) - Move Theme Logic

- move theme logic from Navbar to userSlice

#### Solution (41) - Move Theme Logic

userSlice.js

```js
import { createSlice } from '@reduxjs/toolkit'
import { toast } from 'react-toastify'

const themes = {
  winter: 'winter',
  dracula: 'dracula',
}

const getThemeFromLocalStorage = () => {
  const theme = localStorage.getItem('theme') || themes.winter
  document.documentElement.setAttribute('data-theme', theme)
  return theme
}

const initialState = {
  user: { username: 'coding addict' },
  theme: getThemeFromLocalStorage(),
}

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    loginUser: (state, action) => {
      console.log('login')
    },
    logoutUser: (state) => {
      console.log('logout')
    },
    toggleTheme: (state) => {
      const { dracula, winter } = themes
      state.theme = state.theme === dracula ? winter : dracula
      document.documentElement.setAttribute('data-theme', state.theme)
      localStorage.setItem('theme', state.theme)
    },
  },
})

export const { loginUser, logoutUser, toggleTheme } = userSlice.actions

export default userSlice.reducer
```

Navbar.js

```js
import { BsCart3, BsMoonFill, BsSunFill } from 'react-icons/bs'
import { FaBarsStaggered } from 'react-icons/fa6'
import { NavLink } from 'react-router-dom'
import NavLinks from './NavLinks'

import { useDispatch, useSelector } from 'react-redux'
import { toggleTheme } from '../features/user/userSlice'

const Navbar = () => {
  const numItemsInCart = useSelector((state) => state.cartState.numItemsInCart)

  const dispatch = useDispatch()
  const handleTheme = () => {
    dispatch(toggleTheme())
  }
  return (
    <nav className="bg-base-200">
      <div className="navbar align-element ">
        <div className="navbar-start">
          {/* Title */}
          <NavLink
            to="/"
            className="hidden lg:flex btn btn-primary text-3xl items-center "
          >
            C
          </NavLink>
          {/* DROPDOWN */}
          <div className="dropdown">
            <label tabIndex={0} className="btn btn-ghost lg:hidden">
              <FaBarsStaggered className="h-6 w-6" />
            </label>
            <ul
              tabIndex={0}
              className="menu menu-sm dropdown-content mt-3 z-[1] p-2 shadow bg-base-200 rounded-box w-52"
            >
              <NavLinks />
            </ul>
          </div>
        </div>
        <div className="navbar-center hidden lg:flex">
          <ul className="menu menu-horizontal ">
            <NavLinks />
          </ul>
        </div>
        <div className="navbar-end">
          {/* THEME ICONS */}
          <label className="swap swap-rotate ">
            {/* this hidden checkbox controls the state */}
            <input type="checkbox" onChange={handleTheme} />

            {/* sun icon */}
            <BsSunFill className="swap-on h-4 w-4" />

            {/* moon icon */}
            <BsMoonFill className="swap-off h-4 w-4" />
          </label>
          {/* CART LINK*/}
          <NavLink to="cart" className="btn btn-ghost btn-circle btn-md ml-4">
            <div className="indicator">
              <BsCart3 className="h-6 w-6" />
              <span className="badge badge-sm badge-primary indicator-item">
                {numItemsInCart}
              </span>
            </div>
          </NavLink>
        </div>
      </div>
    </nav>
  )
}
export default Navbar
```

#### Additional Logic use selector access to html

Navbar.jsx

```js
import { useDispatch, useSelector } from 'react-redux';
import { toggleTheme } from '../features/user/userSlice';

const Navbar = () => {
  const theme = useSelector((state) => state.userState.theme);
  const isDarkTheme = theme === 'dracula';

  return (...
          {/* THEME SETUP */}
          <label className='swap swap-rotate'>
            <input
              type='checkbox'
              onChange={handleTheme}
              defaultChecked={isDarkTheme}
            />
            {/* sun icon*/}
            <BsSunFill className='swap-on h-4 w-4' />
            {/* moon icon*/}
            <BsMoonFill className='swap-off h-4 w-4' />
          </label>
          );
          ...
};
export default Navbar;
```

#### Challenge (42) - Setup Logout and Access User

- setup logout reducer
- access user in Header, NavLinks and Cart Page

#### Solution (42) - Setup Logout And Access User

userSlice.js

```js
logoutUser: (state) => {
      state.user = null;
      // localStorage.clear()
      localStorage.removeItem('user');
      toast.success('Logged out successfully');
    },
```

Header.js

```js
import { useDispaDtch, useSelector } from 'react-redux'
import { Link, useNavigate } from 'react-router-dom'
import { logoutUser } from '../features/user/userSlice'
import { clearCart } from '../features/cart/cartSlice'
const Header = () => {
  const navigate = useNavigate()
  const dispatch = useDispatch()
  const user = useSelector((state) => state.userState.user)

  const handleLogout = () => {
    navigate('/')
    dispatch(clearCart())
    dispatch(logoutUser())
  }
  return (
    <header className=" bg-neutral py-2 text-neutral-content ">
      <div className="align-element flex justify-center sm:justify-end ">
        {user ? (
          <div className="flex gap-x-2 sm:gap-x-8 items-center">
            <p className="text-xs sm:text-sm">Hello, {user.username}</p>
            <button
              className="btn btn-xs btn-outline btn-primary "
              onClick={handleLogout}
            >
              logout
            </button>
          </div>
        ) : (
          <div className="flex gap-x-6 justify-center items-center">
            <Link to="/login" className="link link-hover text-xs sm:text-sm">
              Sign in / Guest
            </Link>
            <Link to="/register" className="link link-hover text-xs sm:text-sm">
              Create an Account
            </Link>
          </div>
        )}
      </div>
    </header>
  )
}
export default Header
```

#### Solution (42) - Setup Logout And Access User

NavLinks.jsx

```js
import { useSelector } from 'react-redux'
import { NavLink } from 'react-router-dom'

const NavLinks = () => {
  const user = useSelector((state) => state.userState.user)

  return (
    <>
      {links.map((link) => {
        const { id, url, text } = link
        if ((url === 'checkout' || url === 'orders') && !user) return null
        return (
          <li key={id}>
            <NavLink className="capitalize" to={url}>
              {text}
            </NavLink>
          </li>
        )
      })}
    </>
  )
}
export default NavLinks
```

```js
const Cart = () => {
  // temp
  const { user } = useSelector((state) => state.userState)
}
```

#### Challenge (43) - Register User

- [API DOCS](https://documenter.getpostman.com/view/18152321/2s9Xy5KpTi)
- docs - register request
- test in Thunder Client
- setup action in Register
- add action to Register route in App.jsx

##### Register.js

- Import Dependencies:

  - Import `redirect` from `'react-router-dom'`.
  - Import `customFetch` from `'../utils'`.
  - Import `toast` from `'react-toastify'`.

- Define an asynchronous function named `action` that takes an object with a property named `request` as its parameter.

- Inside the `action` function:

  - Use the `request` object to get form data using the `formData` method.
  - Convert the form data to an object using `Object.fromEntries(formData)` and store it in the `data` variable.

- Use a `try` block to handle the registration process:

  - Send a POST request using `customFetch.post` to the `/auth/local/register` endpoint with the `data`.
  - If the request is successful:
    - Display a success toast message using `toast.success`.
    - Redirect the user to the `/login` page using `redirect('/login')`.

- Use a `catch` block to handle errors:
  - If an error occurs, extract the error message from the response data, if available, or provide a default error message.
  - Display the error message using `toast.error`.
  - Return `null` to indicate that an error occurred.

#### Solution (43) - Register User

Register.jsx

```js
import { FormInput, SubmitBtn } from '../components'
import { Form, redirect, Link } from 'react-router-dom'

import { customFetch } from '../utils'
import { toast } from 'react-toastify'
export const action = async ({ request }) => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)
  try {
    const response = await customFetch.post('/auth/local/register', data)
    toast.success('account created successfully')
    return redirect('/login')
  } catch (error) {
    const errorMessage =
      error?.response?.data?.error?.message ||
      'please double check your credentials'

    toast.error(errorMessage)
    return null
  }
}

const Register = () => {
  return (
    <section className="h-screen grid place-items-center">
      <Form
        method="POST"
        className="card w-96 py-8 px-8 bg-base-100 shadow-lg flex flex-col gap-y-4"
      >
        <h4 className="text-center text-3xl font-bold">Register</h4>
        <FormInput type="text" label="username" name="username" />
        <FormInput type="email" label="email" name="email" />
        <FormInput type="password" label="password" name="password" />
        <div className="mt-4">
          <SubmitBtn text="register" />
        </div>

        <p className="text-center">
          Already a member?
          <Link
            to="/login"
            className="ml-2 link link-hover link-primary capitalize"
          >
            login
          </Link>
        </p>
      </Form>
    </section>
  )
}
export default Register
```

#### Challenge (44) - Login Setup

- [API DOCS](https://documenter.getpostman.com/view/18152321/2s9Xy5KpTi)
- docs - login request
- test in Thunder Client
- setup action in login and access store

#### Solution (44) - Login Setup

App.jsx

```js
import { action as loginAction } from './pages/Login';

import { store } from './store';
{
    path: '/login',
    element: <Login />,
    errorElement: <Error />,
    action: loginAction(store),
  },
```

Login.jsx

```js
export const action =
  (store) =>
  async ({ request }) => {
    console.log(store)
    return store
  }
```

##### Challenge (45) - Login User

##### Login.jsx

- Import Dependencies:

  - Import `redirect` from `'react-router-dom'`.
  - Import `customFetch` from `'../utils'`.
  - Import `toast` from `'react-toastify'`.
  - Import `loginUser` from `'../features/user/userSlice'`.
  - Import `useDispatch` from `'react-redux'`.

- Define a function named `action` that takes a parameter `store` and returns an asynchronous function that takes an object with a property named `request`.

- Inside the inner asynchronous function:

  - Use the `request` object to get form data using the `formData` method.
  - Convert the form data to an object using `Object.fromEntries(formData)` and store it in the `data` variable.

- Use a `try` block to handle the login process:

  - Send a POST request using `customFetch.post` to the `/auth/local` endpoint with the `data`.
  - If the request is successful:
    - Dispatch the `loginUser` action with the response data using `store.dispatch`.
    - Display a success toast message using `toast.success`.
    - Redirect the user to the home page using `redirect('/')`.

- Use a `catch` block to handle errors:
  - If an error occurs, log the error to the console.
  - Extract the error message from the response data, if available, or provide a default error message.
  - Display the error message using `toast.error`.
  - Return `null` to indicate that an error occurred.

##### Solution (45) - Login User

userSlice.js

```js
loginUser: (state, action) => {
      console.log(action.payload);
    },
```

Login.jsx

```js
import { FormInput, SubmitBtn } from '../components'
import { Form, Link, redirect, useNavigate } from 'react-router-dom'
import { customFetch } from '../utils'
import { toast } from 'react-toastify'
import { loginUser } from '../features/user/userSlice'
import { useDispatch } from 'react-redux'

export const action =
  (store) =>
  async ({ request }) => {
    const formData = await request.formData()
    const data = Object.fromEntries(formData)
    try {
      const response = await customFetch.post('/auth/local', data)

      store.dispatch(loginUser(response.data))
      toast.success('logged in successfully')
      return redirect('/')
    } catch (error) {
      console.log(error)
      const errorMessage =
        error?.response?.data?.error?.message ||
        'please double check your credentials'

      toast.error(errorMessage)
      return null
    }
  }
```

userSlice.js

```js
const getUserFromLocalStorage = () => {
  return JSON.parse(localStorage.getItem('user')) || null;
};

const initialState = {
  user: getUserFromLocalStorage(),
  theme: getThemeFromLocalStorage(),
};

loginUser: (state, action) => {
      const user = { ...action.payload.user, token: action.payload.jwt };
      state.user = user;
      localStorage.setItem('user', JSON.stringify(user));
    },
```

#### Challenge (46) - Demo User

- remove defaultValue from inputs

##### loginAsGuestUser

- Create a function named `loginAsGuestUser`.
- Mark the function as `async` to indicate that it contains asynchronous operations.
- Wrap the entire function body in a `try` block to handle potential errors.
- Inside the `try` block, use the `customFetch.post` method to send a POST request.
- Provide the endpoint URL `/auth/local`.
- Pass an object with `identifier` and `password` properties as the request body.
- Assign the response from the `customFetch.post` call to the `response` variable.- If the request is successful, dispatch an action (e.g., `loginUser`) with the `response.data`.
- Use the `toast.success` method to display a success message (e.g., 'welcome guest user').
- If the action dispatch and toast success are successful, use the `navigate` function to navigate to a specific route (e.g., `'/'`).
- If any error occurs within the `try` block, it will be caught by the `catch` block.
- Inside the `catch` block, use `console.log` to log the error for debugging purposes.
- Display an error message using the `toast.error` method to notify the user about the login error.

#### Solution (46) - Demo User

Login.jsx

```js
const Login = () => {
  const dispatch = useDispatch()
  const navigate = useNavigate()
  const loginAsGuestUser = async () => {
    try {
      const response = await convenient.post('/auth/local', {
        identifier: 'test@test.com',
        password: 'secret',
      })
      dispatch(loginUser(response.data))
      toast.success('welcome guest user')
      navigate('/')
    } catch (error) {
      console.log(error)
      toast.error('guest user login error.please try later.')
    }
  }
}

;<button
  type="button"
  className="btn btn-secondary btn-block"
  onClick={loginAsGuestUser}
>
  guest user
</button>
```
