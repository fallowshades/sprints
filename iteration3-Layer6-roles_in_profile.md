# iteration3 layer 6 roles in profile

[extend-role-extend-login]

{...larger obj}

[container-map-with-extra-parts-in-layout]

- map card
- map data with extra child elements

count occurence of key in key:value pair groups

## Test user

### (1)Front static async

#### 1. Test User

- create test user
- feel free to use one of the chatGPT options

```json
{
  "name": "Zippy",
  "email": "test@test.com",
  "password": "secret123",
  "lastName": "ShakeAndBake",
  "location": "Codeville"
}
{
  "name": "Chuckleberry",
  "email": "test@test.com",
  "password": "secret123",
  "lastName": "Gigglepants",
  "location": "Laughterland"
}

{
  "name": "Bubbles McLaughster",
  "email": "test@test.com",
  "password": "secret123",
  "lastName": "Ticklebottom",
  "location": "Giggle City"
}


{
  "name": "Gigglesworth",
  "email": "test@test.com",
  "password": "secret123",
  "lastName": "Snickerdoodle",
  "location": "Chuckleburg"
}
```

#### 2. Test User - Login Page

```js
import { useNavigate } from 'react-router-dom';

const Login = () => {
  const navigate = useNavigate();
  const loginDemoUser = async () => {
    const data = {
      email: 'test@test.com',
      password: 'secret123',
    };
    try {
      await customFetch.post('/auth/login', data);
      toast.success('take a test drive');
      navigate('/dashboard');
    } catch (error) {
      toast.error(error?.response?.data?.msg);
    }
  };
  return (
    <Wrapper>
      ...
        <button type='button' className='btn btn-block' onClick={loginDemoUser}>
          explore the app
        </button>
        ...
      </Form>
    </Wrapper>
  );
};
export default Login;
```

#### 3. Test User - Restrict Access

authMiddleware

```js
import {
  BadRequestError,
} from '../errors/customErrors.js';

export const authenticateUser = (req, res, next) => {
  ...
  try {
    const { userId, role } = verifyJWT(token);
    const testUser = userId === 'testUserId';
    req.user = { userId, role, testUser };
    next();
  }
  ....
};

export const checkForTestUser = (req, res, next) => {
  if (req.user.testUser) {
    throw new BadRequestError('Demo User. Read Only!');
  }
  next();
};

```

- add to updateUser, createAchievement, updateAchievement, deleteAchievement

### (2)backend accumulate \* users before ctrl

#### 4. Mock Data

