# iteration 1 layer

## e(2)

### (2)

#### Product Model

- [] create Product.js in models folder
- [] create Schema
- [] name : {type:String}
- [] price: {type:Number}
- [] description: {type:String}
- [] image: {type:String}
- [] category: {type:String}
- [] company: {type:String}
- [] colors: {type:[]}
- [] featured: {type:Boolean}
- [] freeShipping: {type:Boolean}
- [] inventory:{type:Number}
- [] averageRating:{type:Number}
- [] user
- [] set timestamps
- [] export Product model

SignModel

```js
const mongoose = require('mongoose')

const ProductSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      trim: true,
      required: [true, 'Please provide product name'],
      maxlength: [100, 'Name can not be more than 100 characters'],
    },
    price: {
      type: Number,
      required: [true, 'Please provide product price'],
      default: 0,
    },
    description: {
      type: String,
      required: [true, 'Please provide product description'],
      maxlength: [1000, 'Description can not be more than 1000 characters'],
    },
    image: {
      type: String,
      default: '/uploads/example.jpeg',
    },
    category: {
      type: String,
      required: [true, 'Please provide product category'],
      enum: ['office', 'kitchen', 'bedroom'],
    },
    company: {
      type: String,
      required: [true, 'Please provide company'],
      enum: {
        values: ['ikea', 'liddy', 'marcos'],
        message: '{VALUE} is not supported',
      },
    },
    colors: {
      type: [String],
      default: ['#222'],
      required: true,
    },
    featured: {
      type: Boolean,
      default: false,
    },
    freeShipping: {
      type: Boolean,
      default: false,
    },
    inventory: {
      type: Number,
      required: true,
      default: 15,
    },
    averageRating: {
      type: Number,
      default: 0,
    },
    numOfReviews: {
      type: Number,
      default: 0,
    },
    user: {
      type: mongoose.Types.ObjectId,
      ref: 'User',
      required: true,
    },
  },
  { timestamps: true, toJSON: { virtuals: true }, toObject: { virtuals: true } }
)
/*ProductSchema.virtual('reviews', {
  ref: 'Review',
  localField: '_id',
  foreignField: 'product',
  justOne: false,
});

ProductSchema.pre('remove', async function (next) {
  await this.model('Review').deleteMany({ product: this._id });
});
*/
module.exports = mongoose.model('Product', ProductSchema)
```

#### Product Structure

- [] add productController file in controllers
- [] export (createProduct, getAllProducts,
  getSingleProduct, updateProduct, deleteProduct, uploadImage) functions
- [] res.send('function name')
- [] setup productRoutes file in routes
- [] import all controllers
- [] only getAllProducts and getSingleProduct accessible to public
- [] rest only by admin (setup middlewares)
- [] typical setup
- [] router.route('/uploadImage').post(uploadImage)
- [] import productRoutes as productRouter in the app.js
- [] setup app.use('/api/v1/products', productRouter)

ProductController

```js
const Product = require('../models/Product')
const { StatusCodes } = require('http-status-codes')
const CustomError = require('../errors')
const path = require('path')

const createProduct = async (req, res) => {
  res.send('')
}
const getAllProducts = async (req, res) => {
  res.send('')
}
const getSingleProduct = async (req, res) => {
  res.send('')
}
const updateProduct = async (req, res) => {
  res.send('')
}
const deleteProduct = async (req, res) => {
  res.send('')
}
const uploadImage = async (req, res) => {
  res.send('')
}

module.exports = {
  createProduct,
  getAllProducts,
  getSingleProduct,
  updateProduct,
  deleteProduct,
  uploadImage,
}
```

productRouter

```js
const express = require('express')
const router = express.Router()
const {
  authenticateUser,
  authorizePermissions,
} = require('../middleware/authentication')

const {
  createProduct,
  getAllProducts,
  getSingleProduct,
  updateProduct,
  deleteProduct,
  uploadImage,
} = require('../controllers/productController')

const { getSingleProductReviews } = require('../controllers/reviewController')

router
  .route('/')
  .post([authenticateUser, authorizePermissions('admin')], createProduct)
  .get(getAllProducts)

router
  .route('/uploadImage')
  .post([authenticateUser, authorizePermissions('admin')], uploadImage)

router
  .route('/:id')
  .get(getSingleProduct)
  .patch([authenticateUser, authorizePermissions('admin')], updateProduct)
  .delete([authenticateUser, authorizePermissions('admin')], deleteProduct)

router.route('/:id/reviews').get(getSingleProductReviews)

module.exports = router
```

app

```js
const productRouter = require('./routes/productRoutes')
app.use('/api/v1/products', productRouter)
```

### (4)

#### Product Routes in Postman

#### Create Product

- [] create user property on req.body and set it equal to userId (req.user)
- [] pass req.body into Product.create
- [] send back the product

#### Remaining Controllers (apart from uploadImage)

- [] getAllProducts
- [] getSingleProduct
- [] updateProduct
- [] deleteProduct
- [] typical CRUD, utilize (task or job) project
- [] remember we check already for role 'admin'

#### Upload Image

- [] if some question, re-watch 07-file-upload
- [] images folder with two images
