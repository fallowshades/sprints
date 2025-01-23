# Proper remote set up for identification

[search]

- all ||id (req,res)

[scale]

- couter factory
- bubble up wrapper
- ord schema interoret send inout obj

## SDK to remote architecture

### Successive flow

#### 1. Basic CRUD

- create jobs array where each item is an object with following properties
  id, description, position
- create routes to handle - create, read, update and delete functionalities

#### 2. Get All Achievements

[Nanoid](https://www.npmjs.com/package/nanoid)

The nanoid package is a software library used for generating unique and compact identifiers in web applications or databases. It creates short and URL-safe IDs by combining random characters from a set of 64 characters. Nanoid is a popular choice due to its simplicity, efficiency, and collision-resistant nature.

```sh
npm i nanoid@4.0.2
```

server.js

```js
import { nanoid } from 'nanoid'

let achievements = [
  { id: nanoid(), description: 'start intro', track: false },
  { id: nanoid(), description: 'complete intro', track: false },
]

app.get('/api/v1/achievements', (req, res) => {
  res.status(200).json({ achievements })
})
```

#### 3. Create, FindOne, Modify and Delete

```js
// CREATE JOB (ignore other than string for now)

app.post('/api/v1/achievements', (req, res) => {
  const { description, track } = req.body
  if (!description || !track) {
    return res.status(400).json({ msg: 'please provide company and position' })
  }
  const id = nanoid(10)
  console.log(id)
  const achievement = { id, description, track }
  achievements.push(achievement)
  res.status(200).json({ achievement })
})

// GET SINGLE JOB

app.get('/api/v1/achievements/:id', (req, res) => {
  const { id } = req.params
  const job = achievement.find((job) => achievement.id === id)
  if (!achievement) {
    return res.status(404).json({ msg: `no job with id ${id}` })
  }
  res.status(200).json({ job })
})

// EDIT JOB

app.patch('/api/v1/achievements/:id', (req, res) => {
  const { description, track } = req.body
  if (!description || !track) {
    return res.status(400).json({ msg: 'please provide description and track' })
  }
  const { id } = req.params
  const achievement = achievements.find((achievement) => achievement.id === id)
  if (!achievement) {
    return res.status(404).json({ msg: `no achievement with id ${id}` })
  }

  achievement.description = description
  achievement.track = track
  res.status(200).json({ msg: 'achievement modified', achievement })
})

// DELETE achievement

app.delete('/api/v1/achievements/:id', (req, res) => {
  const { id } = req.params
  const achievement = achievements.find((achievement) => achievement.id === id)
  if (!achievement) {
    return res.status(404).json({ msg: `no achievement with id ${id}` })
  }
  const newAchievements = achievements.filter(
    (achievement) => achievement.id !== id
  )
  achievement = newAchievements

  res.status(200).json({ msg: 'achievement deleted' })
})
```

### Erroneous flow

#### 4. Not Found Middleware

```js
app.use('*', (req, res) => {
  res.status(404).json({ msg: 'not found' })
})
```

#### 5. Error Middleware

```js
app.use((err, req, res, next) => {
  console.log(err)
  res.status(500).json({ msg: 'something went wrong' })
})
```

#### 6. Not Found and Error Middleware

The "not found" middleware in Express.js is used when a request is made to a route that does not exist. It catches these requests and responds with a 404 status code, indicating that the requested resource was not found.

On the other hand, the "error" middleware in Express.js is used to handle any errors that occur during the processing of a request. It is typically used to catch unexpected errors or exceptions that are not explicitly handled in the application code. It logs the error and sends a 500 status code, indicating an internal server error.

In summary, the "not found" middleware is specifically designed to handle requests for non-existent routes, while the "error" middleware is a catch-all for handling unexpected errors that occur during request processing.

- make a request to "/achievements"

```js
// GET ALL achievements
app.get('/api/v1/achievements', (req, res) => {
  // console.log(achievements);
  res.status(200).json({ jobs })
})

// GET SINGLE JOB
app.get('/api/v1/achievements/:id', (req, res) => {
  const { id } = req.params
  const achievement = achievements.find((achievement) => job.id === id)
  if (!achievement) {
    throw new Error('no achievement with that id')
    return res.status(404).json({ msg: `no achievement with id ${id}` })
  }
  res.status(200).json({ achievement })
})
```

#### 7. Controller and Router

setup controllers and router

controllers/achievementController.js

```js
import { nanoid } from 'nanoid'

let achievements = [
  { id: nanoid(), description: 'start', track: 'front-end developer' },
  { id: nanoid(), description: 'end', track: 'back-end developer' },
]

export const getAllAchievements = async (req, res) => {
  res.status(200).json({ achievements })
}

export const createAchievement = async (req, res) => {
  const { description, track } = req.body

  if (!description || !track) {
    return res.status(400).json({ msg: 'please provide description and track' })
  }
  const id = nanoid(10)
  const achievement = { id, description, track }
  achievements.push(achievement)
  res.status(200).json({ achievement })
}

export const getAchievement = async (req, res) => {
  const { id } = req.params
  const achievement = achievements.find((achievement) => achievement.id === id)
  if (!achievement) {
    // throw new Error('no achievement with that id');
    return res.status(404).json({ msg: `no achievement with id ${id}` })
  }
  res.status(200).json({ achievement })
}

export const updateAchievement = async (req, res) => {
  const { description, track } = req.body
  if (!description || !track) {
    return res.status(400).json({ msg: 'please provide description and track' })
  }
  const { id } = req.params
  const achievement = achievements.find((achievement) => achievement.id === id)
  if (!achievement) {
    return res.status(404).json({ msg: `no achievement with id ${id}` })
  }

  achievement.description = description
  achievement.track = track
  res.status(200).json({ msg: 'achievement modified', achievement })
}

export const deleteAchievement = async (req, res) => {
  const { id } = req.params
  const achievement = achievements.find((achievement) => achievement.id === id)
  if (!achievement) {
    return res.status(404).json({ msg: `no job with id ${id}` })
  }
  const newAchievements = achievements.filter(
    (achievement) => achievement.id !== id
  )
  achievements = newAchievements

  res.status(200).json({ msg: 'achievement deleted' })
}
```

routes/AchievementRouter.js

```js
import { Router } from 'express'
const router = Router()

import {
  getAllAchievements,
  getAchievement,
  createAchievement,
  updateAchievement,
  deleteAchievement,
} from '../controllers/achievementController.js'

// router.get('/', getAllJobs);
// router.post('/', createJob);

router.route('/').get(getAllAchievement).post(createAchievement)
router
  .route('/:id')
  .get(getAchievement)
  .patch(updateAchievement)
  .delete(deleteAchievement)

export default router
```

server.js

```js
import achievementRouter from './routes/achievementRouter.js'
app.use('/api/v1/achievements', achievementRouter)
```

## Uncerntainty in-box try-catch

### Connection dependence

#### 8. MongoDB

[MongoDb](https://www.mongodb.com/)

MongoDB is a popular NoSQL database that provides a flexible and scalable approach to storing and retrieving data. It uses a document-oriented model, where data is organized into collections of JSON-like documents. MongoDB offers high performance, horizontal scalability, and easy integration with modern development frameworks, making it suitable for handling diverse data types and handling large-scale applications.

MongoDB Atlas is a fully managed cloud database service provided by MongoDB, offering automated deployment, scaling, and monitoring of MongoDB clusters, allowing developers to focus on building their applications without worrying about infrastructure management.

#### 9. Mongoosejs

[Mongoose](https://mongoosejs.com/)

Mongoose is an Object Data Modeling (ODM) library for Node.js that provides a straightforward and elegant way to interact with MongoDB. It allows developers to define schemas and models for their data, providing structure and validation. Mongoose also offers features like data querying, middleware, and support for data relationships, making it a powerful tool for building MongoDB-based applications.

```sh
npm i mongoose@7.0.5
```

server.js

```js
import mongoose from 'mongoose'

try {
  await mongoose.connect(process.env.MONGO_URL)
  app.listen(port, () => {
    console.log(`server running on PORT ${port}....`)
  })
} catch (error) {
  console.log(error)
  process.exit(1)
}
```

#### 10. Achievement Model

models/achievementModel.js

enum - data type represents a field with a predefined set of values

```js
import mongoose from 'mongoose'

const AchievementSchema = new mongoose.Schema(
  {
    description: String,
    //uh mb later
    Status: {
      type: String,
      enum: ['inactive', 'activated', 'complete'],
      default: 'pending',
    },
    points: {
      type: Number,
      default: 0,
    },

    type: {
      type: String,
      enum: [
        'progressive',
        'exploration',
        'time-based',
        'skill-based',
        'social',
        'collection',
        'storyline',
        'event',
        'hidden',
        'lifetime',
      ],
      required: true,
      default: 'exploration',
    },
    dateOfCompletion: {
      type: Date,
      default: null,
    },
  },
  { timestamps: true }
)

export default mongoose.model('achievement', AchievementSchema)
```

#### 11. Create achievement with routes

achievementController.js

```js
import Achievement from '../models/achievementModel.js'

export const createAchievement = async (req, res) => {
  const { description } = req.body
  const achievement = await Achievement.create({ description })
  res.status(201).json({ achievement })
}
```

achievementRouter.js

```js
import { Router } from 'express'
const router = Router()

import { createAchievement } from '../controllers/achievementController.js'

router.route('/').post(createAchievement)

export default router
```

server.js

```js
import achievementRouter from './routes/achievementRouter.js'
app.use('/api/v1/achievements', achievementRouter)
```

#### 12. Try / Catch (meh i used express-async-error before)

jobController.js

```js
export const createAchievement = async (req, res) => {
  const { description } = req.body
  try {
    const achievement = await Achievement.create('something')
    res.status(201).json({ job })
  } catch (error) {
    res.status(500).json({ msg: 'server error' })
  }
}
```

#### 13. express-async-errors

The "express-async-errors" package is an Express.js middleware that helps handle errors that occur within asynchronous functions. It catches unhandled errors inside async/await functions and forwards them to Express.js's error handling middleware, preventing the Node.js process from crashing. It simplifies error handling in Express.js applications by allowing you to write asynchronous code without worrying about manually catching and forwarding errors.

[Express Async Errors](https://www.npmjs.com/package/express-async-errors)

```sh
npm i express-async-errors@3.1.1
```

- setup import at the top !!!

  server.js

```js
import 'express-async-errors'
```

AchievementController.js

```js
export const createAchievement = async (req, res) => {
  const { description } = req.body

  const achievement = await Achievement.create({ description })
  res.status(201).json({ achievement })
}
```

#### 14. Get All Achievements

achievementsController.js

```js
export const getAllAchievements = async (req, res) => {
  const achievement = await Achievement.find({})
  res.status(200).json({ achievement })
}
```

achievementRouter.js

```js
router.route('/').get(getAllAchievements).post(createAchievement)
```

#### 15. Get Single Achievement

```js
export const getAchievement = async (req, res) => {
  const { id } = req.params
  const achievement = await Achievement.findById(id)
  if (!achievement) {
    return res.status(404).json({ msg: `no achievement with id ${id}` })
  }
  res.status(200).json({ achievement })
}
```

achievementRouter.js

```js
router.route('/id').get(getAchievement)
```

#### 16. Delete Achievement

AchievementController.js

```js
export const deleteAchievement = async (req, res) => {
  const { id } = req.params
  const removedAchievement = await Achievement.findByIdAndDelete(id)

  if (!removedAchievement) {
    return res.status(404).json({ msg: `no job with id ${id}` })
  }
  res.status(200).json({ achievement: removedAchievement })
}
```

achievementRouter.js

```js
router.route('/:id').get(getAchievement).delete(deleteAchievement)
```

#### 17. Update Achievement

```js
export const updateAchievement = async (req, res) => {
  const { id } = req.params

  const updatedAchievement = await Achievement.findByIdAndUpdate(id, req.body, {
    new: true,
  })

  if (!updatedAchievement) {
    return res.status(404).json({ msg: `no achievement with id ${id}` })
  }

  res.status(200).json({ description: updatedAchievement })
}
```

achievementRouter.js

```js
router
  .route('/:id')
  .get(getAchievement)
  .delete(deleteAchievement)
  .patch(updateAchievement)
```

### Expressive module application also middleware uncerntainty

#### 18 Status Codes

A library for HTTP status codes is useful because it provides a comprehensive and standardized set of codes that represent the outcome of HTTP requests. It allows developers to easily understand and handle different scenarios during web development, such as successful responses, client or server errors, redirects, and more. By using a status code library, developers can ensure consistent and reliable communication between servers and clients, leading to better error handling and improved user experience.

[Http Status Codes](https://www.npmjs.com/package/http-status-codes)

```sh
npm i http-status-codes@2.2.0

```

200 OK OK
201 CREATED Created

400 BAD_REQUEST Bad Request
401 UNAUTHORIZED Unauthorized

403 FORBIDDEN Forbidden
404 NOT_FOUND Not Found

500 INTERNAL_SERVER_ERROR Internal Server Error

- refactor 200 response in all controllers

achivementsController.js

```js
res.status(StatusCodes.OK).json({ achivements })
```

createAchivements

```js
res.status(StatusCodes.CREATED).json({ achivements })
```

#### 19. Custom Error Class

achievementsController

```js
export const getAchievement = async (req, res) => {
  ....
  if (!achivements) {
    throw new Error('no achivement with that id');
    // return res.status(404).json({ msg: `no achivement with id ${id}` });
  }
  ...
};

```

errors/customErrors.js

```js
import { StatusCodes } from 'http-status-codes'
export class NotFoundError extends Error {
  constructor(message) {
    super(message)
    this.name = 'NotFoundError'
    this.statusCode = StatusCodes.NOT_FOUND
  }
}
```

This code defines a custom error class NotFoundError that extends the built-in Error class in JavaScript. The NotFoundError class is designed to be used when a requested resource is not found, and it includes a status code of 404 to indicate this.

Here's a breakdown of the code:

class NotFoundError extends Error: This line defines a new class NotFoundError that extends the built-in Error class. This means that NotFoundError inherits all of the properties and methods of the Error class, and can also define its own properties and methods.

constructor(message): This is the constructor method for the NotFoundError class, which is called when a new instance of the class is created. The message parameter is the error message that will be displayed when the error is thrown.

super(message): This line calls the constructor of the Error class and passes the message parameter to it. This sets the error message for the NotFoundError instance.

this.name = "NotFoundError": This line sets the name property of the NotFoundError instance to "NotFoundError". This is a built-in property of the Error class that specifies the name of the error.

this.statusCode = 404: This line sets the statusCode property of the NotFoundError instance to 404. This is a custom property that is specific to the NotFoundError class and indicates the HTTP status code that should be returned when this error occurs.

By creating a custom error class like NotFoundError, you can provide more specific error messages and properties to help with debugging and error handling in your application.

#### 20. Custom Error

achievementController.js

```js
import { NotFoundError } from '../errors/customErrors.js'

 if (!achievement) {
    throw new NotFoundError('no achievement with that id')
```

middleware/errorHandlerMiddleware.js

```js
import { StatusCodes } from 'http-status-codes'
const errorHandlerMiddleware = (err, req, res, next) => {
  console.log(err)
  const statusCode = err.statusCode || StatusCodes.INTERNAL_SERVER_ERROR
  const msg = err.message || 'Something went wrong, try again later'

  res.status(statusCode).json({ msg })
}

export default errorHandlerMiddleware
```

server.js

```js
import errorHandlerMiddleware from './middleware/errorHandlerMiddleware.js'

app.use(errorHandlerMiddleware)
```

#### 21. Bad Request Error

400 BAD_REQUEST Bad Request
401 UNAUTHORIZED Unauthorized
403 FORBIDDEN Forbidden
404 NOT_FOUND Not Found

customErrors.js

```js
export class BadRequestError extends Error {
  constructor(message) {
    super(message)
    this.name = 'BadRequestError'
    this.statusCode = StatusCodes.BAD_REQUEST
  }
}
export class UnauthenticatedError extends Error {
  constructor(message) {
    super(message)
    this.name = 'UnauthenticatedError'
    this.statusCode = StatusCodes.UNAUTHORIZED
  }
}
export class UnauthorizedError extends Error {
  constructor(message) {
    super(message)
    this.name = 'UnauthorizedError'
    this.statusCode = StatusCodes.FORBIDDEN
  }
}
```
