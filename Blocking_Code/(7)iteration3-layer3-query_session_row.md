### Query session of rows

### (1) Session post set info ge owned items

### (2)proper response success and errors

#### Job Model

- Job Model

```js
Job.js

import mongoose from 'mongoose'

const JobSchema = new mongoose.Schema(
  {
    company: {
      type: String,
      required: [true, 'Please provide company name'],
      maxlength: 50,
    },
    position: {
      type: String,
      required: [true, 'Please provide position'],
      maxlength: 100,
    },
    status: {
      type: String,
      enum: ['interview', 'declined', 'pending'],
      default: 'pending',
    },

    jobType: {
      type: String,
      enum: ['full-time', 'part-time', 'remote', 'internship'],
      default: 'full-time',
    },
    jobLocation: {
      type: String,
      default: 'my city',
      required: true,
    },
    createdBy: {
      type: mongoose.Types.ObjectId,
      ref: 'User',
      required: [true, 'Please provide user'],
    },
  },
  { timestamps: true }
)

export default mongoose.model('Job', JobSchema)
```

#### Create Job

```js
jobsController.js

import Job from '../models/Job.js'
import { StatusCodes } from 'http-status-codes'
import { BadRequestError, NotFoundError } from '../errors/index.js'

const createJob = async (req, res) => {
  const { position, company } = req.body

  if (!position || !company) {
    throw new BadRequestError('Please Provide All Values')
  }

  req.body.createdBy = req.user.userId

  const job = await Job.create(req.body)
  res.status(StatusCodes.CREATED).json({ job })
}
```

#### Job State Values

```js
appContext.js
const initialState = {
  isEditing: false,
  editJobId: '',
  position: '',
  company: '',
  // jobLocation
  jobTypeOptions: ['full-time', 'part-time', 'remote', 'internship'],
  jobType: 'full-time',
  statusOptions: ['pending', 'interview', 'declined'],
  status: 'pending',
}
```

#### AddJob Page - Setup

```js
import { FormRow, Alert } from '../../components'
import { useAppContext } from '../../context/appContext'
import Wrapper from '../../assets/wrappers/DashboardFormPage'
const AddJob = () => {
  const {
    isEditing,
    showAlert,
    displayAlert,
    position,
    company,
    jobLocation,
    jobType,
    jobTypeOptions,
    status,
    statusOptions,
  } = useAppContext()

  const handleSubmit = (e) => {
    e.preventDefault()

    if (!position || !company || !jobLocation) {
      displayAlert()
      return
    }
    console.log('create job')
  }

  const handleJobInput = (e) => {
    const name = e.target.name
    const value = e.target.value
    console.log(`${name}:${value}`)
  }

  return (
    <Wrapper>
      <form className="form">
        <h3>{isEditing ? 'edit job' : 'add job'} </h3>
        {showAlert && <Alert />}

        {/* position */}
        <div className="form-center">
          <FormRow
            type="text"
            name="position"
            value={position}
            handleChange={handleJobInput}
          />
          {/* company */}
          <FormRow
            type="text"
            name="company"
            value={company}
            handleChange={handleJobInput}
          />
          {/* location */}
          <FormRow
            type="text"
            labelText="location"
            name="jobLocation"
            value={jobLocation}
            handleChange={handleJobInput}
          />
          {/* job type */}

          {/* job status */}

          <div className="btn-container">
            <button
              className="btn btn-block submit-btn"
              type="submit"
              onClick={handleSubmit}
            >
              submit
            </button>
          </div>
        </div>
      </form>
    </Wrapper>
  )
}

export default AddJob
```

#### Select Input

```js
return (
  // job type
  <div className="form-row">
    <label htmlFor="jobType" className="form-label">
      job type
    </label>

    <select
      name="jobType"
      value={jobType}
      onChange={handleJobInput}
      className="form-select"
    >
      {jobTypeOptions.map((itemValue, index) => {
        return (
          <option key={index} value={itemValue}>
            {itemValue}
          </option>
        )
      })}
    </select>
  </div>
)
```

