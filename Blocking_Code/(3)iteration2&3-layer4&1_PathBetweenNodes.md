## Path Between Nodes

### (1) Secure package untampered

#### Hash Passwords

- one way street, only compare hashed values
- [bcrypt.js](https://www.npmjs.com/package/bcryptjs)

```sh
npm install bcryptjs
```

- User Model
- import bcrypt from 'bcryptjs'
- await genSalt(10)
- await hash(password , salt)
- await compare(requestPassword , currentPassword)
- [mongoose middleware](https://mongoosejs.com/docs/middleware.html)
- UserSchema.pre('save',async function(){
  "this" points to instance created by UserSchema
  })

#### Mongoose - Custom Instance Methods

[Custom Instance Methods](https://mongoosejs.com/docs/guide.html#methods)

- UserSchema.methods.createJWT = function(){console.log(this)}
- register controller
- right after User.create()
- invoke user.createJWT()

#### JWT

- token
- [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)

```sh
npm install jsonwebtoken
```

- User Model
- import jwt from 'jsonwebtoken'
- jwt.sign(payload,secret,options)
- createJWT

```js
return jwt.sign({ userId: this._id }, 'jwtSecret', { expiresIn: '1d' })
```

```js
return jwt.sign({ userId: this._id }, process.env.JWT_SECRET, {
  expiresIn: process.env.JWT_LIFETIME,
})
```

#### JWT_SECRET and JWT_LIFETIME

- [Keys Generator](https://www.allkeysgenerator.com/)
- RESTART SERVER!!!!

#### Complete Register

- password : {select:false}
- complete response

### (2) Secure proxy route between package delivery

#### Concurrently

- front-end and backend (server)
- run separate terminals
- [concurrently](https://www.npmjs.com/package/concurrently)

```sh
npm install concurrently --save-dev

```

- package.json

```js
// --kill-others switch, all commands are killed if one dies
// --prefix client - folder
// cd client && npm start
// escape quotes

"scripts": {
    "server": "nodemon server --ignore client",
    "client": "npm start --prefix client",
    "start": "concurrently --kill-others-on-fail \"npm run server\" \" npm run client\""
  },
```

#### Cors Error

[Cors Error](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

- two fixes (cors package and proxy)

#### Cors Package

[cors package](https://www.npmjs.com/package/cors)

```sh
npm install cors
```

```js
import cors from 'cors'

app.use(cors())
```

#### Proxy

- access from anywhere
- don't want to use full url

[cra proxy](https://create-react-app.dev/docs/proxying-api-requests-in-development/)

```js
"proxy":"http://localhost:5000"
```

- my preference to remove trailing slash /
- restart app
