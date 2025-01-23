# put inside iteration

## Columns in table

### (1)Satisfy Data dependence

#### Populate Database

- create populate.js in the root

```js
populate.js

import { readFile } from 'fs/promises'

import dotenv from 'dotenv'
dotenv.config()

import connectDB from './db/connect.js'
import Job from './models/Job.js'

const start = async () => {
  try {
    await connectDB(process.env.MONGO_URL)
    await Job.deleteMany()

    const jsonProducts = JSON.parse(
      await readFile(new URL('./mock-data.json', import.meta.url))
    )
    await Job.create(jsonProducts)
    console.log('Success!!!!')
    process.exit(0)
  } catch (error) {
    console.log(error)
    process.exit(1)
  }
}

start()
```

#### Show Stats - Structure

- aggregation pipeline
- step by step
- [Aggregation Pipeline](https://docs.mongodb.com/manual/core/aggregation-pipeline/)

```js
jobsController.js

import mongoose from 'mongoose'

const showStats = async (req, res) => {
  let stats = await Job.aggregate([
    { $match: { createdBy: mongoose.Types.ObjectId(req.user.userId) } },
    { $group: { _id: '$status', count: { $sum: 1 } } },
  ])

  res.status(StatusCodes.OK).json({ stats })
}
```

### (2) Derive column from table

#### Show Stats - Object Setup

- [Reduce Basics](https://youtu.be/3WkW9nrS2mw)
- [Reduce Object Example ](https://youtu.be/5BFkp8JjLEY)

```js
jobsController.js

const showStats = async (req, res) => {
  let stats = await Job.aggregate([
    { $match: { createdBy: mongoose.Types.ObjectId(req.user.userId) } },
    { $group: { _id: '$status', count: { $sum: 1 } } },
  ])

  stats = stats.reduce((acc, curr) => {
    const { _id: title, count } = curr
    acc[title] = count
    return acc
  }, {})

  res.status(StatusCodes.OK).json({ stats })
}
```

#### Show Stats - Default Stats

```js
jobsController.js

const showStats = async (req, res) => {
  let stats = await Job.aggregate([
    { $match: { createdBy: mongoose.Types.ObjectId(req.user.userId) } },
    { $group: { _id: '$status', count: { $sum: 1 } } },
  ])
  stats = stats.reduce((acc, curr) => {
    const { _id: title, count } = curr
    acc[title] = count
    return acc
  }, {})

  const defaultStats = {
    pending: stats.pending || 0,
    interview: stats.interview || 0,
    declined: stats.declined || 0,
  }
  let monthlyApplications = []
  res.status(StatusCodes.OK).json({ defaultStats, monthlyApplications })
}
```

#### Show Stats - Function Setup

```js
actions.js

export const SHOW_STATS_BEGIN = 'SHOW_STATS_BEGIN'
export const SHOW_STATS_SUCCESS = 'SHOW_STATS_SUCCESS'
```

```js
appContext.js

const initialState = {
  stats: {},
  monthlyApplications: []

}

const showStats = async () => {
    dispatch({ type: SHOW_STATS_BEGIN })
    try {
      const { data } = await authFetch('/jobs/stats')
      dispatch({
        type: SHOW_STATS_SUCCESS,
        payload: {
          stats: data.defaultStats,
          monthlyApplications: data.monthlyApplications,
        },
      })
    } catch (error) {
console.log(error.response)
      // logoutUser()
    }

clearAlert()
  }
  value={{showStats}}
```

```js
reducers.js
if (action.type === SHOW_STATS_BEGIN) {
  return { ...state, isLoading: true, showAlert: false }
}
if (action.type === SHOW_STATS_SUCCESS) {
  return {
    ...state,
    isLoading: false,
    stats: action.payload.stats,
    monthlyApplications: action.payload.monthlyApplications,
  }
}
```

#### Stats Page - Structure

- components
- StatsContainer.js
- ChartsContainer.js
- StatsItem.js
- simple return
- import/export index.js

```js
Stats.js

import { useEffect } from 'react'
import { useAppContext } from '../../context/appContext'
import { StatsContainer, Loading, ChartsContainer } from '../../components'

const Stats = () => {
  const { showStats, isLoading, monthlyApplications } = useAppContext()
  useEffect(() => {
    showStats()
  }, [])

  if (isLoading) {
    return <Loading center />
  }

  return (
    <>
      <StatsContainer />
      {monthlyApplications.length > 0 && <ChartsContainer />}
    </>
  )
}

export default Stats
```

#### StatsContainer

```js
StatsContainer.js

import { useAppContext } from '../context/appContext'
import StatItem from './StatItem'
import { FaSuitcaseRolling, FaCalendarCheck, FaBug } from 'react-icons/fa'
import Wrapper from '../assets/wrappers/StatsContainer'
const StatsContainer = () => {
  const { stats } = useAppContext()
  const defaultStats = [
    {
      title: 'pending applications',
      count: stats.pending || 0,
      icon: <FaSuitcaseRolling />,
      color: '#e9b949',
      bcg: '#fcefc7',
    },
    {
      title: 'interviews scheduled',
      count: stats.interview || 0,
      icon: <FaCalendarCheck />,
      color: '#647acb',
      bcg: '#e0e8f9',
    },
    {
      title: 'jobs declined',
      count: stats.declined || 0,
      icon: <FaBug />,
      color: '#d66a6a',
      bcg: '#ffeeee',
    },
  ]

  return (
    <Wrapper>
      {defaultStats.map((item, index) => {
        return <StatItem key={index} {...item} />
      })}
    </Wrapper>
  )
}

export default StatsContainer
```

#### StatItem

```js
StatItem.js

import Wrapper from '../assets/wrappers/StatItem'

function StatItem({ count, title, icon, color, bcg }) {
  return (
    <Wrapper color={color} bcg={bcg}>
      <header>
        <span className="count">{count}</span>
        <div className="icon">{icon}</div>
      </header>
      <h5 className="title">{title}</h5>
    </Wrapper>
  )
}

export default StatItem
```

### (3) Further processing of column

#### Aggregate Jobs Based on Year and Month

```js
jobsController.js

let monthlyApplications = await Job.aggregate([
  { $match: { createdBy: mongoose.Types.ObjectId(req.user.userId) } },
  {
    $group: {
      _id: {
        year: {
          $year: '$createdAt',
        },
        month: {
          $month: '$createdAt',
        },
      },
      count: { $sum: 1 },
    },
  },
  { $sort: { '_id.year': -1, '_id.month': -1 } },
  { $limit: 6 },
])
```

#### Refactor Data

- install moment.js on the SERVER

```sh
npm install moment

```

```js
jobsController.js

import moment from 'moment'

monthlyApplications = monthlyApplications
  .map((item) => {
    const {
      _id: { year, month },
      count,
    } = item
    // accepts 0-11
    const date = moment()
      .month(month - 1)
      .year(year)
      .format('MMM Y')
    return { date, count }
  })
  .reverse()
```

#### Charts Container

- BarChart.js
- AreaChart.js

```js
ChartsContainer.js
import React, { useState } from 'react'

import BarChart from './BarChart'
import AreaChart from './AreaChart'
import { useAppContext } from '../context/appContext'
import Wrapper from '../assets/wrappers/ChartsContainer'

export default function ChartsContainer() {
  const [barChart, setBarChart] = useState(true)
  const { monthlyApplications: data } = useAppContext()

  return (
    <Wrapper>
      <h4>Monthly Applications</h4>

      <button type="button" onClick={() => setBarChart(!barChart)}>
        {barChart ? 'AreaChart' : 'BarChart'}
      </button>
      {barChart ? <BarChart data={data} /> : <AreaChart data={data} />}
    </Wrapper>
  )
}
```

#### Recharts Library

- install in the Client!!!

[Recharts](https://recharts.org)

```sh
npm install recharts
```

#### Bar Chart

```js
BarChart.js

import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
} from 'recharts'

const BarChartComponent = ({ data }) => {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <BarChart
        data={data}
        margin={{
          top: 50,
        }}
      >
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis allowDecimals={false} />
        <Tooltip />
        <Bar dataKey="count" fill="#2cb1bc" barSize={75} />
      </BarChart>
    </ResponsiveContainer>
  )
}
```

#### Area Chart

```js
import {
  ResponsiveContainer,
  AreaChart,
  Area,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
} from 'recharts'

const AreaChartComponent = ({ data }) => {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <AreaChart
        data={data}
        margin={{
          top: 50,
        }}
      >
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis allowDecimals={false} />
        <Tooltip />
        <Area type="monotone" dataKey="count" stroke="#2cb1bc" fill="#bef8fd" />
      </AreaChart>
    </ResponsiveContainer>
  )
}
```
