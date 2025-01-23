## Challenge (47) - Checkout Page Setup

- create CheckoutForm component

### Checkout.jsx

- Import Dependencies:

  - Import `useSelector` from `'react-redux'`.
  - Import `CheckoutForm`, `SectionTitle`, and `CartTotals` from `'../components'`.

- Create the `Checkout` component:

  - Inside the component, use `useSelector` to access the `cartTotal` from the Redux store.
  - Check if the `cartTotal` is empty.
  - If the `cartTotal` is empty, return a `SectionTitle` component with the text 'Your cart is empty'.
  - If the `cartTotal` is not empty:
    - Return a `SectionTitle` component with the text 'Place your order'.
    - Render a `<div>` element with the class name 'mt-8 grid gap-8 md:grid-cols-2 items-start'.
    - Inside the `<div>`, render the `CheckoutForm` component and the `CartTotals` component.

- Export the `Checkout` component as the default export.

## Solution (47) - Checkout Page Setup

Checkout.jsx

```js
import { useSelector } from 'react-redux'
import { CheckoutForm, SectionTitle, CartTotals } from '../components'

//  const cartItems = useSelector((state) => state.cartState)

if (cartItems.cartTotal === 0) {
  const Checkout = () => {
    const cartItems = useSelector((state) => state.cartState.cartTotal)
    if (cartTotal.length === 0) {
      return <SectionTitle text="Your cart is empty" />
    }
  }
  return (
    <>
      <SectionTitle text="Place your order" />
      <div className="mt-8 grid gap-8  md:grid-cols-2 items-start">
        <CheckoutForm />
        <CartTotals />
      </div>
    </>
  )
}
export default Checkout
```

## Challenge (48) - Restrict Access

App.jsx

- in App.jsx import loader from Checkout page
- pass store into the checkoutLoader
- if no user redirect to login

### Checkout.jsx

- Import Dependencies:

  - Import `redirect` from `'react-router-dom'`.
  - Import `toast` from `'react-toastify'`.

- Create a `loader` function:

  - The `loader` function takes a `store` as a parameter.
  - Inside the `loader` function:
    - Get the `user` from the Redux store using `store.getState().userState.user`.
    - Check if the `user` is falsy (not logged in).
    - If the `user` is falsy:
      - Display a toast warning message using `toast.warn()` with the text 'You must be logged in to checkout'.
      - Return `redirect('/login')` to redirect the user to the login page.
    - If the `user` is truthy (logged in):
      - Return `null`.

- Export the `loader` function.

## Solution (48) - Restrict Access

App.jsx

```js
import { loader as checkoutLoader } from './pages/Checkout';

import { store } from './store';

const router = createBrowserRouter([
  {
   ....
      {
        path: 'checkout',
        element: <Checkout />,
        loader: checkoutLoader(store),

      },

  },

]);

```

Checkout.jsx

```js
import { redirect } from 'react-router-dom'
import { toast } from 'react-toastify'

export const loader = (store) => async () => {
  const user = store.getState().userState.user

  if (!user) {
    toast.warn('You must be logged in to checkout')
    return redirect('/login')
  }
  return null
}
```

## Challenge (49) - CheckoutForm

- [API DOCS](https://documenter.getpostman.com/view/18152321/2s9Xy5KpTi)
- docs - create order request
- test in Thunder Client

### App.jsx

- in App.jsx import action from CheckoutForm.jsx
- pass store into the checkoutAction

## CheckoutForm.jsx

- Import Dependencies:

  - Import `Form` and `redirect` from `'react-router-dom'`.
  - Import `FormInput` and `SubmitBtn` from appropriate paths.
  - Import other required utilities and actions.

- Create an `action` function:

  - The `action` function takes a `store` as a parameter and returns an asynchronous function that takes a `request` parameter.
  - Inside the async function:
    - Await `request.formData()` to get form data.
    - Destructure the `name` and `address` properties from the form data using `Object.fromEntries(formData)`.
    - Get the `user` from the Redux store using `store.getState().userState.user`.
    - Get the `cartItems`, `orderTotal`, and `numItemsInCart` from the Redux store using `store.getState().cartState`.
    - Create an `info` object containing the gathered information.
    - Try to make a POST request to '/orders' with the `info` data and the user's token in the headers.
    - If successful:
      - Dispatch the `clearCart()` action using `store.dispatch(clearCart())` to clear the cart.
      - Display a success toast message using `toast.success()` with the text 'order placed successfully'.
      - Return `redirect('/orders')` to redirect the user to the orders page.
    - If there's an error:
      - Log the error.
      - Get the error message from the response data or provide a default message.
      - Display an error toast message using `toast.error()` with the error message.
      - Return `null`.

- Create a `CheckoutForm` component:

  - Inside the component:
    - Use the `Form` component from 'react-router-dom' to create a form.
    - Display a heading for the shipping information.
    - Use the `FormInput` component to create input fields for the first name and address.
    - Use the `SubmitBtn` component to create a submit button with the text 'Place Your Order'.

- Export the `CheckoutForm` component.

## Solution (49) - CheckoutForm

App.jsx

```js
import { action as checkoutAction } from './components/CheckoutForm'
import { store } from './store'

const router = createBrowserRouter([
  {
    path: '/',
    element: <HomeLayout />,
    errorElement: <Error />,
    children: [
      {
        path: 'checkout',
        element: <Checkout />,
        loader: checkoutLoader(store),
        action: checkoutAction(store),
      },
    ],
  },
])
```

CheckoutForm.jsx

```js
import { Form, redirect } from 'react-router-dom'
import FormInput from './FormInput'
import SubmitBtn from './SubmitBtn'
import { customFetch, formatPrice } from '../utils'
import { toast } from 'react-toastify'
import { clearCart } from '../features/cart/cartSlice'

export const action =
  (store) =>
  async ({ request }) => {
    const formData = await request.formData()
    const { name, address } = Object.fromEntries(formData)
    const user = store.getState().userState.user
    const { cartItems, orderTotal, numItemsInCart } = store.getState().cartState

    const info = {
      name,
      address,
      chargeTotal: orderTotal,
      orderTotal: formatPrice(orderTotal),
      cartItems,
      numItemsInCart,
    }
    try {
      const response = await customFetch.post(
        '/orders',
        { data: info },
        {
          headers: {
            Authorization: `Bearer ${user.token}`,
          },
        }
      )
      store.dispatch(clearCart())
      toast.success('order placed successfully')
      return redirect('/orders')
    } catch (error) {
      console.log(error)
      const errorMessage =
        error?.response?.data?.error?.message ||
        'there was an error placing your order'

      toast.error(errorMessage)
      return null
    }
  }
const CheckoutForm = () => {
  return (
    <Form method="POST" className="flex flex-col gap-y-4">
      <h4 className="font-medium text-xl">Shipping Information</h4>
      <FormInput label="first name" name="name" type="text" />
      <FormInput label="address" name="address" type="text" />
      <div className="mt-4">
        <SubmitBtn text="Place Your Order" />
      </div>
    </Form>
  )
}
export default CheckoutForm
```

## Challenge (50) - Auth Error

- handle auth errors
- check for response.status
  - if status === 401 redirect to login

## Solution (50) - Auth Error

CheckoutForm.jsx

```js

 catch (error) {
  console.log(error);
  const errorMessage =
    error?.response?.data?.error?.message ||
    'there was an error placing your order';
  toast.error(errorMessage);
  if (error?.response?.status === 401 || 403) return redirect('/login');

  return null;
}
```
