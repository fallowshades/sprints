# Iteration 4 layer 1: Deploy

## \* path consideration

#### Local Build

- remove default values from inputs in Register and Login
- navigate to client and build front-end

```sh
cd hamoriaClient && npm run build
```

- copy/paste all the files/folders

  - from client/dist
  - to server(root)/public

- in server.js point to index.html
  https://developer.mozilla.org/en-US/docs/Glossary/SPA (bc SPA only 1 page)

```js
app.get('*', (req, res) => {
  res.sendFile(path.resolve(__dirname, './public', 'index.html'))
})
```

#### Deploy On Render

[Render](https://render.com/)

- sign up of for account
- create git repository

#### Build Front-End on Render

- add script
- change path

package.json

```js
 "scripts": {
    "setup-production-app": "npm i && cd client && npm i && npm run build",
  },
```

server.js

```js
app.use(express.static(path.resolve(__dirname, './hamoriaClient/dist')))
app.get('*', (req, res) => {
  res.sendFile(path.resolve(__dirname, './public/dist', 'index.html'))
})
```

#### Test Locally

- remove client/dist and client/node_modules
- remove node_modules and package-lock.json (optional)
- run "npm run setup-production-app", followed by "node server"

#### Test in Production

- change build command on render

```sh
npm run setup-production-app
```

- push up to github

## consider earlier

### in production

#### 12. Upload Image As Buffer

- remove public folder

```sh
npm i datauri@4.1.0
```

middleware/multerMiddleware.js

```js
import multer from 'multer'
import DataParser from 'datauri/parser.js'
import path from 'path'

const bufferStorage = multer.memoryStorage()
const upload = multer({ bufferStorage })

const parser = new DataParser()

export const formatImage = (file) => {
  const fileExtension = path.extname(file.originalname).toString()
  return parser.format(fileExtension, file.buffer).content
}

export default upload
```

controller/userController.js

```js
import { formatImage } from '../middleware/multerMiddleware.js'

export const updateUser = async (req, res) => {
  const newUser = { ...req.body }
  delete newUser.password
  if (req.file) {
    const file = formatImage(req.file)
    const response = await cloudinary.v2.uploader.upload(file)
    newUser.avatar = response.secure_url
    newUser.avatarPublicId = response.public_id
  }
  const updatedUser = await User.findByIdAndUpdate(req.user.userId, newUser)

  if (req.file && updatedUser.avatarPublicId) {
    await cloudinary.v2.uploader.destroy(updatedUser.avatarPublicId)
  }
  res.status(StatusCodes.OK).json({ msg: 'update user' })
}
```
