# Iteration 3 layer 5

[batch-findIndex-byName]

- path and array indexing/name search

[personalize-unlink]

- unlink valid obj

## both front/back is in control

### (1)Concern deevelopment || production

#### 1. Avatar Image

- get two images from pexels

[pexels](https://www.pexels.com/search/person/)

#### 2. Setup Public Folder

server.js

```js
import { dirname } from 'path'
import { fileURLToPath } from 'url'
import path from 'path'

const __dirname = dirname(fileURLToPath(import.meta.url))

app.use(express.static(path.resolve(__dirname, './public')))
```

- http://localhost:5100/imageName

### (2)Concerns over local space / \_Directory

#### 3. Profile Page - Initial Setup

- remove jobs,users from DB
- add avatar property in the user model

models/UserModel.js

```js
const UserSchema = new mongoose.Schema({
  avatar: String,
  avatarPublicId: String,
})
```

#### 4. Profile Page - Structure

pages/Profile.jsx

```js
import { FormRow } from '../components'
import Wrapper from '../assets/wrappers/DashboardFormPage'
import { useOutletContext } from 'react-router-dom'
import { useNavigation, Form } from 'react-router-dom'
import customFetch from '../utils/customFetch'
import { toast } from 'react-toastify'

const Profile = () => {
  const { user } = useOutletContext()
  const { name, lastName, email, location } = user
  const navigation = useNavigation()
  const isSubmitting = navigation.state === 'submitting'
  return (
    <Wrapper>
      <Form method='post' className='form' encType='multipart/form-data'>
        <h4 className='form-title'>profile</h4>

        <div className='form-center'>
          <div className='form-row'>
            <label htmlFor='image' className='form-label'>
              Select an image file (max 0.5 MB):
            </label>
            <input
              type='file'
              id='avatar'
              name='avatar'
              className='form-input'
              accept='image/*'
            />
          </div>
          <FormRow type='text' name='name' defaultValue={name} />
          <FormRow
            type='text'
            labelText='last name'
            name='lastName'
            defaultValue={lastName}
          />
          <FormRow type='email' name='email' defaultValue={email} />
          <FormRow type='text' name='location' defaultValue={location} />
          <button
            className='btn btn-block form-btn'
            type='submit'
            disabled={isSubmitting}
          >
            {isSubmitting ? 'submitting...' : 'save changes'}
          </button>
        </div>
      </Form>
    </Wrapper>
  )
}

export default Profile
```

#### 5. Profile Page - Action

- import/export action (App.jsx)

```js
export const action = async ({ request }) => {
  const formData = await request.formData()

  const file = formData.get('avatar')
  if (file && file.size > 500000) {
    toast.error('Image size too large')
    return null
  }

  try {
    await customFetch.patch('/users/update-user', formData)
    toast.success('Profile updated successfully')
  } catch (error) {
    toast.error(error?.response?.data?.msg)
  }
  return null
}
```

## Path to presentational dst

### (1)How ctrl data w string to Cloudinary

#### 6. Update User - Server

```sh
npm i multer@1.4.5
```

Multer is a popular middleware package for handling multipart/form-data in Node.js web applications. It is commonly used for handling file uploads. Multer simplifies the process of accepting and storing files submitted through HTTP requests by providing an easy-to-use API. It integrates seamlessly with Express.js and allows developers to define upload destinations, file size limits, and other configurations.

- create middleware/multerMiddleware.js
- setup multer

```js
import multer from 'multer'

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    // set the directory where uploaded files will be stored
    cb(null, 'public/uploads')
  },
  filename: (req, file, cb) => {
    const fileName = file.originalname
    // set the name of the uploaded file
    cb(null, fileName)
  },
})
const upload = multer({ storage })

export default upload
```

routes/userRouter.js

```js
import upload from '../middleware/multerMiddleware.js'

router.patch(
  '/update-user',
  upload.single('avatar'),
  validateUpdateUserInput,
  updateUser
)
```

First, the multer package is imported.

Then, a storage object is created using multer.diskStorage(). This object specifies the configuration for storing uploaded files. In this case, the destination function determines the directory where the uploaded files will be saved, which is set to 'public/uploads'. The filename function defines the name of the uploaded file, which is set to the original filename.

Next, a multer middleware is created by passing the storage object as a configuration option. This multer middleware will be used to handle file uploads in the application.

In this case, upload is an instance of the Multer middleware that was created earlier. The .single() method is called on this instance to indicate that only one file will be uploaded. The argument 'avatar' specifies the name of the field in the HTTP request that corresponds to the uploaded file.

When this middleware is used in an HTTP route handler, it will process the incoming request and extract the file attached to the 'avatar' field. Multer will then save the file according to the specified storage configuration, which includes the destination directory and filename logic defined earlier. The uploaded file can be accessed in the route handler using req.file.

#### 7. Cloudinary - Create Account/Get API Keys

[Cloudinary](https://cloudinary.com/)

Cloudinary is a cloud-based media management platform that helps businesses store, optimize, and deliver images and videos across the web. It provides developers with an easy way to upload, manipulate, and serve media assets, enabling faster and more efficient delivery of visual content on websites and applications. Cloudinary also offers features like automatic resizing, format conversion, and responsive delivery to ensure optimal user experiences across different devices and network conditions.

.env

```sh
CLOUD_NAME=
CLOUD_API_KEY=
CLOUD_API_SECRET=
```

#### 8. Cloudinary - Setup Instance

```sh
npm i cloudinary@1.37.3
```

server

```js
import cloudinary from 'cloudinary'

cloudinary.config({
  cloud_name: process.env.CLOUD_NAME,
  api_key: process.env.CLOUD_API_KEY,
  api_secret: process.env.CLOUD_API_SECRET,
})
```

#### 9. Update User Controller

controllers/userController.js

```js
import cloudinary from 'cloudinary'
import { promises as fs } from 'fs'

export const updateUser = async (req, res) => {
  const newUser = { ...req.body }
  delete newUser.password
  if (req.file) {
    const response = await cloudinary.v2.uploader.upload(req.file.path)
    await fs.unlink(req.file.path)
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

### (2)Accept attribute

#### 10. Logout Container

```js
{
  user.avatar ? (
    <img src={user.avatar} alt='avatar' className='img' />
  ) : (
    <FaUserCircle />
  )
}
```

#### 11. Submit Btn Component

- create component SubmitBtn (export/import)
- add all classes, including'.form-btn'
- setup in Register,Login, AddJob, EditJob, Profile
- make sure to add formBtn prop

```js
import { useNavigation } from 'react-router-dom'
const SubmitBtn = ({ formBtn }) => {
  const navigation = useNavigation()
  const isSubmitting = navigation.state === 'submitting'
  return (
    <button
      type='submit'
      className={`btn btn-block ${formBtn && 'form-btn'}`}
      disabled={isSubmitting}
    >
      {isSubmitting ? 'submitting...' : 'submit'}
    </button>
  )
}
export default SubmitBtn
```

## Src transmition (um other approach)

(3)Challange size presentational (content type ctrl session)'

### (1)Rdy dst persist -> bc hosted limitations

### (2)Method to transmit (callback)

### (3)Connect onto additional node ( redundancy?)

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
