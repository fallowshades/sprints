#### Challenge (23) - Products Page (Setup)

- create following components and render in products page
  - Filters
  - SignsContainer
  - PaginationContainer
- in products page loader fetch all products

##### Products.jsx

1. Import Dependencies:

   - Import `Filters`, `PaginationContainer`, and `ProductsContainer` from `'../components'`.
   - Import `customFetch` from `'../utils'`.

2. Define URL and Loader Function:

   - Define a constant `url` containing the URL path to fetch products from.
   - Define a loader function that fetches product data from the defined URL.
   - Use `customFetch` to fetch the product data from the `url`.
   - Extract products and meta information from the response and return them.

3. Create Products Component:

   - Define a functional component named `Products`.

4. Component Structure:

   - Return a `Fragment` element (`<>...</>`) to wrap the component content.

5. Filters Component:

   - Include the `Filters` component to allow users to apply filters to the product list.

6. ProductsContainer Component:

   - Include the `ProductsContainer` component to display the list of products.

7. PaginationContainer Component:

   - Include the `PaginationContainer` component to manage product list pagination.

8. Export Products Component:
   - Export the `Products` component as the default export of the module.

#### Solution (23) - Products Page (Setup)

- import and setup loader in app.jsx

Products.jsx

```js
import { Filters, PaginationContainer, ProductsContainer } from '../components'
import { customFetch } from '../utils'

const url = '/products'
export const loader = async ({ request }) => {
  const response = await customFetch(url)

  const products = response.data.data
  const meta = response.data.meta
  return { products, meta }
}

const Products = () => {
  return (
    <>
      <Filters />
      <ProductsContainer />
      <PaginationContainer />
    </>
  )
}
export default Products
```

#### Challenge (24) - Products Container

- create ProductsList and render products in one column
- setup header (with total jobs and toggle buttons)
- toggle between ProductsGrid and ProductsList

##### ProductsList.jsx

1. Import Dependencies:

   - Import `formatPrice` from `'../utils'`.
   - Import `Link` and `useLoaderData` from `'react-router-dom'`.

2. Create ProductList Component:

   - Define a functional component named `ProductList`.

3. Component Structure:

   - Return a `div` element containing a list of products.

4. Loop Through Products:

   - Use the `useLoaderData` hook to get the `products` data from the loader.
   - Use the `map` function to loop through each product in the `products` array.

5. Product Link:

   - For each product, create a `Link` element that links to the individual product page.
   - Use the `product.id` as the link path (`to={`/products/${product.id}`}`).
   - Add CSS classes to style the link and apply hover effects.

6. Product Image:

   - Display the product image inside an `img` element.
   - Apply appropriate classes for styling and responsive design.
   - Add hover effect to the image using CSS classes.

7. Product Details:

   - Display the product title and company using `h3` and `h4` elements.
   - Add classes for font styles and responsiveness.

8. Product Price:

   - Display the formatted price using the `formatPrice` function.
   - Use a `p` element with appropriate classes for styling.

9. Export ProductList Component:
   - Export the `ProductList` component as the default export of the module.

##### ProductsContainer.jsx

1. Import Dependencies:

   - Import `useLoaderData` from `'react-router-dom'`.
   - Import `ProductsGrid` and `ProductsList` from their respective paths.
   - Import `useState` from `'react'`.
   - Import `BsFillGridFill` and `BsList` from `'react-icons/bs'`.

2. Create ProductsContainer Component:

   - Define a functional component named `ProductsContainer`.

3. Component Structure:

   - Return a `div` element containing the products container.

4. Total Products Count:

   - Use the `useLoaderData` hook to get the `meta` data from the loader.
   - Extract the `total` count of products from `meta.pagination`.
   - Use a conditional statement to handle the plural form of the word "product".

5. Layout State and Styles:

   - Use the `useState` hook to manage the layout state (grid or list).
   - Create a helper function `setActiveStyles` to generate the CSS classes based on the active layout.
   - Return appropriate CSS classes for active and inactive layouts.

