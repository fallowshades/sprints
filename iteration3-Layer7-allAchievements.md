# Iteration3 layer 7

[lll]

## Params -> send request

### (1) get all affect pipieline

#### 1. Get All Achievements - Server

achievement model

```js
const AchievementSchema = new mongoose.Schema({
  name: {
    type: String,
    default: 'NaN',
  },
})
```

achievementsController.js

Query parameters, also known as query strings or URL parameters, are used to pass information to a web server through the URL of a webpage. They are typically appended to the end of a URL after a question mark (?) and separated by ampersands (&). Query parameters consist of a key-value pair, where the key represents the parameter name and the value represents the corresponding data being passed. They are commonly used in web applications to provide additional context or parameters for server-side processing or to filter and sort data.

```js
export const getAllAchievements = async (req, res) => {
  const { search, status, type, sort } = req.query

  const queryObject = {
    createdBy: req.user.userId,
  }

  if (search) {
    queryObject.$or = [
      { description: { $regex: search, $options: 'i' } },
      { name: { $regex: search, $options: 'i' } },
    ]
  }
  if (status && status !== 'all') {
    queryObject.status = status
  }
  if (type && type !== 'all') {
    queryObject.type = type
  }

  const sortOptions = {
    newest: '-createdAt',
    oldest: 'createdAt',
    'a-z': 'position',
    'z-a': '-position',
  }

  const sortKey = sortOptions[sort] || sortOptions.newest

  // setup pagination
  const page = Number(req.query.page) || 1
  const limit = Number(req.query.limit) || 10
  const skip = (page - 1) * limit

  const achievements = await Achievement.find(queryObject)
    .sort(sortKey)
    .skip(skip)
    .limit(limit)

  const totalAchievements = await Achievement.countDocuments(queryObject)
  const numOfPages = Math.ceil(totalAchievements / limit)

  res
    .status(StatusCodes.OK)
    .json({ totalAchievements, numOfPages, currentPage: page, achievements })
}
```

#### 2. Search Container

- setup log in AllAchievements loader

```js
import { FormRow, FormRowSelect, SubmitBtn } from '.'
import Wrapper from '../assets/wrappers/DashboardFormPage'
import { Form, useSubmit, Link } from 'react-router-dom'
import {
  ACHIEVEMENT_TYPE,
  ACHIEVEMENT_STATUS,
  ACHIEVEMENT_SORT_BY,
} from '../../../utils/constants'
import { useAllAchievementsContext } from '../pages/AllAchievements'

const SearchContainer = () => {
  return (
    <Wrapper>
      <Form className='form'>
        <h5 className='form-title'>search form</h5>
        <div className='form-center'>
          {/* search position */}

          <FormRow type='search' name='search' defaultValue='a' />
          <FormRowSelect
            labelText='achievement status'
            name='achievementStatus'
            list={['all', ...Object.values(ACHIEVEMENT_STATUS)]}
            defaultValue='all'
          />
          <FormRowSelect
            labelText='achievements type'
            name='achievementType'
            list={['all', ...Object.values(ACHIEVEMENT_TYPE)]}
            defaultValue='all'
          />
          <FormRowSelect
            name='sort'
            defaultValue='newest'
            list={[...Object.values(ACHIEVEMENT_SORT_BY)]}
          />

          <Link
            to='/dashboard/all-achievements'
            className='btn form-btn delete-btn'
          >
            Reset Search Values
          </Link>
          {/* TEMP!!!! */}
          <SubmitBtn formBtn />
        </div>
      </Form>
    </Wrapper>
  )
}

export default SearchContainer
```

#### 3. All Achievements Loader

AllAchievements.jsx

