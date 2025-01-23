## 1st exchange

### (1) Connect to network with intention to identify

#### Register User - Setup

```js
appContext.js

const initialState = {
  user: null,
  token: null,
  userLocation: '',
}
```

- actions.js REGISTER_USER_BEGIN,SUCCESS,ERROR
- import reducer,appContext

```js
appContext.js
const registerUser = async (currentUser) => {
  console.log(currentUser)
}
```

- import in Register.js

```js
Register.js

const currentUser = { name, email, password }
if (isMember) {
  console.log('already a member')
} else {
  registerUser(currentUser)
}

return (
  <button type="submit" className="btn btn-block" disabled={isLoading}>
    submit
  </button>
)
```

#### Axios

- [axios docs](https://axios-http.com/docs/intro)
- stop app
- cd client

```sh
npm install axios
```

- cd ..
- restart app

#### Register User - Complete

```js
appContext.js

import axios from 'axios'

const registerUser = async (currentUser) => {
  dispatch({ type: REGISTER_USER_BEGIN })
  try {
    const response = await axios.post('/api/v1/auth/register', currentUser)
    console.log(response)
    const { user, token, location } = response.data
    dispatch({
      type: REGISTER_USER_SUCCESS,
      payload: {
        user,
        token,
        location,
      },
    })

    // will add later
    // addUserToLocalStorage({
    //   user,
    //   token,
    //   location,
    // })
  } catch (error) {
    console.log(error.response)
    dispatch({
      type: REGISTER_USER_ERROR,
      payload: { msg: error.response.data.msg },
    })
  }
  clearAlert()
}
```

```js
reducer.js
if (action.type === REGISTER_USER_BEGIN) {
  return { ...state, isLoading: true }
}
if (action.type === REGISTER_USER_SUCCESS) {
  return {
    ...state,
    user: action.payload.user,
    token: action.payload.token,
    userLocation: action.payload.location,
    jobLocation: action.payload.location,
    isLoading: false,
    showAlert: true,
    alertType: 'success',
    alertText: 'User Created! Redirecting...',
  }
}
if (action.type === REGISTER_USER_ERROR) {
  return {
    ...state,
    isLoading: false,
    showAlert: true,
    alertType: 'danger',
    alertText: action.payload.msg,
  }
}
```

### (2) Response to identified user action to access subset

#### Navigate To Dashboard

```js
Register.js
import { useEffect } from 'react'
import { useNavigate } from 'react-router-dom'

const Register = () => {
  const { user } = useAppContext()
  const navigate = useNavigate()

  useEffect(() => {
    if (user) {
      setTimeout(() => {
        navigate('/')
      }, 3000)
    }
  }, [user, navigate])
}
```

#### Local Storage

```js
appContext.js
const addUserToLocalStorage = ({ user, token, location }) => {
  localStorage.setItem('user', JSON.stringify(user))
  localStorage.setItem('token', token)
  localStorage.setItem('location', location)
}

const removeUserFromLocalStorage = () => {
  localStorage.removeItem('token')
  localStorage.removeItem('user')
  localStorage.removeItem('location')
}

const registerUser = async (currentUser) => {
  // in try block
  addUserToLocalStorage({
    user,
    token,
    location,
  })
}

// set as default
const token = localStorage.getItem('token')
const user = localStorage.getItem('user')
const userLocation = localStorage.getItem('location')

const initialState = {
  user: user ? JSON.parse(user) : null,
  token: token,
  userLocation: userLocation || '',
  jobLocation: userLocation || '',
}
```

### (3) Token noted in morgan for various users in production and development log ins

#### Morgan Package

- http logger middleware for node.js
- [morgan docs](https://www.npmjs.com/package/morgan)

```sh
npm install morgan
```

```js
import morgan from 'morgan'

if (process.env.NODE_ENV !== 'production') {
  app.use(morgan('dev'))
}
```

#### UnauthenticatedError

- unauthenticated.js in errors
- import/export

```js
import { StatusCodes } from 'http-status-codes'
import CustomAPIError from './custom-api.js'

class UnauthenticatedError extends CustomAPIError {
  constructor(message) {
    super(message)
    this.statusCode = StatusCodes.UNAUTHORIZED
  }
}
```

#### Compare Password

```js
User.js in models

UserSchema.methods.comparePassword = async function (candidatePassword) {
  const isMatch = await bcrypt.compare(candidatePassword, this.password)
  return isMatch
}
```

```js
authController.js
const login = async (req, res) => {
  const { email, password } = req.body
  if (!email || !password) {
    throw new BadRequestError('Please provide all values')
  }
  const user = await User.findOne({ email }).select('+password')

  if (!user) {
    throw new UnauthenticatedError('Invalid Credentials')
  }
  const isPasswordCorrect = await user.comparePassword(password)
  if (!isPasswordCorrect) {
    throw new UnauthenticatedError('Invalid Credentials')
  }
  const token = user.createJWT()
  user.password = undefined
  res.status(StatusCodes.OK).json({ user, token, location: user.location })
}
```

- test in Postman

#### Login User - Setup

- actions.js LOGIN_USER_BEGIN,SUCCESS,ERROR
- import reducer,appContext

```js
appContext.js
const loginUser = async (currentUser) => {
  console.log(currentUser)
}
```

- import in Register.js

```js
Register.js

if (isMember) {
  loginUser(currentUser)
} else {
  registerUser(currentUser)
}
```

#### Login User - Complete

```js
appContext.js
const loginUser = async (currentUser) => {
  dispatch({ type: LOGIN_USER_BEGIN })
  try {
    const { data } = await axios.post('/api/v1/auth/login', currentUser)
    const { user, token, location } = data

    dispatch({
      type: LOGIN_USER_SUCCESS,
      payload: { user, token, location },
    })

    addUserToLocalStorage({ user, token, location })
  } catch (error) {
    dispatch({
      type: LOGIN_USER_ERROR,
      payload: { msg: error.response.data.msg },
    })
  }
  clearAlert()
}
```

```js
reducer.js

if (action.type === LOGIN_USER_BEGIN) {
  return {
    ...state,
    isLoading: true,
  }
}
if (action.type === LOGIN_USER_SUCCESS) {
  return {
    ...state,
    isLoading: false,
    user: action.payload.user,
    token: action.payload.token,
    userLocation: action.payload.location,
    jobLocation: action.payload.location,
    showAlert: true,
    alertType: 'success',
    alertText: 'Login Successful! Redirecting...',
  }
}
if (action.type === LOGIN_USER_ERROR) {
  return {
    ...state,
    isLoading: false,
    showAlert: true,
    alertType: 'danger',
    alertText: action.payload.msg,
  }
}
```

#### Refactor

```js
actions.js
export const SETUP_USER_BEGIN = 'SETUP_USER_BEGIN'
export const SETUP_USER_SUCCESS = 'SETUP_USER_SUCCESS'
export const SETUP_USER_ERROR = 'SETUP_USER_ERROR'
```

```js
appContext.js

const setupUser = async ({ currentUser, endPoint, alertText }) => {
  dispatch({ type: SETUP_USER_BEGIN })
  try {
    const { data } = await axios.post(`/api/v1/auth/${endPoint}`, currentUser)

    const { user, token, location } = data
    dispatch({
      type: SETUP_USER_SUCCESS,
      payload: { user, token, location, alertText },
    })
    addUserToLocalStorage({ user, token, location })
  } catch (error) {
    dispatch({
      type: SETUP_USER_ERROR,
      payload: { msg: error.response.data.msg },
    })
  }
  clearAlert()
}
```

```js
reducer.js
if (action.type === SETUP_USER_BEGIN) {
  return { ...state, isLoading: true }
}
if (action.type === SETUP_USER_SUCCESS) {
  return {
    ...state,
    isLoading: false,
    token: action.payload.token,
    user: action.payload.user,
    userLocation: action.payload.location,
    jobLocation: action.payload.location,
    showAlert: true,
    alertType: 'success',
    alertText: action.payload.alertText,
  }
}
if (action.type === SETUP_USER_ERROR) {
  return {
    ...state,
    isLoading: false,
    showAlert: true,
    alertType: 'danger',
    alertText: action.payload.msg,
  }
}
```

- import/export

```js
Register.js

const onSubmit = (e) => {
  e.preventDefault()
  const { name, email, password, isMember } = values
  if (!email || !password || (!isMember && !name)) {
    displayAlert()
    return
  }
  const currentUser = { name, email, password }
  if (isMember) {
    setupUser({
      currentUser,
      endPoint: 'login',
      alertText: 'Login Successful! Redirecting...',
    })
  } else {
    setupUser({
      currentUser,
      endPoint: 'register',
      alertText: 'User Created! Redirecting...',
    })
  }
}
```
