# Iteration 3 layer 4 Admin

[wrapped-in-obj]

- the session
- the css obj

## g

### count only if role and path

#### Admin Page

pages/Admin.jsx

```js
import { FaSuitcaseRolling, FaCalendarCheck } from 'react-icons/fa'

import { useLoaderData, redirect } from 'react-router-dom'
import customFetch from '../utils/customFetch'
import Wrapper from '../assets/wrappers/StatsContainer'
import { toast } from 'react-toastify'
export const loader = async () => {
  try {
    const response = await customFetch.get('/users/admin/app-stats')
    return response.data
  } catch (error) {
    toast.error('You are not authorized to view this page')
    return redirect('/dashboard')
  }
}

const Admin = () => {
  const { users, jobs } = useLoaderData()

  return (
    <Wrapper>
      <h2>admin page</h2>
    </Wrapper>
  )
}
export default Admin
```

App.jsx

```js
import { loader as adminLoader } from './pages/Admin';

{
  path: 'admin',
  element: <Admin />,
  loader: adminLoader,
},

```

NavLinks.jsx

```js
{
  links.map((link) => {
    const { text, path, icon } = link
    const { role } = user
    if (role !== 'admin' && path === 'admin') return
  })
}
```

#### StatItem Component

- create StatItem.jsx
- import/export

  StatItem.jsx

```js
import Wrapper from '../assets/wrappers/StatItem'

const StatItem = ({ count, title, icon, color, bcg }) => {
  return (
    <Wrapper color={color} bcg={bcg}>
      <header>
        <span className='count'>{count}</span>
        <span className='icon'>{icon}</span>
      </header>
      <h5 className='title'>{title}</h5>
    </Wrapper>
  )
}

export default StatItem
```

Admin.jsx

```js
import { StatItem } from '../components'

const Admin = () => {
  const { users, achievements } = useLoaderData()

  return (
    <Wrapper>
      <StatItem
        title='current users'
        count={users}
        color='#e9b949'
        bcg='#fcefc7'
        icon={<FaSuitcaseRolling />}
      />
      <StatItem
        title='total achievement'
        count={achievement}
        color='#647acb'
        bcg='#e0e8f9'
        icon={<FaCalendarCheck />}
      />
    </Wrapper>
  )
}
export default Admin
```

#### Admin - CSS (optional)

wrappers/StatsContainer.js

```js
import styled from 'styled-components'

const Wrapper = styled.section`
  display: grid;
  row-gap: 2rem;
  @media (min-width: 768px) {
    grid-template-columns: 1fr 1fr;
    column-gap: 1rem;
  }
  @media (min-width: 1120px) {
    grid-template-columns: 1fr 1fr 1fr;
    column-gap: 1rem;
  }
`
export default Wrapper
```

wrappers/StatItem.js

```js
import styled from 'styled-components'

const Wrapper = styled.article`
  padding: 2rem;
  background: var(--background-secondary-color);
  border-radius: var(--border-radius);
  border-bottom: 5px solid ${(props) => props.color};
  header {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .count {
    display: block;
    font-weight: 700;
    font-size: 50px;
    color: ${(props) => props.color};
    line-height: 2;
  }
  .title {
    margin: 0;
    text-transform: capitalize;
    letter-spacing: var(--letter-spacing);
    text-align: left;
    margin-top: 0.5rem;
    font-size: 1.25rem;
  }
  .icon {
    width: 70px;
    height: 60px;
    background: ${(props) => props.bcg};
    border-radius: var(--border-radius);
    display: flex;
    align-items: center;
    justify-content: center;
    svg {
      font-size: 2rem;
      color: ${(props) => props.color};
    }
  }
`

export default Wrapper
```
