# Second layer and beyond

## Partition logic of

### 1. Layout fixed/relatife 2 column setup

#### 1. Dashboard Pages

App.jsx

```jsx
 {
        path: 'dashboard',
        element: <DashboardLayout />,
        children: [
          {
            index: true,
            element: <Achievement />,
          },
          { path: 'stats', element: <Stats /> },
          {
            path: 'all-achievements',
            element: <AllAchievements />,
          },

          {
            path: 'profile',
            element: <Profile />,
          },
          {
            path: 'admin',
            element: <Admin />,
          },
        ],
      },
```

Dashboard.jsx

```jsx
import { Outlet } from 'react-router-dom'

const DashboardLayout = () => {
  return (
    <div>
      <Outlet />
    </div>
  )
}
export default DashboardLayout
```

#### 2. Navbar, BigSidebar and SmallSidebar

- in components create :
  Navbar.jsx
  BigSidebar.jsx
  SmallSidebar.jsx

DashboardLayout.jsx

```jsx
import { Outlet } from 'react-router-dom'

import Wrapper from '../assets/wrappers/Dashboard'
import { Navbar, BigSidebar, SmallSidebar } from '../components'

const Dashboard = () => {
  return (
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
  )
}

export default Dashboard
```

#### 3. Dashboard Layout - CSS (optional)

assets/wrappers/DashboardLayout.jsx

```js
import styled from 'styled-components'

const Wrapper = styled.section`
  .dashboard {
    display: grid;
    grid-template-columns: 1fr;
  }
  .dashboard-page {
    width: 90vw;
    margin: 0 auto;
    padding: 2rem 0;
  }
  @media (min-width: 992px) {
    .dashboard {
      grid-template-columns: auto 1fr;
    }
    .dashboard-page {
      width: 90%;
    }
  }
`
export default Wrapper
```

### 2. Main wrapper for (n-scalled prop / m- scalled nav)

#### .4 Dashboard Context

```jsx
import { Outlet } from 'react-router-dom'

import Wrapper from '../assets/wrappers/Dashboard'
import { Navbar, BigSidebar, SmallSidebar } from '../components'

import { useState, createContext, useContext } from 'react'
const DashboardContext = createContext()
const DashboardLayout = () => {
  // temp
  const user = { name: 'john' }

  const [showSidebar, setShowSidebar] = useState(false)
  const [isDarkTheme, setIsDarkTheme] = useState(false)

  const toggleDarkTheme = () => {
    console.log('toggle dark theme')
  }

  const toggleSidebar = () => {
    setShowSidebar(!showSidebar)
  }

  const logoutUser = async () => {
    console.log('logout user')
  }
  return (
    <DashboardContext.Provider
      value={{
        user,
        showSidebar,
        isDarkTheme,
        toggleDarkTheme,
        toggleSidebar,
        logoutUser,
      }}
    >
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
    </DashboardContext.Provider>
  )
}

export const useDashboardContext = () => useContext(DashboardContext)

export default Dashboard
```

#### .5 React Icons