#### FormRowSelect

- create FormRowSelect in components
- setup import/export

```js
const FormRowSelect = ({ labelText, name, value, handleChange, list }) => {
  return (
    <div className="form-row">
      <label htmlFor={name} className="form-label">
        {labelText || name}
      </label>

      <select
        name={name}
        value={value}
        onChange={handleChange}
        className="form-select"
      >
        {list.map((itemValue, index) => {
          return (
            <option key={index} value={itemValue}>
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

```js
AddJob.js

return (
  <>
    {/* job status */}

    <FormRowSelect
      name="status"
      value={status}
      handleChange={handleJobInput}
      list={statusOptions}
    />

    {/* job type */}
    <FormRowSelect
      labelText="type"
      name="jobType"
      value={jobType}
      handleChange={handleJobInput}
      list={jobTypeOptions}
    />
  </>
)
```

#### Change State Values With Handle Change

- [JS Nuggets Dynamic Object Keys](https://youtu.be/_qxCYtWm0tw)

```js
actions.js

export const HANDLE_CHANGE = 'HANDLE_CHANGE'
```

```js
appContext.js

const handleChange = ({ name, value }) => {
  dispatch({
    type: HANDLE_CHANGE,
    payload: { name, value },
  })
}

value={{handleChange}}
```

```js
reducer.js

if (action.type === HANDLE_CHANGE) {
  return { ...state, [action.payload.name]: action.payload.value }
}
```

```js
AddJob.js

const { handleChange } = useAppContext()

const handleJobInput = (e) => {
  handleChange({ name: e.target.name, value: e.target.value })
}
```

#### Clear Values

```js
actions.js

export const CLEAR_VALUES = 'CLEAR_VALUES'
```

```js
appContext.js

const clearValues = () => {
    dispatch({ type: CLEAR_VALUES })
  }

value={{clearValues}}
```

```js
reducer.js

if (action.type === CLEAR_VALUES) {
  const initialState = {
    isEditing: false,
    editJobId: '',
    position: '',
    company: '',
    jobLocation: state.userLocation,
    jobType: 'full-time',
    status: 'pending',
  }
  return { ...state, ...initialState }
}
```

```js
AddJob.js

const { clearValues } = useAppContext()

return (
  <div className="btn-container">
    {/* submit button */}

    <button
      className="btn btn-block clear-btn"
      onClick={(e) => {
        e.preventDefault()
        clearValues()
      }}
    >
      clear
    </button>
  </div>
)
```

#### ()Create Job

```js
actions.js

export const CREATE_JOB_BEGIN = 'CREATE_JOB_BEGIN'
export const CREATE_JOB_SUCCESS = 'CREATE_JOB_SUCCESS'
export const CREATE_JOB_ERROR = 'CREATE_JOB_ERROR'
```

```js
appContext.js

const createJob = async () => {
  dispatch({ type: CREATE_JOB_BEGIN })
  try {
    const { position, company, jobLocation, jobType, status } = state

    await authFetch.post('/jobs', {
      company,
      position,
      jobLocation,
      jobType,
      status,
    })
    dispatch({
      type: CREATE_JOB_SUCCESS,
    })
    // call function instead clearValues()
    dispatch({ type: CLEAR_VALUES })
  } catch (error) {
    if (error.response.status === 401) return
    dispatch({
      type: CREATE_JOB_ERROR,
      payload: { msg: error.response.data.msg },
    })
  }
  clearAlert()
}
```

```js
AddJob.js

const { createJob } = useAppContext()

const handleSubmit = (e) => {
  e.preventDefault()
  // while testing

  // if (!position || !company || !jobLocation) {
  //   displayAlert()
  //   return
  // }
  if (isEditing) {
    // eventually editJob()
    return
  }
  createJob()
}
```

```js
reducer.js

