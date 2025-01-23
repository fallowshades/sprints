# ö

## User data known

### (1) Access User in a stateless fashion

#### Authenticate User Setup

- create auth.js in <b>middleware</b>

```js
const auth = async (req, res, next) => {
  console.log('authenticate user')
  next()
}

export default auth
```

```js
authRoutes.js

import authenticateUser from '../middleware/auth.js'

router.route('/updateUser').patch(authenticateUser, updateUser)
```

- two options

```js
server.js

import authenticateUser from './middleware/auth.js'
app.use('/api/v1/jobs', authenticateUser, jobsRouter)
```

```js
jobsRoutes.js

import authenticateUser from './middleware/auth.js'

// all routes !!!!

router.route('/stats').get(authenticateUser, showStats)
```

#### Auth - Bearer Schema

```js
Postman

Headers

Authorization: Bearer <token>

```

```js
auth.js

const auth = async (req, res, next) => {
  const headers = req.headers
  const authHeader = req.headers.authorization
  console.log(headers)
  console.log(authHeader)
  next()
}
```

#### Postman - Set Token Programmatically

- register and login routes
- Tests

```js
const jsonData = pm.response.json()
pm.globals.set('token', jsonData.token)

Type: Bearer

Token: {
  {
    token
  }
}
```

#### Unauthenticated Error

```js
auth.js

import { UnAuthenticatedError } from '../errors/index.js'

const auth = async (req, res, next) => {
  const authHeader = req.headers.authorization

  if (!authHeader) {
    // why, well is it 400 or 404?
    // actually 401
    throw new UnAuthenticatedError('Authentication Invalid')
  }

  next()
}
```

#### Auth Middleware

```js
import jwt from 'jsonwebtoken'
import { UnAuthenticatedError } from '../errors/index.js'

const auth = async (req, res, next) => {
  // check header
  const authHeader = req.headers.authorization
  if (!authHeader || !authHeader.startsWith('Bearer')) {
    throw new UnauthenticatedError('Authentication invalid')
  }
  const token = authHeader.split(' ')[1]

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET)
    // console.log(payload)
    // attach the user request object
    // req.user = payload
    req.user = { userId: payload.userId }
    next()
  } catch (error) {
    throw new UnauthenticatedError('Authentication invalid')
  }
}

export default auth
```

#### Update User

```js
const updateUser = async (req, res) => {
  const { email, name, lastName, location } = req.body
  if (!email || !name || !lastName || !location) {
    throw new BadRequestError('Please provide all values')
  }

  const user = await User.findOne({ _id: req.user.userId })

  user.email = email
  user.name = name
  user.lastName = lastName
  user.location = location

  await user.save()

  // various setups
  // in this case only id
  // if other properties included, must re-generate

  const token = user.createJWT()
  res.status(StatusCodes.OK).json({
    user,
    token,
    location: user.location,
  })
}
```

#### Modified Paths

- user.save() vs User.findOneAndUpdate

```js
User.js

UserSchema.pre('save', async function () {
  console.log(this.modifiedPaths())
  console.log(this.isModified('name'))

  // if (!this.isModified('password')) return
  // const salt = await bcrypt.genSalt(10)
  // this.password = await bcrypt.hash(this.password, salt)
})
```

### (2)Proper feedback cycle of personal modifiable data

#### Profile Page

```js
appContext.js

const updateUser = async (currentUser) => {
  console.log(currentUser)
}

value={{updateUser}}
```

```js
Profile.js

import { useState } from 'react'
import { FormRow, Alert } from '../../components'
import { useAppContext } from '../../context/appContext'
import Wrapper from '../../assets/wrappers/DashboardFormPage'

const Profile = () => {
  const { user, showAlert, displayAlert, updateUser, isLoading } =
    useAppContext()
  const [name, setName] = useState(user?.name)
  const [email, setEmail] = useState(user?.email)
  const [lastName, setLastName] = useState(user?.lastName)
  const [location, setLocation] = useState(user?.location)

  const handleSubmit = (e) => {
    e.preventDefault()
    if (!name || !email || !lastName || !location) {
      // test and remove temporary
      displayAlert()
      return
    }

    updateUser({ name, email, lastName, location })
  }
  return (
    <Wrapper>
      <form className="form" onSubmit={handleSubmit}>
        <h3>profile </h3>
        {showAlert && <Alert />}

        {/* name */}
        <div className="form-center">
          <FormRow
            type="text"
            name="name"
            value={name}
            handleChange={(e) => setName(e.target.value)}
          />
          <FormRow
            labelText="last name"
            type="text"
            name="lastName"
            value={lastName}
            handleChange={(e) => setLastName(e.target.value)}
          />
          <FormRow
            type="email"
            name="email"
            value={email}
            handleChange={(e) => setEmail(e.target.value)}
          />

          <FormRow
            type="text"
            name="location"
            value={location}
            handleChange={(e) => setLocation(e.target.value)}
          />
          <button className="btn btn-block" type="submit" disabled={isLoading}>
            {isLoading ? 'Please Wait...' : 'save changes'}
          </button>
        </div>
      </form>
    </Wrapper>
  )
}

export default Profile
```

