# prefix v0.5.1

##

### prefix schema

utils/constants.js

```js
export const HAND_VARIANTS = {
  TUMHAND: 'Tumhand',
  HOLD: 'hold',
  SPRET: 'spret',
  MEASURE: 'measure',
  KROK2: 'krok2',
  NYP: 'nyp',
  KLO: 'klo',
  R: 'r',
  KROK: 'krok',
  G: 'g',
  I: 'i',
  VINKEL: 'vinkel',
  N: 'n',
  V: 'v',
  E: 'e',
  D: 'd',
  STRAIGHTMEASURE: 'straightMeasure',
  J: 'j',
  TUPP: 'tupp',
  L: 'l',
  P: 'p',
  HAND4: 'hand4',
  T: 't',
  H: 'h',
  A: 'a',
  O: 'o',
  S: 's',
  TUMVINKEL: 'tumvinkel',
  B: 'b',
  FLYG: 'flyg',
}
```

prefixModel.js

```js
import mongoose from 'mongoose'
import { HAND_VARIANTS } from '../utils/constants.js'

const PrefixSchema = new mongoose.Schema({
  position: { type: String, required: true },
  hand: { type: String, enum: [HAND_VARIANTS], required: true },
})

export default mongoose.model('prefix', PrefixSchema)
```

### Order structure

#### CRUD for who

prefixController.js

```js
import { StatusCodes } from 'http-status-codes'
import PrefixModel from '../models/prefixModel.js'
import 'express-async-errors'

export const createPrefix = async (req, res) => {
  res.send('create prefix')
}

export const getAllPrefix = async (req, res) => {
  res.send('get all prefixes')
}

export const getSinglePrefix = async (req, res) => {
  res.send('get single prefix')
}

export const updatePrefix = async (req, res) => {
  res.send('update prefix')
}

export const deletePrefix = async (req, res) => {
  res.send('delete prefix')
}
```

#### privilaged order routes

prefixRouter.js

```js
import { Router } from 'express'

import {
  getAllPrefixes,
  getSinglePrefix,
  createPrefix,
  updatePrefix,
  deletePrefix,
} from '../controllers/prefixController.js'

const router = Router()

router.route('/').post(createPrefix).get(getAllPrefixes)

router
  .route('/:id')
  .get(getSinglePrefix)
  .patch(updatePrefix)
  .delete(deletePrefix)
export default router
```

App.js

```js
import prefixRouter from './routes/prefixRouter.js'

app.use('/api/v1/prefixes', prefixRouter)
```