if (action.type === CREATE_JOB_BEGIN) {
  return { ...state, isLoading: true }
}
if (action.type === CREATE_JOB_SUCCESS) {
  return {
    ...state,
    isLoading: false,
    showAlert: true,
    alertType: 'success',
    alertText: 'New Job Created!',
  }
}
if (action.type === CREATE_JOB_ERROR) {
  return {
    ...state,
    isLoading: false,
    showAlert: true,
    alertType: 'danger',
    alertText: action.payload.msg,
  }
}
```

#### Get All Jobs

```js
jobsController.js

const getAllJobs = async (req, res) => {
  const jobs = await Job.find({ createdBy: req.user.userId })

  res
    .status(StatusCodes.OK)
    .json({ jobs, totalJobs: jobs.length, numOfPages: 1 })
}
```

#### Jobs State Values

```js
appContext.js

const initialState = {
  jobs: [],
  totalJobs: 0,
  numOfPages: 1,
  page: 1,
}
```

#### Get All Jobs Request

```js
actions.js
export const GET_JOBS_BEGIN = 'GET_JOBS_BEGIN'
export const GET_JOBS_SUCCESS = 'GET_JOBS_SUCCESS'
```

```js
appContext.js

import React, { useReducer, useContext, useEffect } from 'react'

const getJobs = async () => {
  let url = `/jobs`

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
    console.log(error.response)
    logoutUser()
  }
  clearAlert()
}

useEffect(() => {
  getJobs()
}, [])

value={{getJobs}}

```

```js
reducer.js

if (action.type === GET_JOBS_BEGIN) {
  return { ...state, isLoading: true, showAlert: false }
}
if (action.type === GET_JOBS_SUCCESS) {
  return {
    ...state,
    isLoading: false,
    jobs: action.payload.jobs,
    totalJobs: action.payload.totalJobs,
    numOfPages: action.payload.numOfPages,
  }
}
```

#### AllJobs Page Setup

- create
- SearchContainer export
- JobsContainer export
- Job
- JobInfo

```js
AllJobs.js

import { JobsContainer, SearchContainer } from '../../components'
const AllJobs = () => {
  return (
    <>
      <SearchContainer />
      <JobsContainer />
    </>
  )
}

export default AllJobs
```

```js
JobsContainer.js
import { useAppContext } from '../context/appContext'
import { useEffect } from 'react'
import Loading from './Loading'
import Job from './Job'
import Wrapper from '../assets/wrappers/JobsContainer'

const JobsContainer = () => {
  const { getJobs, jobs, isLoading, page, totalJobs } = useAppContext()
  useEffect(() => {
    getJobs()
  }, [])

  if (isLoading) {
    return <Loading center />
  }
  if (jobs.length === 0) {
    return (
      <Wrapper>
        <h2>No jobs to display...</h2>
      </Wrapper>
    )
  }
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
    </Wrapper>
  )
}

export default JobsContainer
```

```js
Job.js

import moment from 'moment'

const Job = ({ company }) => {
  return <h5>{company}</h5>
}

export default Job
```

#### Moment.js

- Format Dates
- [moment.js](https://momentjs.com/)

- stop server
- cd client

```sh
npm install moment

```

```js
Job.js

import moment from 'moment'

const Job = ({ company, createdAt }) => {
  let date = moment(createdAt)
  date = date.format('MMM Do, YYYY')
  return (
    <div>
      <h5>{company}</h5>
      <h5>{date}</h5>
    </div>
  )
}

export default Job
```

#### Job Component - Setup

```js
appContext.js

const setEditJob = (id) => {
  console.log(`set edit job : ${id}`)
}
const deleteJob = (id) =>{
  console.log(`delete : ${id}`)
}
value={{setEditJob,deleteJob}}
```

```js
Job.js

import { FaLocationArrow, FaBriefcase, FaCalendarAlt } from 'react-icons/fa'
import { Link } from 'react-router-dom'
import { useAppContext } from '../context/appContext'
import Wrapper from '../assets/wrappers/Job'
import JobInfo from './JobInfo'

