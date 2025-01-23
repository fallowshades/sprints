# Iteration 3 layer 2 Effective validate data

## 1 layer behave as 2 layer existence check

### Valid value in closed range

#### 1. Validation Layer

[Express Validator](https://express-validator.github.io/docs/)

```sh
npm i express-validator@7.0.1
```

#### 2. Test Route

server.js

```js
app.post('/api/v1/test', (req, res) => {
  const { name } = req.body
  res.json({ msg: `hello ${name}` })
})
```

#### 3. Express Validator

```js
import { body, validationResult } from 'express-validator'

app.post(
  '/api/v1/test',
  [body('name').notEmpty().withMessage('name is required')],
  (req, res) => {
    const errors = validationResult(req)
    if (!errors.isEmpty()) {
      const errorMessages = errors.array().map((error) => error.msg)
      return res.status(400).json({ errors: errorMessages })
    }
    next()
  },
  (req, res) => {
    const { name } = req.body
    res.json({ msg: `hello ${name}` })
  }
)
```

#### 4. Validation Middleware

middleware/validationMiddleware.js

```js
import { body, validationResult } from 'express-validator'
import { BadRequestError } from '../errors/customErrors'
const withValidationErrors = (validateValues) => {
  return [
    validateValues,
    (req, res, next) => {
      const errors = validationResult(req)
      if (!errors.isEmpty()) {
        const errorMessages = errors.array().map((error) => error.msg)
        throw new BadRequestError(errorMessages)
      }
      next()
    },
  ]
}

export const validateTest = withValidationErrors([
  body('name')
    .notEmpty()
    .withMessage('name is required')
    .isLength({ min: 3, max: 50 })
    .withMessage('name must be between 3 and 50 characters long')
    .trim(),
])
```

belive he console.log

#### 5. Remove Test Case From Server

### check achievement

#### 6. Setup Constants

utils/constants.js

```js
export const ACHIEVEMENT_STATUS = {
  INACTIVE: 'inactive',
  ACTIVATED: 'activated',
  COMPLETE: 'complete',
}

export const ACHIEVEMENT_TYPE = {
  PROGRESSIVE: 'progressive',
  EXPLORATION: 'exploration',
  TIME_BASED: 'time-based',
  SKILL_BASED: 'skill-based',
  SOCIAL: 'social',
  COLLECTION: 'collection',
  STORYLINE: 'storyline',
  EVENT: 'event',
  HIDDEN: 'hidden',
  LIFETIME: 'lifetime',
}
```

models/AchievementModel.js

```js
import mongoose from 'mongoose'
import { ACHIEVEMENT_STATUS, ACHIEVEMENT_TYPE } from '../utils/constants.js'
const AchievementSchema = new mongoose.Schema(
  {
    description: String,
    //uh mb later
    status: {
      type: String,
      enum: [ACHIEVEMENT_STATUS],
      default: ACHIEVEMENT_STATUS.INACTIVE,
    },
    points: {
      type: Number,
      default: 0,
    },

    type: {
      type: String,
      enum: [ACHIEVEMENT_STATUS],
      default: ACHIEVEMENT_TYPE.EXPLORATION,
    },
    dateOfCompletion: {
      type: Date,
      default: null,
    },
  },
  { timestamps: true }
)
```

#### 7. Validate Create Achievement

validationMiddleware.js

```js
import { ACHIEVEMENT_STATUS, ACHIEVEMENT_TYPE } from '../utils/constants.js'

export const validateAchievementInput = withValidationErrors([
  body('description').notEmpty().withMessage('description is required'),
  body('completion_status')
    .isIn(Object.values(ACHIEVEMENT_STATUS))
    .withMessage('invalid status value'),
  body('jobType')
    .isIn(Object.values(ACHIEVEMENT_TYPE))
    .withMessage('invalid job type'),
])
```

```js
import { validateAchievementInput } from '../middleware/validationMiddleware.js'

router
  .route('/')
  .get(getAllAchievements)
  .post(validateAchievementInput, createAchievement)

router
  .route('/:id')
  .get(getAchievement)
  .delete(deleteAchievement)
  .patch(validateAchievementInput, updateAchievement)
```

- create Achievement request

```json
{
  "company": "coding addict",
  "position": "backend-end",
  "jobStatus": "pending",
  "jobType": "full-time",
  "jobLocation": "florida"
}
```

#### 8. Validate ID Parameter

validationMiddleware.js

```js
import mongoose from 'mongoose'
import { param } from 'express-validator'

export const validateIdParam = withValidationErrors([
  param('id')
    .custom((value) => mongoose.Types.ObjectId.isValid(value))
    .withMessage('invalid MongoDB id'),
])
```

```js
export const validateIdParam = withValidationErrors([
  param('id').custom(async (value) => {
    const isValidId = mongoose.Types.ObjectId.isValid(value)
    if (!isValidId) throw new BadRequestError('invalid MongoDB id')
    const achievement = await Achievement.findById(value)
    if (!achievement)
      throw new NotFoundError(`no achievement with id : ${value}`)
  }),
])
```

```js
import { body, param, validationResult } from 'express-validator'
import { BadRequestError, NotFoundError } from '../errors/customErrors.js'
import { ACHIEVEMENT_STATUS, ACHIEVEMENT_TYPE } from '../utils/constants.js'
import mongoose from 'mongoose'
import Achievement from '../models/achievementModel.js'

const withValidationErrors = (validateValues) => {
  return [
    validateValues,
    (req, res, next) => {
      const errors = validationResult(req)
      if (!errors.isEmpty()) {
        const errorMessages = errors.array().map((error) => error.msg)
        if (errorMessages[0].startsWith('no achievement')) {
          throw new NotFoundError(errorMessages)
        }
        throw new BadRequestError(errorMessages)
      }
      next()
    },
  ]
}
```

achievementRouter.js

```js
import {
  validateAchievementInput,
  validateIdParam,
} from '../middleware/validationMiddleware.js'

router
  .route('/:id')
  .get(validateIdParam, getAchievement)
  .delete(validateIdParam, deleteAchievement)
```

- remove NotFoundError from getJob, updateJob, deleteJob controllers

#### 9. Clean DB

## user 1 element existence check may throw

### identifiable data on DB (common, admin)

#### .10 User Model

models/UserModel.js

```js
import mongoose from 'mongoose'

const UserSchema = new mongoose.Schema({
  name: String,
  email: String,
  password: String,
  lastName: {
    type: String,
    default: 'lastName',
  },
  location: {
    type: String,
    default: 'my city',
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user',
  },
})

export default mongoose.model('User', UserSchema)
```

#### 11. User Controller and Router

controllers/authController.js

```js
export const register = async (req, res) => {
  res.send('register')
}
export const login = async (req, res) => {
  res.send('login')
}
```

routers/authRouter.js

```js
import { Router } from 'express'
import { register, login } from '../controllers/authController.js'
const router = Router()

router.post('/register', register)
router.post('/login', login)

export default router
```

server.js

```js
import authRouter from './routes/authRouter.js'

app.use('/api/v1/auth', authRouter)
```

#### 12. Create User - Initial Setup

authController.js

```js
import { StatusCodes } from 'http-status-codes'
import User from '../models/UserModel.js'

export const register = async (req, res) => {
  const user = await User.create(req.body)
  res.status(StatusCodes.CREATED).json({ user })
}
```

- register user request

```json
{
  "name": "name",
  "email": "name@gmail.com",
  "password": "secret123",
  "lastName": "lastName",
  "location": "my city"
}
```

#### 13. Validate User

validationMiddleware.js

```js
import User from '../models/UserModel.js'

export const validateRegisterInput = withValidationErrors([
  body('name').notEmpty().withMessage('name is required'),
  body('email')
    .notEmpty()
    .withMessage('email is required')
    .isEmail()
    .withMessage('invalid email format')
    .custom(async (email) => {
      const user = await User.findOne({ email })
      if (user) {
        throw new BadRequestError('email already exists')
      }
    }),
  body('password')
    .notEmpty()
    .withMessage('password is required')
    .isLength({ min: 8 })
    .withMessage('password must be at least 8 characters long'),
  body('location').notEmpty().withMessage('location is required'),
  body('lastName').notEmpty().withMessage('last name is required'),
])
```

authRouter.js

```js
import { validateRegisterInput } from '../middleware/validationMiddleware.js'

router.post('/register', validateRegisterInput, register)
```

#### 14. Admin Role

authController.js

```js
// first registered user is an admin
const isFirstAccount = (await User.countDocuments()) === 0
req.body.role = isFirstAccount ? 'admin' : 'user'

const user = await User.create(req.body)
```