[React Icons](https://react-icons.github.io/react-icons/)

```sh
npm install react-icons@4.8.0
```

Navbar.jsx

```jsx
import { FaHome } from 'react-icons/fa'
const Navbar = () => {
  return (
    <div>
      <h2>navbar</h2>
      <FaHome />
    </div>
  )
}
```

#### .6 Navbar - Initial Setup

```jsx
import Wrapper from '../assets/wrappers/Navbar'
import { FaAlignLeft } from 'react-icons/fa'
import Logo from './Logo'

import { useDashboardContext } from '../pages/DashboardLayout'
const Navbar = () => {
  const { toggleSidebar } = useDashboardContext()
  return (
    <Wrapper>
      <div className="nav-center">
        <button type="button" className="toggle-btn" onClick={toggleSidebar}>
          <FaAlignLeft />
        </button>
        <div>
          <Logo />
          <h4 className="logo-text">dashboard</h4>
        </div>
        <div className="btn-container">toggle/logout</div>
      </div>
    </Wrapper>
  )
}

export default Navbar
```

#### .7 Navbar CSS (optional)

assets/wrappers/Navbar.js

```js
import styled from 'styled-components'

const Wrapper = styled.nav`
  height: var(--nav-height);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 1px 0px 0px rgba(0, 0, 0, 0.1);
  background: var(--background-secondary-color);
  .logo {
    display: flex;
    align-items: center;
    width: 100px;
  }
  .nav-center {
    display: flex;
    width: 90vw;
    align-items: center;
    justify-content: space-between;
  }
  .toggle-btn {
    background: transparent;
    border-color: transparent;
    font-size: 1.75rem;
    color: var(--primary-500);
    cursor: pointer;
    display: flex;
    align-items: center;
  }
  .btn-container {
    display: flex;
    align-items: center;
  }

  .logo-text {
    display: none;
  }
  @media (min-width: 992px) {
    position: sticky;
    top: 0;

    .nav-center {
      width: 90%;
    }
    .logo {
      display: none;
    }
    .logo-text {
      display: block;
    }
  }
`
export default Wrapper
```

### 3. m-scalled nav extend context

#### 8. Links

- create src/utils/links.jsx

```jsx
import React from 'react'

import { IoBarChartSharp } from 'react-icons/io5'
import { MdQueryStats } from 'react-icons/md'
import { FaWpforms } from 'react-icons/fa'
import { ImProfile } from 'react-icons/im'
import { MdAdminPanelSettings } from 'react-icons/md'

const links = [
  { text: 'add Achievement', path: '.', icon: <FaWpforms /> },
  {
    text: 'all Achievements',
    path: 'all-achievements',
    icon: <MdQueryStats />,
  },
  { text: 'stats', path: 'stats', icon: <IoBarChartSharp /> },
  { text: 'profile', path: 'profile', icon: <ImProfile /> },
  { text: 'admin', path: 'admin', icon: <MdAdminPanelSettings /> },
]

export default links
```

- in a second, we will discuss why '.' in "add job"

#### 9. SmallSidebar

SmallSidebar

```jsx
import Wrapper from '../assets/wrappers/SmallSidebar'
import { FaTimes } from 'react-icons/fa'

import Logo from './Logo'
import { NavLink } from 'react-router-dom'
import links from '../utils/links'
import { useDashboardContext } from '../pages/DashboardLayout'

const SmallSidebar = () => {
  const { showSidebar, toggleSidebar } = useDashboardContext()
  return (
    <Wrapper>
      <div
        className={
          showSidebar ? 'sidebar-container show-sidebar' : 'sidebar-container'
        }
      >
        <div className="content">
          <button type="button" className="close-btn" onClick={toggleSidebar}>
            <FaTimes />
          </button>
          <header>
            <Logo />
          </header>
          <div className="nav-links">
            {links.map((link) => {
              const { text, path, icon } = link

              return (
                <NavLink
                  to={path}
                  key={text}
                  className="nav-link"
                  onClick={toggleSidebar}
                  // will discuss in a second
                  end
                >
                  <span className="icon">{icon}</span>
                  {text}
                </NavLink>
              )
            })}
          </div>
        </div>
      </div>
    </Wrapper>
  )
}

export default SmallSidebar
```

- cover '.' path ,active class and 'end' prop

#### 10. Small Sidebar CSS (optional)

assets/wrappers/SmallSidebar.js

```js
import styled from 'styled-components'

const Wrapper = styled.aside`
  @media (min-width: 992px) {
    display: none;
  }
  .sidebar-container {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: -1;
    opacity: 0;
    transition: var(--transition);
    visibility: hidden;
  }
  .show-sidebar {
    z-index: 99;
    opacity: 1;
    visibility: visible;
  }
  .content {
    background: var(--background-secondary-color);
    width: var(--fluid-width);
    height: 95vh;
    border-radius: var(--border-radius);
    padding: 4rem 2rem;
    position: relative;
    display: flex;
    align-items: center;
    flex-direction: column;
  }
  .close-btn {
    position: absolute;
    top: 10px;
    left: 10px;
    background: transparent;
    border-color: transparent;
    font-size: 2rem;
    color: var(--red-dark);
    cursor: pointer;
  }
  .nav-links {
    padding-top: 2rem;
    display: flex;
    flex-direction: column;
  }
  .nav-link {
    display: flex;
    align-items: center;
    color: var(--text-secondary-color);
    padding: 1rem 0;
    text-transform: capitalize;
    transition: var(--transition);
  }
  .nav-link:hover {
    color: var(--primary-500);
  }

  .icon {
    font-size: 1.5rem;
    margin-right: 1rem;
    display: grid;
    place-items: center;
  }
  .active {
    color: var(--primary-500);
  }
`
export default Wrapper
```

#### 11. NavLinks

- components/NavLinks.jsx

```jsx
import { useDashboardContext } from '../pages/DashboardLayout'
import links from '../utils/links'
import { NavLink } from 'react-router-dom'

const NavLinks = () => {
  const { user, toggleSidebar } = useDashboardContext()

  return (
    <div className="nav-links">
      {links.map((link) => {
        const { text, path, icon } = link
        // admin user
        return (
          <NavLink
            to={path}
            key={text}
            onClick={toggleSidebar}
            className="nav-link"
            end
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

### 4. Contitional container

#### 12. Big Sidebar

```jsx
import NavLinks from './NavLinks'
import Logo from '../components/Logo'
import Wrapper from '../assets/wrappers/BigSidebar'
import { useDashboardContext } from '../pages/DashboardLayout'

const BigSidebar = () => {
  const { showSidebar } = useDashboardContext()
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
          <NavLinks isBigSidebar />
        </div>
      </div>
    </Wrapper>
  )
}