const Job = ({
  _id,
  position,
  company,
  jobLocation,
  jobType,
  createdAt,
  status,
}) => {
  const { setEditJob, deleteJob } = useAppContext()

  let date = moment(createdAt)
  date = date.format('MMM Do, YYYY')

  return (
    <Wrapper>
      <header>
        <div className="main-icon">{company.charAt(0)}</div>
        <div className="info">
          <h5>{position}</h5>
          <p>{company}</p>
        </div>
      </header>
      <div className="content">
        {/* content center later */}
        <footer>
          <div className="actions">
            <Link
              to="/add-job"
              onClick={() => setEditJob(_id)}
              className="btn edit-btn"
            >
              Edit
            </Link>
            <button
              type="button"
              className="btn delete-btn"
              onClick={() => deleteJob(_id)}
            >
              Delete
            </button>
          </div>
        </footer>
      </div>
    </Wrapper>
  )
}

export default Job
```

#### JobInfo

```js
JobInfo.js

import Wrapper from '../assets/wrappers/JobInfo'

const JobInfo = ({ icon, text }) => {
  return (
    <Wrapper>
      <span className="icon">{icon}</span>
      <span className="text">{text}</span>
    </Wrapper>
  )
}

export default JobInfo
```

```js
Job.js
return (
  <div className="content">
    <div className="content-center">
      <JobInfo icon={<FaLocationArrow />} text={jobLocation} />
      <JobInfo icon={<FaCalendarAlt />} text={date} />
      <JobInfo icon={<FaBriefcase />} text={jobType} />
      <div className={`status ${status}`}>{status}</div>
    </div>
    {/* footer content */}
  </div>
)
```

### (3)Further processing with id trigger and validated

#### SetEditJob

```js
actions.js
export const SET_EDIT_JOB = 'SET_EDIT_JOB'
```

```js
appContext.js

const setEditJob = (id) => {
  dispatch({ type: SET_EDIT_JOB, payload: { id } })
}
const editJob = () => {
  console.log('edit job')
}
value={{editJob}}
```

```js
reducer.js

if (action.type === SET_EDIT_JOB) {
  const job = state.jobs.find((job) => job._id === action.payload.id)
  const { _id, position, company, jobLocation, jobType, status } = job
  return {
    ...state,
    isEditing: true,
    editJobId: _id,
    position,
    company,
    jobLocation,
    jobType,
    status,
  }
}
```

```js
AddJob.js
const { isEditing, editJob } = useAppContext()
const handleSubmit = (e) => {
  e.preventDefault()

  if (!position || !company || !jobLocation) {
    displayAlert()
    return
  }
  if (isEditing) {
    editJob()
    return
  }
  createJob()
}
```

#### Edit Job - Server

```js
jobsController.js

const updateJob = async (req, res) => {
  const { id: jobId } = req.params

  const { company, position } = req.body

  if (!company || !position) {
    throw new BadRequestError('Please Provide All Values')
  }

  const job = await Job.findOne({ _id: jobId })

  if (!job) {
    throw new NotFoundError(`No job with id ${jobId}`)
  }

  // check permissions

  const updatedJob = await Job.findOneAndUpdate({ _id: jobId }, req.body, {
    new: true,
    runValidators: true,
  })

  res.status(StatusCodes.OK).json({ updatedJob })
}
```

#### Alternative Approach

- optional
- multiple approaches
- different setups
- course Q&A

```js
jobsController.js
const updateJob = async (req, res) => {
  const { id: jobId } = req.params
  const { company, position, jobLocation } = req.body

  if (!position || !company) {
    throw new BadRequestError('Please provide all values')
  }
  const job = await Job.findOne({ _id: jobId })

  if (!job) {
    throw new NotFoundError(`No job with id :${jobId}`)
  }

  // check permissions

  // alternative approach

  job.position = position
  job.company = company
  job.jobLocation = jobLocation

  await job.save()
  res.status(StatusCodes.OK).json({ job })
}
```

#### Check Permissions

```js
jobsController.js

