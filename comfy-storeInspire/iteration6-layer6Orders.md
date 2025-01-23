## Challenge (51) - Orders Request Overview

- [API DOCS](https://documenter.getpostman.com/view/18152321/2s9Xy5KpTi)
- docs - orders request
- test in Thunder Client

## Solution (51) - Orders Request Overview

## Challenge (52) - Orders Page Setup

- create components/OrdersList.jsx (export)
- create loader (import/setup in App.jsx and provide store)
- restrict access to page
- make a request to get all users
- grab all the query params
- return orders and meta

### Orders.jsx

1. **Import Dependencies:**

   - Import the required modules and components from 'react-router-dom', 'react-toastify', and other custom files.

2. **Define Loader Function:**

   - Create a loader function that takes the `store` parameter and an object with a `request` property.
   - Within the loader function:
     - Retrieve the user information from the Redux store.
     - Check if the user is logged in. If not, display a warning toast and redirect to the login page.
     - Parse query parameters from the URL.
     - Use the `customFetch` utility to make a GET request to the '/orders' endpoint.
     - Handle successful responses by returning the fetched orders data and meta information.
     - Handle errors by displaying appropriate error messages using toast and optionally redirecting if unauthorized.
     - Return `null` if an error occurs.

3. **Define Orders Component:**

   - Create a functional component named `Orders`.
   - Within the component:
     - Return JSX that displays a heading element with the text "orders".

4. **Export Loader Function:**

   - Export the defined loader function.

5. **Export Orders Component:**
   - Export the `Orders` component as the default export of the module.

## Solution (52) - Orders Page Setup

App.jsx

```js
import { loader as ordersLoader } from './pages/Orders';

{
  path: 'orders',
  element: <Orders />,
  loader: ordersLoader(store),
},
```

Orders.jsx

```js
import { redirect, useLoaderData } from 'react-router-dom'
import { toast } from 'react-toastify'
import { customFetch } from '../utils'
import { OrdersList, PaginationContainer, SectionTitle } from '../components'

export const loader =
  (store) =>
  async ({ request }) => {
    const user = store.getState().userState.user

    if (!user) {
      toast.warn('You must be logged in to view orders')
      return redirect('/login')
    }
    const params = Object.fromEntries([
      ...new URL(request.url).searchParams.entries(),
    ])
    try {
      const response = await customFetch.get('/orders', {
        params,
        headers: {
          Authorization: `Bearer ${user.token}`,
        },
      })

      return { orders: response.data.data, meta: response.data.meta }
    } catch (error) {
      console.log(error)
      const errorMessage =
        error?.response?.data?.error?.message ||
        'there was an error accessing your orders'

      toast.error(errorMessage)
      if (error?.response?.status === 401 || 403) return redirect('/login')

      return null
    }
  }
const Orders = () => {
  return <h1 className="text-3xl">orders</h1>
}
export default Orders
```

## Challenge (53) - Render Orders

### Orders.jsx

- Import Dependencies:

  - Import necessary components and hooks from your project's dependencies.

- Define Component:

  - Define a functional component named `Orders`.

- Fetch Data:

  - Use the `useLoaderData` hook to access data from the loader context.
  - Check if the `meta.pagination.total` value is less than 1 to determine if there are no orders.

- Conditional Rendering:

  - If there are no orders, return a component, such as `<SectionTitle />`, with a message like 'Please make an order'.

- Orders Rendering:

  - If there are orders, return the following components:
    - `<SectionTitle />` with the text 'Your Orders'.
    - `<OrdersList />` component to display the list of orders.
    - `<PaginationContainer />` component for handling pagination.

- Export Component:
  - Export the `Orders` component as the default export.

### OrdersList.jsx

- Import Dependencies:

  - Import `useLoaderData` from `'react-router-dom'`.
  - Import `day` and `advancedFormat` from `'dayjs'`.
  - Extend dayjs with the `advancedFormat` plugin using `day.extend(advancedFormat)`.

- Create the `OrdersList` component:

  - Inside the component:
    - Use the `useLoaderData()` hook to get data from loader data.
    - Destructure `orders` and `meta` from the loaded data.
    - Return the orders list interface:
      - Display the total number of orders using the `meta.pagination.total` value.
      - Create a table to display order information.
      - Define table headings: Name, Address, Products, Cost, and Date.
      - Use `.map()` to iterate over each order and generate table rows:
        - Destructure relevant attributes from the `order` object.
        - Format the `createdAt` date using dayjs to display in 'hh:mm a - MMM Do, YYYY' format.
        - Return a table row with the extracted data.

- Export the `OrdersList` component.

## Solution (53) - Render Orders

Orders.jsx

```js
const Orders = () => {
  const { meta } = useLoaderData()
  if (meta.pagination.total < 1) {
    return <SectionTitle text="Please make an order" />
  }
  return (
    <>
      <SectionTitle text="Your Orders" />
      <OrdersList />
      <PaginationContainer />
    </>
  )
}
export default Orders
```

OrdersList.jsx

```js
import { useLoaderData } from 'react-router-dom'
import day from 'dayjs'
import advancedFormat from 'dayjs/plugin/advancedFormat'
day.extend(advancedFormat)

const OrdersList = () => {
  const { orders, meta } = useLoaderData()
  return (
    <div className="mt-8">
      <h4 className="mb-4 capitalize">
        total orders : {meta.pagination.total}
      </h4>
      <div className="overflow-x-auto ">
        <table className="table table-zebra">
          {/* head */}
          <thead>
            <tr>
              <th>Name</th>
              <th>Address</th>
              <th>Products</th>
              <th>Cost</th>
              <th className="hidden sm:block">Date</th>
            </tr>
          </thead>
          <tbody>
            {orders.map((order) => {
              const id = order.id
              const { name, address, numItemsInCart, orderTotal, createdAt } =
                order.attributes

              const date = day(createdAt).format('hh:mm a - MMM Do, YYYY ')
              return (
                <tr key={id}>
                  <td>{name}</td>
                  <td>{address}</td>
                  <td>{numItemsInCart}</td>
                  <td>{orderTotal}</td>
                  <td className="hidden sm:block">{date}</td>
                </tr>
              )
            })}
          </tbody>
        </table>
      </div>
    </div>
  )
}
export default OrdersList
```

## Challenge (54) - Complex Pagination

- create ComplexPaginationContainer.jsx
- render in Orders.jsx

### ComplexPaginationContainer.jsx

- Import Dependencies:

  - Import `useLoaderData`, `useLocation`, and `useNavigate` from `'react-router-dom'`.

- Create the `ComplexPaginationContainer` component:

  - Inside the component:
    - Use the `useLoaderData()` hook to get data from loader data.
    - Destructure `meta.pagination` to get `pageCount` and `page`.
    - Use the `useLocation()` hook to get the current location's search and pathname.
    - Use the `useNavigate()` hook to get the navigation function.
    - Create a `handlePageChange` function that:
      - Constructs a new URLSearchParams from the current search.
      - Sets the 'page' parameter in the search to the specified `pageNumber`.
      - Uses the navigate function to change the URL with the updated search.
    - Create an `addPageButton` function that returns a button element with the appropriate classes and onClick handler.
    - Create a `renderPageButtons` function that generates an array of page buttons, including ellipses and the current page button.
    - Use conditional checks to handle rendering of first, last, and ellipsis buttons.
    - Return `null` if `pageCount` is less than 2.
    - Return the pagination interface containing the "Prev," page buttons, and "Next" buttons.

- Export the `ComplexPaginationContainer` component.

## Solution (54) - Complex Pagination

ComplexPaginationContainer.jsx

```js
import { useLoaderData, useLocation, useNavigate } from 'react-router-dom'

const ComplexPaginationContainer = () => {
  const { meta } = useLoaderData()
  const { pageCount, page } = meta.pagination

  const { search, pathname } = useLocation()
  const navigate = useNavigate()
  const handlePageChange = (pageNumber) => {
    const searchParams = new URLSearchParams(search)
    searchParams.set('page', pageNumber)
    navigate(`${pathname}?${searchParams.toString()}`)
  }

  const addPageButton = ({ pageNumber, activeClass }) => {
    return (
      <button
        key={pageNumber}
        onClick={() => handlePageChange(pageNumber)}
        className={`btn btn-xs sm:btn-md border-none join-item ${
          activeClass ? 'bg-base-300 border-base-300 ' : ''
        }`}
      >
        {pageNumber}
      </button>
    )
  }

  const renderPageButtons = () => {
    const pageButtons = []
    // first button
    pageButtons.push(addPageButton({ pageNumber: 1, activeClass: page === 1 }))

    // dots
    if (page > 2) {
      pageButtons.push(
        <button className="join-item btn btn-xs sm:btn-md" key="dots-1">
          ...
        </button>
      )
    }

    // active/current page
    if (page !== 1 && page !== pageCount) {
      pageButtons.push(addPageButton({ pageNumber: page, activeClass: true }))
    }
    // dots
    if (page < pageCount - 1) {
      pageButtons.push(
        <button className="join-item btn btn-xs sm:btn-md" key="dots-2">
          ...
        </button>
      )
    }

    // last button
    pageButtons.push(
      addPageButton({ pageNumber: pageCount, activeClass: page === pageCount })
    )
    return pageButtons
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
        {renderPageButtons()}
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
export default ComplexPaginationContainer
```
