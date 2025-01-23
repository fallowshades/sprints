# Iteration 3 layer 3 Add Achievement

[create]

[read]
hook
time
what.method('typeOfFormat)

[dynamics]
all or 1 (acctually 1)

## Support tree for object transfer to set

### object request/response session

#### 1. AddAchievement - Structure

pages/AddAchievement.jsx

```js
//element
import { FormRow } from '../components'
import Wrapper from '../assets/wrappers/DashboardFormPage'
import { useOutletContext } from 'react-router-dom'
// mapping
import { ACHIEVEMENT_STATUS, ACHIEVEMENT_TYPE } from '../../../utils/constants'
//network
import { Form, useNavigation, redirect } from 'react-router-dom'
import { toast } from 'react-toastify'
import customFetch from '../utils/customFetch'

const AddAchievement = () => {
  const { user } = useOutletContext()
  const navigation = useNavigation()
  const isSubmitting = navigation.state === 'submitting'

  return (
    <Wrapper>
      <Form method='post' className='form'>
        <h4 className='form-title'>add achievement</h4>
        <div className='form-center'>
          <FormRow type='text' name='description' />
          <FormRow type='text' name='points' />
          <FormRow
            type='text'
            labelText='achievement user attachment'
            name='createdBy'
            defaultValue={user.createdBy}
          />

          <button
            type='submit'
            className='btn btn-block form-btn '
            disabled={isSubmitting}
          >
            {isSubmitting ? 'submitting...' : 'submit'}
          </button>
        </div>
      </Form>
    </Wrapper>
  )
}

export default AddAchievement
```

dont try to send request yet

#### 2. Select Input

```js
<div className='form-row'>
  <label htmlFor='status' className='form-label'>
    Achievement status
  </label>
  <select
    name='status'
    id='status'
    className='form-select'
    defaultValue={ACHIEVEMENT_TYPE.EXPLORATION}
  >
    {Object.values(ACHIEVEMENT_TYPE).map((itemValue) => {
      return (
        <option key={itemValue} value={itemValue}>
          {itemValue}
        </option>
      )
    })}
  </select>
</div>
```

#### 3. FormRowSelect Component

components/FormRowSelect.jsx

```js
const FormRowSelect = ({ name, labelText, list, defaultValue = '' }) => {
  return (
    <div className='form-row'>
      <label htmlFor={name} className='form-label'>
        {labelText || name}
      </label>
      <select
        name={name}
        id={name}
        className='form-select'
        defaultValue={defaultValue}
      >
        {list.map((itemValue) => {
          return (
            <option key={itemValue} value={itemValue}>
              {itemValue}
            </option>
          )
        })}
      </select>
    </div>
  )
}
export default FormRowSelect
```

pages/AddAchievement.jsx

```js
<FormRowSelect
  labelText='achievement  status'
  name='status'
  defaultValue={ACHIEVEMENT_STATUS.INACTIVE}
  list={Object.values(ACHIEVEMENT_STATUS)}
  />
<FormRowSelect
  name='type'
  labelText='achievement type'
  defaultValue={ACHIEVEMENT_TYPE.EXPLORATION}
  list={Object.values(ACHIEVEMENT_TYPE)}
  />
```

### Heavy load master --> slave

#### 4. Create Achievement

AddAchievement.jsx

```js
export const action = async ({ request }) => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)

  try {
    await customFetch.post('/achievements', data)
    toast.success('achievement added successfully')
    return null
  } catch (error) {
    toast.error(error?.response?.data?.msg)
    return error
  }
}
```

#### 5. Pending Class and Redirect

wrappers/BigSidebar.js

```css
.pending {
  background: var(--background-color);
}
```

AddAchievement.jsx

```js
export const action = async ({ request }) => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)

  try {
    await customFetch.post('/achievements', data)
    toast.success('achievement added successfully')
    return redirect('all-achievements')
  } catch (error) {
    toast.error(error?.response?.data?.msg)
    return error
  }
}
```

#### 6. Add Achievement - CSS(optional)

wrappers/DashboardFormPage.js

```js
import styled from 'styled-components'

const Wrapper = styled.section`
  border-radius: var(--border-radius);
  width: 100%;
  background: var(--background-secondary-color);
  padding: 3rem 2rem 4rem;
  box-shadow: var(--shadow-2);
  .form-title {
    margin-bottom: 2rem;
  }

  .form {
    margin: 0;
    border-radius: 0;
    box-shadow: none;
    padding: 0;
    max-width: 100%;
    width: 100%;
  }
  .form-row {
    margin-bottom: 0;
  }
  .form-center {
    display: grid;
    row-gap: 1rem;
  }
  .form-btn {
    align-self: end;
    margin-top: 1rem;
    display: grid;
    place-items: center;
  }

  @media (min-width: 992px) {
    .form-center {
      grid-template-columns: 1fr 1fr;
      align-items: center;
      column-gap: 1rem;
    }
  }
  @media (min-width: 1120px) {
    .form-center {
      grid-template-columns: 1fr 1fr 1fr;
    }
  }
`

export default Wrapper
```

## Create transfer lifecycle

### both master and a slave to request its creators data

#### 7. All Achievements - Structure

- create AchievementsContainer and SearchContainer (export)
- handle loader in App.jsx

```js
import { toast } from 'react-toastify'
import { AchievementsContainer, SearchContainer } from '../components'
import customFetch from '../utils/customFetch'
import { useLoaderData } from 'react-router-dom'
import { useContext, createContext } from 'react'

export const loader = async ({ request }) => {
  try {
    const { data } = await customFetch.get('/achievements')
    return {
      data,
    }
  } catch (error) {
    toast.error(error?.response?.data?.msg)
    return error
  }
}

const AllAchievement = () => {
  const { data } = useLoaderData()

  return (
    <>
      <SearchContainer />
      <AchievementsContainer />
    </>
  )
}
export default AllAchievement
```

#### 8. Setup All Achievements Context

```js
const AllAchievementsContext = createContext()

const AllAchievement = () => {
  const { data } = useLoaderData()

  return (
    <AllAchievementContext.Provider value={{ data }}>
      <SearchContainer />
      <AchievementContainer />
    </AllAchievementContext.Provider>
  )
}

export const useAllAchievementsContext = () => useContext(AllAchievementContext)
```

## Context help map presentational data

### l

#### 9. Render Achievements

- create Achievement.jsx

AchievementsContainer.jsx

```js
import Achievement from './Achievement'
import Wrapper from '../assets/wrappers/AchievementsContainer'

import { useAllAchievementsContext } from '../pages/AllAchievements'

const AchievementsContainer = () => {
  const { data } = useAllAchievementsContext()
  const { achievements } = data
  if (achievements.length === 0) {
    return (
      <Wrapper>
        <h2>No Achievements to display...</h2>
      </Wrapper>
    )
  }

  return (
    <Wrapper>
      <div className='achievements'>
        {achievements.map((achievement) => {
          return <Achievement key={achievement._id} {...Achievement} />
        })}
      </div>
    </Wrapper>
  )
}

export default AchievementContainer
```

#### 10. AchievementContainer - CSS (optional)

wrappers/AchievementContainer.js

```js
import styled from 'styled-components'

const Wrapper = styled.section`
  margin-top: 4rem;
  h2 {
    text-transform: none;
  }
  & > h5 {
    font-weight: 700;
    margin-bottom: 1.5rem;
  }
  .achievements {
    display: grid;
    grid-template-columns: 1fr;
    row-gap: 2rem;
  }
  @media (min-width: 1120px) {
    .achievements {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 2rem;
    }
  }
`
export default Wrapper
```

### Understandable

#### 11. Dayjs

```sh
npm i dayjs@1.11.7
```

[Dayjs Docs](https://day.js.org/docs/en/installation/installation)

#### 12. Achievement Component

- create AchievementInfo component

```js
import { FaLocationArrow, FaBriefcase, FaCalendarAlt } from 'react-icons/fa'
import { Link } from 'react-router-dom'
import Wrapper from '../assets/wrappers/Achievement'
import AchievementInfo from './AchievementInfo'
import { Form } from 'react-router-dom'
import day from 'dayjs'
import advancedFormat from 'dayjs/plugin/advancedFormat'
day.extend(advancedFormat)

const Achievement = ({
  _id,
  description,
  status,
  points,
  type,
  dateOfCompletion,
  createdAt,
}) => {
  const date = day(createdAt).format('MMM Do, YYYY')

  return (
    <Wrapper>
      <header>
        <div className='main-icon'>{description.charAt(0)}</div>
        <div className='info'>
          <h5>{description}</h5>
          <p>{points}</p>
        </div>
      </header>
      <div className='content'>
        <div className='content-center'>
          <AchievementInfo icon={<FaLocationArrow />} text={date} />
          <AchievementInfo icon={<FaCalendarAlt />} text={points} />
          <AchievementInfo icon={<FaBriefcase />} text={type} />
          <div className={`status ${status}`}>{status}</div>
        </div>

        <footer className='actions'>
          <Link className='btn edit-btn'>Edit</Link>
          <Form>
            <button type='submit' className='btn delete-btn'>
              Delete
            </button>
          </Form>
        </footer>
      </div>
    </Wrapper>
  )
}

export default Achievement
```

#### 13. AchievementInfo Component

```js
import Wrapper from '../assets/wrappers/AchievementInfo'

const AchievementInfo = ({ icon, text }) => {
  return (
    <Wrapper>
      <span className='achievement-icon'>{icon}</span>
      <span className='achievement-text'>{text}</span>
    </Wrapper>
  )
}

export default AchievementInfo
```

#### 14. AchievementInfo - CSS (optional)

wrappers/AchievementInfo.js

```js
import styled from 'styled-components'

const Wrapper = styled.div`
  display: flex;
  align-items: center;

  .achievement-icon {
    font-size: 1rem;
    margin-right: 1rem;
    display: flex;
    align-items: center;
    svg {
      color: var(--text-secondary-color);
    }
  }
  .achievement-text {
    text-transform: capitalize;
    letter-spacing: var(--letter-spacing);
  }
`
export default Wrapper
```

#### 15. Achievement - CSS (optional)

```js
import styled from 'styled-components'

const Wrapper = styled.article`
  background: var(--background-secondary-color);
  border-radius: var(--border-radius);
  display: grid;
  grid-template-rows: 1fr auto;
  box-shadow: var(--shadow-2);

  header {
    padding: 1rem 1.5rem;
    border-bottom: 1px solid var(--grey-100);
    display: grid;
    grid-template-columns: auto 1fr;
    align-items: center;
  }
  .main-icon {
    width: 60px;
    height: 60px;
    display: grid;
    place-items: center;
    background: var(--primary-500);
    border-radius: var(--border-radius);
    font-size: 1.5rem;
    font-weight: 700;
    text-transform: uppercase;
    color: var(--white);
    margin-right: 2rem;
  }
  .info {
    h5 {
      margin-bottom: 0.5rem;
    }
    p {
      margin: 0;
      text-transform: capitalize;
      color: var(--text-secondary-color);
      letter-spacing: var(--letter-spacing);
    }
  }

  .content {
    padding: 1rem 1.5rem;
  }
  .content-center {
    display: grid;
    margin-top: 1rem;
    margin-bottom: 1.5rem;
    grid-template-columns: 1fr;
    row-gap: 1.5rem;
    align-items: center;
    @media (min-width: 576px) {
      grid-template-columns: 1fr 1fr;
    }
  }

  .status {
    border-radius: var(--border-radius);
    text-transform: capitalize;
    letter-spacing: var(--letter-spacing);
    text-align: center;
    width: 100px;
    height: 30px;
    display: grid;
    align-items: center;
  }
  .actions {
    margin-top: 1rem;
    display: flex;
    align-items: center;
  }
  .edit-btn,
  .delete-btn {
    height: 30px;
    font-size: 0.85rem;
    display: flex;
    align-items: center;
  }
  .edit-btn {
    margin-right: 0.5rem;
  }
`

export default Wrapper
```

### Dynamic updates

#### 16. Achievement Achievement - Setup

Achievement.jsx

```js
<Link to={`../edit-achievement/${_id}`} className='btn edit-btn'>
  Edit
</Link>
```

pages/EditAchievement.jsx

```js
import { FormRow, FormRowSelect } from '../components'
import Wrapper from '../assets/wrappers/DashboardFormPage'
import { useLoaderData } from 'react-router-dom'
import { ACHIEVEMENT_STATUS, ACHIEVEMENT_TYPE } from '../../../utils/constants'
import { Form, useNavigation, redirect } from 'react-router-dom'
import { toast } from 'react-toastify'
import customFetch from '../utils/customFetch'

export const loader = async () => {
  return null
}
export const action = async () => {
  return null
}

const EditAchievement = () => {
  return <h1>EditAchievement Page</h1>
}
export default EditAchievement
```

- import EditAchievement page
  App.jsx

```js
import { loader as editAchievementLoader } from './pages/EditAchievement';
import { action as editAchievementAction } from './pages/EditAchievement';


{
  path: 'edit-achievement/:id',
  element: <EditAchievement />,
  loader: editAchievementLoader,
  action: editAchievementAction,
},
```

pages/EditAchievement.jsx

```js
export const loader = async ({ params }) => {
  try {
    const { data } = await customFetch.get(`/achievements/${params.id}`)
    return data
  } catch (error) {
    toast.error(error.response.data.msg)
    return redirect('/dashboard/all-achievements')
  }
}
export const action = async () => {
  return null
}

const EditAchievement = () => {
  const params = useParams()
  console.log(params)
  const { achievement } = useLoaderData()

  const navigation = useNavigation()
  const isSubmitting = navigation.state === 'submitting'
  return <h1>EditAchievement Page</h1>
}
export default EditAchievement
```

#### 16. Edit Achievement - Complete

```js
export const action = async ({ request, params }) => {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)

  try {
    await customFetch.patch(`/achievements/${params.id}`, data)
    toast.success('Achievement edited successfully')
    return redirect('/dashboard/all-achievements')
  } catch (error) {
    toast.error(error.response.data.msg)
    return error
  }
}

const EditAchievement = () => {
  const { achievement } = useLoaderData()

  const navigation = useNavigation()
  const isSubmitting = navigation.state === 'submitting'

  return (
    <Wrapper>
      <Form method='post' className='form'>
        <h4 className='form-title'>edit achievement</h4>
        <div className='form-center'>
          <FormRow
            type='text'
            name='description'
            defaultValue={achievement.description}
          />
          <FormRow
            type='text'
            name='points'
            defaultValue={achievement.points}
          />
          <FormRow
            type='text'
            labelText='achievement type'
            name='type'
            defaultValue={achievement.type}
          />

          <FormRowSelect
            name='status'
            labelText='achievement status'
            defaultValue={achievement.status}
            list={Object.values(ACHIEVEMENT_STATUS)}
          />
          <FormRowSelect
            name='achievementType'
            labelText='achievement type'
            defaultValue={achievement.type}
            list={Object.values(ACHIEVEMENT_TYPE)}
          />
          <button
            type='submit'
            className='btn btn-block form-btn '
            disabled={isSubmitting}
          >
            {isSubmitting ? 'submitting...' : 'submit'}
          </button>
        </div>
      </Form>
    </Wrapper>
  )
}

export default EditAchievement
```

#### 17. Delete Achievement

Achievement.jsx

```js
<Form method='post' action={`../delete-achievement/${_id}`}>
  <button type='submit' className='btn delete-btn'>
    Delete
  </button>
</Form>
```

pages/DeleteAchievement.jsx

```js
import { redirect } from 'react-router-dom'
import customFetch from '../utils/customFetch'
import { toast } from 'react-toastify'

export async function action({ params }) {
  try {
    await customFetch.delete(`/achievements/${params.id}`)
    toast.success('Achievement deleted successfully')
  } catch (error) {
    toast.error(error.response.data.msg)
  }
  return redirect('/dashboard/all-achievements')
}
```

App.jsx

```js
import { action as deleteAchievementAction } from './pages/DeleteAchievement';

 { path: 'delete-achievement/:id', action: deleteAchievementAction },
```