6. Header Section:

   - Create a `div` for the header section containing the product count and layout buttons.
   - Display the total number of products using the extracted `totalProducts` count.
   - Create a button for grid layout and a button for list layout.
   - Attach click event handlers to the buttons to set the layout state.

7. Products Display:

   - Create a `div` to display the products.
   - Use conditional rendering to handle cases where no products match the search or when products are present.
   - If no products match the search, display a message.
   - If products are present and the layout is 'grid', display the `ProductsGrid` component.
   - If products are present and the layout is 'list', display the `ProductsList` component.

8. Export ProductsContainer Component:
   - Export the `ProductsContainer` component as the default export of the module.

#### Solution (24) - Products Container

[]destruct id,image,title: enum, dollarsAmount:int, company: string

ProductsList.jsx

```js
import { formatPrice } from '../utils'
import { Link, useLoaderData } from 'react-router-dom'

const ProductList = () => {
  const { products } = useLoaderData()
  return (
    <div className="mt-12 grid gap-y-8">
      {products.map((product) => {
        const { title, price, image, company } = product.attributes
        const dollarsAmount = formatPrice(price)

        return (
          <Link
            key={product.id}
            to={`/products/${product.id}`}
            className="p-8 rounded-lg flex flex-col sm:flex-row gap-y-4 flex-wrap bg-base-100 shadow-xl hover:shadow-2xl duration-300 group"
          >
            <img
              src={image}
              alt={title}
              className="h-24 w-24 rounded-lg sm:h-32 sm:w-32 object-cover group-hover:scale-105 transition duration-300"
            />
            <div className="ml-0 sm:ml-16">
              <h3 className="capitalize font-medium text-lg">{title}</h3>
              <h4 className="capitalize text-md text-neutral-content">
                {company}
              </h4>

              {/* COLOR */}
            </div>

            <p className="font-medium ml-0 sm:ml-auto text-lg">
              {dollarsAmount}
            </p>
          </Link>
        )
      })}
    </div>
  )
}

export default ProductList
```

ProductsContainer.jsx

```js
import { useLoaderData } from 'react-router-dom'
import ProductsGrid from './ProductsGrid'
import ProductsList from './ProductsList'
import { useState } from 'react'
import { BsFillGridFill, BsList } from 'react-icons/bs'

const ProductsContainer = () => {
  const { meta } = useLoaderData()
  const totalProducts = meta.pagination.total
  const [layout, setLayout] = useState('grid')

  const setActiveStyles = (pattern) => {
    return `text-xl btn btn-circle btn-sm ${
      pattern === layout
        ? 'btn-primary text-primary-content'
        : 'btn-ghost text-base-content'
    }`
  }

  return (
    <>
      {/* HEADER */}
      <div className="flex justify-between items-center mt-8 border-b border-base-300 pb-5">
        <h4 className="font-medium text-md">
          {totalProducts} product{totalProducts > 1 && 's'}
        </h4>
        <div className="flex gap-x-2">
          <button
            onClick={() => setLayout('grid')}
            className={setActiveStyles('grid')}
          >
            <BsFillGridFill />
          </button>

          <button
            onClick={() => setLayout('list')}
            className={setActiveStyles('list')}
          >
            <BsList />
          </button>
        </div>
      </div>

      {/* PRODUCTS */}
      <div>
        {totalProducts === 0 ? (
          <h5 className="text-2xl mt-16">
            Sorry, no products matched your search...
          </h5>
        ) : layout === 'grid' ? (
          <ProductsGrid />
        ) : (
          <ProductsList />
        )}
      </div>
    </>
  )
}

export default ProductsContainer
```

## Challenge (25) - Filters (Search Input)

- add size to prop FormInput.jsx
- render search input, submit button and reset button

## Solution (25) - Filters (Search Input)

FormInput.jsx

