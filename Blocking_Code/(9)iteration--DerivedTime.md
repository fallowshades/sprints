## Derived time

### (1) Ctrl outside presentation on all/ duplicate data versions

#### Filter

#### Get All Jobs - Initial Setup

```js
jobsController.js

const getAllJobs = async (req, res) => {
  const { search, status, jobType, sort } = req.query

  const queryObject = {
    createdBy: req.user.userId,
  }

  // NO AWAIT
  let result = Job.find(queryObject)

  // chain sort conditions

  const jobs = await result

  res
    .status(StatusCodes.OK)
    .json({ jobs, totalJobs: jobs.length, numOfPages: 1 })
}
```

#### Status

```js
jobsController.js

const getAllJobs = async (req, res) => {
  const { search, status, jobType, sort } = req.query

  const queryObject = {
    createdBy: req.user.userId,
  }

  if (status !== 'all') {
    queryObject.status = status
  }

  // NO AWAIT
  let result = Job.find(queryObject)

  // chain sort conditions

  const jobs = await result

  res
    .status(StatusCodes.OK)
    .json({ jobs, totalJobs: jobs.length, numOfPages: 1 })
}
```

#### JobType

```js
jobsController.js

const getAllJobs = async (req, res) => {
  const { search, status, jobType, sort } = req.query

  const queryObject = {
    createdBy: req.user.userId,
  }

  if (status !== 'all') {
    queryObject.status = status
  }
  if (jobType !== 'all') {
    queryObject.jobType = jobType
  }
  // NO AWAIT
  let result = Job.find(queryObject)

  // chain sort conditions

  const jobs = await result

  res
    .status(StatusCodes.OK)
    .json({ jobs, totalJobs: jobs.length, numOfPages: 1 })
}
```

#### Search

```js
jobsController.js

const getAllJobs = async (req, res) => {
  const { search, status, jobType, sort } = req.query

  const queryObject = {
    createdBy: req.user.userId,
  }

  if (status !== 'all') {
    queryObject.status = status
  }
  if (jobType !== 'all') {
    queryObject.jobType = jobType
  }
  if (search) {
    queryObject.position = { $regex: search, $options: 'i' }
  }
  // NO AWAIT
  let result = Job.find(queryObject)

  // chain sort conditions
  if (sort === 'latest') {
    result = result.sort('-createdAt')
  }
  if (sort === 'oldest') {
    result = result.sort('createdAt')
  }
  if (sort === 'a-z') {
    result = result.sort('position')
  }
  if (sort === 'z-a') {
    result = result.sort('-position')
  }
  const jobs = await result

  res
    .status(StatusCodes.OK)
    .json({ jobs, totalJobs: jobs.length, numOfPages: 1 })
}
```

#### Search Context Setup

```js
appContext.js

const initialState = {
  jobType: 'full-time',
  jobTypeOptions: ['full-time', 'part-time', 'remote', 'internship'],
  status: 'pending',
  statusOptions: ['pending', 'interview', 'declined']
  //
  //
  //
  search: '',
  searchStatus: 'all',
  searchType: 'all',
  sort: 'latest',
  sortOptions: ['latest', 'oldest', 'a-z', 'z-a'],
}

const clearFilters = () =>{
console.log('clear filters')
}

value={{clearFilters}}

// remember this function :)
const handleChange = ({ name, value }) => {
    dispatch({
      type: HANDLE_CHANGE,
      payload: { name, value },
    })
  }

```

#### Search Container - Setup

```js
SearchContainer.js

import { FormRow, FormRowSelect } from '.'
import { useAppContext } from '../context/appContext'
import Wrapper from '../assets/wrappers/SearchContainer'
const SearchContainer = () => {
  const {
    isLoading,
    search,
    searchStatus,
    searchType,
    sort,
    sortOptions,
    statusOptions,
    jobTypeOptions,
    handleChange,
    clearFilters,
  } = useAppContext()

  const handleSearch = (e) => {
    if (isLoading) return
    handleChange({ name: e.target.name, value: e.target.value })
  }

  return (
    <Wrapper>
      <form className="form">
        <h4>search form</h4>
        {/* search position */}
        <div className="form-center">
          <FormRow
            type="text"
            name="search"
            value={search}
            handleChange={handleSearch}
          ></FormRow>
          {/* rest of the inputs */}
        </div>
      </form>
    </Wrapper>
  )
}

export default SearchContainer
```

#### Search Container - Complete

