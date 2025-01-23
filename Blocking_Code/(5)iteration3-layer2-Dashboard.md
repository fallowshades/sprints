## Global context navigate resources

### (1) Two layers shared/not connec tidentified response to subset

#### Nested Pages in React Router 6

#### Dashboard pages

- delete Dashboard.js
- fix imports/exports
- replace in home route

```js
<Route path="/" element={<div>dashboard</div>} />
```

- create <b>dashboard</b> directory in pages
- create AddJob,AllJobs,Profile,Stats,SharedLayout, index.js
- setup basic returns

```js
return <h1>Add Job Page</h1>
```

- export all with index.js (just like components)
- import all pages in App.js

#### Nested Structure

```js
App.js

<Route path='/' >
  <Route path="stats" element={<Stats />} />
  <Route path='all-jobs' element={<AllJobs />}></Route>
  <Route path='add-job' element={<AddJob />}></Route>
  <Route path='profile' element={<Profile />}></Route>
</Route>
```

#### Shared Layout

```js
App.js

<Route path='/' element={<SharedLayout/>} >
```

```js
SharedLayout.js

import { Outlet, Link } from 'react-router-dom'
import Wrapper from '../../assets/wrappers/SharedLayout'

const SharedLayout = () => {
  return (
    <Wrapper>
      <nav>
        <Link to="all-jobs">all jobs</Link>
        <Link to="add-job">all jobs</Link>
      </nav>
      <Outlet />
    </Wrapper>
  )
}

export default SharedLayout
```

```js
App.js

<Route index element={<Stats/>} >
```

#### Protected Route

- create ProtectedRoute.js in pages
- import/export
- wrap SharedLayout in App.js

```js
<Route
  path="/"
  element={
    <ProtectedRoute>
      <SharedLayout />
    </ProtectedRoute>
  }
/>
```

```js
ProtectedRoute.js

import { Navigate } from 'react-router-dom'
import { useAppContext } from '../context/appContext'

const ProtectedRoute = ({ children }) => {
  const { user } = useAppContext()
  if (!user) {
    return <Navigate to="/landing" />
  }
  return children
}
```

### (2) Add all components aiding user to navigate

#### Navbar, SmallSidebar, BigSidebar

- create Navbar, SmallSidebar, BigSidebar in components
- import Wrappers from assets/wrappers
- simple return
- import/export

```js
SharedLayout.js

import { Outlet } from 'react-router-dom'
import { Navbar, SmallSidebar, BigSidebar } from '../../components'
import Wrapper from '../../assets/wrappers/SharedLayout'

const SharedLayout = () => {
  const { user } = useAppContext()
  return (
    <>
      <Wrapper>
        <main className="dashboard">
          <SmallSidebar />
          <BigSidebar />
          <div>
            <Navbar />
            <div className="dashboard-page">
              <Outlet />
            </div>
          </div>
        </main>
      </Wrapper>
    </>
  )
}

export default SharedLayout
```

#### React Icons

