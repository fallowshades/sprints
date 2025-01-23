## Deploy

### (1)security in production

#### Production Setup - Fix Warnings and logoutUser

- getJobs,deleteJob,showStats - invoke logoutUser()
- fix warnings

```sh
// eslint-disable-next-line
```

#### Production Setup - Build Front-End Application

- create front-end production application

```js
package.json
"scripts": {
    "build-client": "cd client && npm run build",
    "server": "nodemon server.js --ignore client",
    "client": "cd client && npm run start",
    "start": "concurrently --kill-others-on-fail \"npm run server\" \"npm run client\""

  },

```

```js
server.js

import { dirname } from 'path'
import { fileURLToPath } from 'url'
import path from 'path'

const __dirname = dirname(fileURLToPath(import.meta.url))

// only when ready to deploy
app.use(express.static(path.resolve(__dirname, './client/build')))

// routes
app.use('/api/v1/auth', authRouter)
app.use('/api/v1/jobs', authenticateUser, jobsRouter)

// only when ready to deploy
app.get('*', function (request, response) {
  response.sendFile(path.resolve(__dirname, './client/build', 'index.html'))
})
```

#### Security Packages

- remove log in the error-handler
- [helmet](https://www.npmjs.com/package/helmet)
  Helmet helps you secure your Express apps by setting various HTTP headers.
- [xss-clean](https://www.npmjs.com/package/xss-clean)
  Node.js Connect middleware to sanitize user input coming from POST body, GET queries, and url params.
- [express-mongo-sanitize](https://www.npmjs.com/package/express-mongo-sanitize)
  Sanitizes user-supplied data to prevent MongoDB Operator Injection.
- [express-rate-limit](https://www.npmjs.com/package/express-rate-limit)
  Basic rate-limiting middleware for Express.

```sh
npm install helmet xss-clean express-mongo-sanitize express-rate-limit
```

```js
server.js

import helmet from 'helmet'
import xss from 'xss-clean'
import mongoSanitize from 'express-mongo-sanitize'

app.use(express.json())
app.use(helmet())
app.use(xss())
app.use(mongoSanitize())
```

#### Limit Requests

```js
authRoutes.js

import rateLimiter from 'express-rate-limit'

const apiLimiter = rateLimiter({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10,
  message: 'Too many requests from this IP, please try again after 15 minutes',
})

router.route('/register').post(apiLimiter, register)
router.route('/login').post(apiLimiter, login)
```

#### Alternative Search with Debounce

client/components/SearchContainer.js

```js

import { useState, useMemo } from 'react';
const SearchContainer = () => {
  const [localSearch, setLocalSearch] = useState('');
  const {
    ....
  } = useAppContext();
  const handleSearch = (e) => {
    handleChange({ name: e.target.name, value: e.target.value });
  };
  const handleSubmit = (e) => {
    e.preventDefault();
    clearFilters();
  };
  const debounce = () => {
    let timeoutID;
    return (e) => {
      setLocalSearch(e.target.value);
      clearTimeout(timeoutID);
      timeoutID = setTimeout(() => {
        handleChange({ name: e.target.name, value: e.target.value });
      }, 1000);
    };
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    setLocalSearch('');
    clearFilters();
  };

  const optimizedDebounce = useMemo(() => debounce(), []);
  return (
    <Wrapper>
      <form className='form'>
        <h4>search form</h4>
        <div className='form-center'>
          {/* search position */}

          <FormRow
            type='text'
            name='search'
            value={localSearch}
            handleChange={optimizedDebounce}
          />
         ........
        </div>
      </form>
    </Wrapper>
  );
};

export default SearchContainer;
```
