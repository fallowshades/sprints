# itteration 2 layer 1 blacck and white box

[file-system]

- express.json

[container-w-port]

- npm init -y --> type module

## white box is safly monitored

### l

#### 1. Folder Setup

- IMPORTANT !!!!
- remove existing .git folder (if any) from client

Mac

```sh
rm -rf .git
```

Windows

```sh
rmdir -Force -Recurse .git
```

```sh
rd /s /q .git
```

- Windows commands were shared by students and I have not personally tested them.
- git status should return :
  "fatal: Not a git repository (or any of the parent directories): .git"
- create jobify directory
- copy/paste client
- move README to root

#### 2. Setup Server

- create package.json

````sh
npm init -y
```cd ..

- create and test server.js

```sh
node server
````

#### 3. ES6 Modules

package.json

```json
  "type": "module",
```

Create test.js and implement named import

test.js

```js
export const value = 42
```

server.js

```js
import { value } from './test.js'
console.log(value)
```

- don't forget about .js extension
- for named imports, names must match

#### 4. Source Control

- create .gitignore
- copy values from client/.gitignore
- create Github Repo (optional)

#### 5. Install Packages and Setup Install Script

```sh
npm install bcryptjs@2.4.3 concurrently@8.0.1 cookie-parser@1.4.6 dayjs@1.11.7 dotenv@16.0.3 express@4.18.2 express-async-errors@3.1.1 express-validator@7.0.1 http-status-codes@2.2.0 jsonwebtoken@9.0.0 mongoose@7.0.5 morgan@1.10.0 multer@1.4.5-lts.1 nanoid@4.0.2 nodemon@2.0.22 cloudinary@1.37.3 dayjs@1.11.9 datauri@4.1.0

```

bcryptjs@2.4.3
concurrently@8.0.1
cookie-parser@1.4.6
dayjs@1.11.7
dotenv@16.0.3
express@4.18.2
express-async-errors@3.1.1
express-validator@7.0.1
http-status-codes@2.2.0
jsonwebtoken@9.0.0
mongoose@7.0.5
morgan@1.10.0
multer@1.4.5-lts.1
nanoid@4.0.2
nodemon@2.0.22
cloudinary@1.37.3
dayjs@1.11.9
datauri@4.1.0

package.json

```json
"scripts": {
    "setup-project": "npm i && cd client && npm i"
  },
```

- install packages in root and client

```sh
npm run setup-project
```

## remote black boxes

### 1. Tester (thunder Client /morgan, accept json)

#### 6. Setup Basic Express

- install express and nodemon.
- setup a basic server which listening on PORT=5100
- create a basic home route which sends back "hello world"
- setup a script with nodemon package.

[Express Docs](https://expressjs.com/)

Express is a fast and minimalist web application framework for Node.js. It simplifies the process of building web applications by providing a robust set of features for handling HTTP requests, routing, middleware, and more. Express allows you to create server-side applications and APIs easily, with a focus on simplicity and flexibility.

[Nodemon Docs](https://nodemon.io/)

Nodemon is a development tool that improves the developer experience. It monitors your Node.js application for any changes in the code and automatically restarts the server whenever a change is detected. This eliminates the need to manually restart the server after every code modification, making the development process more efficient and productive. Nodemon is commonly used during development to save time and avoid the hassle of manual server restarts.

```sh
npm i express@4.18.2 nodemon@2.0.22
```

server.js

```js
import express from 'express'
const app = express()

app.get('/', (req, res) => {
  res.send('Hello World')
})

app.listen(5100, () => {
  console.log('server running....')
})
```

package.json

```json
"scripts": {
    "dev": "nodemon server.js"
  },
```

#### 7. Thunder Client

Thunder Client is a popular Visual Studio Code extension that facilitates API testing and debugging. It provides a user-friendly interface for making HTTP requests and viewing the responses, allowing developers to easily test APIs, examine headers, and inspect JSON/XML payloads. Thunder Client offers features such as environment variables, request history, and the ability to save and organize requests for efficient development workflows.

[Thunder Client](https://www.thunderclient.com/)

- install and test home route

#### 8. Accept JSON

Setup express middleware to accept json

server

```js
app.use(express.json())

app.post('/', (req, res) => {
  console.log(req)

  res.json({ message: 'Data received', data: req.body })
})
```

### Architect

#### 9. Morgan and Dotenv

[Morgan](https://www.npmjs.com/package/morgan)

HTTP request logger middleware for node.js

[Dotenv](https://www.npmjs.com/package/dotenv)

Dotenv is a zero-dependency module that loads environment variables from a .env file into process.env.

```sh
npm i morgan@1.10.0 dotenv@16.0.3
```

```js
import morgan from 'morgan'

app.use(morgan('dev'))
```

- create .env file in the root
- add PORT and NODE_ENV
- add .env to .gitignore

server.js

```js
import * as dotenv from 'dotenv'
dotenv.config()

if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'))
}

const port = process.env.PORT || 5100
app.listen(port, () => {
  console.log(`server running on PORT ${port}....`)
})
```

### Developer (convenience restart/fetch)

#### 10. New Features

- fetch API
- global await (top-level await)
- watch mode

```js
try {
  const response = await fetch(
    'https://www.course-api.com/react-useReducer-cart-project'
  )
  const cartData = await response.json()
  console.log(cartData)
} catch (error) {
  console.log(error)
}
```

package.json

```json
 "scripts": {
    "watch": "node --watch server.js "
  },
```
