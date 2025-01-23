# l

## ll

### Load with lazy axios .data.data approach and hero

#### Challenge (19) - Landing Loader

- setup ErrorElement
- add to Loading Page
- setup a loader
- fetch only featured products
- return products

##### ErrorElement.jsx

1. Create ErrorElement Component:

   - Define a functional component named `ErrorElement`.

2. Import Dependencies:

   - Import the `useRouteError` hook from `'react-router-dom'`.

3. Get Route Error:

   - Inside the component, use the `useRouteError` hook to retrieve the error information from the current route.

4. Display Error Message:

   - Return an `h4` element with the classes `font-bold` and `text-4xl`.
   - Set the content of the `h4` element to "there was an error..."

5. Export ErrorElement Component:
   - Export the `ErrorElement` component as the default export of the module.

#### Solution (19) - Landing Loader

ErrorElement.jsx

```js
import { useRouteError } from 'react-router-dom'
const ErrorElement = () => {
  const error = useRouteError()
  console.log(error)

  return <h4 className="font-bold text-4xl">there was an error... </h4>
}
export default ErrorElement
```

App.jsx

```js
import { ErrorElement } from './components'
// loaders
import { loader as landingLoader } from './pages/Landing'
// actions

const router = createBrowserRouter([
  {
    path: '/',
    element: <HomeLayout />,
    errorElement: <Error />,
    children: [
      {
        index: true,
        element: <Landing />,
        loader: landingLoader,
        errorElement: ErrorElement,
      },
    ],
  },
])
```

Landing.js

```js
import { Hero } from '../components'

import { customFetch } from '../utils'
const url = '/products?featured=true'

export const loader = async () => {
  const response = await customFetch(url)
  console.log(response)
  const products = response.data.data
  return { products }
}

const Landing = () => {
  return (
    <>
      <Hero />
    </>
  )
}
export default Landing
```