[Mockaroo ](https://www.mockaroo.com/)

```json
{
  "description": "Cogidoo",
  "status": "Help Desk Technician",
  "points": "Vyksa",
  "type": "pending",
  "dateOfCompletion": "part-time",
  "createdAt": "2022-07-25T21:26:23Z"
}
```

- rename and save json in utils

#### 5. Populate DB

- create populate.js
- setup for test user and admin

```js
import { readFile } from 'fs/promises'
import mongoose from 'mongoose'
import dotenv from 'dotenv'
dotenv.config()

import Achievement from './models/AchievementModel.js'
import User from './models/UserModel.js'
try {
  await mongoose.connect(process.env.MONGO_URL)
  // const user = await User.findOne({ email: 'john@gmail.com' });
  const user = await User.findOne({ email: 'test@test.com' })

  const jsonAchievements = JSON.parse(
    await readFile(new URL('./utils/mockAchievementData.json', import.meta.url))
  )
  const achievements = jsonAchievements.map((achievement) => {
    return { ...achievement, createdBy: user._id }
  })
  await Achievement.deleteMany({ createdBy: user._id })
  await Achievement.create(achievements)
  console.log('Success!!!')
  process.exit(0)
} catch (error) {
  console.log(error)
  process.exit(1)
}
```

## Stats

### (1) Populate /stat set up

#### 6. Stats - Setup

- create controller
- setup route and thunder client
- install/setup dayjs on the server

achievementController.js

```js
import mongoose from 'mongoose'
import day from 'dayjs'

export const showStats = async (req, res) => {
  const defaultStats = {
    pending: 22,
    interview: 11,
    declined: 4,
  }

  let monthlyApplications = [
    {
      date: 'May 23',
      count: 12,
    },
    {
      date: 'Jun 23',
      count: 9,
    },
    {
      date: 'Jul 23',
      count: 3,
    },
  ]
  res.status(StatusCodes.OK).json({ defaultStats, monthlyApplications })
}
```

#### 7. Stats - Complete Server Functionality

[MongoDB Docs](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/)

The MongoDB aggregation pipeline is like a factory line for data. Data enters, it goes through different stages like cleaning, sorting, or grouping, and comes out at the end changed in some way. It's a way to process data inside MongoDB.

achievementController.js

```js
export const showStats = async (req, res) => {
  let stats = await Achievement.aggregate([
    { $match: { createdBy: new mongoose.Types.ObjectId(req.user.userId) } },
    { $group: { _id: '$status', count: { $sum: 1 } } },
  ])
  stats = stats.reduce((acc, curr) => {
    const { _id: title, count } = curr
    acc[title] = count
    return acc
  }, {})

  const defaultStats = {
    inactive: stats.inactive || 0,
    activated: stats.activated || 0,
    complete: stats.complete || 0,
  }

  let monthlyApplications = await Achievement.aggregate([
    { $match: { createdBy: new mongoose.Types.ObjectId(req.user.userId) } },
    {
      $group: {
        _id: { year: { $year: '$createdAt' }, month: { $month: '$createdAt' } },
        count: { $sum: 1 },
      },
    },
    { $sort: { '_id.year': -1, '_id.month': -1 } },
    { $limit: 6 },
  ])
  monthlyApplications = monthlyApplications
    .map((item) => {
      const {
        _id: { year, month },
        count,
      } = item

      const date = day()
        .month(month - 1)
        .year(year)
        .format('MMM YY')
      return { date, count }
    })
    .reverse()

  res.status(StatusCodes.OK).json({ defaultStats, monthlyApplications })
}
```

#### Commentary

```js
let stats = await Achievement.aggregate([
  { $match: { createdBy: new mongoose.Types.ObjectId(req.user.userId) } },
  { $group: { _id: '$status', count: { $sum: 1 } } },
])
```

let stats = await Achievement.aggregate([ ... ]); This line says we're going to perform an aggregation operation on the Achievement collection in MongoDB and save the result in a variable called stats. The await keyword is used to wait for the operation to finish before continuing, as the operation is asynchronous (i.e., it runs in the background).

{ $match: { createdBy: new mongoose.Types.ObjectId(req.user.userId) } } This is the first stage of the pipeline. It filters the Achievements so that only the ones created by the user specified by req.user.userId are passed to the next stage. The new mongoose.Types.ObjectId(req.user.userId) part converts req.user.userId into an ObjectId (which is the format MongoDB uses for ids).

{ $group: { _id: '$achievementStatus', count: { $sum: 1 } } } This is the second stage of the pipeline. It groups the remaining Achievements by their status (the achievementStatus field). For each group, it calculates the count of achievements by adding 1 for each achievement ({ $sum: 1 }), and stores this in a field called count.

```js
let monthlyApplications = await Achievement.aggregate([
  { $match: { createdBy: new mongoose.Types.ObjectId(req.user.userId) } },
  {
    $group: {
      _id: { year: { $year: '$createdAt' }, month: { $month: '$createdAt' } },
      count: { $sum: 1 },
    },
  },
  { $sort: { '_id.year': -1, '_id.month': -1 } },
  { $limit: 6 },
])
```

let monthlyApplications = await Achievement.aggregate([ ... ]); This line indicates that an aggregation operation will be performed on the Achievement collection in MongoDB. The result will be stored in the variable monthlyApplications. The await keyword ensures that the code waits for this operation to complete before proceeding, as it is an asynchronous operation.

{ $match: { createdBy: new mongoose.Types.ObjectId(req.user.userId) } } This is the first stage of the pipeline. It filters the achievements to only those created by the user identified by req.user.userId.

{ $group: { _id: { year: { $year: '$createdAt' }, month: { $month: '$createdAt' } }, count: { $sum: 1 } } } This is the second stage of the pipeline. It groups the remaining achievements based on the year and month when they were created. For each group, it calculates the count of achievements by adding 1 for each achievement in the group.

{ $sort: { '\_id.year': -1, '\_id.month': -1 } } This is the third stage of the pipeline. It sorts the groups by year and month in descending order. The -1 indicates descending order. So it starts with the most recent year and month.

{ $limit: 6 } This is the fourth and last stage of the pipeline. It limits the output to the top 6 groups, after sorting. This is effectively getting the achievement count for the last 6 months.

So, monthlyApplications will be an array with up to 6 elements, each representing the number of achievements created by the user in a specific month and year. The array will be sorted by year and month, starting with the most recent.

### (2) map loading stats

#### 8. Stats - Front-End Setup

- create four components
- StatsContainer and ChartsContainer (import/export)
- AreaChart, BarChart (local)

pages/Stats.jsx

```js
import { ChartsContainer, StatsContainer } from '../components'
import customFetch from '../utils/customFetch'
import { useLoaderData } from 'react-router-dom'
export const loader = async () => {
  try {
    const response = await customFetch.get('/achievements/stats')
    return response.data
  } catch (error) {
    return error
  }
}

const Stats = () => {
  const { defaultStats, monthlyApplications } = useLoaderData()
  return (
    <>
      <StatsContainer defaultStats={defaultStats} />
      {monthlyApplications?.length > 0 && (
        <ChartsContainer data={monthlyApplications} />
      )}
    </>
  )
}
export default Stats
```

App.jsx

```js
import { loader as statsLoader } from './pages/Stats'
 { path: 'stats', element: <Stats />, loader: statsLoader },
```

#### 9. Stats Container

```js
import { FaSuitcaseRolling, FaCalendarCheck, FaBug } from 'react-icons/fa'
import Wrapper from '../assets/wrappers/StatsContainer'
import StatItem from './StatItem'
const StatsContainer = ({ defaultStats }) => {
  const stats = [
    {
      title: 'pending applications',
      count: defaultStats?.inactive || 0,
      icon: <FaSuitcaseRolling />,
      color: '#f59e0b',
      bcg: '#fef3c7',
    },
    {
      title: 'interviews scheduled',
      count: defaultStats?.active || 0,
      icon: <FaCalendarCheck />,
      color: '#647acb',
      bcg: '#e0e8f9',
    },
    {
      title: 'achievements declined',
      count: defaultStats?.complete || 0,
      icon: <FaBug />,
      color: '#d66a6a',
      bcg: '#ffeeee',
    },
  ]
  return (
    <Wrapper>
      {stats.map((item) => {
        return <StatItem key={item.title} {...item} />
      })}
    </Wrapper>
  )
}
export default StatsContainer
```

### (3) charts container

### charts

#### 10. ChartsContainer

```js
import { useState } from 'react'

import BarChart from './BarChart'
import AreaChart from './AreaChart'
import Wrapper from '../assets/wrappers/ChartsContainer'

const ChartsContainer = ({ data }) => {
  const [barChart, setBarChart] = useState(true)

  return (
    <Wrapper>
      <h4>Monthly Applications</h4>
      <button type='button' onClick={() => setBarChart(!barChart)}>
        {barChart ? 'Area Chart' : 'Bar Chart'}
      </button>
      {barChart ? <BarChart data={data} /> : <AreaChart data={data} />}
    </Wrapper>
  )
}

export default ChartsContainer
```

#### 11. Charts

[recharts](https://recharts.org/en-US/)

- in the client

```sh
npm i recharts@2.5.0
```

#### 12. Area Chart

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
    <ResponsiveContainer width='100%' height={300}>
      <AreaChart data={data} margin={{ top: 50 }}>
        <CartesianGrid strokeDasharray='3 3' />
        <XAxis dataKey='date' />
        <YAxis allowDecimals={false} />
        <Tooltip />
        <Area type='monotone' dataKey='count' stroke='#2cb1bc' fill='#bef8fd' />
      </AreaChart>
    </ResponsiveContainer>
  )
}

export default AreaChartComponent
```

#### 13. Bar Chart

```js
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
    <ResponsiveContainer width='100%' height={300}>
      <BarChart data={data} margin={{ top: 50 }}>
        <CartesianGrid strokeDasharray='3 3 ' />
        <XAxis dataKey='date' />
        <YAxis allowDecimals={false} />
        <Tooltip />
        <Bar dataKey='count' fill='#2cb1bc' barSize={75} />
      </BarChart>
    </ResponsiveContainer>
  )
}

export default BarChartComponent
```

#### 14. Charts CSS (optional)

wrappers/ChartsContainer.js

```js
import styled from 'styled-components'

const Wrapper = styled.section`
  margin-top: 4rem;
  text-align: center;
  button {
    background: transparent;
    border-color: transparent;
    text-transform: capitalize;
    color: var(--primary-5ma00);
    font-size: 1.25rem;
    cursor: pointer;
  }
  h4 {
    text-align: center;
    margin-bottom: 0.75rem;
  }
`

export default Wrapper
```