#### Bearer Token - Manual Approach

```js
appContext.js

const updaterUser = async (currentUser) => {
  try {
    const { data } = await axios.patch('/api/v1/auth/updateUser', currentUser, {
      headers: {
        Authorization: `Bearer ${state.token}`,
      },
    })
    console.log(data)
  } catch (error) {
    console.log(error.response)
  }
}
```

#### Axios - Global Setup

<!-- IMPORTANT  -->

In current axios version,
common property returns undefined,
so we don't use it anymore!!!

```js
appContext.js

axios.defaults.headers['Authorization'] = `Bearer ${state.token}`
```

#### Axios - Setup Instance

```js
AppContext.js

const authFetch = axios.create({
  baseURL: '/api/v1',
  headers: {
    Authorization: `Bearer ${state.token}`,
  },
})

const updaterUser = async (currentUser) => {
  try {
    const { data } = await authFetch.patch('/auth/updateUser', currentUser)
  } catch (error) {
    console.log(error.response)
  }
}
```

#### Axios - Interceptors

- will use instance, but can use axios instead

<!-- IMPORTANT  -->

In current axios version,
common property returns undefined,
so we don't use it anymore!!!

```js
appContext.js

// response interceptor
authFetch.interceptors.request.use(
  (config) => {
    config.headers['Authorization'] = `Bearer ${state.token}`
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)
// response interceptor
authFetch.interceptors.response.use(
  (response) => {
    return response
  },
  (error) => {
    console.log(error.response)
    if (error.response.status === 401) {
      console.log('AUTH ERROR')
    }
    return Promise.reject(error)
  }
)
```

#### Update User

```js
actions.js
export const UPDATE_USER_BEGIN = 'UPDATE_USER_BEGIN'
export const UPDATE_USER_SUCCESS = 'UPDATE_USER_SUCCESS'
export const UPDATE_USER_ERROR = 'UPDATE_USER_ERROR'
```

```js
appContext.js

const updateUser = async (currentUser) => {
  dispatch({ type: UPDATE_USER_BEGIN })
  try {
    const { data } = await authFetch.patch('/auth/updateUser', currentUser)

    // no token
    const { user, location, token } = data

    dispatch({
      type: UPDATE_USER_SUCCESS,
      payload: { user, location, token },
    })

    addUserToLocalStorage({ user, location, token })
  } catch (error) {
    dispatch({
      type: UPDATE_USER_ERROR,
      payload: { msg: error.response.data.msg },
    })
  }
  clearAlert()
}
```

```js
reducer.js
if (action.type === UPDATE_USER_BEGIN) {
  return { ...state, isLoading: true }
}

if (action.type === UPDATE_USER_SUCCESS) {
  return {
    ...state,
    isLoading: false,
    token:action.payload.token
    user: action.payload.user,
    userLocation: action.payload.location,
    jobLocation: action.payload.location,
    showAlert: true,
    alertType: 'success',
    alertText: 'User Profile Updated!',
  }
}
if (action.type === UPDATE_USER_ERROR) {
  return {
    ...state,
    isLoading: false,
    showAlert: true,
    alertType: 'danger',
    alertText: action.payload.msg,
  }
}
```

#### 401 Error - Logout User

```js
appContext.js
// response interceptor
authFetch.interceptors.response.use(
  (response) => {
    return response
  },
  (error) => {
    if (error.response.status === 401) {
      logoutUser()
    }
    return Promise.reject(error)
  }
)

const updateUser = async (currentUser) => {
  dispatch({ type: UPDATE_USER_BEGIN })
  try {
    const { data } = await authFetch.patch('/auth/updateUser', currentUser)

    // no token
    const { user, location } = data

    dispatch({
      type: UPDATE_USER_SUCCESS,
      payload: { user, location, token },
    })

    addUserToLocalStorage({ user, location, token: initialState.token })
  } catch (error) {
    if (error.response.status !== 401) {
      dispatch({
        type: UPDATE_USER_ERROR,
        payload: { msg: error.response.data.msg },
      })
    }
  }
  clearAlert()
}
```
