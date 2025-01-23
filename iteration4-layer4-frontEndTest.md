### check sign

#### 6. Setup Constants

utils/constants.js

```js
export const SIGN_CATEGORY = {
  WORK: 'work',
  EAT: 'eat',
  SLEEP: 'sleep',
}

export const SIGN_COMPANY = {
  STANDARD: 'ikea',
  FRIEND: 'liddy',
  COMPETITOR: 'marcos',
}
```

!!! wanring med enum: {}

models/SignModel.js

```js
import mongoose from 'mongoose'
import { SIGN_COMPANY, SIGN_CATEGORY } from '../utils/constants.js'
const SignSchema = new mongoose.Schema(

   category: {
      type: String,
      required: [true, 'Please provide product category'],
      enum: [SIGN_CATEGORY],
    },
    company: {
      type: String,
      required: [true, 'Please provide company'],
      enum: {
        values: [SIGN_COMPANY],
        message: '{VALUE} is not supported',
      },

  },
  { timestamps: true }
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
```

#### 7. Validate Create Sign

validationMiddleware.js

```js
import { SIGN_COMPANY, SIGN_CATEGORY } from '../utils/constants.js'

export const validateSignInput = withValidationErrors([
  body('title').notEmpty().withMessage('title is required'),
  body('description').notEmpty().withMessage('description is required'),
  body('description')
    .isIn(Object.values(SIGN_CATEGORY))
    .withMessage('invalid category value'),
  body('company')
    .isIn(Object.values(SIGN_COMPANY))
    .withMessage('invalid company type'),
])
```

signRouter

```js
import { validateSignInput } from '../middleware/validationMiddleware.js'

router.route('/').get(getAllSigns).post(validateSignInput, createSign)

router
  .route('/:id')
  .get(getSign)
  .delete(deleteSign)
  .patch(validateSignInput, updateSign)
```

- create Sign request

```json
{
  "title": "tst",
  "price": 1,
  "description": "des",
  "category": "work",
  "company": "ikea",
  "colors": ["#ff000", "#00ff00", "#0000ff"]
}
```

#### 8. Validate ID Parameter

validationMiddleware.js

-- A request using the driver checking id

```js
import mongoose from 'mongoose'
import { param } from 'express-validator'

export const validateIdParam = withValidationErrors([
  param('id')
    .custom((value) => mongoose.Types.ObjectId.isValid(value))
    .withMessage('invalid MongoDB id'),
])
```

---the request is async that must throw before invalid request to db
--throw before or after id
--the async is in argument since inside withValidationErrors

```js
export const validateIdParam = withValidationErrors([
  param('id').custom(async (value) => {
    const isValidId = mongoose.Types.ObjectId.isValid(value)
    if (!isValidId) throw new BadRequestError('invalid MongoDB id')
    const sign = await Sign.findById(value)
    if (!sign) throw new NotFoundError(`no sign with id : ${value}`)
  }),
])
```

--go back to base function to manage errors that bubble up

```js
import { body, param, validationResult } from 'express-validator'
import { BadRequestError, NotFoundError } from '../errors/customErrors.js'
import { SIGN_COMPANY, SIGN_CATEGORY } from '../utils/constants.js'
import mongoose from 'mongoose'
import Sign from '../models/signModel.js'

const withValidationErrors = (validateValues) => {
  return [
    validateValues,
    (req, res, next) => {
      const errors = validationResult(req)
      if (!errors.isEmpty()) {
        const errorMessages = errors.array().map((error) => error.msg)
        if (errorMessages[0].startsWith('no sign')) {
          throw new NotFoundError(errorMessages)
        }
        throw new BadRequestError(errorMessages)
      }
      next()
    },
  ]
}
```

signRouter.js

```js
import {
  validateSignInput,
  validateIdParam,
} from '../middleware/validationMiddleware.js'

router
  .route('/:id')
  .get(validateIdParam, getSign)
  .delete(validateIdParam, deleteSign)
```

- remove NotFoundError from getJob, updateJob, deleteJob controllers

#### 9. Clean DB