```js
import { toast } from 'react-toastify'
import { AchievementsContainer, SearchContainer } from '../components'
import customFetch from '../utils/customFetch'
import { useLoaderData } from 'react-router-dom'
import { useContext, createContext } from 'react'
const AllAchievementsContext = createContext()
export const loader = async ({ request }) => {
  try {
    const params = Object.fromEntries([
      ...new URL(request.url).searchParams.entries(), ////
    ])

    const { data } = await customFetch.get('/achievements', {
      params, ////////
    })

    return {
      data,
      searchValues: { ...params }, //////////
    }
  } catch (error) {
    toast.error(error.response.data.msg)
    return error
  }
}

const AllAchievements = () => {
  const { data, searchValues } = useLoaderData()
  //
  return (
    <AllAchievementsContext.Provider value={{ data, searchValues }}>
      <SearchContainer />
      <AchievementsContainer />
    </AllAchievementsContext.Provider>
  )
}
export default AllAchievements

export const useAllAchievementsContext = () =>
  useContext(AllAchievementsContext)
```

```js
const params = Object.fromEntries([
  ...new URL(request.url).searchParams.entries(),
])
```

new URL(request.url): This creates a new URL object by passing the request.url to the URL constructor. The URL object provides various methods and properties to work with URLs.

.searchParams: The searchParams property of the URL object gives you access to the query parameters in the URL. It is an instance of the URLSearchParams class, which provides methods to manipulate and access the parameters.

.entries(): The entries() method of searchParams returns an iterator containing arrays of key-value pairs for each query parameter. Each array contains two elements: the parameter name and its corresponding value.

([...new URL(request.url).searchParams.entries()]): The spread operator ... is used to convert the iterator obtained from searchParams.entries() into an array. This allows us to pass the array to the Object.fromEntries() method.

Object.fromEntries(): This static method creates an object from an array of key-value pairs. It takes an iterable (in this case, the array of parameter key-value pairs) and returns a new object where the keys and values are derived from the iterable.

Putting it all together, the code retrieves the URL from the request.url property, extracts the search parameters using the searchParams property, converts them into an array of key-value pairs using entries(), and finally uses Object.fromEntries() to create an object with the parameter names as keys and their corresponding values. The resulting object, params, contains all the search parameters from the URL.

#### 4. Submit Form Programmatically

- setup default values from the context
- remove SubmitBtn
- add onChange to FormRow, FormRowSelect and all inputs

SearchContainer.js

```js
import { FormRow, FormRowSelect } from '.'
import Wrapper from '../assets/wrappers/DashboardFormPage'
import { Form, useSubmit, Link } from 'react-router-dom'
import {
  ACHIEVEMENT_TYPE,
  ACHIEVEMENT_STATUS,
  ACHIEVEMENT_SORT_BY,
} from '../../../utils/constants'
import { useAllAchievementsContext } from '../pages/AllAchievements'
const SearchContainer = () => {
  const { searchValues } = useAllAchievementsContext()
  const { search, status, type, sort } = searchValues

  const submit = useSubmit()

  return (
    <Wrapper>
      <Form className='form'>
        <h5 className='form-title'>search form</h5>
        <div className='form-center'>
          {/* search position */}

          <FormRow
            type='search'
            name='search'
            defaultValue={search}
            onChange={(e) => {
              submit(e.currentTarget.form)
            }}
          />
          <FormRowSelect
            labelText='achievement status'
            name='status'
            list={['all', ...Object.values(ACHIEVEMENT_STATUS)]}
            defaultValue={status}
            onChange={(e) => {
              submit(e.currentTarget.form)
            }}
          />
          <FormRowSelect
            labelText='achievement type'
            name='type'
            defaultValue={type}
            list={['all', ...Object.values(ACHIEVEMENT_TYPE)]}
            onChange={(e) => {
              submit(e.currentTarget.form)
            }}
          />
          <FormRowSelect
            name='sort'
            defaultValue={sort}
            list={[...Object.values(ACHIEVEMENT_SORT_BY)]}
            onChange={(e) => {
              submit(e.currentTarget.form)
            }}
          />
          <Link
            to='/dashboard/all-achievements'
            className='btn form-btn delete-btn'
          >
            Reset Search Values
          </Link>
        </div>
      </Form>
    </Wrapper>
  )
}

export default SearchContainer
```