[React Icons](https://react-icons.github.io/react-icons/)

```sh
npm install react-icons
```

```js
Navbar.js

import Wrapper from '../assets/wrappers/Navbar'
import {FaHome} from 'react-icons/fa'
const Navbar = () => {
  return (
    <Wrapper>
      <h4>navbar</h4>
      <FaHome>
    </Wrapper>
  )
}

export default Navbar

```

#### Navbar Setup

```js
Navbar.js

import { useState } from 'react'
import { FaAlignLeft, FaUserCircle, FaCaretDown } from 'react-icons/fa'
import { useAppContext } from '../context/appContext'
import Logo from './Logo'
import Wrapper from '../assets/wrappers/Navbar'
const Navbar = () => {
  return (
    <Wrapper>
      <div className="nav-center">
        <button
          className="toggle-btn"
          onClick={() => console.log('toggle sidebar')}
        >
          <FaAlignLeft />
        </button>

        <div>
          <Logo />
          <h3 className="logo-text">dashboard</h3>
        </div>

        <div className="btn-container">
          <button className="btn" onClick={() => console.log('show logout')}>
            <FaUserCircle />
            john
            <FaCaretDown />
          </button>
          <div className="dropdown show-dropdown">
            <button
              onClick={() => console.log('logout user')}
              className="dropdown-btn"
            >
              logout
            </button>
          </div>
        </div>
      </div>
    </Wrapper>
  )
}

export default Navbar
```

#### Toggle Sidebar

```js
actions.js

export const TOGGLE_SIDEBAR = 'TOGGLE_SIDEBAR'
```

- import/export

```js
appContext.js

const initialState = {
  showSidebar: false,
}

const toggleSidebar = () => {
  dispatch({ type: TOGGLE_SIDEBAR })
}
```

```js
reducer.js

if (action.type === TOGGLE_SIDEBAR) {
  return { ...state, showSidebar: !state.showSidebar }
}
```

```js
Navbar.js

const { toggleSidebar } = useAppContext()

return (
  <button className="toggle-btn" onClick={toggleSidebar}>
    <FaAlignLeft />
  </button>
)
```

#### Toggle Dropdown

```js
Navbar.js

const [showLogout, setShowLogout] = useState(false)

<div className='btn-container'>
  <button className='btn' onClick={() => setShowLogout(!showLogout)}>
    <FaUserCircle />
      {user.name}
    <FaCaretDown />
  </button>
  <div className={showLogout ? 'dropdown show-dropdown' : 'dropdown'}>
    <button onClick={() => logoutUser()} className='dropdown-btn'>
      logout
    </button>
  </div>
</div>

```

#### Logout User

```js
actions.js

export const LOGOUT_USER = 'LOGOUT_USER'
```

- import/export

```js
appContext.js

const logoutUser = () => {
  dispatch({ type: LOGOUT_USER })
  removeUserFromLocalStorage()
}

value={{logoutUser}}
```

```js
reducer.js

import { initialState } from './appContext'

if (action.type === LOGOUT_USER) {
  return {
    ...initialState,
    user: null,
    token: null,
    userLocation: '',
    jobLocation: '',
  }
}
```

```js
Navbar.js

const { user, logoutUser, toggleSidebar } = useAppContext()

return (
  <div className="btn-container">
    <button className="btn" onClick={() => setShowLogout(!showLogout)}>
      <FaUserCircle />
      {user.name}
      {user && user.name}
      {user?.name} // optional chaining
      <FaCaretDown />
    </button>
    <div className={showLogout ? 'dropdown show-dropdown' : 'dropdown'}>
      <button onClick={logoutUser} className="dropdown-btn">
        logout
      </button>
    </div>
  </div>
)
```

#### Setup Links

- create <b>utils</b>in the <b>src</b>
- setup links.js

```js
import { IoBarChartSharp } from 'react-icons/io5'
import { MdQueryStats } from 'react-icons/md'
import { FaWpforms } from 'react-icons/fa'
import { ImProfile } from 'react-icons/im'

const links = [
  {
    id: 1,
    text: 'stats',
    path: '/',
    icon: <IoBarChartSharp />,
  },
  {
    id: 2,
    text: 'all jobs',
    path: 'all-jobs',
    icon: <MdQueryStats />,
  },
  {
    id: 3,
    text: 'add job',
    path: 'add-job',
    icon: <FaWpforms />,
  },
  {
    id: 4,
    text: 'profile',
    path: 'profile',
    icon: <ImProfile />,
  },
]

export default links
```

#### Small Sidebar - Setup

```js
SmallSidebar.js

import Wrapper from '../assets/wrappers/SmallSidebar'
import { FaTimes } from 'react-icons/fa'
import { useAppContext } from '../context/appContext'
import links from '../utils/links'
import { NavLink } from 'react-router-dom'
import Logo from './Logo'

export const SmallSidebar = () => {
  return (
    <Wrapper>
      <div className="sidebar-container show-sidebar">
        <div className="content">
          <button className="close-btn" onClick={() => console.log('toggle')}>
            <FaTimes />
          </button>
          <header>
            <Logo />
          </header>
          <div className="nav-links">nav links</div>
        </div>
      </div>
    </Wrapper>
  )
}

export default SmallSidebar
```

#### Small Sidebar - Toggle

```js
SmallSidebar.js

const { showSidebar, toggleSidebar } = useAppContext()
```

```js
SmallSidebar.js

return (
  <div
    className={
      showSidebar ? 'sidebar-container show-sidebar' : 'sidebar-container'
    }
  ></div>
)
```

```js
SmallSidebar.js

return (
  <button className="close-btn" onClick={toggleSidebar}>
    <FaTimes />
  </button>
)
```

#### Small Sidebar - Nav Links

```js
SmallSidebar.js

import { NavLink } from 'react-router-dom'

return (
  <div className="nav-links">
    {links.map((link) => {
      const { text, path, id, icon } = link

      return (
        <NavLink
          to={path}
          className={({ isActive }) =>
            isActive ? 'nav-link active' : 'nav-link'
          }
          key={id}
          onClick={toggleSidebar}
        >
          <span className="icon">{icon}</span>
          {text}
        </NavLink>
      )
    })}
  </div>
)
```

#### Nav Links Component

- in <b>components</b> create NavLinks.js
- styles still set from Wrapper
- also can setup in links.js, preference

```js
import { NavLink } from 'react-router-dom'
import links from '../utils/links'

const NavLinks = ({ toggleSidebar }) => {
  return (
    <div className="nav-links">
      {links.map((link) => {
        const { text, path, id, icon } = link

        return (
          <NavLink
            to={path}
            key={id}
            onClick={toggleSidebar}
            className={({ isActive }) =>
              isActive ? 'nav-link active' : 'nav-link'
            }
          >
            <span className="icon">{icon}</span>
            {text}
          </NavLink>
        )
      })}
    </div>
  )
}

export default NavLinks
```

```js
SmallSidebar.js

import NavLinks from './NavLinks'

return <NavLinks toggleSidebar={toggleSidebar}>
```

#### Big Sidebar

```js
import { useAppContext } from '../context/appContext'
import NavLinks from './NavLinks'
import Logo from '../components/Logo'
import Wrapper from '../assets/wrappers/BigSidebar'

const BigSidebar = () => {
  const { showSidebar } = useAppContext()
  return (
    <Wrapper>
      <div
        className={
          showSidebar ? 'sidebar-container ' : 'sidebar-container show-sidebar'
        }
      >
        <div className="content">
          <header>
            <Logo />
          </header>
          <NavLinks />
        </div>
      </div>
    </Wrapper>
  )
}

export default BigSidebar
```

#### REACT ROUTER UPDATE !!!

- [Stack Overflow](https://stackoverflow.com/questions/70644361/react-router-dom-v6-shows-active-for-index-as-well-as-other-subroutes)

```js
<NavLink
to={path}
key={id}
onClick={toggleSidebar}
className={({ isActive }) =>
isActive ? 'nav-link active' : 'nav-link'}


end
>
```
