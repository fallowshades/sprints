# word v0.5.2

##

### word schema

wordModel.js

```js
import mongoose from 'mongoose'

const WordSchema = new mongoose.Schema({
  word: {
    type: String,
    required: true,
  },
  subgroup: {
    type: String,
    required: true,
  },
  subsection: {
    type: String,
    required: true,
  },
})

export default mongoose.model('word', WordSchema)
```

### Order structure

#### CRUD for who

wordController.js

```js
import { StatusCodes } from 'http-status-codes'
import wordModel from '../models/wordModel.js'
import 'express-async-errors'

export const createWord = async (req, res) => {
  res.send('create word')
}

export const getAllWords = async (req, res) => {
  res.send('get all words')
}

export const getSingleWord = async (req, res) => {
  res.send('get single word')
}

export const updateWord = async (req, res) => {
  res.send('update word')
}

export const deleteWord = async (req, res) => {
  res.send('delete word')
}
```

#### privilaged order routes

wordRouter.js

```js
import { Router } from 'express'

import {
  getAllWords,
  getSingleWord,
  createWord,
  updateWord,
  deleteWord,
} from '../controllers/wordController.js'

const router = Router()

router.route('/').post(createWord).get(getAllWords)

router.route('/:id').get(getSingleWord).patch(updateWord).delete(deleteWord)
export default router
```

App.js

```js
import wordRouter from './routes/wordRouter.js'

app.use('/api/v1/words', wordRouter)
```
