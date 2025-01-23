## Transport with secure sockets kind of

### (1)Cookies

#### Test User - Initial Setup

- create new user (test user)
- populate DB with jobs
- create a login button

client/pages/Register.js

```js
const Register = () => {
  return (
    <Wrapper className="full-page">
      <form className="form" onSubmit={onSubmit}>
        <button type="submit" className="btn btn-block" disabled={isLoading}>
          submit
        </button>
        <button
          type="button"
          className="btn btn-block btn-hipster"
          disabled={isLoading}
          onClick={() => {
            setupUser({
              currentUser: { email: 'testUser@test.com', password: 'secret' },
              endPoint: 'login',
              alertText: 'Login Successful! Redirecting...',
            })
          }}
        >
          {isLoading ? 'loading...' : 'demo app'}
        </button>
      </form>
    </Wrapper>
  )
}
export default Register
```

#### Test User - Restrict Access (server)

- check for test user in auth middleware
- create new property on user object (true/false)
- create new middleware (testUser)
- check for test user, if true send back BadRequest Error
- add testUser middleware in front of routes you want to restrict access to

middleware/auth.js

```js
import jwt from 'jsonwebtoken'
import { UnAuthenticatedError } from '../errors/index.js'

UnAuthenticatedError
const auth = async (req, res, next) => {
  const authHeader = req.headers.authorization
  if (!authHeader || !authHeader.startsWith('Bearer')) {
    throw new UnAuthenticatedError('Authentication Invalid')
  }
  const token = authHeader.split(' ')[1]
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET)
    // TEST USER
    const testUser = payload.userId === 'testUserID'
    req.user = { userId: payload.userId, testUser }
    // TEST USER
    next()
  } catch (error) {
    throw new UnAuthenticatedError('Authentication Invalid')
  }
}

export default auth
```

middleware/testUser

```js
import { BadRequestError } from '../errors/index.js'

const testUser = (req, res, next) => {
  if (req.user.testUser) {
    throw new BadRequestError('Test User. Read Only!')
  }
  next()
}

export default testUser
```

routes/jobsRoutes

```js
import express from 'express'
const router = express.Router()

import {
  createJob,
  deleteJob,
  getAllJobs,
  updateJob,
  showStats,
} from '../controllers/jobsController.js'

import testUser from '../middleware/testUser.js'

router.route('/').post(testUser, createJob).get(getAllJobs)
// remember about :id
router.route('/stats').get(showStats)
router.route('/:id').delete(testUser, deleteJob).patch(testUser, updateJob)

export default router
```

routes/authRoutes

```js
import express from 'express'
const router = express.Router()

import rateLimiter from 'express-rate-limit'
const apiLimiter = rateLimiter({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10,
  message: 'Too many requests from this IP, please try again after 15 minutes',
})

import { register, login, updateUser } from '../controllers/authController.js'
import authenticateUser from '../middleware/auth.js'
import testUser from '../middleware/testUser.js'
router.route('/register').post(apiLimiter, register)
router.route('/login').post(apiLimiter, login)
router.route('/updateUser').patch(authenticateUser, testUser, updateUser)

export default router
```

#### Store JWT in Cookie

- BE PREPARED TO REFACTOR CODE !!!
- PLEASE DON'T RUSH THROUGH THESES VIDEOS
- CHECK FEW TIMES BEFORE REMOVING/ADDING CODE

#### Attach Cookies to Login response

controllers/authController.js

```js
// login controller

const token = user.createJWT()

const oneDay = 1000 * 60 * 60 * 24

res.cookie('token', token, {
  httpOnly: true,
  expires: new Date(Date.now() + oneDay),
  secure: process.env.NODE_ENV === 'production',
})
```

#### Setup Function in Utils

- create attachCookies.js

```js
const attachCookie = ({ res, token }) => {
  const oneDay = 1000 * 60 * 60 * 24

  res.cookie('token', token, {
    httpOnly: true,
    expires: new Date(Date.now() + oneDay),
    secure: process.env.NODE_ENV === 'production',
  })
}

export default attachCookie
```

- import in authController.js
- invoke in register/login/updateUser

```js
import attachCookie from '../utils/attachCookie.js'

attachCookie({ res, token })
```

#### Parse Cookie Coming Back from the Front-End

- install cookie-parser (server)

```sh
npm install cookie-parser
```

server.js

```js
import cookieParser from 'cookie-parser'

app.use(express.json())
app.use(cookieParser())
```

middleware/auth.js

```js
const auth = async (req, res, next) => {
  console.log(req.cookies)
  ....
}
```

#### Refactor Auth Middleware

middleware/auth.js

```js
const auth = async (req, res, next) => {
  const token = req.cookies.token
  if (!token) {
    throw new UnAuthenticatedError('Authentication Invalid')
  }
  // rest of the code
}
```

#### SERVER - Remove Token from JSON Response

controllers/authController

register/login/updateUser

```js
res.status(StatusCodes.OK).json({ user, location: user.location })
```

- test the APP