```js
const FormInput = ({ label, name, type, defaultValue, size }) => {
  return (
    <div className="form-control">
      <label htmlFor={name} className="label">
        <span className="label-text capitalize">{label}</span>
      </label>
      <input
        type={type}
        name={name}
        defaultValue={defaultValue}
        className={`input input-bordered ${size}`}
      />
    </div>
  )
}
export default FormInput
```

Filters.jsx

```js
import { Form, useLoaderData, Link } from 'react-router-dom'
import FormInput from './FormInput'

const Filters = () => {
  return (
    <Form className="bg-base-200 rounded-md px-8 py-4 grid gap-x-4 gap-y-8 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 items-center">
      {/* SEARCH */}
      <FormInput
        type="search"
        label="search product"
        name="search"
        size="input-sm"
      />
      {/* BUTTONS */}
      <button type="submit" className="btn btn-primary btn-sm ">
        search
      </button>
      <Link to="/products" className="btn btn-accent btn-sm">
        reset
      </Link>
    </Form>
  )
}
export default Filters
```

## Challenge (26) - Filters (Select Input)

- setup input for select input
- render for categories, companies and order
- companies and categories values are located in meta

### FormSelect.jsx

1. Create FormSelect Component:

   - Define a functional component named `FormSelect`.

2. Component Structure:

   - Return a `div` element containing the form select input.

3. Props:

   - Accept the following props: `label`, `name`, `list`, `defaultValue`, and `size`.

4. Label:

   - Create a `label` element with a `for` attribute matching the `name` prop.
   - Display the capitalized label text using the `label` prop.

5. Select Input:

   - Create a `select` element for the input field.
   - Set the `name` and `id` attributes to the value of the `name` prop.
   - Apply the appropriate CSS classes for the select input using the `size` prop.
   - Set the `defaultValue` of the select input using the `defaultValue` prop.

6. Options:

   - Map through the `list` prop array to generate individual `option` elements.
   - Use each item in the `list` as the `key` and `value` attributes of the `option` element.

7. Export FormSelect Component:
   - Export the `FormSelect` component as the default export of the module.

#### Solution (26) - Filters (Select Input)

FormSelect.jsx

```js
const FormSelect = ({ label, name, list, defaultValue, size }) => {
  return (
    <div className="form-control">
      <label htmlFor={name} className="label">
        <span className="label-text capitalize">{label}</span>
      </label>
      <select
        name={name}
        id={name}
        className={`select select-bordered ${size}`}
        defaultValue={defaultValue}
      >
        {list.map((item) => {
          return (
            <option key={item} value={item}>
              {item}
            </option>
          )
        })}
      </select>
    </div>
  )
}
export default FormSelect
```

Filters.jsx

```js
const { meta } = useLoaderData()

{
  /* CATEGORIES */
}
;<FormSelect
  label="select category"
  name="category"
  list={meta.categories}
  size="select-sm"
/>
{
  /* COMPANIES */
}
;<FormSelect
  label="select company"
  name="company"
  list={meta.companies}
  size="select-sm"
/>
{
  /* ORDER */
}
;<FormSelect
  label="sort by"
  name="order"
  list={['a-z', 'z-a', 'high', 'low']}
  size="select-sm"
/>
```

#### Challenge (27) - Filters (Price)

- create range input (hint: you will need local state)

##### FormRange.jsx

1. Create FormRange Component:

   - Define a functional component named `FormRange`.

2. Component Structure:

   - Return a `div` element containing the form range input and related elements.

3. Props:

   - Accept the following props: `label`, `name`, and `size`.

4. Default Values:

   - Define default values for `step`, `maxPrice`, and `selectedPrice`.

5. Label and Selected Price Display:

   - Create a `label` element with a `for` attribute matching the `name` prop.
   - Display the capitalized label text using the `label` prop.
   - Display the selected price using the `formatPrice` function.

