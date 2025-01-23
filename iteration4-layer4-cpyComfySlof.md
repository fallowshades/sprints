#### 7. Controller and Router

setup controllers and router

controllers/signController.js

signs

```js
import { nanoid } from 'nanoid'

let signs = [
  { id: nanoid(), description: 'start', track: 'front-end developer' },
  { id: nanoid(), description: 'end', track: 'back-end developer' },
]

export const getAllSigns = async (req, res) => {
  res.status(200).json({ signs })
}

export const createSign = async (req, res) => {
  const { description, track } = req.body

  if (!description || !track) {
    return res.status(400).json({ msg: 'please provide description and track' })
  }
  const id = nanoid(10)
  const sign = { id, description, track }
  signs.push(sign)
  res.status(200).json({ sign })
}

export const getSign = async (req, res) => {
  const { id } = req.params
  const sign = signs.find((sign) => sign.id === id)
  if (!sign) {
    // throw new Error('no sign with that id');
    return res.status(404).json({ msg: `no sign with id ${id}` })
  }
  res.status(200).json({ sign })
}

export const updateSign = async (req, res) => {
  const { description, track } = req.body
  if (!description || !track) {
    return res.status(400).json({ msg: 'please provide description and track' })
  }
  const { id } = req.params
  const sign = signs.find((sign) => sign.id === id)
  if (!sign) {
    return res.status(404).json({ msg: `no sign with id ${id}` })
  }

  sign.description = description
  sign.track = track
  res.status(200).json({ msg: 'sign modified', sign })
}

export const deleteSign = async (req, res) => {
  const { id } = req.params
  const sign = signs.find((sign) => sign.id === id)
  if (!sign) {
    return res.status(404).json({ msg: `no job with id ${id}` })
  }
  const newSigns = signs.filter((sign) => sign.id !== id)
  signs = newASigns

  res.status(200).json({ msg: 'sign deleted' })
}
```

routes/SignRouter.js

```js
import { Router } from 'express'
const router = Router()

import {
  getAllSigns,
  getSign,
  createSign,
  updateSign,
  deleteSign,
} from '../controllers/signController.js'

// router.get('/', getAllJobs);
// router.post('/', createJob);

router.route('/').get(getAllSign).post(createSign)
router.route('/:id').get(getSign).patch(updateSign).delete(deleteSign)

export default router
```

server.js

```js
import signRouter from './routes/signRouter.js'
app.use('/api/v1/signs', signRouter)
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

#### 10. Sign Model

models/signModel.js

enum - data type represents a field with a predefined set of values

```js
import mongoose from 'mongoose'

const SignSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      trim: true,
      required: [true, 'Please provide product name'],
      maxlength: [100, 'Name can not be more than 100 characters'],
    },
    price: {
      type: Number,
      required: [true, 'Please provide product price'],
      default: 0,
    },
    description: {
      type: String,
      required: [true, 'Please provide product description'],
      maxlength: [1000, 'Description can not be more than 1000 characters'],
    },
    image: {
      type: String,
      default: '/uploads/example.jpeg',
    },
    category: {
      type: String,
      required: [true, 'Please provide product category'],
      enum: ['office', 'kitchen', 'bedroom'],
    },
    company: {
      type: String,
      required: [true, 'Please provide company'],
      enum: {
        values: ['ikea', 'liddy', 'marcos'],
        message: '{VALUE} is not supported',
      },
    },
    colors: {
      type: [String],
      default: ['#222'],
      required: true,
    },
    featured: {
      type: Boolean,
      default: false,
    },
    freeShipping: {
      type: Boolean,
      default: false,
    },
  },
  { timestamps: true }
)

export default mongoose.model('sign', SignSchema)
```

#### 11. Create sign with routes

signController.js

```js
import Sign from '../models/signModel.js'

export const createSign = async (req, res) => {
  const { title, price, description, category, company, colors } = req.body
  const sign = await Sign.create({
    title,
    price,
    description,
    category,
    company,
    colors,
  })
  res.status(201).json({ sign })
}
```

signRouter.js

```js
import { Router } from 'express'
const router = Router()

import { createSign } from '../controllers/signController.js'

router.route('/').post(createSign)

export default router
```

server.js

```js
import SignRouter from './routes/signRouter.js'
app.use('/api/v1/signs', signRouter)
```

#### 12. Try / Catch (meh i used express-async-error before)

jobController.js

```js
export const createSign = async (req, res) => {
  const { description } = req.body
  try {
    const sign = await Sign.create('something')
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

SignController.js

```js
export const createSign = async (req, res) => {
  const { description } = req.body

  const sign = await Sign.create({ description })
  res.status(201).json({ sign })
}
```

#### 14. Get All Signs

signsController.js

```js
export const getAllSigns = async (req, res) => {
  const sign = await Sign.find({})
  res.status(200).json({ sign })
}
```

signRouter.js

```js
router.route('/').get(getAllSigns).post(createSign)
```

#### 15. Get Single Sign

```js
export const getSign = async (req, res) => {
  const { id } = req.params
  const sign = await Sign.findById(id)
  if (!sign) {
    return res.status(404).json({ msg: `no sign with id ${id}` })
  }
  res.status(200).json({ sign })
}
```

signRouter.js

```js
router.route('/id').get(getSign
```

#### 16. Delete Sign

SignController.js

```js
export const deleteSign = async (req, res) => {
  const { id } = req.params
  const removedSign = await Sign.findByIdAndDelete(id)

  if (!removedSign) {
    return res.status(404).json({ msg: `no job with id ${id}` })
  }
  res.status(200).json({ sign: removedSign })
}
```

signRouter.js

```js
router.route('/:id').get(getSign).delete(deleteSign)
```

#### 17. Update Sign

```js
export const updateSign = async (req, res) => {
  const { id } = req.params

  const updatedSign = await Sign.findByIdAndUpdate(id, req.body, {
    new: true,
  })

  if (!updatedSign) {
    return res.status(404).json({ msg: `no sign with id ${id}` })
  }

  res.status(200).json({ description: updatedSign })
}
```

signRouter.js

```js
router.route('/:id').get(getSign).delete(deleteSign).patch(updateSign)
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

SignsController.js

```js
res.status(StatusCodes.OK).json({ signs })
```

createSigns

```js
res.status(StatusCodes.CREATED).json({ signs })
```

#### 19. Custom Error Class

signsController

```js
export const getSign = async (req, res) => {
  ....
  if (!signs) {
    throw new Error('no sign with that id');
    // return res.status(404).json({ msg: `no sign with id ${id}` });
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

signController.js

```js
import { NotFoundError } from '../errors/customErrors.js'

 if (!sign) {
    throw new NotFoundError('no sign with that id')
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