const updateJob = async (req, res) => {
  const { id: jobId } = req.params
  const { company, position, status } = req.body

  if (!position || !company) {
    throw new BadRequestError('Please provide all values')
  }
  const job = await Job.findOne({ _id: jobId })

  if (!job) {
    throw new NotFoundError(`No job with id :${jobId}`)
  }

  // check permissions
  // req.user.userId (string) === job.createdBy(object)
  // throw new UnAuthenticatedError('Not authorized to access this route')

  // console.log(typeof req.user.userId)
  // console.log(typeof job.createdBy)

  checkPermissions(req.user, job.createdBy)

  const updatedJob = await Job.findOneAndUpdate({ _id: jobId }, req.body, {
    new: true,
    runValidators: true,
  })

  res.status(StatusCodes.OK).json({ updatedJob })
}
```

- utils folder
- checkPermissions.js
- import in jobsController.js

```js
checkPermissions.js

import { UnAuthenticatedError } from '../errors/index.js'

const checkPermissions = (requestUser, resourceUserId) => {
  // if (requestUser.role === 'admin') return
  if (requestUser.userId === resourceUserId.toString()) return
  throw new UnauthorizedError('Not authorized to access this route')
}

export default checkPermissions
```

#### Remove/Delete Job

```js
jobsController.js

const deleteJob = async (req, res) => {
  const { id: jobId } = req.params

  const job = await Job.findOne({ _id: jobId })

  if (!job) {
    throw new NotFoundError(`No job with id : ${jobId}`)
  }

  checkPermissions(req.user, job.createdBy)

  await job.remove()
  res.status(StatusCodes.OK).json({ msg: 'Success! Job removed' })
}
```

#### Delete Job - Front-End

```js
actions.js

export const DELETE_JOB_BEGIN = 'DELETE_JOB_BEGIN'
```

```js
appContext.js

const deleteJob = async (jobId) => {
  dispatch({ type: DELETE_JOB_BEGIN })
  try {
    await authFetch.delete(`/jobs/${jobId}`)
    getJobs()
  } catch (error) {
    logoutUser()
  }
}
```

```js
reducer.js

if (action.type === DELETE_JOB_BEGIN) {
  return { ...state, isLoading: true }
}
```

#### Edit Job - Front-End

```js
actions.js
export const EDIT_JOB_BEGIN = 'EDIT_JOB_BEGIN'
export const EDIT_JOB_SUCCESS = 'EDIT_JOB_SUCCESS'
export const EDIT_JOB_ERROR = 'EDIT_JOB_ERROR'
```

```js
appContext.js
const editJob = async () => {
  dispatch({ type: EDIT_JOB_BEGIN })
  try {
    const { position, company, jobLocation, jobType, status } = state

    await authFetch.patch(`/jobs/${state.editJobId}`, {
      company,
      position,
      jobLocation,
      jobType,
      status,
    })
    dispatch({
      type: EDIT_JOB_SUCCESS,
    })
    dispatch({ type: CLEAR_VALUES })
  } catch (error) {
    if (error.response.status === 401) return
    dispatch({
      type: EDIT_JOB_ERROR,
      payload: { msg: error.response.data.msg },
    })
  }
  clearAlert()
}
```

```js
reducer.js

if (action.type === EDIT_JOB_BEGIN) {
  return { ...state, isLoading: true }
}
if (action.type === EDIT_JOB_SUCCESS) {
  return {
    ...state,
    isLoading: false,
    showAlert: true,
    alertType: 'success',
    alertText: 'Job Updated!',
  }
}
if (action.type === EDIT_JOB_ERROR) {
  return {
    ...state,
    isLoading: false,
    showAlert: true,
    alertType: 'danger',
    alertText: action.payload.msg,
  }
}
```

#### Create More Jobs

- [Mockaroo](https://www.mockaroo.com/)
- setup mock-data.json in the root