export default BigSidebar
```

```jsx
const NavLinks = ({ isBigSidebar }) => {
  const { user, toggleSidebar } = useDashboardContext()

  return (
    <div className="nav-links">
      {links.map((link) => {
        const { text, path, icon } = link
        // admin user
        return (
          <NavLink
            to={path}
            key={text}
            onClick={isBigSidebar ? null : toggleSidebar}
            className="nav-link"
            end
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

#### 13. BigSidebar CSS (optional)

assets/wrappers/BigSidebar.js

```js
import styled from 'styled-components'

const Wrapper = styled.aside`
  display: none;
  @media (min-width: 992px) {
    display: block;
    box-shadow: 1px 0px 0px 0px rgba(0, 0, 0, 0.1);
    .sidebar-container {
      background: var(--background-secondary-color);
      min-height: 100vh;
      height: 100%;
      width: 250px;
      margin-left: -250px;
      transition: margin-left 0.3s ease-in-out;
    }
    .content {
      position: sticky;
      top: 0;
    }
    .show-sidebar {
      margin-left: 0;
    }
    header {
      height: 6rem;
      display: flex;
      align-items: center;
      padding-left: 2.5rem;
    }
    .nav-links {
      padding-top: 2rem;
      display: flex;
      flex-direction: column;
    }
    .nav-link {
      display: flex;
      align-items: center;
      color: var(--text-secondary-color);
      padding: 1rem 0;
      padding-left: 2.5rem;
      text-transform: capitalize;
      transition: padding-left 0.3s ease-in-out;
    }
    .nav-link:hover {
      padding-left: 3rem;
      color: var(--primary-500);
      transition: var(--transition);
    }

    .icon {
      font-size: 1.5rem;
      margin-right: 1rem;
      display: grid;
      place-items: center;
    }
    .active {
      color: var(--primary-500);
    }
  }
`
export default Wrapper
```

#### 14. LogoutContainer

components/LogoutContainer.jsx

```jsx
import { FaUserCircle, FaCaretDown } from 'react-icons/fa'
import Wrapper from '../assets/wrappers/LogoutContainer'
import { useState } from 'react'
import { useDashboardContext } from '../pages/DashboardLayout'

const LogoutContainer = () => {
  const [showLogout, setShowLogout] = useState(false)
  const { user, logoutUser } = useDashboardContext()

  return (
    <Wrapper>
      <button
        type="button"
        className="btn logout-btn"
        onClick={() => setShowLogout(!showLogout)}
      >
        {user.avatar ? (
          <img src={user.avatar} alt="avatar" className="img" />
        ) : (
          <FaUserCircle />
        )}

        {user?.name}
        <FaCaretDown />
      </button>
      <div className={showLogout ? 'dropdown show-dropdown' : 'dropdown'}>
        <button type="button" className="dropdown-btn" onClick={logoutUser}>
          logout
        </button>
      </div>
    </Wrapper>
  )
}

export default LogoutContainer
```

component/Navbar.jsx

```jsx
import LogoutContainer from './LogoutContainer'

const Navbar = () => {
  const { toggleSidebar } = useDashboardContext()

  return (
    <Wrapper>
      <div className="nav-center">
        <button type="button" className="toggle-btn" onClick={toggleSidebar}>
          <FaAlignLeft />
        </button>
        <div>
          <Logo />
          <h4 className="logo-text">dashboard</h4>
        </div>
        <div className="btn-container">
          <LogoutContainer />
        </div>
      </div>
    </Wrapper>
  )
}
export default Navbar
```

#### 15. LogoutContainer CSS (optional)

assets/wrappers/LogoutContainer.js

```js
import styled from 'styled-components'

const Wrapper = styled.div`
  position: relative;

  .logout-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0 0.5rem;
  }
  .img {
    width: 25px;
    height: 25px;
    border-radius: 50%;
  }
  .dropdown {
    position: absolute;
    top: 45px;
    left: 0;
    width: 100%;
    box-shadow: var(--shadow-2);
    text-align: center;
    visibility: hidden;
    border-radius: var(--border-radius);
    background: var(--primary-500);
  }
  .show-dropdown {
    visibility: visible;
  }
  .dropdown-btn {
    border-radius: var(--border-radius);
    padding: 0.5rem;
    background: transparent;
    border-color: transparent;
    color: var(--white);
    letter-spacing: var(--letter-spacing);
    text-transform: capitalize;
    cursor: pointer;
    width: 100%;
    height: 100%;
  }
`

export default Wrapper
```

## Navigate and local storage

### 1. insert conditional logic in useState (when (time) couple)

#### 16. ThemeToggle

components/ThemeToggle.jsx

```jsx
import { BsFillSunFill, BsFillMoonFill } from 'react-icons/bs'
import Wrapper from '../assets/wrappers/ThemeToggle'
import { useDashboardContext } from '../pages/DashboardLayout'

const ThemeToggle = () => {
  const { isDarkTheme, toggleDarkTheme } = useDashboardContext()
  return (
    <Wrapper onClick={toggleDarkTheme}>
      {isDarkTheme ? (
        <BsFillSunFill className="toggle-icon" />
      ) : (
        <BsFillMoonFill className="toggle-icon" />
      )}
    </Wrapper>
  )
}

export default ThemeToggle
```

Navbar.jsx

```jsx
<div className="btn-container">
  <ThemeToggle />
</div>
```

#### 17. ThemeToggle CSS (optional)

assets/wrappers/ThemeToggle.js

```js
import styled from 'styled-components'

const Wrapper = styled.div`
  background: transparent;
  border-color: transparent;
  width: 3.5rem;
  height: 2rem;
  display: grid;
  place-items: center;
  cursor: pointer;

  .toggle-icon {
    font-size: 1.15rem;
    color: var(--text-color);
  }
`
export default Wrapper
```

#### 18. Dark Theme - Logic

DashboardLayout.jsx

```jsx
const toggleDarkTheme = () => {
  const newDarkTheme = !isDarkTheme
  setIsDarkTheme(newDarkTheme)
  document.body.classList.toggle('dark-theme', newDarkTheme)
  localStorage.setItem('darkTheme', newDarkTheme)
}
```

#### 19. Access Theme

App.jsx

```jsx
const checkDefaultTheme = () => {
  const isDarkTheme =
    localStorage.getItem('darkTheme') === 'true'
  document.body.classList.toggle('dark-theme', isDarkTheme);
  return isDarkTheme;
};

const isDarkThemeEnabled = checkDefaultTheme();

{
path: 'dashboard',
element: <DashboardLayout isDarkThemeEnabled={isDarkThemeEnabled} />,
}
```

DashboardLayout.jsx

```jsx
const Dashboard = ({ isDarkThemeEnabled }) => {
  const [isDarkTheme, setIsDarkTheme] = useState(isDarkThemeEnabled)
}
```

#### 20. Dark Theme CSS

index.css

```css
:root {
  /* DARK MODE */

  --dark-mode-bg-color: #333;
  --dark-mode-text-color: #f0f0f0;
  --dark-mode-bg-secondary-color: #3f3f3f;
  --dark-mode-text-secondary-color: var(--grey-300);

  --background-color: var(--grey-50);
  --text-color: var(--grey-900);
  --background-secondary-color: var(--white);
  --text-secondary-color: var(--grey-500);
}

.dark-theme {
  --text-color: var(--dark-mode-text-color);
  --background-color: var(--dark-mode-bg-color);
  --text-secondary-color: var(--dark-mode-text-secondary-color);
  --background-secondary-color: var(--dark-mode-bg-secondary-color);
}

body {
  background: var(--background-color);
  color: var(--text-color);
}
```