```js
SearchContainer.js

import { FormRow, FormRowSelect } from '.'
import { useAppContext } from '../context/appContext'
import Wrapper from '../assets/wrappers/SearchContainer'

const SearchContainer = () => {
  const {
    isLoading,
    search,
    handleChange,
    searchStatus,
    statusOptions,
    jobTypeOptions,
    searchType,
    clearFilters,
    sort,
    sortOptions,
  } = useAppContext()

  const handleSearch = (e) => {
    if (isLoading) return
    handleChange({ name: e.target.name, value: e.target.value })
  }
  const handleSubmit = (e) => {
    e.preventDefault()
    clearFilters()
  }
  return (
    <Wrapper>
      <form className="form">
        <h4>search form</h4>
        {/* search position */}
        <div className="form-center">
          <FormRow
            type="text"
            name="search"
            value={search}
            handleChange={handleSearch}
          ></FormRow>
          {/* search by status */}
          <FormRowSelect
            labelText="job status"
            name="searchStatus"
            value={searchStatus}
            handleChange={handleSearch}
            list={['all', ...statusOptions]}
          ></FormRowSelect>
          {/* search by type */}

          <FormRowSelect
            labelText="job type"
            name="searchType"
            value={searchType}
            handleChange={handleSearch}
            list={['all', ...jobTypeOptions]}
          ></FormRowSelect>
          {/* sort */}

          <FormRowSelect
            name="sort"
            value={sort}
            handleChange={handleSearch}
            list={sortOptions}
          ></FormRowSelect>
          <button
            className="btn btn-block btn-danger"
            disabled={isLoading}
            onClick={handleSubmit}
          >
            clear filters
          </button>
        </div>
      </form>
    </Wrapper>
  )
}

export default SearchContainer
```

#### Clear Filters

```js
actions.js

export const CLEAR_FILTERS = 'CLEAR_FILTERS'
```

```js
appContext.js

const clearFilters = () => {
  dispatch({ type: CLEAR_FILTERS })
}
```

```js
reducer.js

if (action.type === CLEAR_FILTERS) {
  return {
    ...state,
    search: '',
    searchStatus: 'all',
    searchType: 'all',
    sort: 'latest',
  }
}
```

#### Refactor Get All Jobs

```js
const getJobs = async () => {
  // will add page later
  const { search, searchStatus, searchType, sort } = state
  let url = `/jobs?status=${searchStatus}&jobType=${searchType}&sort=${sort}`
  if (search) {
    url = url + `&search=${search}`
  }
  dispatch({ type: GET_JOBS_BEGIN })
  try {
    const { data } = await authFetch(url)
    const { jobs, totalJobs, numOfPages } = data
    dispatch({
      type: GET_JOBS_SUCCESS,
      payload: {
        jobs,
        totalJobs,
        numOfPages,
      },
    })
  } catch (error) {
    // logoutUser()
  }
  clearAlert()
}
```

```js
JobsContainer.js

const JobsContainer = () => {
  const {
    getJobs,
    jobs,
    isLoading,
    page,
    totalJobs,
    search,
    searchStatus,
    searchType,
    sort,

  } = useAppContext()
  useEffect(() => {
    getJobs()
  }, [ search, searchStatus, searchType, sort])

```

### (2)Relational stats about data

#### Limit and Skip

```js
jobsController.js

const getAllJobs = async (req, res) => {
  const { search, status, jobType, sort } = req.query
  const queryObject = {
    createdBy: req.user.userId,
  }
  if (search) {
    queryObject.position = { $regex: search, $options: 'i' }
  }
  if (status !== 'all') {
    queryObject.status = status
  }
  if (jobType !== 'all') {
    queryObject.jobType = jobType
  }
  let result = Job.find(queryObject)

  if (sort === 'latest') {
    result = result.sort('-createdAt')
  }
  if (sort === 'oldest') {
    result = result.sort('createdAt')
  }
  if (sort === 'a-z') {
    result = result.sort('position')
  }
  if (sort === 'z-a') {
    result = result.sort('-position')
  }

  const totalJobs = await result

  // setup pagination
  const limit = 10
  const skip = 1

  result = result.skip(skip).limit(limit)
  // 23
  // 4 7 7 7 2
  const jobs = await result
  res
    .status(StatusCodes.OK)
    .json({ jobs, totalJobs: jobs.length, numOfPages: 1 })
}
```

#### Page and Limit

```js
jobsController.js

const getAllJobs = async (req, res) => {
  const { search, status, jobType, sort } = req.query
  const queryObject = {
    createdBy: req.user.userId,
  }
  if (search) {
    queryObject.position = { $regex: search, $options: 'i' }
  }
  if (status !== 'all') {
    queryObject.status = status
  }
  if (jobType !== 'all') {
    queryObject.jobType = jobType
  }
  let result = Job.find(queryObject)

  if (sort === 'latest') {
    result = result.sort('-createdAt')
  }
  if (sort === 'oldest') {
    result = result.sort('createdAt')
  }
  if (sort === 'a-z') {
    result = result.sort('position')
  }
  if (sort === 'z-a') {
    result = result.sort('-position')
  }

  // setup pagination
  const page = Number(req.query.page) || 1
  const limit = Number(req.query.limit) || 10
  const skip = (page - 1) * limit //10
  result = result.skip(skip).limit(limit)
  // 75
  // 10 10 10 10 10 10 10 5
  const jobs = await result
  res
    .status(StatusCodes.OK)
    .json({ jobs, totalJobs: jobs.length, numOfPages: 1 })
}
```

