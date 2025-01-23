## iteration 2- backNode

### (1) Achieve task level parallelism with ability to communicate (built in event Emitter/driver)

#### Setup Server

- stop the dev server in client
- cd ..
- start setting up our server
- setup package.json

```sh
npm init -y
```

- create server.js
- console.log('server running...')

```sh
node server
```

#### ES6 vs CommonJS

```js
CommonJS

const express = require('express')
const app = express()
```

```js
ES6

import express from 'express'
const app = express()
```

- file extension .mjs

```js
package.json

"type":"module"
```

#### Nodemon and Basic Express Server

```sh
npm install nodemon --save-dev
```

```js
package.json

"start":"nodemon server"

```

```sh
npm install express
```

```js
import express from 'express'
const app = express()

app.get('/', (req, res) => {
  res.send('Welcome!')
})

const port = process.env.PORT || 5000

app.listen(port, () => console.log(`Server is listening on port ${port}...`))
```

#### Not Found Middleware

- in the root create <b>middleware</b> folder
- not-found.js
- setup function
- return 404 with message 'Route does not exist'
- import in server.js
- make sure to use .js extension
- place after home route

#### Error Middleware

- in the middleware create error-handler.js
- setup function
- accept 4 parameters, first one error
- log error
- return 500
- json({msg:'there was an error'})
- import in the server.js
- make sure to use .js extension
- place it last
- eventually handle Mongoose Errors, just like in the node-express
- showcase with async errors

#### ENV Variables

```sh
npm install dotenv
```

- import dotenv from 'dotenv'
- dotenv.config()

- create .env
- PORT=4000
- .gitignore
- /node_modules
- .env

#### Connect to MongoDB

- switched back to PORT=5000
- remove Error from '/'

- existing MongoDB Atlas Account

```sh
npm install mongoose
```

- create <b>db</b> folder
- create connect.js
- setup connectDB(url)
- in server.js create start() function
- get connection string
- setup as MONGO_URL in .env
- provide credentials and DB Name

### (2) Validate user permitted access user related data subsets

#### Auth Controller and Route Structure

- create <b>controllers</b>
- authController.js
- create async functions

```js
export { register, login, updateUser }
```

- return res.send('function name')
- create <b>routes</b> folder
- authRoutes.js
- setup express router
- import functions from authController.js

```js
router.route('/register').post(register)
router.route('/login').post(login)
router.route('/updateUser').patch(updateUser)

export default router
```

- import authRouter in server.js

```js
app.use('/api/v1/auth', authRouter)
```

#### Jobs Controller and Route Structure

- jobsController.js
- create async functions

```js
export { createJob, deleteJob, getAllJobs, updateJob, showStats }
```

- return res.send('function name')

- jobsRoutes.js
- setup express router
- import functions from jobsController.js

```js
router.route('/').post(createJob).get(getAllJobs)
// place before :id
router.route('/stats').get(showStats)
router.route('/:id').delete(deleteJob).patch(updateJob)

export default router
```

- in server.js jobsRouter

```js
app.use('/api/v1/jobs', jobsRouter)
```

#### Postman

- URL global var
- JOBIFY Collection
- auth and jobs folders
- setup routes

#### User Model

- <b>models</b> folder
- User.js
- setup schema
- name, email, password, lastName, location
- all {type:String}

### (3) Able to exchange explici responses

#### Validate Email

```js
validate:{
  validator:(field)=> {return 2 > 1},
  message:'Please provide valid email'
  }
```

- [Validator Package](https://www.npmjs.com/package/validator)

```sh
npm install validator
```

- import in User.js
- validator.isEmail

#### Register User - Initial Setup

- authController
- import User model
- setup temporary try/catch
- await User.create(req.body)
- if success 201 with json({user}) (temp)
- if error 500 with json({msg:'there was an error'})

#### Pass Error to Error Handler

- next(error)

#### Express-Async-Errors Package

- remove try/catch
- [Express-Async-Errors](https://www.npmjs.com/package/express-async-errors)

```sh
npm install express-async-errors

```

- in server.js
- import 'express-async-errors'

- use throw Error('error') instead of next(error)

#### Http Status Codes

- constants for status codes
- personal preference
- provides consistency
- less bugs
- easier to read/manage

- [Http Status Codes](https://www.npmjs.com/package/http-status-codes)

```sh
npm install http-status-codes
```

- import/setup in authController and error-handler
- setup defaultError

#### Custom Errors

#### Refactor Errors

- create errors folder
- create custom-api, bad-request, not-found, index.js files
- add proper imports
- setup index.js just like in the front-end
- import {BadRequestError} in authController
- gotcha "errors/index.js"
