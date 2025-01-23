## Hosted Project

[E-Commerce API Render URL](https://node-course-e-commerce.onrender.com/)

#### Setup Basic Express Server

- [] import express and assign to variable
- [] setup start port variable (5000) and start function

```js
const express = require('express');
const app = express();
const port = process.env.PORT || 5000;

const start = async () => {
    try{
        app.listen(port, console.log("working))
    }catch(error){
        console.log(err)
    }
}
```

#### Connect To DB

- [] get connection string
- [] setup .env with MONGO_URL variable and assign the value
- [] import 'dotenv' and setup package
- [] import connect() and invoke in the starter
- [] restart the server
- [] mongoose V6 info

#### Basic Routes and Middleware

- [] setup / GET Route
- [] setup express.json() middleware
- [] setup 404 and errorHandler middleware
- [] import 'exress-async-errors' package

#### 404 vs ErrorHandler Middleware

#### Morgan Pacakge

- [Morgan Package](https://www.npmjs.com/package/morgan)

#### User Model

- [] create models folder and User.js file
- [] create schema with name,email, password (all type:String)
- [] export mongoose model

#### Validator Package

- [Validator](https://www.npmjs.com/package/validator)
