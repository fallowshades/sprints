# Orientation v0.5.3

##

### Reference schema

utils/constats.js

```js
export const TOUCH_TYPE = {
  LINGER: 'bibehållen',
  MEDIAL: 'medial',
  ANGLE: 'vinkel',
  CROSS: 'kors',
  CROSSED: 'korsar',
  DRAG: 'drar',
  BRAIDING: 'flätande',
  CLOSED: 'sluten',
  HOOKED: 'hakas',
  HOOK: 'hakade',
  PINCH: 'nyper',
  NULL: null,
  EMPTY: '',
}

export const FACE_EXPRESSION = {
  PUH: 'puh',
  P: 'p',
  M: 'm',
  CHEEKS: '()',
  Y: 'y',
  F: 'f',
  A: 'a',
  Ä: 'ä',
  L: 'l',
  U: 'u',
  O: 'o',
  T: 't',
  S: 's',
  TH: 'th',
  PM: 'pm',
  PF: 'pf',
  PAFF: 'paff',
  PI: 'pi',
  PS: 'ps',
  PY: 'py',
  OPS: 'ops',
  OP: 'op',
  PU: 'pu',
  PÄ: 'pä',
  AP: 'ap',
  AS: 'as',
  ASAS: 'asas',
  ACHEAKS: 'a()',
  FA: 'fs',
  FSFS: 'fsfs',
  FT: 'ft',
  LS: 'ls',
  NULL: null,
  EMPTY: '',
}
```

referenceModel.js

```js
import mongoose from 'mongoose'
import { TOUCH_TYPE, FACE_EXPRESSION } from '../utils/constants.js'

const ReferenceSchema = new mongoose.Schema({
  active_hand: {
    type: String,
    required: true,
  },
  active_hand2: {
    type: String,
    required: true,
  },
  passive_hand2: {
    type: String,
    enum: [TOUCH_TYPE],
    default: TOUCH_TYPE.NULL,
  },
  singlehandform: {
    type: String,
    enum: [FACE_EXPRESSION],
    default: FACE_EXPRESSION.NULL,
  },
  transform: {
    type: String,
  },
})

export default mongoose.model('reference', ReferenceSchema)
```

### Order structure

#### CRUD for who

referenceController.js

```js
import { StatusCodes } from 'http-status-codes'
import referenceModel from '../models/referenceModel.js'
import 'express-async-errors'

export const createReference = async (req, res) => {
  res.send('create reference')
}

export const getAllReferences = async (req, res) => {
  res.send('get all reference')
}

export const getSingleReference = async (req, res) => {
  res.send('get single reference')
}

export const updateReference = async (req, res) => {
  res.send('update reference')
}

export const deleteReference = async (req, res) => {
  res.send('delete reference')
}
```

#### privilaged order routes

referenceRouter.js

```js
import { Router } from 'express'

import {
  getAllReferences,
  getSingleReference,
  createReference,
  updateReference,
  deleteReference,
} from '../controllers/referenceController.js'

const router = Router()

router.route('/').post(createReference).get(getAllReferences)

router
  .route('/:id')
  .get(getSingleReference)
  .patch(updateReference)
  .delete(deleteReference)
export default router
```

App.js

```js
import referenceRouter from './routes/referenceRouter.js'

app.use('/api/v1/references', referenceRouter)
```
