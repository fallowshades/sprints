# identifying workload

[ignore]

[done-every-private-session]

[that-one-user-type]

- require that role type

## User modification

### Get current id

#### 1. User Routes

controllers/userController.js

```js
import { StatusCodes } from 'http-status-codes'
import User from '../models/UserModel.js'
import Achievement from '../models/AchievementModel.js'

export const getCurrentUser = async (req, res) => {
  res.status(StatusCodes.OK).json({ msg: 'get current user' })
}

export const getApplicationStats = async (req, res) => {
  res.status(StatusCodes.OK).json({ msg: 'application stats' })
}

export const updateUser = async (req, res) => {
  res.status(StatusCodes.OK).json({ msg: 'update user' })
}
```

routes/userRouter.js

```js
import { Router } from 'express'
const router = Router()

import {
  getCurrentUser,
  getApplicationStats,
  updateUser,
} from '../controllers/userController.js'

router.get('/current-user', getCurrentUser)
router.get('/admin/app-stats', getApplicationStats)
router.patch('/update-user', updateUser)
export default router
```

server.js

```js
import userRouter from './routers/userRoutes.js'

app.use('/api/v1/users', authenticateUser, userRouter)
```

#### 2. Get Current User

```js
export const getCurrentUser = async (req, res) => {
  const user = await User.findOne({ _id: req.user.userId })
  res.status(StatusCodes.OK).json({ user })
}
```

#### 3. Remove Password

models/UserModel.js

```js
UserSchema.methods.toJSON = function () {
  var obj = this.toObject()
  delete obj.password
  return obj
}
```

```js
export const getCurrentUser = async (req, res) => {
  const user = await User.findOne({ _id: req.user.userId })
  const userWithoutPassword = user.toJSON()
  res.status(StatusCodes.OK).json({ user: userWithoutPassword })
}
```

#### 4. Update User

middleware/validationMiddleware.js

```js
export const validateUpdateUserInput = withValidationErrors([
  body('name').notEmpty().withMessage('name is required'),
  body('email')
    .notEmpty()
    .withMessage('email is required')
    .isEmail()
    .withMessage('invalid email format')
    .custom(async (email, { req }) => {
      const user = await User.findOne({ email })
      if (user && user._id.toString() !== req.user.userId) {
        throw new Error('email already exists')
      }
    }),
  body('lastName').notEmpty().withMessage('last name is required'),
  body('location').notEmpty().withMessage('location is required'),
])
```

controllers/userController.js

```js
export const updateUser = async (req, res) => {
  const updatedUser = await User.findByIdAndUpdate(req.user.userId, req.body)
  res.status(StatusCodes.OK).json({ msg: 'user updated' })
}
```

routes/userRouter.js

```js
import { validateUpdateUserInput } from '../middleware/validationMiddleware.js'
router.patch('/update-user', validateUpdateUserInput, updateUser)
```

```json
{
  "name": "john",
  "email": "john@gmail.com",
  "lastName": "smith",
  "location": "florida"
}
```

### Permission granted on application state data

#### 5. Application Stats

controllers/userController.js

```js
export const getApplicationStats = async (req, res) => {
  const users = await User.countDocuments()
  const achievement = await Achievement.countDocuments()
  res.status(StatusCodes.OK).json({ users, achievement })
}
```

middleware/

```js
export const authorizePermissions = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      throw new UnauthorizedError('Unauthorized to access this route')
    }
    next()
  }
}
```

routes/userRouter.js

```js
import { authorizePermissions } from '../middleware/authMiddleware.js'

router.get('/admin/app-stats', [
  authorizePermissions('admin'),
  getApplicationStats,
])
```

## path between

### secure path between front and back

#### 6. Setup Proxy

- only in dev env
- a must since cookies are sent back to the same server
- spin up both servers (our own and vite dev)

- server

```sh
npm run dev
```

- vite dev server

```sh
cd client && npm run dev
```

server.js

```js
app.get('/api/v1/test', (req, res) => {
  res.json({ msg: 'test route' })
})
```

client/src/main.jsx

```js
fetch('http://localhost:5100/api/v1/test')
  .then((res) => res.json())
  .then((data) => console.log(data))
```

client/vite.config.js

```js
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:5100/api',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
})
```

main.jsx

```js
fetch('/api/v1/test')
  .then((res) => res.json())
  .then((data) => console.log(data))
```

This code configures a proxy rule for the development server, specifically for requests that start with /api. Let's go through each property:

'/api': This is the path to match. If a request is made to the development server with a path that starts with /api, the proxy rule will be applied.
target: 'http://localhost:5100/api': This specifies the target URL where the requests will be redirected. In this case, any request that matches the /api path will be forwarded to http://localhost:5100/api.

changeOrigin: true: When set to true, this property changes the origin of the request to match the target URL. This can be useful when working with CORS (Cross-Origin Resource Sharing) restrictions.

rewrite: (path) => path.replace(/^\/api/, ''): This property allows you to modify the path of the request before it is forwarded to the target. In this case, the rewrite function uses a regular expression (/^\/api/) to remove the /api prefix from the path. For example, if a request is made to /api/users, the rewritten path will be /users.

To summarize, these lines of code configure a proxy rule for requests starting with /api on the development server. The requests will be redirected to http://localhost:5100/api, with the /api prefix removed from the path.

#### 7. Concurrently

The concurrently npm package is a utility that allows you to run multiple commands concurrently in the same terminal window. It provides a convenient way to execute multiple tasks or processes simultaneously.

```sh
npm i concurrently@8.0.1
```

```json
"scripts": {
    "setup-project": "npm i && cd client && npm i",
    "server": "nodemon server",
    "client": "cd client && npm run dev",
    "dev": "concurrently --kill-others-on-fail \" npm run server\" \" npm run client\""
  },
```

By default, when a command fails, concurrently continues running the remaining commands. However, when --kill-others-on-fail is specified, if any of the commands fail, concurrently will immediately terminate all the other running commands.

#### 8. Axios

Axios is a popular JavaScript library that simplifies the process of making HTTP requests from web browsers or Node.js. It provides a simple and elegant API for performing asynchronous HTTP requests, supporting features such as making GET, POST, PUT, and DELETE requests, handling request and response headers, handling request cancellation, and more.

[Axios Docs](https://axios-http.com/docs/intro)

```sh
npm i axios@1.3.6
```

main.jsx

```js
import axios from 'axios'

const data = await axios.get('/api/v1/test')
console.log(data)
```

#### Custom Instance

utils/customFetch.js

```js
import axios from 'axios'
const customFetch = axios.create({
  baseURL: '/api/v1',
})

export default customFetch
```

main.jsx

```js
import customFetch from './utils/customFetch.js'

const data = await customFetch.get('/test')
console.log(data)
```
