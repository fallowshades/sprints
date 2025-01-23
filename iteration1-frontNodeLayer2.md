# Hamoria

## front layer 1

## Data context supported n-way communication for its children

### Main section simple form flow (verticaly scaled)

#### Router

[React Router](https://reactrouter.com/en/main)

- version 6.4 brought significant changes (loader and action)
- pages as independent entities
- less need for global state
- more pages

#### 1. Setup Router

- all my examples will include version !!!

```sh
npm i react-router-dom@6.10.0
```

App.jsx

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom'

const router = createBrowserRouter([
  {
    path: '/',
    element: <h1>home</h1>,
  },
  {
    path: '/about',
    element: (
      <div>
        <h2>about page</h2>
      </div>
    ),
  },
])

const App = () => {
  return <RouterProvider router={router} />
}
export default App
```

#### 2. Create Pages

- create src/pages directory
- setup index.js and following pages :

---bind to user/role
AddAchievements.jsx
AllAchievements.jsx
Stats.jsx
Admin.jsx

--dynamic rules
DeleteAchievement.jsx
EditAchievement.jsx

---container of containers (who --> what)
DashboardLayout.jsx
Error.jsx

--How to add privilage(move to privilaged layer)
HomeLayout.jsx
Landing.jsx
Login.jsx
Profile.jsx
Register.jsx

```jsx
const AddJob = () => {
  return <h1>AddJob</h1>
}
export default AddJob
```

#### 3. Index

App.jsx

```jsx
import HomeLayout from '../ pages/HomeLayout'
```

pages/index.js

```js
export { default as DashboardLayout } from './DashboardLayout'
export { default as Landing } from './Landing'
export { default as HomeLayout } from './HomeLayout'
export { default as Register } from './Register'
export { default as Login } from './Login'
export { default as Error } from './Error'
export { default as Stats } from './Stats'
export { default as AllAchievements } from './AllAchievements'
export { default as AddAchievement } from './AddAchievement'
export { default as EditAchievement } from './EditAchievement'
export { default as Profile } from './Profile'
export { default as Admin } from './Admin'
```

App.jsx

```jsx
import {
  HomeLayout,
  Landing,
  Register,
  Login,
  DashboardLayout,
  Error,
} from './pages'

const router = createBrowserRouter([
  {
    path: '/',
    element: <HomeLayout />,
  },
  {
    path: '/register',
    element: <Register />,
  },
  {
    path: '/login',
    element: <Login />,
  },
  {
    path: '/dashboard',
    element: <DashboardLayout />,
  },
])
```

### interior points

#### 4. Link Component

- navigate around project
- client side routing

Register.jsx

```jsx
import { Link } from 'react-router-dom'

const Register = () => {
  return (
    <div>
      <h1>Register</h1>
      <Link to="/login">Login Page</Link>
    </div>
  )
}
export default Register
```

Login.jsx

```jsx
import { Link } from 'react-router-dom'

const Login = () => {
  return (
    <div>
      <h1>Login</h1>
      <Link to="/register">Register Page</Link>
    </div>
  )
}
export default Login
```

#### 5. Nested Routes

- what about Navbar?
- decide on root (parent route)
- make path relative !!!(refracture link reference)!!!
- for time being only home layout will be visible

App.jsx

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <HomeLayout />,
    children: [
      {
        path: 'register',
        element: <Register />,
      },
      {
        path: 'login',
        element: <Login />,
      },
      {
        path: 'dashboard',
        element: <DashboardLayout />,
      },
    ],
  },
])
```

HomeLayout.jsx

```jsx
import { Outlet } from 'react-router-dom'

const HomeLayout = () => {
  return (
    <>
      {/* add things like Navbar */}
      {/* <h1>home layout</h1> */}
      <Outlet />
    </>
  )
}
export default HomeLayout
```

#### 6. Index (Home) Page

App.jsx

```jsx
{
    path: '/',
    element: <HomeLayout />,
    children: [
      {
        index: true,
        element: <Landing />,
      },
...
      ]
}
```

### Error bubble up homeLayout (horizontal, verticle, shared, index)

#### 7. Error Page

- bubbles up

App.jsx

```jsx
{
    path: '/',
    element: <HomeLayout />,
    errorElement: <Error />,
    ...
}
```

Error.jsx

```jsx
import { Link, useRouteError } from 'react-router-dom'

const Error = () => {
  const error = useRouteError()
  console.log(error)
  return (
    <div>
      <h1>Error Page !!!</h1>
      <Link to="/dashboard">back home</Link>
    </div>
  )
}
export default Error
```

#### 14. Error Page

Error.jsx

```jsx
import { Link, useRouteError } from 'react-router-dom'
import img from '../assets/images/not-found.svg'
import Wrapper from '../assets/wrappers/ErrorPage'

const Error = () => {
  const error = useRouteError()
  console.log(error)
  if (error.status === 404) {
    return (
      <Wrapper>
        <div>
          <img src={img} alt="not found" />
          <h3>Ohh! page not found</h3>
          <p>We can't seem to find the page you're looking for</p>
          <Link to="/dashboard">back home</Link>
        </div>
      </Wrapper>
    )
  }
  return (
    <Wrapper>
      <div>
        <h3>something went wrong</h3>
      </div>
    </Wrapper>
  )
}

export default Error
```

#### 15. Error Page CSS (optional)

assets/wrappers/Error.js

```js
import styled from 'styled-components'

const Wrapper = styled.main`
  min-height: 100vh;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  img {
    width: 90vw;
    max-width: 600px;
    display: block;
    margin-bottom: 2rem;
    margin-top: -3rem;
  }
  h3 {
    margin-bottom: 0.5rem;
  }
  p {
    line-height: 1.5;
    margin-top: 0.5rem;
    margin-bottom: 1rem;
    color: var(--text-secondary-color);
  }
  a {
    color: var(--primary-500);
    text-transform: capitalize;
  }
`

export default Wrapper
```

## shared responsibillities for parent node

### Navigate --> 1 active node in array of children

#### 8. Styled Components

- CSS in JS
- Styled Components
- have logic and styles in component
- no name collisions
- apply javascript logic
- [Styled Components Docs](https://styled-components.com/)
- [Styled Components Course](https://www.udemy.com/course/styled-components-tutorial-and-project-course/?referralCode=9DABB172FCB2625B663F)

```sh
npm install styled-components@5.3.10
```

```js
import styled from 'styled-components'

const El = styled.el`
  // styles go here
`
```

- no name collisions, since unique class
- vscode-styled-components extension
- colors and bugs

Landing.jsx

```jsx
import styled from 'styled-components'

const Landing = () => {
  return (
    <div>
      <h1>Landing</h1>
      <StyledButton>Click Me</StyledButton>
    </div>
  )
}

const StyledButton = styled.button`
  background-color: red;
  color: white;
`
export default Landing
```

#### 9. Style Entire React Component

```js
const Wrapper = styled.el``

const Component = () => {
  return (
    <Wrapper>
      <h1> Component</h1>
    </Wrapper>
  )
}
```

- only responsible for styling
- wrappers folder in assets

Landing.jsx

```jsx
import styled from 'styled-components'

const Landing = () => {
  return (
    <Wrapper>
      <h1>Landing</h1>
      <div className="content">some content</div>
    </Wrapper>
  )
}

const Wrapper = styled.div`
  background-color: red;
  h1 {
    color: white;
  }
  .content {
    background-color: blue;
    color: yellow;
  }
`
export default Landing
```

#### 10 Landing Page

```jsx
import main from '../assets/images/main.svg'
import { Link } from 'react-router-dom'
import logo from '../assets/images/logo.svg'
import styled from 'styled-components'
const Landing = () => {
  return (
    <StyledWrapper>
      <nav>
        <img src={logo} alt="jobify" className="logo" />
      </nav>
      <div className="container page">
        {/* info */}
        <div className="info">
          <h1>
            job <span>tracking</span> app
          </h1>
          <p>
            I'm baby wayfarers hoodie next level taiyaki brooklyn cliche blue
            bottle single-origin coffee chia. Aesthetic post-ironic venmo,
            quinoa lo-fi tote bag adaptogen everyday carry meggings +1 brunch
            narwhal.
          </p>
          <Link to="/register" className="btn register-link">
            Register
          </Link>
          <Link to="/login" className="btn">
            Login / Demo User
          </Link>
        </div>
        <img src={main} alt="job hunt" className="img main-img" />
      </div>
    </StyledWrapper>
  )
}

const StyledWrapper = styled.section`
  nav {
    width: var(--fluid-width);
    max-width: var(--max-width);
    margin: 0 auto;
    height: var(--nav-height);
    display: flex;
    align-items: center;
  }
  .page {
    min-height: calc(100vh - var(--nav-height));
    display: grid;
    align-items: center;
    margin-top: -3rem;
  }
  h1 {
    font-weight: 700;
    span {
      color: var(--primary-500);
    }
    margin-bottom: 1.5rem;
  }
  p {
    line-height: 2;
    color: var(--text-secondary-color);
    margin-bottom: 1.5rem;
    max-width: 35em;
  }
  .register-link {
    margin-right: 1rem;
  }
  .main-img {
    display: none;
  }
  .btn {
    padding: 0.75rem 1rem;
  }
  @media (min-width: 992px) {
    .page {
      grid-template-columns: 1fr 400px;
      column-gap: 3rem;
    }
    .main-img {
      display: block;
    }
  }
`

export default Landing
```

#### 11. Assets/Wrappers

- css optional

  Landing.jsx

```jsx
import Wrapper from '../assets/wrappers/LandingPage'
```

#### 12. Logo Component

- create src/components/Logo.jsx
- import logo and setup component
- in components setup index.js import/export (just like pages)
- replace in Landing

  Logo.jsx

```jsx
import logo from '../assets/images/logo.svg'

const Logo = () => {
  return <img src={logo} alt="jobify" className="logo" />
}

export default Logo
```

#### 13. Logo and Images

- logo built in Figma
- [Cool Images](https://undraw.co/)

### Navigation depend on user privilage

#### 16. Register Page

Register.jsx

```jsx
import { Logo } from '../components'
import Wrapper from '../assets/wrappers/RegisterAndLoginPage'
import { Link } from 'react-router-dom'

const Register = () => {
  return (
    <Wrapper>
      <form className="form">
        <Logo />
        <h4>Register</h4>
        <div className="form-row">
          <label htmlFor="name" className="form-label">
            name
          </label>
          <input
            type="text"
            id="name"
            name="name"
            className="form-input"
            defaultValue="john"
            required
          />
        </div>

        <button type="submit" className="btn btn-block">
          submit
        </button>
        <p>
          Already a member?
          <Link to="/login" className="member-btn">
            Login
          </Link>
        </p>
      </form>
    </Wrapper>
  )
}
export default Register
```

- required attribute

  In HTML, the "required" attribute is used to indicate that a form input field must be filled out before the form can be submitted. It is typically applied to input elements such as text fields, checkboxes, and radio buttons. When the "required" attribute is added to an input element, the browser will prevent form submission if the field is left empty, providing a validation message to prompt the user to enter the required information.

- default value

In React, the defaultValue prop is used to set the initial or default value of an input component. It is similar to the value attribute in HTML, but with a slightly different behavior.

#### 17. FormRow Component

- create components/FormRow.jsx (export/import)

FormRow.jsx

```jsx
const FormRow = ({ type, name, labelText, defaultValue = '' }) => {
  return (
    <div className="form-row">
      <label htmlFor={name} className="form-label">
        {labelText || name}
      </label>
      <input
        type={type}
        id={name}
        name={name}
        className="form-input"
        defaultValue={defaultValue}
        required
      />
    </div>
  )
}

export default FormRow
```

Register.jsx

```jsx
import { Logo, FormRow } from '../components'
import Wrapper from '../assets/wrappers/RegisterAndLoginPage'
import { Link } from 'react-router-dom'

const Register = () => {
  return (
    <Wrapper>
      <form className="form">
        <Logo />
        <h4>Register</h4>
        <FormRow type="text" name="name" />
        <FormRow type="text" name="lastName" labelText="last name" />
        <FormRow type="text" name="location" />
        <FormRow type="email" name="email" />

        <FormRow type="password" name="password" />

        <button type="submit" className="btn btn-block">
          submit
        </button>
        <p>
          Already a member?
          <Link to="/login" className="member-btn">
            Login
          </Link>
        </p>
      </form>
    </Wrapper>
  )
}
export default Register
```

#### 18. Login Page

Login Page

```jsx
import { Logo, FormRow } from '../components'
import Wrapper from '../assets/wrappers/RegisterAndLoginPage'

import { Link } from 'react-router-dom'

const Login = () => {
  return (
    <Wrapper>
      <form className="form">
        <Logo />
        <h4>Login</h4>
        <FormRow type="email" name="email" defaultValue="john@gmail.com" />
        <FormRow type="password" name="password" defaultValue="secret123" />
        <button type="submit" className="btn btn-block">
          submit
        </button>
        <button type="button" className="btn btn-block">
          explore the app
        </button>
        <p>
          Not a member yet?
          <Link to="/register" className="member-btn">
            Register
          </Link>
        </p>
      </form>
    </Wrapper>
  )
}
export default Login
```

#### 19. Register and Login CSS (optional)

assets/wrappers/RegisterAndLoginPage.js

```js
import styled from 'styled-components'

const Wrapper = styled.section`
  min-height: 100vh;
  display: grid;
  align-items: center;
  .logo {
    display: block;
    margin: 0 auto;
    margin-bottom: 1.38rem;
  }
  .form {
    max-width: 400px;
    border-top: 5px solid var(--primary-500);
  }

  h4 {
    text-align: center;
    margin-bottom: 1.38rem;
  }
  p {
    margin-top: 1rem;2. 
    text-align: center;
    line-height: 1.5;
  }
  .btn {
    margin-top: 1rem;
  }
  .member-btn {
    color: var(--primary-500);
    letter-spacing: var(--letter-spacing);
    margin-left: 0.25rem;
  }
`
export default Wrapper
```
