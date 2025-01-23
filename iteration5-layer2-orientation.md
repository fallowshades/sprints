# Orientation v0.5.1

##

### Orientation schema

utils/constants.js

```js
export const ORIENTATION = {
  INWARD: 'inward',
  DOWNWARD: 'downward',
  FORWARD: 'forward',
  REAGAINST: 'reAgainst',
  UPPWARD: 'uppward',
  NULL: null,
  EMPTY: '',
}
```

orientationModel.js

```js
import mongoose from 'mongoose'
import { ORIENTATION } from '../utils/constants.js'

const OrientationSchema = new mongoose.Schema({
  position: { type: String, required: true },
  hand: { type: String, enum: [ORIENTATION], required: true },
})

export default mongoose.model('orientation', OrientationSchema)
```

### Order structure

#### CRUD for who

orientationController.js

```js
import { StatusCodes } from 'http-status-codes'
import orientationModel from '../models/orientationModel.js'
import 'express-async-errors'

export const createOrientation = async (req, res) => {
  res.send('create orientation')
}

export const getAllOrientations = async (req, res) => {
  res.send('get all orientation')
}

export const getSingleOrientation = async (req, res) => {
  res.send('get single orientation')
}

export const updateOrientation = async (req, res) => {
  res.send('update orientation')
}

export const deleteOrientation = async (req, res) => {
  res.send('delete orientation')
}
```

#### privilaged order routes

orientationRouter.js

```js
import { Router } from 'express'

import {
  getAllOrientations,
  getSingleOrientation,
  createOrientation,
  updateOrientation,
  deleteOrientation,
} from '../controllers/orientationController.js'

const router = Router()

router.route('/').post(createOrientation).get(getAllOrientations)

router
  .route('/:id')
  .get(getSingleOrientation)
  .patch(updateOrientation)
  .delete(deleteOrientation)
export default router
```

App.js

```js
import orientationRouter from './routes/orientationRouter.js'

app.use('/api/v1/orientations', orientationRouter)
```
