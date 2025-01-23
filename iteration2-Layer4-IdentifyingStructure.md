# Identiying workload

[encrypt-register->login...]

before pass body on create --> comparison enable (req,res)

[wrapped-client-side-session]
pass id and role into session token --> enable personalisation

[session-personalized]
parse and verify to access passed extended values

## secure data identification

### secure identity on DB

#### 1. Hash Passwords

[bcryptjs](https://www.npmjs.com/package/bcryptjs)

```sh
npm i bcryptjs@2.4.3

```

authController.js

```js
import bcrypt from 'bcryptjs'

const register = async (req, res) => {
  // a random value that is added to the password before hashing
  const salt = await bcrypt.genSalt(10)
  const hashedPassword = await bcrypt.hash(req.body.password, salt)
  req.body.password = hashedPassword

  const user = await User.create(req.body)
}
```

const salt = await bcrypt.genSalt(10);
This line generates a random "salt" value that will be used to hash the password. A salt is a random value that is added to the password before hashing, which helps to make the resulting hash more resistant to attacks like dictionary attacks and rainbow table attacks. The genSalt() function in bcrypt generates a random salt value using a specified "cost" value. The cost value determines how much CPU time is needed to calculate the hash, and higher cost values result in stronger hashes that are more resistant to attacks.

In this example, a cost value of 10 is used to generate the salt. This is a good default value that provides a good balance between security and performance. However, you may need to adjust the cost value based on the specific needs of your application.

const hashedPassword = await bcrypt.hash(password, salt);
This line uses the generated salt value to hash the password. The hash() function in bcrypt takes two arguments: the password to be hashed, and the salt value to use for the hash. It then calculates the hash value using a one-way hash function and the specified salt value.

The resulting hash value is a string that represents the hashed password. This string can then be stored in a database or other storage mechanism to be compared against the user's password when they log in.

By using a salt value and a one-way hash function, bcrypt helps to ensure that user passwords are stored securely and are resistant to attacks like password cracking and brute-force attacks.

##### 2. BCRYPT VS BCRYPTJS

bcrypt and bcryptjs are both popular libraries for hashing passwords in Node.js applications. However, bcryptjs is considered to be a better choice for a few reasons:

Cross-platform compatibility: bcrypt is a native Node.js module that uses C++ bindings, which can make it difficult to install and use on some platforms. bcryptjs, on the other hand, is a pure JavaScript implementation that works on any platform.

Security: While both bcrypt and bcryptjs use the same underlying algorithm for hashing passwords, bcryptjs is designed to be more resistant to certain types of attacks, such as side-channel attacks.

Ease of use: bcryptjs has a simpler and more intuitive API than bcrypt, which can make it easier to use and integrate into your application.

Overall, while bcrypt and bcryptjs are both good choices for hashing passwords in Node.js applications, bcryptjs is considered to be a better choice for its cross-platform compatibility, improved security, ease of use, and ongoing maintenance.

#### 3. Setup Password Utils

utils/passwordUtils.js

```js
import bcrypt from 'bcryptjs'

export async function hashPassword(password) {
  const salt = await bcrypt.genSalt(10)
  const hashedPassword = await bcrypt.hash(password, salt)
  return hashedPassword
}
```

authController.js

```js
import { hashPassword } from '../utils/passwordUtils.js'

const register = async (req, res) => {
  const hashedPassword = await hashPassword(req.body.password)
  req.body.password = hashedPassword

  const user = await User.create(req.body)
  res.status(StatusCodes.CREATED).json({ msg: 'user created' })
}
```

#### 4. Login User

- login user request

```json
{
  "email": "john@gmail.com",
  "password": "secret123"
}
```

validationMiddleware.js

```js
export const validateLoginInput = withValidationErrors([
  body('email')
    .notEmpty()
    .withMessage('email is required')
    .isEmail()
    .withMessage('invalid email format'),
  body('password').notEmpty().withMessage('password is required'),
])
```

authRouter.js

```js
import { validateLoginInput } from '../middleware/validationMiddleware.js'

router.post('/login', validateLoginInput, login)
```

#### 5. Unauthenticated Error

authController.js

```js
import { UnauthenticatedError } from '../errors/customErrors.js'

export const login = async (req, res) => {
  // check if user exists
  // check if password is correct

  const user = await User.findOne({ email: req.body.email })
  if (!user) throw new UnauthenticatedError('invalid credentials')

  res.send('login route')
}
```

#### 6. Compare Password

passwordUtils.js

```js
export async function comparePassword(password, hashedPassword) {
  const isMatch = await bcrypt.compare(password, hashedPassword)
  return isMatch
}
```

authController.js

```js
import { hashPassword, comparePassword } from '../utils/passwordUtils.js'

const login = async (req, res) => {
  // check if user exists
  // check if password is correct

  const user = await User.findOne({ email: req.body.email })

  if (!user) throw new UnauthenticatedError('invalid credentials')

  const isPasswordCorrect = await comparePassword(
    req.body.password,
    user.password
  )

  if (!isPasswordCorrect) throw new UnauthenticatedError('invalid credentials')
  res.send('login route')
}
```

Refactor

```js
const isValidUser =
  user && (await comparePassword(req.body.password, user.password))
if (!isValidUser) throw new UnauthenticatedError('invalid credentials')
```

### bind token to cookie and item subsets to user

#### 7. JSON Web Token

A JSON Web Token (JWT) is a compact and secure way of transmitting data between parties. It is often used to authenticate and authorize users in web applications and APIs. JWTs contain information about the user and additional metadata, and can be used to securely transmit this information

[Useful Resource](https://jwt.io/introduction)

```sh
npm i jsonwebtoken@9.0.0
```

utils/tokenUtils.js

```js
import jwt from 'jsonwebtoken'

export const createJWT = (payload) => {
  const token = jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN,
  })
  return token
}
```

JWT_SECRET represents the secret key used to sign the JWT. When creating a JWT, the payload (data) is signed with this secret key to generate a unique token. The secret key should be kept secure and should not be disclosed to unauthorized parties.

JWT_EXPIRES_IN specifies the expiration time for the JWT. It determines how long the token remains valid before it expires. The value of JWT_EXPIRES_IN is typically provided as a duration, such as "1h" for one hour or "7d" for seven days. Once the token expires, it is no longer considered valid and can't be used for authentication or authorization purposes.

These environment variables (JWT_SECRET and JWT_EXPIRES_IN) are read from the system environment during runtime, allowing for flexibility in configuration without modifying the code.

authController.js

```js
import { createJWT } from '../utils/tokenUtils.js'

const token = createJWT({ userId: user._id, role: user.role })
console.log(token)
```

#### 8. Test JWT (optional)

[JWT](https://jwt.io/)

#### 9. ENV Variables

- RESTART SERVER!!!!

.env

```js
JWT_SECRET=
JWT_EXPIRES_IN=
```

#### 10. HTTP Only Cookie

An HTTP-only cookie is a cookie that can't be accessed by JavaScript running in the browser. It is designed to help prevent cross-site scripting (XSS) attacks, which can be used to steal cookies and other sensitive information.

##### 11. HTTP Only Cookie VS Local Storage

An HTTP-only cookie is a type of cookie that is designed to be inaccessible to JavaScript running in the browser. It is primarily used for authentication purposes and is a more secure way of storing sensitive information like user tokens. Local storage, on the other hand, is a browser-based storage mechanism that is accessible to JavaScript, and is used to store application data like preferences or user-generated content. While local storage is convenient, it is not a secure way of storing sensitive information as it can be accessed and modified by JavaScript running in the browser.

authControllers.js

```js
const oneDay = 1000 * 60 * 60 * 24

res.cookie('token', token, {
  httpOnly: true,
  expires: new Date(Date.now() + oneDay),
  secure: process.env.NODE_ENV === 'production',
})

res.status(StatusCodes.CREATED).json({ msg: 'user logged in' })
```

```js
const oneDay = 1000 * 60 * 60 * 24
```

This line defines a constant oneDay that represents the number of milliseconds in a day. This value is used later to set the expiration time for the cookie.

```js
res.cookie('token', token, {...});:
```

This line sets a cookie with the name "token" and a value of token, which is the JWT that was generated for the user. The ... represents an object containing additional options for the cookie.

httpOnly: true: This option makes the cookie inaccessible to JavaScript running in the browser. This helps to prevent cross-site scripting (XSS) attacks, which can be used to steal cookies and other sensitive information.

expires: new Date(Date.now() + oneDay): This option sets the expiration time for the cookie. In this case, the cookie will expire one day from the current time (as represented by Date.now() + oneDay).

secure: process.env.NODE_ENV === 'production': This option determines whether the cookie should be marked as secure or not. If the NODE_ENV environment variable is set to "production", then the cookie is marked as secure, which means it can only be transmitted over HTTPS. This helps to prevent man-in-the-middle (MITM) attacks, which can intercept and modify cookies that are transmitted over unsecured connections.

AchievementController.js

```js
export const getAllAchievements = async (req, res) => {
  console.log(req)
  const achievement = await Achievement.find({})
  res.status(StatusCodes.OK).json({ achievement })
}
```

#### 12. Clean DB

#### 13. Connect User and Job

models/AchievementModel.js

```js
const AchievementSchema = new mongoose.Schema(
  {
    ....
    createdBy: {
      type: mongoose.Types.ObjectId,
      ref: 'User',
    },
  },
  { timestamps: true }
);
```

controllers/achievementController.js

```js
dont try to create yet, need to access user first.
```

## Higher order data concern (boundary pts)

### identifiable data set is accessible

#### 14. Auth Middleware

middleware/authMiddleware.js

```js
export const authenticateUser = async (req, res, next) => {
  console.log('auth middleware')
  next()
}
```

server.js

```js
import { authenticateUser } from './middleware/authMiddleware.js'

app.use('/api/v1/achievements', authenticateUser, achievementRouter)
```

##### 15. Cookie Parser

[Cookie Parser](https://www.npmjs.com/package/cookie-parser)

```sh
npm i cookie-parser@1.4.6
```

server.js

```js
import cookieParser from 'cookie-parser'
app.use(cookieParser())
```

#### 16. Access Token

authMiddleware.js

```js
import { UnauthenticatedError } from '../errors/customErrors.js'
export const authenticateUser = async (req, res, next) => {
  const { token } = req.cookies
  if (!token) {
    throw new UnauthenticatedError('authentication invalid')
  }
  next()
}
```

#### 17. Verify Token

utils/tokenUtils.js

```js
export const verifyJWT = (token) => {
  const decoded = jwt.verify(token, process.env.JWT_SECRET)
  return decoded
}
```

authMiddleware.js

```js
import { UnauthenticatedError } from '../errors/customErrors.js'
import { verifyJWT } from '../utils/tokenUtils.js'

export const authenticateUser = async (req, res, next) => {
  const { token } = req.cookies
  if (!token) {
    throw new UnauthenticatedError('authentication invalid')
  }

  try {
    const { userId, role } = verifyJWT(token)
    req.user = { userId, role }
    next()
  } catch (error) {
    throw new UnauthenticatedError('authentication invalid')
  }
}
```

AchievementController.js

```js
export const getAllAchievements = async (req, res) => {
  console.log(req.user)
  const achievement = await Achievement.find({ createdBy: req.user.userId })
  res.status(StatusCodes.OK).json({ achievement })
}
```

```js
export const createAchievement = async (req, res) => {
  req.body.createdBy = req.user.userId
  const achievement = await Achievement.create(req.body)
  res.status(StatusCodes.CREATED).json({ achievement })
}
```

- use user to construct a create Achievement request

```json
{
    "description":"do this",
    "status":"inactive",
    "points":0,
    "type":"progressive",
    "dateOfCompletion":null,
    "createdBy",""
}
```

### Permission lifeCycle

#### 18. Check Permissions

validationMiddleware.js

```js
const withValidationErrors = (validateValues) => {
  return [
    validateValues,
    (req, res, next) => {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
       ...
        if (errorMessages[0].startsWith('not authorized')) {
          throw new UnauthorizedError('not authorized to access this route');
        }

        throw new BadRequestError(errorMessages);
      }
      next();
    },
  ];
};
```

```js
import {
  BadRequestError,
  NotFoundError,
  UnauthorizedError,
} from '../errors/customErrors.js'

export const validateIdParam = withValidationErrors([
  param('id').custom(async (value, { req }) => {
    const isValidMongoId = mongoose.Types.ObjectId.isValid(value)
    if (!isValidMongoId) throw new BadRequestError('invalid MongoDB id')
    const job = await Job.findById(value)
    if (!job) throw new NotFoundError(`no job with id ${value}`)
    const isAdmin = req.user.role === 'admin'
    const isOwner = req.user.userId === job.createdBy.toString()
    if (!isAdmin && !isOwner)
      throw UnauthorizedError('not authorized to access this route')
  }),
])
```

- need log in correct user before access

create user

```json
{
  "name": "name",
  "email": "name3@gmail.com",
  "password": "secret123",
  "lastName": "lastName",
  "location": "my city"
}
```

Log in user

```json
{
  "email": "name3@gmail.com",
  "password": "secret123"
}
```

create achievements --> can access

```json
{
  "description": "do this",
  "status": "inactive",
  "points": 0,
  "type": "progressive",
  "dateOfCompletion": null,
  "createdBy": "652293ca9abc320592693e3e"
}
```

identifying workload must include createdBy (by being logged in)

```json
{
  "createdBy": "6516c7e3d81989b44aa337d2"
}
```

#### 19. Logout User

controllers/authController.js

```js
export const logout = (req, res) => {
  res.cookie('token', 'logout', {
    httpOnly: true,
    expires: new Date(Date.now()),
  })
  res.status(StatusCodes.OK).json({ msg: 'user logged out!' })
}
```

routes/authRouter.js

```js
import { Router } from 'express'
const router = Router()
import { logout } from '../controllers/authController.js'

router.get('/logout', logout)

export default router
```