#### FRONT-END Remove Token from CONTEXT

- PLEASE BE CAREFUL WHEN MAKING THESE UPDATES
  client/context/appContext

- remove

```js
const token = localStorage.getItem('token')
const user = localStorage.getItem('user')
const userLocation = localStorage.getItem('location')
```

- fix initial state

```js
const initialState = {
  // remove token all together
  user: null,
  userLocation: '',
  jobLocation: '',
}
```

- remove request interceptor

```js
authFetch.interceptors.request.use(
  (config) => {
    config.headers.common['Authorization'] = `Bearer ${state.token}`
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)
```

- remove both addToLocalStorage and removeFromLocalStorage functions
- remove from setupUser and updateUser (token and local storage functions)
- remove from the reducer token (COMMAND + F)

```js
const logoutUser = async () => {
  dispatch({ type: LOGOUT_USER })
  // remove local storage code
}
```

#### Test Expiration

```js
expires: new Date(Date.now() + 5000),
```

#### GET Current User Route

controllers/authController.js

```js
const getCurrentUser = async (req, res) => {
  const user = await User.findOne({ _id: req.user.userId })
  res.status(StatusCodes.OK).json({ user, location: user.location })
}

export { register, login, updateUser, getCurrentUser }
```

routes/authRoutes.js

```js
import {
  register,
  login,
  updateUser,
  getCurrentUser,
} from '../controllers/authController.js'

router.route('/register').post(apiLimiter, register)
router.route('/login').post(apiLimiter, login)
router.route('/updateUser').patch(authenticateUser, testUser, updateUser)
router.route('/getCurrentUser').get(authenticateUser, getCurrentUser)
```

#### GET Current User - Front-End

actions.js

```js
export const GET_CURRENT_USER_BEGIN = 'GET_CURRENT_USER_BEGIN'
export const GET_CURRENT_USER_SUCCESS = 'GET_CURRENT_USER_SUCCESS'
```

- setup imports (appContext and reducer)

#### GET Current User Request

- first set the state value (default TRUE !!!)
  appContext.js

```js
const initialState = {
  userLoading: true,
}

const getCurrentUser = async () => {
  dispatch({ type: GET_CURRENT_USER_BEGIN })
  try {
    const { data } = await authFetch('/auth/getCurrentUser')
    const { user, location } = data

    dispatch({
      type: GET_CURRENT_USER_SUCCESS,
      payload: { user, location },
    })
  } catch (error) {
    if (error.response.status === 401) return
    logoutUser()
  }
}
useEffect(() => {
  getCurrentUser()
}, [])
```

reducer.js

```js
if (action.type === GET_CURRENT_USER_BEGIN) {
  return { ...state, userLoading: true, showAlert: false }
}
if (action.type === GET_CURRENT_USER_SUCCESS) {
  return {
    ...state,
    userLoading: false,
    user: action.payload.user,
    userLocation: action.payload.location,
    jobLocation: action.payload.location,
  }
}
```

```js
if (action.type === LOGOUT_USER) {
  return {
    ...initialState,
    userLoading: false,
  }
}
```

#### Protected Route FIX

```js
import Loading from '../components/Loading'

const ProtectedRoute = ({ children }) => {
  const { user, userLoading } = useAppContext()

  if (userLoading) return <Loading />

  if (!user) {
    return <Navigate to="/landing" />
  }
  return children
}

export default ProtectedRoute
```

#### ()Landing Page

```js
import { Navigate } from 'react-router-dom'
import { useAppContext } from '../context/appContext'

const Landing = () => {
  const { user } = useAppContext()
  return (
    <React.Fragment>
      {user && <Navigate to="/" />}
      <Wrapper>// rest of the code..........</Wrapper>
    </React.Fragment>
  )
}

export default Landing
```

#### Logout Route

controllers/authController

```js
const logout = async (req, res) => {
  res.cookie('token', 'logout', {
    httpOnly: true,
    expires: new Date(Date.now() + 1000),
  })
  res.status(StatusCodes.OK).json({ msg: 'user logged out!' })
}
```

routes/authRoutes

```js
import {
  register,
  login,
  updateUser,
  getCurrentUser,
  logout,
} from '../controllers/authController.js'

router.route('/register').post(apiLimiter, register)
router.route('/login').post(apiLimiter, login)
router.get('/logout', logout)
// rest of the code ....
```

#### Logout - Front-End

appContext.js

```js
const logoutUser = async () => {
  await authFetch.get('/auth/logout')
  dispatch({ type: LOGOUT_USER })
}
```

### (2)Horizontal sesion

#### Prepare for Deployment

- in client remove build and node_modules
- in server remove node_modules

package.json

```json

"scripts":{
  "setup-production":"npm run install-client && npm run build-client && npm install",
  "install-client":"cd client && npm install",
}



```

- node server
- APP NEEDS TO WORK LOCALLY !!!

#### Github Repo

- create new repo
- remove all existing repos (CHECK CLIENT !!!)
- in the root
- git init
- git add .
- git commit -m "first commit"
- push up to Github