### (2)Tool life cycle mounted

#### 5. Debounce

[JS Nuggets - Debounce](https://youtu.be/tYx6pXdvt1s)

In JavaScript, debounce is a way to limit how often a function gets called. It helps prevent rapid or repeated function executions by introducing a delay. This is useful for tasks like handling user input, where you want to wait for a pause before triggering an action to avoid unnecessary processing.

```js
const debounce = (onChange) => {
  let timeout
  return (e) => {
    const form = e.currentTarget.form
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      onChange(form)
    }, 2000)
  }
}
;<FormRow
  type='search'
  name='search'
  defaultValue={search}
  onChange={debounce((form) => {
    submit(form)
  })}
/>
```

## Pagination of response data

### (1)lots

#### 6. Pagination - Setup

- create PageBtnContainer

AchievementsContainer.jsx

```js
import Achievement from './Achievement'
import Wrapper from '../assets/wrappers/AchievementsContainer'
import PageBtnContainer from './PageBtnContainer'
import { useAllAchievementsContext } from '../pages/AllAchievements'

const AchievementsContainer = () => {
  const { data } = useAllAchievementsContext()
  const { achievements, totalAchievements, numOfPages } = data
  if (achievements.length === 0) {
    return (
      <Wrapper>
        <h2>No achievements to display...</h2>
      </Wrapper>
    )
  }

  return (
    <Wrapper>
      <h5>
        {totalAchievements} achievement{achievements.length > 1 && 's'} found
      </h5>
      <div className='achievements'>
        {achievement.map((achievements) => {
          return <Achievements key={achievement._id} {...achievement} />
        })}
      </div>
      {numOfPages > 1 && <PageBtnContainer />}
    </Wrapper>
  )
}

export default AchievementsContainer
```

#### 7. Basic PageBtnContainer

```js
import { HiChevronDoubleLeft, HiChevronDoubleRight } from 'react-icons/hi'
import Wrapper from '../assets/wrappers/PageBtnContainer'
import { useLocation, Link, useNavigate } from 'react-router-dom'
import { useAllAchievementsContext } from '../pages/AllAchievements'

const PageBtnContainer = () => {
  const {
    data: { numOfPages, currentPage },
  } = useAllAchievementsContext()
  const { search, pathname } = useLocation()
  const navigate = useNavigate()
  const pages = Array.from({ length: numOfPages }, (_, index) => index + 1) //1st btn

  const handlePageChange = (pageNumber) => {
    const searchParams = new URLSearchParams(search)
    searchParams.set('page', pageNumber)
    navigate(`${pathname}?${searchParams.toString()}`)
  }

  return (
    <Wrapper>
      <button
        className='btn prev-btn'
        onClick={() => {
          let prevPage = currentPage - 1
          if (prevPage < 1) prevPage = numOfPages
          handlePageChange(prevPage)
        }}
      >
        <HiChevronDoubleLeft />
        prev
      </button>
      {/* 1 btns*/}
      <div className='btn-container'>
        {pages.map((pageNumber) => (
          <button
            className={`btn page-btn ${pageNumber === currentPage && 'active'}`}
            key={pageNumber}
            onClick={() => handlePageChange(pageNumber)}
          >
            {pageNumber}
          </button>
        ))}
      </div>
      <button
        className='btn next-btn'
        onClick={() => {
          let nextPage = currentPage + 1
          if (nextPage > numOfPages) nextPage = 1
          handlePageChange(nextPage)
        }}
      >
        next
        <HiChevronDoubleRight />
      </button>
    </Wrapper>
  )
}

export default PageBtnContainer
```

#### 8. Complex - PageBtnContainer

```js
import { HiChevronDoubleLeft, HiChevronDoubleRight } from 'react-icons/hi'
import Wrapper from '../assets/wrappers/PageBtnContainer'
import { useLocation, Link, useNavigate } from 'react-router-dom'
import { useAllAchievementsContext } from '../pages/AllAchievements'

const PageBtnContainer = () => {
  const {
    data: { numOfPages, currentPage },
  } = useAllAchievementsContext()
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
        className={`btn page-btn ${activeClass && 'active'}`}
        key={pageNumber}
        onClick={() => handlePageChange(pageNumber)}
      >
        {pageNumber}
      </button>
    )
  }

  const renderPageButtons = () => {
    const pageButtons = []

    // Add the first page button
    pageButtons.push(
      addPageButton({ pageNumber: 1, activeClass: currentPage === 1 })
    )
    // Add the dots before the current page if there are more than 3 pages
    if (currentPage > 3) {
      pageButtons.push(
        <span className='page-btn dots' key='dots-1'>
          ....
        </span>
      )
    }
    // one before current page
    if (currentPage !== 1 && currentPage !== 2) {
      pageButtons.push(
        addPageButton({ pageNumber: currentPage - 1, activeClass: false })
      )
    }

    // Add the current page button
    if (currentPage !== 1 && currentPage !== numOfPages) {
      pageButtons.push(
        addPageButton({ pageNumber: currentPage, activeClass: true })
      )
    }

    // one after current page
    if (currentPage !== numOfPages && currentPage !== numOfPages - 1) {
      pageButtons.push(
        addPageButton({ pageNumber: currentPage + 1, activeClass: false })
      )
    }
    if (currentPage < numOfPages - 2) {
      pageButtons.push(
        <span className=' page-btn dots' key='dots+1'>
          ....
        </span>
      )
    }

    // Add the last page button
    pageButtons.push(
      addPageButton({
        pageNumber: numOfPages,
        activeClass: currentPage === numOfPages,
      })
    )

    return pageButtons
  }

  return (
    <Wrapper>
      <button
        className='prev-btn'
        onClick={() => {
          let prevPage = currentPage - 1
          if (prevPage < 1) prevPage = numOfPages
          handlePageChange(prevPage)
        }}
      >
        <HiChevronDoubleLeft />
        prev
      </button>
      <div className='btn-container'>{renderPageButtons()}</div>
      <button
        className='btn next-btn'
        onClick={() => {
          let nextPage = currentPage + 1
          if (nextPage > numOfPages) nextPage = 1
          handlePageChange(nextPage)
        }}
      >
        next
        <HiChevronDoubleRight />
      </button>
    </Wrapper>
  )
}

export default PageBtnContainer
```

#### 9. PageBtnContainer CSS (optional)

wrappers/PageBtnContainer.js

```js
import styled from 'styled-components'

const Wrapper = styled.section`
  height: 6rem;
  margin-top: 2rem;
  display: flex;
  align-items: center;
  justify-content: end;
  flex-wrap: wrap;
  gap: 1rem;
  .btn-container {
    background: var(--background-secondary-color);
    border-radius: var(--border-radius);
    display: flex;
  }
  .page-btn {
    background: transparent;
    border-color: transparent;
    width: 50px;
    height: 40px;
    font-weight: 700;
    font-size: 1.25rem;
    color: var(--primary-500);
    border-radius: var(--border-radius);
    cursor:pointer:
  }
  .active{
    background:var(--primary-500);
        color: var(--white);

  }
  .prev-btn,.next-btn{
    background: var(--background-secondary-color);
    border-color: transparent;
        border-radius: var(--border-radius);

    width: 100px;
    height: 40px;
        color: var(--primary-500);
text-transform:capitalize;
letter-spacing:var(--letter-spacing);
display:flex;
align-items:center;
justify-content:center;
gap:0.5rem;
cursor:pointer;
  }
  .prev-btn:hover,.next-btn:hover{
    background:var(--primary-500);
        color: var(--white);
        transition:var(--transition);
  }
.dots{
  display:grid;
  place-items:center;
  cursor:text;
}
`
export default Wrapper
```
