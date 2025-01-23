# Iteration 4 layer: 2 Optimization

## Stacked logic of 2 layers

### (1)f() in arg for shared behavior

#### Setup Global Loading

- create loading component (import/export)
- check for loading in DashboardLayout page

components/Loading.jsx

```js
const Loading = () => {
  return <div className="loading"></div>
}

export default Loading
```

DashboardLayout.jsx

```js
import { useNavigation } from 'react-router-dom'
import { Loading } from '../components'

const DashboardLayout = ({ isDarkThemeEnabled }) => {
  const navigation = useNavigation()
  const isPageLoading = navigation.state === 'loading'

  return (
    <Wrapper>
      ...
      <div className="dashboard-page">
        {isPageLoading ? <Loading /> : <Outlet context={{ user }} />}
      </div>
      ...
    </Wrapper>
  )
}
```

#### React Query

React Query is a powerful library that simplifies data fetching, caching, and synchronization in React applications. It provides a declarative and intuitive way to manage remote data by abstracting away the complex logic of fetching and caching data from APIs. React Query offers features like automatic background data refetching, optimistic updates, pagination support, and more, making it easier to build performant and responsive applications that rely on fetching and manipulating data.

[React Query Docs](https://tanstack.com/query/v4/docs/react/overview)

- in the client

```sh
npm i @tanstack/react-query@4.29.5 @tanstack/react-query-devtools@4.29.6
```

App.jsx

```js
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,
    },
  },
})

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

#### Page Error Element

- create components/ErrorElement

```js
import { useRouteError } from 'react-router-dom'

const Error = () => {
  const error = useRouteError()
  console.log(error)
  return <h4>There was an error...</h4>
}
export default ErrorElement
```

Stats.jsx

```js
export const loader = async () => {
  const response = await customFetch.get('/achievements/stats')
  return response.data
}
```

App.jsx

```js
{
  path: 'stats',
  element: <Stats />,
  loader: statsLoader,
  errorElement: <h4>There was an error...</h4>
},
```

```js
{
  path: 'stats',
  element: <Stats />,
  loader: statsLoader,
  errorElement: <ErrorElement />,
},
```

### (2)connect

### (3)dashboard when data null

#### First Query

- navigate to stats

Stats.jsx

```js
import { ChartsContainer, StatsContainer } from '../components'
import customFetch from '../utils/customFetch'
import { useLoaderData } from 'react-router-dom'
import { useQuery } from '@tanstack/react-query'

export const loader = async () => {
  return null
}

const Stats = () => {
  const response = useQuery({
    queryKey: ['stats'],
    queryFn: () => customFetch.get('/achievements/stats'),
  })
  console.log(response)
  if (response.isLoading) {
    return <h1>Loading...</h1>
  }
  return <h1>react query</h1>
  return (
    <>
      <StatsContainer defaultStats={defaultStats} />
      {monthlyApplications?.length > 1 && (
        <ChartsContainer data={monthlyApplications} />
      )}
    </>
  )
}
export default Stats
```

```js
const data = useQuery({
  queryKey: ['stats'],
  queryFn: () => customFetch.get('/achievements/stats'),
})
```

const data = useQuery({ ... });: This line declares a constant variable named data and assigns it the result of the useQuery hook. The useQuery hook is provided by React Query and is used to perform data fetching.

queryKey: ['stats'],: The queryKey property is an array that serves as a unique identifier for the query. In this case, the query key is set to ['stats'], indicating that this query is fetching statistics related to achievements.

queryFn: () => customFetch.get('/achievements/stats'),: omFetch.get('/achievements/stats'). The customFetch object is likely a custom wrapper around the fetch function or an external HTTP client library, used to make the actual API request to retrieve achievement statistics.In React Query, the quThe queryFn property specifies the function that will be executed when the query is triggered. In this case, it uses an arrow function that calls custeryFn property expects a function that returns a promise. The promise should resolve with the data you want to fetch and store in the query cache.

customFetch.get('/achievements/stats'): This line is making an HTTP GET request to the /achievements/stats endpoint, which is the API route that provides the achievement statistics data.

#### Get Stats with React Query

```js
const statsQuery = {
  queryKey: ['stats'],
  queryFn: async () => {
    const response = await customFetch.get('/achievements/stats')
    return response.data
  },
}

export const loader = async () => {
  return null
}

const Stats = () => {
  const { isLoading, isError, data } = useQuery(statsQuery)

  if (isLoading) return <h4>Loading...</h4>
  if (isError) return <h4>Error...</h4>
  // after loading/error or ?.
  const { defaultStats, monthlyApplications } = data

  return (
    <>
      <StatsContainer defaultStats={defaultStats} />
      {monthlyApplications?.length > 1 && (
        <ChartsContainer data={monthlyApplications} />
      )}
    </>
  )
}
export default Stats
```

#### React Query in Stats Loader

App.jsx

```js
{
  path: 'stats',
  element: <Stats />,
  loader: statsLoader(queryClient),
  errorElement: <ErrorElement />,
},
```

Stats.jsx

```js
import { ChartsContainer, StatsContainer } from '../components'
import customFetch from '../utils/customFetch'
import { useQuery } from '@tanstack/react-query'

const statsQuery = {
  queryKey: ['stats'],
  queryFn: async () => {
    const response = await customFetch.get('/achievements/stats')
    return response.data
  },
}

export const loader = (queryClient) => async () => {
  const data = await queryClient.ensureQueryData(statsQuery)
  return data
}

const Stats = () => {
  const { data } = useQuery(statsQuery)
  const { defaultStats, monthlyApplications } = data

  return (
    <>
      <StatsContainer defaultStats={defaultStats} />
      {monthlyApplications?.length > 1 && (
        <ChartsContainer data={monthlyApplications} />
      )}
    </>
  )
}
export default Stats
```

#### React Query for Current User

App.jsx

```js
      {
        path: 'dashboard',
        element: (
          <DashboardLayout
            isDarkThemeEnabled={isDarkThemeEnabled}
            queryClient={queryClient}
          />
        ),
```

DashboardLayout.jsx

```js
import { useQuery } from '@tanstack/react-query'

const userQuery = {
  queryKey: ['user'],
  queryFn: async () => {
    const { data } = await customFetch.get('/users/current-user')
    return data
  },
}

export const loader = (queryClient) => async () => {
  try {
    return await queryClient.ensureQueryData(userQuery)
  } catch (error) {
    return redirect('/')
  }
}

const Dashboard = ({ prefersDarkMode, queryClient }) => {
  const { user } = useQuery(userQuery)?.data
}
```

## return handle load

### ()

#### Invalidate Queries

App.jsx

```js
    {
        path: 'login',
        element: <Login />,
        action: loginAction(queryClient),
      },
```

Login.jsx

```js
export const action =
  (queryClient) =>
  async ({ request }) => {
    const formData = await request.formData()
    const data = Object.fromEntries(formData)
    try {
      await axios.post('/api/v1/auth/login', data)
      queryClient.invalidateQueries()
      toast.success('Login successful')
      return redirect('/dashboard')
    } catch (error) {
      toast.error(error.response.data.msg)
      return error
    }
  }
```

DashboardLayout.jsx

```js
const logoutUser = async () => {
  navigate('/')
  await customFetch.get('/auth/logout')
  queryClient.invalidateQueries()
  toast.success('Logging out...')
}
```

Profile.jsx

```js
export const action =
  (queryClient) =>
  async ({ request }) => {
    const formData = await request.formData()
    const file = formData.get('avatar')
    if (file && file.size > 500000) {
      toast.error('Image size too large')
      return null
    }
    try {
      await customFetch.patch('/users/update-user', formData)
      queryClient.invalidateQueries(['user'])
      toast.success('Profile updated successfully')
    } catch (error) {
      toast.error(error?.response?.data?.msg)
      return null
    }
  }
```

## all achievements

#### All Achievements Query

app.jsx

```js

```

AllAchievements.jsx

```js
import { toast } from 'react-toastify'
import { AchievementsContainer, SearchContainer } from '../components'
import customFetch from '../utils/customFetch'
import { useLoaderData } from 'react-router-dom'
import { useContext, createContext } from 'react'
import { useQuery } from '@tanstack/react-query'
const AllAchievementsContext = createContext()

const allAchievementsQuery = (params) => {
  const { search, status, type, sort, page } = params
  return {
    queryKey: [
      'achievements',
      search ?? 'all',
      atatus ?? 'all',
      type ?? 'all',
      sort ?? 'all',
      page ?? 1,
    ],
    queryFn: async () => {
      const { data } = await customFetch.get('/achievements', {
        params,
      })
      return data
    },
  }
}

export const loader =
  (queryClient) =>
  async ({ request }) => {
    try {
      const params = Object.fromEntries([
        ...new URL(request.url).searchParams.entries(),
      ])
      //mb copied wrong
      // const url = new URL(request.url)
      // const search = url.searchParams.get('search')
      // const status = url.searchParams.get('status')
      // const type = url.searchParams.get('type')
      // const sort = url.searchParams.get('sort')
      // const page = url.searchParams.get('page')
      // const params = { status, type, sort, page }
      // if (search) {
      //   params.search = search
      // }
      await queryClient.ensureQueryData(allAchievementsQuery(params))

      return {
        searchValues: { search, status, type, sort, page },
      }
    } catch (error) {
      toast.error(error.response.data.msg)
      return error
    }
  }

const AllAchievements = () => {
  const { searchValues } = useLoaderData()
  const { data } = useQuery(allAchievementsQuery(searchValues))
  return (
    <AllAchievementContext.Provider value={{ data, searchValues }}>
      <SearchContainer />
      <AchievementsContainer />
    </AllAchievementContext.Provider>
  )
}
export default AllAchievements

export const useAllAchievementsContext = () =>
  useContext(AllAchievementsContext)
```

#### Invalidate Achievements

AddAchievement.jsx

```js
export const action =
  (queryClient) =>
  async ({ request }) => {
    const formData = await request.formData()
    const data = Object.fromEntries(formData)
    try {
      await customFetch.post('/achievements', data)
      queryClient.invalidateQueries(['achievements'])
      toast.success('achievement added successfully ')
      return redirect('all-achievements')
    } catch (error) {
      toast.error(error?.response?.data?.msg)
      return error
    }
  }
```

EditAchievement.jsx

```js
export const action =
  (queryClient) =>
  async ({ request, params }) => {
    const formData = await request.formData()
    const data = Object.fromEntries(formData)
    try {
      await customFetch.patch(`/achievements/${params.id}`, data)
      queryClient.invalidateQueries(['achievements'])
      toast.success('achievement edited successfully')
      return redirect('/dashboard/all-achievements')
    } catch (error) {
      toast.error(error?.response?.data?.msg)
      return error
    }
  }
```

DeleteAchievement.jsx

```js
export const action =
  (queryClient) =>
  async ({ params }) => {
    try {
      await customFetch.delete(`/achievements/${params.id}`)
      queryClient.invalidateQueries(['achievements'])
      toast.success('achievement deleted successfully')
    } catch (error) {
      toast.error(error?.response?.data?.msg)
    }
    return redirect('/dashboard/all-achievements')
  }
```

#### Edit achievement Loader

```js
import { FormRow, FormRowSelect, SubmitBtn } from '../components'
import Wrapper from '../assets/wrappers/DashboardFormPage'
import { useLoaderData, useParams } from 'react-router-dom'
import { STATUS, TYPE } from '../../../utils/constants'
import { Form, redirect } from 'react-router-dom'
import { toast } from 'react-toastify'
import customFetch from '../utils/customFetch'
import { useQuery } from '@tanstack/react-query'

const singleAchievementQuery = (id) => {
  return {
    queryKey: ['achievement', id],
    queryFn: async () => {
      const { data } = await customFetch.get(`/achievements/${id}`)
      return data
    },
  }
}

export const loader =
  (queryClient) =>
  async ({ params }) => {
    try {
      await queryClient.ensureQueryData(singleAchievementQuery(params.id))
      return params.id
    } catch (error) {
      toast.error(error?.response?.data?.msg)
      return redirect('/dashboard/all-achievements')
    }
  }

export const action =
  (queryClient) =>
  async ({ request, params }) => {
    const formData = await request.formData()
    const data = Object.fromEntries(formData)
    try {
      await customFetch.patch(`/achievements/${params.id}`, data)
      queryClient.invalidateQueries(['achievements'])

      toast.success('achievement edited successfully')
      return redirect('/dashboard/all-achievements')
    } catch (error) {
      toast.error(error?.response?.data?.msg)
      return error
    }
  }

const EditAchievement = () => {
  const id = useLoaderData()

  const {
    data: { achievement },
  } = useQuery(singleAchievementQuery(id))

  return (
    <Wrapper>
      <Form method="post" className="form">
        <h4 className="form-title">edit achievement</h4>
        <div className="form-center">
          <FormRow
            type="text"
            name="position"
            defaultValue={achievement.position}
          />
          <FormRow
            type="text"
            name="company"
            defaultValue={achievement.company}
          />
          <FormRow
            type="text"
            name="location"
            labelText="achievement location"
            defaultValue={achievement.location}
          />
          <FormRowSelect
            name="status"
            labelText="achievement status"
            defaultValue={achievement.status}
            list={Object.values(STATUS)}
          />
          <FormRowSelect
            name="type"
            labelText="achievement type"
            defaultValue={achievement.type}
            list={Object.values(TYPE)}
          />
          <SubmitBtn formBtn />
        </div>
      </Form>
    </Wrapper>
  )
}
export default EditAchievement
```

#### Axios Interceptors

DashboardLayout.jsx

```js
const DashboardContext = createContext();

const DashboardLayout = ({ isDarkThemeEnabled,queryClient }) => {
  const [isAuthError, setIsAuthError] = useState(false);

  const logoutUser = async () => {
    await customFetch.get('/auth/logout');
    toast.success('Logging out...');
    navigate('/');
  };

  customFetch.interceptors.response.use(
    (response) => {
      return response;
    },
    (error) => {
      if (error?.response?.status === 401) {
        setIsAuthError(true);
      }
      return Promise.reject(error);
    }
  );
  useEffect(() => {
    if (!isAuthError) return;
    logoutUser();
  }, [isAuthError]);
  return (
    ...
  )
};

```