6. Range Input:

   - Create an `input` element with `type` set to `'range'`.
   - Set the `name`, `min`, `max`, `value`, and `step` attributes.
   - Use the `selectedPrice` state for the `value` attribute.
   - Set the `onChange` event handler to update `selectedPrice`.

7. Min and Max Price Display:

   - Create a `div` element for displaying minimum and maximum price values.
   - Use the `formatPrice` function for formatting and displaying max price.

8. Export FormRange Component:
   - Export the `FormRange` component as the default export of the module.

#### Solution (27) - Filters (Price )

FormRange.jsx

```js
import { formatPrice } from '../utils'
import { useState } from 'react'
const FormRange = ({ label, name, size }) => {
  const step = 1000
  const maxPrice = 100000
  const [selectedPrice, setSelectedPrice] = useState(maxPrice)

  return (
    <div className="form-control">
      <label htmlFor={name} className="label cursor-pointer">
        <span className="label-text capitalize">{label}</span>
        <span>{formatPrice(selectedPrice)}</span>
      </label>
      <input
        type="range"
        name={name}
        min={0}
        max={maxPrice}
        value={selectedPrice}
        onChange={(e) => setSelectedPrice(e.target.value)}
        className={`range range-primary ${size}`}
        step={step}
      />
      <div className="w-full flex justify-between text-xs px-2 mt-2">
        <span className="font-bold text-md">0</span>
        <span className="font-bold text-md">Max : {formatPrice(maxPrice)}</span>
      </div>
    </div>
  )
}
export default FormRange
```

Filters.jsx

```js
{
  /* PRICE */
}
;<FormRange label="select price" name="price" size="range-sm" />
```

#### Challenge (28) - Filters (Shipping)

- create checkbox input

##### FormCheckbox.jsx

1. Create FormCheckbox Component:

   - Define a functional component named `FormCheckbox`.

2. Component Structure:

   - Return a `div` element containing the form checkbox input and related elements.

3. Props:

   - Accept the following props: `label`, `name`, `defaultValue`, and `size`.

4. Label Display:

   - Create a `label` element with a `for` attribute matching the `name` prop.
   - Display the capitalized label text using the `label` prop.

5. Checkbox Input:

   - Create an `input` element with `type` set to `'checkbox'`.
   - Set the `name` attribute to match the `name` prop.
   - Set the `defaultChecked` attribute using the `defaultValue` prop.
   - Use the `size` prop to determine the checkbox size class.

6. Styling and Layout:

   - Apply appropriate classes to style and position the form control items.

7. Export FormCheckbox Component:
   - Export the `FormCheckbox` component as the default export of the module.

#### Solution (28) - Filters (Shipping)

FormCheckbox.jsx

```js
const FormCheckbox = ({ label, name, defaultValue, size }) => {
  return (
    <div className="form-control items-center">
      <label htmlFor={name} className="label cursor-pointer">
        <span className="label-text capitalize">{label}</span>
      </label>
      <input
        type="checkbox"
        name={name}
        defaultChecked={defaultValue}
        className={`checkbox checkbox-primary ${size}`}
      />
    </div>
  )
}
export default FormCheckbox
```

Filters.jsx

```js
{
  /* SHIPPING */
}
;<FormCheckbox label="free shipping" name="shipping" size="checkbox-sm" />
```

### Load and paginate

#### Challenge (29) - Global Loading

- create loading component
- check for loading state in HomeLayout
- toggle between loading and <Outlet>

##### Loading.jsx

1. Create Loading Component:

   - Define a functional component named "Loading".

2. Component Structure:

   - Return a "div" element with CSS classes to center content both vertically and horizontally.

3. Loading Animation:

   - Inside the "div", include a "span" element with the classes "loading loading-ring loading-lg".
   - This applies a loading animation to create the visual effect.

4. Styling:

   - Use the provided CSS classes to style the loading animation.

5. Export Loading Component:
   - Export the "Loading" component as the default export of the module.

##### HomeLayout.jsx