#### Total Jobs and Number Of Pages

```js
jobsController.js

const getAllJobs = async (req, res) => {
  const { search, status, jobType, sort } = req.query
  const queryObject = {
    createdBy: req.user.userId,
  }
  if (search) {
    queryObject.position = { $regex: search, $options: 'i' }
  }
  if (status !== 'all') {
    queryObject.status = status
  }
  if (jobType !== 'all') {
    queryObject.jobType = jobType
  }
  let result = Job.find(queryObject)

  if (sort === 'latest') {
    result = result.sort('-createdAt')
  }
  if (sort === 'oldest') {
    result = result.sort('createdAt')
  }
  if (sort === 'a-z') {
    result = result.sort('position')
  }
  if (sort === 'z-a') {
    result = result.sort('-position')
  }

  // setup pagination
  const page = Number(req.query.page) || 1
  const limit = Number(req.query.limit) || 10
  const skip = (page - 1) * limit

  result = result.skip(skip).limit(limit)

  const jobs = await result

  const totalJobs = await Job.countDocuments(queryObject)
  const numOfPages = Math.ceil(totalJobs / limit)

  res.status(StatusCodes.OK).json({ jobs, totalJobs, numOfPages })
}
```

#### PageBtnContainer Setup

- PageBtnContainer.js

```js
JobsContainer.js

import PageBtnContainer from './PageBtnContainer'

const { numOfPages } = useAppContext()

return (
  <Wrapper>
    <h5>
      {totalJobs} job{jobs.length > 1 && 's'} found
    </h5>
    <div className="jobs">
      {jobs.map((job) => {
        return <Job key={job._id} {...job} />
      })}
    </div>
    {numOfPages > 1 && <PageBtnContainer />}
  </Wrapper>
)
```

#### PageBtnContainer - Structure

```js
PageBtnContainer.js

import { useAppContext } from '../context/appContext'
import { HiChevronDoubleLeft, HiChevronDoubleRight } from 'react-icons/hi'
import Wrapper from '../assets/wrappers/PageBtnContainer'

const PageButtonContainer = () => {
  const { numOfPages, page } = useAppContext()

  const prevPage = () => {
    console.log('prev page')
  }
  const nextPage = () => {
    console.log('next page')
  }

  return (
    <Wrapper>
      <button className="prev-btn" onClick={prevPage}>
        <HiChevronDoubleLeft />
        prev
      </button>

      <div className="btn-container">buttons</div>

      <button className="next-btn" onClick={nextPage}>
        next
        <HiChevronDoubleRight />
      </button>
    </Wrapper>
  )
}

export default PageButtonContainer
```

#### Button Container

- [Array.from] (https://youtu.be/zg1Bv4xubwo)

```js
PageBtnContainer.js

const pages = Array.from({ length: numOfPages }, (_, index) => {
  return index + 1
})

return (
  <div className="btn-container">
    {pages.map((pageNumber) => {
      return (
        <button
          type="button"
          className={pageNumber === page ? 'pageBtn active' : 'pageBtn'}
          key={pageNumber}
          onClick={() => console.log(page)}
        >
          {pageNumber}
        </button>
      )
    })}
  </div>
)
```

#### Change Page

```js
actions.js
export const CHANGE_PAGE = 'CHANGE_PAGE'
```

```js
appContext.js
const changePage = (page) => {
  dispatch({ type: CHANGE_PAGE, payload: { page } })
}
value={{changePage}}
```

```js
reducer.js

if (action.type === CHANGE_PAGE) {
  return { ...state, page: action.payload.page }
}
```

```js
PageBtnContainer.js

const { changePage } = useAppContext()
return (
  <button
    type="button"
    className={pageNumber === page ? 'pageBtn active' : 'pageBtn'}
    key={pageNumber}
    onClick={() => changePage(pageNumber)}
  >
    {pageNumber}
  </button>
)
```

#### Prev and Next Buttons

```js
PageBtnContainer.js
const prevPage = () => {
  let newPage = page - 1
  if (newPage < 1) {
    // newPage = 1
    // alternative
    newPage = numOfPages
  }
  changePage(newPage)
}
const nextPage = () => {
  let newPage = page + 1
  if (newPage > numOfPages) {
    // newPage = numOfPages
    // alternative
    newPage = 1
  }
  changePage(newPage)
}
```

#### Trigger New Page

```js
appContext.js

const getJobs = async () => {
  const { page, search, searchStatus, searchType, sort } = state

  let url = `/jobs?page=${page}&status=${searchStatus}&jobType=${searchType}&sort=${sort}`
  // rest of the code
}
```

```js
JobsContainer.js

const { page } = useAppContext()
useEffect(() => {
  getJobs()
}, [page, search, searchStatus, searchType, sort])
```

```js
reducer.js

if (action.type === HANDLE_CHANGE) {
  // set back to first page

  return { ...state, page: 1, [action.payload.name]: action.payload.value }
}
```
