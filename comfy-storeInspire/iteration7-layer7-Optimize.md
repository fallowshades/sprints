## Challenge (55) - Setup React Query

- import and setup react query in App.jsx
- pass query client down to
  - Landing Page
  - SingleProduct Page
  - Products Page
- refactor loaders

## Solution (55) - Setup React Query

App.jsx

```js
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,
    },
  },
})

const router = createBrowserRouter([
  {
    path: '/',
    element: <HomeLayout />,
    errorElement: <Error />,
    children: [
      {
        index: true,
        element: <Landing />,
        loader: landingLoader(queryClient),
        errorElement: <ErrorElement />,
      },
      {
        path: 'products',
        element: <Products />,
        loader: productsLoader(queryClient),
        errorElement: <ErrorElement />,
      },
      {
        path: 'products/:id',
        element: <SingleProduct />,
        loader: singleProductLoader(queryClient),
        errorElement: <ErrorElement />,
      },
      {
        path: 'checkout',
        element: <Checkout />,
        loader: checkoutLoader(store),
        action: checkoutAction(store, queryClient),
      },
      {
        path: 'orders',
        element: <Orders />,
        loader: ordersLoader(store, queryClient),
      },
    ],
  },
])

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
export default App
```

Landing.js

```js
export const loader = (queryClient) => async () => {
  const response = await customFetch(url)
  const products = response.data.data
  return { products }
}
```

## Challenge (56) - Landing

- setup react query and invoke in loader

## Solution (56) - Landing

Landing.jsx

```js
const featuredProductsQuery = {
  queryKey: ['featuredProducts'],
  queryFn: () => customFetch(url),
}

export const loader = (queryClient) => async () => {
  const response = await queryClient.ensureQueryData(featuredProductsQuery)
  const products = response.data.data
  return { products }
}
```

## Challenge (57) - Single Product

- setup react query and invoke in loader

## Solution (57) - Single Product

SingleProduct.jsx

```js
const singleProductQuery = (id) => {
  return {
    queryKey: ['singleProduct', id],
    queryFn: () => customFetch.get(`/products/${id}`),
  }
}D

export const loader =
  (queryClient) =>
  async ({ params }) => {
    const response = await queryClient.ensureQueryData(
      singleProductQuery(params.id)
    )
    return { product: response.data.data }
  }
```

## Challenge (58) - All Products

- setup react query and invoke in loader

## Solution (58) - All Products

Products.jsx

```js
const allProductsQuery = (queryParams) => {
  const { search, category, company, sort, price, shipping, page } = queryParams

  return {
    queryKey: [
      'products',
      search ?? '',
      category ?? 'all',
      company ?? 'all',
      sort ?? 'a-z',
      price ?? 100000,
      shipping ?? false,
      page ?? 1,
    ],
    queryFn: () =>
      customFetch(url, {
        params: queryParams,
      }),
  }
}

export const loader =
  (queryClient) =>
  async ({ request }) => {
    const params = Object.fromEntries([
      ...new URL(request.url).searchParams.entries(),
    ])
    const response = await queryClient.ensureQueryData(allProductsQuery(params))

    const products = response.data.data
    const meta = response.data.meta

    return { products, meta, params }
  }
```

?? === This operator is known as the nullish coalescing operator in JavaScript. It is a logical operator that returns its right-hand side operand when its left-hand side operand is null or undefined, and otherwise returns its left-hand side operand.

In simpler terms, the ?? operator is used to provide a default value for potentially null or undefined variables.

## Challenge (59) - Orders

setup react query and invoke in loader

## Solution (59) - Orders

```js
import { redirect, useLoaderData } from 'react-router-dom'
import { toast } from 'react-toastify'
import { customFetch } from '../utils'
import {
  OrdersList,
  ComplexPaginationContainer,
  SectionTitle,
} from '../components'

export const ordersQuery = (params, user) => {
  return {
    queryKey: [
      'orders',
      user.username,
      params.page ? parseInt(params.page) : 1,
    ],
    queryFn: () =>
      customFetch.get('/orders', {
        params,
        headers: {
          Authorization: `Bearer ${user.token}`,
        },
      }),
  }
}

export const loader =
  (store, queryClient) =>
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
      const response = await queryClient.ensureQueryData(
        ordersQuery(params, user)
      )

      return {
        orders: response.data.data,
        meta: response.data.meta,
      }
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
  const { meta } = useLoaderData()

  if (meta.pagination.total < 1) {
    return <SectionTitle text="Please make an order" />
  }
  return (
    <>
      <SectionTitle text="Your Orders" />
      <OrdersList />
      <ComplexPaginationContainer />
    </>
  )
}
export default Orders
```

## Challenge (60) - Remove Queries

- remove "orders" query in CheckoutForm and Header

## Solution (60) - Remove Queries

CheckoutForm.jsx

```js
import { Form, redirect } from 'react-router-dom';
import FormInput from './FormInput';
import SubmitBtn from './SubmitBtn';
import { customFetch, formatPrice } from '../utils';
import { toast } from 'react-toastify';
import { clearCart } from '../features/cart/cartSlice';

export const action =
  (store, queryClient) =>
  async ({ request }) => {
    ...
    try {
      const response = await customFetch.post(
        '/orders',
        { data: info },
        {
          headers: {
            Authorization: `Bearer ${user.token}`,
          },
        }
      );
      // remove query
      queryClient.removeQueries(['orders']);
      // rest of the code
      store.dispatch(clearCart());
      toast.success('order placed successfully');
      return redirect('/orders');
    } ...
  };
```

Header.jsx

```js

import { useQueryClient } from '@tanstack/react-query';
const Header = () => {
  const navigate = useNavigate();
  const dispatch = useDispatch();
  const user = useSelector((state) => state.userState.user);
  const queryClient = useQueryClient();
  const handleLogout = async () => {
    navigate('/');
    dispatch(logoutUser());
    dispatch(clearCart());
    queryClient.removeQueries();
  };
  ...
}

```