1. Create HomeLayout Component:

   - Define a functional component named "HomeLayout".

2. Import Dependencies:

   - Import "Outlet" and "useNavigation" from 'react-router-dom'.
   - Import "Navbar", "Loading", and "Header" from '../components'.

3. Component Structure:

   - Return a fragment ('<>...</>') to encapsulate the component's content.

4. UseNavigation Hook:

   - Use the "useNavigation" hook to access the navigation state.
   - Store whether the page is currently loading in "isPageLoading" variable.

5. Conditional Rendering:

   - Use a ternary operator to conditionally render content:
     - If "isPageLoading" is true, render the "Loading" component.
     - Otherwise, render a "section" element with CSS classes and include the "Outlet" component.

6. Header and Navbar:

   - Include the "Header" and "Navbar" components at the beginning of the component.

7. Styling:

   - Apply CSS classes to style the layout and align its elements.

8. Export HomeLayout Component:
   - Export the "HomeLayout" component as the default export of the module.

#### Solution (29) - Global Loading

Loading.jsx

```js
const Loading = () => {
  return (
    <div className="h-screen flex items-center justify-center">
      <span className="loading loading-ring loading-lg" />
    </div>
  )
}
export default Loading
```

```js
import { Outlet, useNavigation } from 'react-router-dom'
import { Navbar, Loading, Header } from '../components'
const HomeLayout = () => {
  const navigation = useNavigation()
  const isPageLoading = navigation.state === 'loading'
  return (
    <>
      <Header />
      <Navbar />
      {isPageLoading ? (
        <Loading />
      ) : (
        <section className="align-element py-20">
          <Outlet />
        </section>
      )}
    </>
  )
}
export default HomeLayout
```

#### Challenge (30) - Setup Params

- explore how to filter products
  [API DOCS](https://documenter.getpostman.com/view/18152321/2s9Xy5KpTi#80a47ff5-cc24-494b-89e0-02cd92acc226)
- test in Thunder Client
- access params in loader
- use params in customFetch
- pass params down
- use params as default values (price in FormRange)
- setup reset button

#### Solution (30) - Setup Params

Products.jsx

```js
export const loader = async ({ request }) => {
  const params = Object.fromEntries([
    ...new URL(request.url).searchParams.entries(),
  ])
  const response = await customFetch(url, { params })

  const products = response.data.data
  const meta = response.data.meta

  return { products, meta, params }
}
```

Filters.jsx

```js
import { Form, useLoaderData, Link } from 'react-router-dom'
import FormInput from './FormInput'
import FormSelect from './FormSelect'
import FormRange from './FormRange'
import FormCheckbox from './FormCheckbox'
const Filters = () => {
  const { meta, params } = useLoaderData()
  const { search, company, category, shipping, order, price } = params
  return (
    <Form className="bg-base-200 rounded-md px-8 py-4 grid gap-x-4 gap-y-8 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 items-center">
      {/* SEARCH */}
      <FormInput
        type="search"
        label="search product"
        name="search"
        defaultValue={search}
        size="input-sm"
      />
      {/* CATEGORIES */}
      <FormSelect
        label="select category"
        name="category"
        list={meta.categories}
        defaultValue={category}
        size="select-sm"
      />
      {/* COMPANIES */}
      <FormSelect
        label="select company"
        name="company"
        list={meta.companies}
        defaultValue={company}
        size="select-sm"
      />
      {/* ORDER */}
      <FormSelect
        label="sort by"
        name="order"
        list={['a-z', 'z-a', 'high', 'low']}
        defaultValue={order}
        size="select-sm"
      />
      {/* PRICE */}
      <FormRange
        label="select price"
        name="price"
        price={price}
        size="range-sm"
      />
      {/* SHIPPING */}
      <FormCheckbox
        label="free shipping"
        name="shipping"
        defaultValue={shipping}
        size="checkbox-sm"
      />
      {/* BUTTONS */}
      <button type="submit" className="btn btn-primary btn-sm">
        search
      </button>
      <Link to="/products" className="btn btn-accent btn-sm">
        reset
      </Link>
    </Form>
  )
}
export default Filters
```

```js
const params = Object.fromEntries([
  ...new URL(request.url).searchParams.entries(),
])
```

It takes a URL string from the request.url property.
It creates a URL object from that URL string.
It extracts the query parameters using the searchParams property.
It converts the query parameters into an iterable of key-value pairs using the entries() method.
It spreads these key-value pairs into an array.
It uses Object.fromEntries() to create a new object where the key-value pairs become properties of the object.

#### Challenge (31) - Pagination

- explore how to paginate
- test in Thunder Client
- access meta
- display next, prev and page buttons
- add page value to query params

#### Building the PaginationContainer Component

1. Import Required Modules:

- Import hooks and modules from `react-router-dom`:
  - `useLoaderData`
  - `useLocation`
  - `useNavigate`

2. Initialize Component:

- Create a functional component named `PaginationContainer`.

3. Retrieve Data with Hooks:

- Use the `useLoaderData` hook to get the `meta` data.
- Destructure `pageCount` and `page` from `meta.pagination`.
- Use the `useLocation` hook to get `search` and `pathname`.
- Use the `useNavigate` hook to get the `navigate` function.

4. Generate Pages Array:

- Create an array called `pages` using `Array.from()`.
  - This represents all the page numbers.

5. Handle Page Change:

- Create a function `handlePageChange` that takes `pageNumber` as an argument.
  - Update the URL's query string parameter `page` with the new page number.
  - Navigate to the updated URL.

6. Conditional Rendering:

- If `pageCount` is less than 2:
  - Return `null`.

7. Render Pagination Component:

- Render a `div` container with the class `mt-16 flex justify-end`.
  - Inside this, render another `div` with class `join`.
    - For "Prev" button:
      - Use the class `btn btn-xs sm:btn-md join-item`.
    - For page numbers:
      - Use the class `btn btn-xs sm:btn-md border-none join-item`.
      - Highlight the current page with classes `bg-base-300 border-base-300`.
    - For "Next" button:
      - Use the class `btn btn-xs sm:btn-md join-item`.

8. Export Component:

- Export the `PaginationContainer` component.

#### Solution (31) - Pagination

PaginationContainer.jsx

```js
import { useLoaderData, useLocation, useNavigate } from 'react-router-dom'

const PaginationContainer = () => {
  const { meta } = useLoaderData()
  const { pageCount, page } = meta.pagination
  const pages = Array.from({ length: pageCount }, (_, index) => {
    return index + 1
  })
  const { search, pathname } = useLocation()
  const navigate = useNavigate()

  const handlePageChange = (pageNumber) => {
    const searchParams = new URLSearchParams(search)
    searchParams.set('page', pageNumber)
    navigate(`${pathname}?${searchParams.toString()}`)
  }

  if (pageCount < 2) return null

  return (
    <div className="mt-16 flex justify-end">
      <div className="join">
        <button
          className="btn btn-xs sm:btn-md join-item"
          onClick={() => {
            let prevPage = page - 1
            if (prevPage < 1) prevPage = pageCount
            handlePageChange(prevPage)
          }}
        >
          Prev
        </button>
        {pages.map((pageNumber) => {
          return (
            <button
              onClick={() => handlePageChange(pageNumber)}
              key={pageNumber}
              className={`btn btn-xs sm:btn-md border-none join-item ${
                pageNumber === page ? 'bg-base-300 border-base-300' : ''
              }`}
            >
              {pageNumber}
            </button>
          )
        })}
        <button
          className="btn btn-xs sm:btn-md join-item"
          onClick={() => {
            let nextPage = page + 1
            if (nextPage > pageCount) nextPage = 1
            handlePageChange(nextPage)
          }}
        >
          Next
        </button>
      </div>
    </div>
  )
}
export default PaginationContainer
```
