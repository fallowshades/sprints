#

##

[pending-from-fetch-wrapper]
- data for related pages
- submit wrapper call mutate f() --> pass values -->actionhandler
- body+pagination whereClause

- prerendered hydration boundary
- search is nested in boundary and push searchparam onto router
- fromdata --> params.set
- list useQuery hook to access * times

[card-w-dynamics]

- action handler is Promise
- btn w isPending network awarness

[page-within]

- can use fetchwrapper for another promise
- hydration boundary prefetch (and check user from clerk)
- usQuery and useMutation w shadcn component library handling zod resolver

## pending logic from fetch wrapper

### CreateJobForm Complete

```tsx
// imports
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { createJobAction } from '@/utils/actions'
import { useToast } from '@/components/ui/use-toast'
import { useRouter } from 'next/navigation'

// logic
const queryClient = useQueryClient()
const { toast } = useToast()
const router = useRouter()
const { mutate, isPending } = useMutation({
  mutationFn: (values: CreateAndEditJobType) => createJobAction(values),
  onSuccess: (data) => {
    if (!data) {
      toast({
        description: 'there was an error',
      })
      return
    }
    toast({ description: 'job created' })
    queryClient.invalidateQueries({ queryKey: ['jobs'] })
    queryClient.invalidateQueries({ queryKey: ['stats'] })
    queryClient.invalidateQueries({ queryKey: ['charts'] })

    router.push('/jobs')
    // form.reset();
  },
})

function onSubmit(values: CreateAndEditJobType) {
  mutate(values)
}
// return
;<Button type="submit" className="self-end capitalize" disabled={isPending}>
  {isPending ? 'loading...' : 'create job'}
</Button>
```

### Challenge - GetAllJobsAction

1. **Define the getAllJobsAction function**

   - Define an asynchronous function named `getAllJobsAction` that takes an object as a parameter.
   - This object should have `search`, `jobStatus`, `page`, and `limit` properties.
   - The `page` and `limit` properties should have default values of 1 and 10, respectively.
   - This function should return a Promise that resolves to an object with `jobs`, `count`, `page`, and `totalPages` properties.

2. **Authenticate the user**

   - Inside the `getAllJobsAction` function, call `authenticateAndRedirect` and store its return value in `userId`.

3. **Define the whereClause object**

   - Define a `whereClause` object with a `clerkId` property that has `userId` as its value.

4. **Modify the whereClause object based on search and jobStatus**

   - If `search` is defined, add an `OR` property to `whereClause` that is an array of objects.
   - Each object in the `OR` array should represent a condition where a field contains the search string.
   - If `jobStatus` is defined and not equal to 'all', add a `status` property to `whereClause` that has `jobStatus` as its value.

5. **Fetch jobs from the database**

   - Use the `prisma.job.findMany` method to fetch jobs from the database.
   - Pass an object to this method with `where` and `orderBy` properties.
   - The `where` property should have `whereClause` as its value.
   - The `orderBy` property should be an object with a `createdAt` property that has 'desc' as its value.
   - Store the return value of this method in `jobs`.

6. **Handle errors**

   - Wrap the database operation in a try-catch block.
   - If an error occurs, log the error to the console and return an object with `jobs`, `count`, `page`, and `totalPages` properties, all of which have 0 or [] as their values.

7. **Return the jobs**

   - After the try-catch block, return an object with `jobs`, `count`, `page`, and `totalPages` properties.

8. **Export the getAllJobsAction function**
   - Export `getAllJobsAction` so it can be used in other parts of your application.

### GetAllJobsAction

- actions

```ts
type GetAllJobsActionTypes = {
  search?: string
  jobStatus?: string
  page?: number
  limit?: number
}

export async function getAllJobsAction({
  search,
  jobStatus,
  page = 1,
  limit = 10,
}: GetAllJobsActionTypes): Promise<{
  jobs: JobType[]
  count: number
  page: number
  totalPages: number
}> {
  const userId = authenticateAndRedirect()

  try {
    let whereClause: Prisma.JobWhereInput = {
      clerkId: userId,
    }
    if (search) {
      whereClause = {
        ...whereClause,
        OR: [
          {
            position: {
              contains: search,
            },
          },
          {
            company: {
              contains: search,
            },
          },
        ],
      }
    }
    if (jobStatus && jobStatus !== 'all') {
      whereClause = {
        ...whereClause,
        status: jobStatus,
      }
    }

    const jobs: JobType[] = await prisma.job.findMany({
      where: whereClause,
      orderBy: {
        createdAt: 'desc',
      },
    })

    return { jobs, count: 0, page: 1, totalPages: 0 }
  } catch (error) {
    console.error(error)
    return { jobs: [], count: 0, page: 1, totalPages: 0 }
  }
}
```

### Challenge - Jobs Page

- create SearchForm, JobsList, JobCard, JobInfo, DeleteJobBtn components
- setup jobs/loading.tsx
- wrap jobs/page in React Query and pre-fetch getAllJobsAction

### Jobs Page

- create SearchForm, JobsList, JobCard, JobInfo, DeleteJobBtn
- setup jobs/loading.tsx

```tsx
function loading() {
  return <h2 className="text-xl font-medium capitalize">loading...</h2>
}
export default loading
```

JobCard.tsx

```tsx
import { JobType } from '@/utils/types'

function JobCard({ job }: { job: JobType }) {
  return <h1 className="text-3xl">JobCard</h1>
}
export default JobCard
```

jobs/page.tsx

```tsx
import JobsList from '@/components/JobsList'
import SearchForm from '@/components/SearchForm'
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query'
import { getAllJobsAction } from '@/utils/actions'

async function AllJobsPage() {
  const queryClient = new QueryClient()

  await queryClient.prefetchQuery({
    queryKey: ['jobs', '', 'all', 1],
    queryFn: () => getAllJobsAction({}),
  })
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <SearchForm />
      <JobsList />
    </HydrationBoundary>
  )
}

export default AllJobsPage
```

### Challenge - SearchForm

1. **Import necessary libraries and components**

   - Import the `Input` and `Button` components from your UI library.
   - Import the `usePathname`, `useRouter`, and `useSearchParams` hooks from `next/navigation`.
   - Import the `Select`, `SelectContent`, `SelectItem`, `SelectTrigger`, and `SelectValue` components from your UI library.
   - Import the `JobStatus` type from your types file.

2. **Define the SearchContainer component**

   - Define a function component named `SearchContainer`.

3. **Use hooks to get necessary data**

   - Inside `SearchContainer`, use the `useSearchParams` hook to get the current search parameters.
   - Use the `get` method of the `searchParams` object to get the `search` and `jobStatus` parameters.
   - Use the `useRouter` hook to get the router object.
   - Use the `usePathname` hook to get the current pathname.

4. **Define the form submission handler**

   - Inside `SearchContainer`, define a function named `handleSubmit` for handling form submission.
   - This function should take an event object as its parameter.
   - Inside this function, prevent the default form submission behavior.
   - Create a new `URLSearchParams` object and a new `FormData` object.
   - Use the `get` method of the `formData` object to get the `search` and `jobStatus` form values.
   - Use the `set` method of the `params` object to set the `search` and `jobStatus` parameters.
   - Use the `push` method of the router object to navigate to the current pathname with the new search parameters.

5. **Create the form UI**

   - In the component's return statement, create the form UI using the form element.
   - Use the `Input` and `Select` components to create the form fields.
   - Use the `Button` component to create the submit button.
   - Pass the `handleSubmit` function as the `onSubmit` prop to the form element.

6. **Export the SearchContainer component**
   - After defining the `SearchContainer` component, export it so it can be used in other parts of your application.

### SearchForm

```tsx
'use client'
import { Input } from './ui/input'
import { usePathname, useRouter, useSearchParams } from 'next/navigation'
import { Button } from './ui/button'

import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'
import { JobStatus } from '@/utils/types'

function SearchContainer() {
  // set default values
  const searchParams = useSearchParams()
  const search = searchParams.get('search') || ''
  const jobStatus = searchParams.get('jobStatus') || 'all'

  const router = useRouter()
  const pathname = usePathname()
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()

    const formData = new FormData(e.currentTarget)
    const search = formData.get('search') as string
    const jobStatus = formData.get('jobStatus') as string
    let params = new URLSearchParams()
    params.set('search', search)
    params.set('jobStatus', jobStatus)

    router.push(`${pathname}?${params.toString()}`)
  }

  return (
    <form
      className="bg-muted mb-16 p-8 grid sm:grid-cols-2 md:grid-cols-3  gap-4 rounded-lg"
      onSubmit={handleSubmit}
    >
      <Input
        type="text"
        placeholder="Search Jobs"
        name="search"
        defaultValue={search}
      />
      <Select defaultValue={jobStatus} name="jobStatus">
        <SelectTrigger>
          <SelectValue />
        </SelectTrigger>
        <SelectContent>
          {['all', ...Object.values(JobStatus)].map((jobStatus) => {
            return (
              <SelectItem key={jobStatus} value={jobStatus}>
                {jobStatus}
              </SelectItem>
            )
          })}
        </SelectContent>
      </Select>
      <Button type="submit">Search</Button>
    </form>
  )
}
export default SearchContainer
```

### Challenge - JobsList

1. **Import necessary libraries and modules**

   - Import the `useSearchParams` hook from `next/navigation`.
   - Import the `getAllJobsAction` function from your actions file.
   - Import the `useQuery` hook from `@tanstack/react-query`.

2. **Define the JobsList component**

   - Define a function component named `JobsList`.

3. **Use hooks to get necessary data**

   - Inside `JobsList`, use the `useSearchParams` hook to get the current search parameters.
   - Use the `get` method of the `searchParams` object to get the `search` and `jobStatus` parameters.
   - If `search` or `jobStatus` is null, default them to an empty string and 'all', respectively.
   - Use the `get` method of the `searchParams` object to get the `page` parameter.
   - If `page` is null, default it to 1.

4. **Fetch the jobs from the server**

   - Use the `useQuery` hook to fetch the jobs from the server.
   - Pass an object to this hook with `queryKey` and `queryFn` properties.
   - The `queryKey` property should be an array with 'jobs', `search`, `jobStatus`, and `pageNumber`.
   - The `queryFn` property should be a function that calls `getAllJobsAction` with an object that has `search`, `jobStatus`, and `page` properties.
   - Store the return value of this hook in `data` and `isPending`.

5. **Handle loading and empty states**

   - If `isPending` is true, return a `h2` element with 'Please Wait...' as its child.
   - If `jobs` is an empty array, return a `h2` element with 'No Jobs Found...' as its child.

6. **Export the JobsList component**
   - After defining the `JobsList` component, export it so it can be used in other parts of your application.

### JobsList

```tsx
'use client'
import JobCard from './JobCard'
import { useSearchParams } from 'next/navigation'
import { getAllJobsAction } from '@/utils/actions'
import { useQuery } from '@tanstack/react-query'

function JobsList() {
  const searchParams = useSearchParams()

  const search = searchParams.get('search') || ''
  const jobStatus = searchParams.get('jobStatus') || 'all'

  const pageNumber = Number(searchParams.get('page')) || 1

  const { data, isPending } = useQuery({
    queryKey: ['jobs', search ?? '', jobStatus, pageNumber],
    queryFn: () => getAllJobsAction({ search, jobStatus, page: pageNumber }),
  })
  const jobs = data?.jobs || []

  if (isPending) return <h2 className="text-xl">Please Wait...</h2>

  if (jobs.length < 1) return <h2 className="text-xl">No Jobs Found...</h2>
  return (
    <>
      {/*button container  */}
      <div className="grid md:grid-cols-2  gap-8">
        {jobs.map((job) => {
          return <JobCard key={job.id} job={job} />
        })}
      </div>
    </>
  )
}
export default JobsList
```

## card w dynamics

### Explore - shadcn/ui badge separator and card components

- install

```sh
npx shadcn-ui@latest add badge separator card

```

[badge](https://ui.shadcn.com/docs/components/badge)
[separator](https://ui.shadcn.com/docs/components/separator)
[card](https://ui.shadcn.com/docs/components/card)

### Challenge - JobCard

1. **Import necessary libraries and components**

   - Import the `JobType` type from your types file.
   - Import the `MapPin`, `Briefcase`, `CalendarDays`, and `RadioTower` components from `lucide-react`.
   - Import the `Link` component from `next/link`.
   - Import the `Card`, `CardContent`, `CardDescription`, `CardFooter`, `CardHeader`, and `CardTitle` components from your UI library.
   - Import the `Separator`, `Button`, `Badge`, `JobInfo`, and `DeleteJobButton` components from your components directory.

2. **Define the JobCard component**

   - Define a function component named `JobCard` that takes an object as a prop.
   - This object should have a `job` property of type `JobType`.

3. **Convert the job's creation date to a locale string**

   - Inside `JobCard`, create a new Date object with `job.createdAt` as its argument.
   - Call the `toLocaleDateString` method on this object and store its return value in `date`.

4. **Create the component UI**

   - In the component's return statement, create the component UI using the `Card`, `CardHeader`, `CardTitle`, `CardDescription`, `Separator`, `CardContent`, `CardFooter`, `Button`, `Link`, and `DeleteJobButton` components.
   - Pass the `job.position` and `job.company` as the children of the `CardTitle` and `CardDescription` components, respectively.
   - Pass the `job.id` as the `href` prop to the `Link` component.
   - Pass the `date` as the child of the `CalendarDays` component.

5. **Export the JobCard component**
   - After defining the `JobCard` component, export it so it can be used in other parts of your application.

### JobCard

JobCard

```tsx
import { JobType } from '@/utils/types'
import { MapPin, Briefcase, CalendarDays, RadioTower } from 'lucide-react'

import Link from 'next/link'
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from '@/components/ui/card'
import { Separator } from './ui/separator'
import { Button } from './ui/button'
import { Badge } from './ui/badge'
import JobInfo from './JobInfo'
import DeleteJobButton from './DeleteJobButton'

function JobCard({ job }: { job: JobType }) {
  const date = new Date(job.createdAt).toLocaleDateString()
  return (
    <Card className="bg-muted">
      <CardHeader>
        <CardTitle>{job.position}</CardTitle>
        <CardDescription>{job.company}</CardDescription>
      </CardHeader>
      <Separator />
      <CardContent>{/* card info */}</CardContent>
      <CardFooter className="flex gap-4">
        <Button asChild size="sm">
          <Link href={`/jobs/${job.id}`}>edit</Link>
        </Button>
        <DeleteJobButton />
      </CardFooter>
    </Card>
  )
}
export default JobCard
```

### Challenge - JobInfo

1. **Define the JobInfo component**

   - Define a function component named `JobInfo` that takes an object as a prop.
   - This object should have `icon` and `text` properties.
   - The `icon` property should be of type `React.ReactNode` and the `text` property should be of type `string`.

2. **Create the component UI**

   - In the component's return statement, create a `div` element with a `className` of 'flex gap-x-2 items-center'.
   - Inside this `div`, render the `icon` and `text` props.

3. **Export the JobInfo component**

   - After defining the `JobInfo` component, export it so it can be used in other parts of your application.

4. **Use the JobInfo component**
   - In the `CardContent` component, use the `JobInfo` component four times.
   - For each `JobInfo` component, pass an `icon` prop and a `text` prop.
   - The `icon` prop should be a `Briefcase`, `MapPin`, `CalendarDays`, or `RadioTower` component.
   - The `text` prop should be `job.mode`, `job.location`, `date`, or `job.status`.
   - Wrap the last `JobInfo` component in a `Badge` component with a `className` of 'w-32 justify-center'.



### JobInfo

JobInfo.tsx

```tsx
function JobInfo({ icon, text }: { icon: React.ReactNode; text: string }) {
  return (
    <div className="flex gap-x-2 items-center">
      {icon}
      {text}
    </div>
  )
}
export default JobInfo
```

JobCard.tsx

```tsx
<CardContent className="mt-4 grid grid-cols-2 gap-4">
  <JobInfo icon={<Briefcase />} text={job.mode} />
  <JobInfo icon={<MapPin />} text={job.location} />
  <JobInfo icon={<CalendarDays />} text={date} />
  <Badge className="w-32  justify-center">
    <JobInfo icon={<RadioTower className="w-4 h-4" />} text={job.status} />
  </Badge>
</CardContent>
```

## redirect to delete action

### Challenge - DeleteJobAction

1. **Define the deleteJobAction function**

   - Define an asynchronous function named `deleteJobAction` that takes a string `id` as a parameter.
   - This function should return a Promise that resolves to a `JobType` object or null.

2. **Authenticate the user**

   - Inside the `deleteJobAction` function, call `authenticateAndRedirect` and store its return value in `userId`.

3. **Delete the job from the database**

   - Use the `prisma.job.delete` method to delete the job from the database.
   - Pass an object to this method with a `where` property.
   - The `where` property should be an object with `id` and `clerkId` properties.
   - The `id` property should have `id` as its value and the `clerkId` property should have `userId` as its value.
   - Store the return value of this method in `job`.

4. **Handle errors**

   - Wrap the database operation in a try-catch block.
   - If an error occurs, return null.

5. **Return the deleted job**

   - After the try-catch block, return `job`.

6. **Export the deleteJobAction function**
   - Export `deleteJobAction` so it can be used in other parts of your application.

### DeleteJobAction

actions

```ts
export async function deleteJobAction(id: string): Promise<JobType | null> {
  const userId = authenticateAndRedirect()

  try {
    const job: JobType = await prisma.job.delete({
      where: {
        id,
        clerkId: userId,
      },
    })
    return job
  } catch (error) {
    return null
  }
}
```

### Challenge - DeleteJobButton

1. **Import necessary libraries and components**

   - Import the `Button`, `Badge`, `JobInfo`, and `useToast` components from your components directory.
   - Import the `useMutation` and `useQueryClient` hooks from `@tanstack/react-query`.
   - Import the `deleteJobAction` function from your actions file.

2. **Define the DeleteJobBtn component**

   - Define a function component named `DeleteJobBtn` that takes an object as a prop.
   - This object should have an `id` property of type string.

3. **Use hooks to get necessary data and functions**

   - Inside `DeleteJobBtn`, use the `useToast` hook to get the `toast` function.
   - Use the `useQueryClient` hook to get the `queryClient` object.
   - Use the `useMutation` hook to get the `mutate` function and `isPending` state.
   - Pass an object to the `useMutation` hook with `mutationFn` and `onSuccess` properties.
   - The `mutationFn` property should be a function that takes `id` as a parameter and calls `deleteJobAction` with `id`.
   - The `onSuccess` property should be a function that takes `data` as a parameter and invalidates the `jobs`, `stats`, and `charts` queries if data is truthy. If data is falsy, it should call `toast` with an object that has a `description` property of 'there was an error'.

4. **Create the component UI**

   - In the component's return statement, create the component UI using the `Button` component.
   - Pass the `mutate` function as the `onClick` prop to the `Button` component.
   - Pass `isPending` as the `loading` prop to the `Button` component.

5. **Export the DeleteJobBtn component**
   - After defining the `DeleteJobBtn` component, export it so it can be used in other parts of your application.

#### DeleteJobButton

```tsx
import { Button } from './ui/button'
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { deleteJobAction } from '@/utils/actions'
import { useToast } from '@/components/ui/use-toast'

function DeleteJobBtn({ id }: { id: string }) {
  const { toast } = useToast()
  const queryClient = useQueryClient()
  const { mutate, isPending } = useMutation({
    mutationFn: (id: string) => deleteJobAction(id),
    onSuccess: (data) => {
      if (!data) {
        toast({
          description: 'there was an error',
        })
        return
      }
      queryClient.invalidateQueries({ queryKey: ['jobs'] })
      queryClient.invalidateQueries({ queryKey: ['stats'] })
      queryClient.invalidateQueries({ queryKey: ['charts'] })

      toast({ description: 'job removed' })
    },
  })
  return (
    <Button
      size="sm"
      disabled={isPending}
      onClick={() => {
        mutate(id)
      }}
    >
      {isPending ? 'deleting...' : 'delete'}
    </Button>
  )
}
export default DeleteJobBtn
```

## edit in linked page

### single job

#### Challenge - GetSingleJobAction

1. **Define the getSingleJobAction function**

   - Define an asynchronous function named `getSingleJobAction` that takes a string `id` as a parameter.
   - This function should return a Promise that resolves to a `JobType` object or null.

2. **Authenticate the user**

   - Inside the `getSingleJobAction` function, call `authenticateAndRedirect` and store its return value in `userId`.

3. **Fetch the job from the database**

   - Use the `prisma.job.findUnique` method to fetch the job from the database.
   - Pass an object to this method with a `where` property.
   - The `where` property should be an object with `id` and `clerkId` properties.
   - The `id` property should have `id` as its value and the `clerkId` property should have `userId` as its value.
   - Store the return value of this method in `job`.

4. **Handle errors**

   - Wrap the database operation in a try-catch block.
   - If an error occurs, set `job` to null.

5. **Redirect if the job is not found**

   - After the try-catch block, check if `job` is falsy.
   - If `job` is falsy, call `redirect` with '/jobs' as its argument.

6. **Return the fetched job**

   - After the if statement, return `job`.

7. **Export the getSingleJobAction function**
   - Export `getSingleJobAction` so it can be used in other parts of your application.

#### GetSingleJobAction

```tsx
export async function getSingleJobAction(id: string): Promise<JobType | null> {
  let job: JobType | null = null
  const userId = authenticateAndRedirect()

  try {
    job = await prisma.job.findUnique({
      where: {
        id,
        clerkId: userId,
      },
    })
  } catch (error) {
    job = null
  }
  if (!job) {
    redirect('/jobs')
  }
  return job
}
```


#### Challenge - SingleJob Page

- create single job page (dynamic)
- create EditJobForm which accepts jobId props (string)

1. **Import necessary libraries and components**

   - Import the `EditJobForm` component from your components directory.
   - Import the `getSingleJobAction` function from your actions file.
   - Import the `dehydrate`, `HydrationBoundary`, and `QueryClient` components from `@tanstack/react-query`.

2. **Define the JobDetailPage component**

   - Define an asynchronous function component named `JobDetailPage` that takes an object as a prop.
   - This object should have a `params` property, which is also an object with an `id` property of type string.

3. **Create a new query client**

   - Inside `JobDetailPage`, create a new `QueryClient` instance and store it in `queryClient`.

4. **Prefetch the job data**

   - Use the `prefetchQuery` method of `queryClient` to prefetch the job data.
   - Pass an object to this method with `queryKey` and `queryFn` properties.
   - The `queryKey` property should be an array with 'job' and `params.id`.
   - The `queryFn` property should be a function that calls `getSingleJobAction` with `params.id`.

5. **Create the component UI**

   - In the component's return statement, create the component UI using the `HydrationBoundary` and `EditJobForm` components.
   - Pass the result of calling `dehydrate` with `queryClient` as the `state` prop to `HydrationBoundary`.
   - Pass `params.id` as the `jobId` prop to `EditJobForm`.

6. **Export the JobDetailPage component**
   - After defining the `JobDetailPage` component, export it so it can be used in other parts of your application.

#### SingleJob Page

jobs/[id]/page.tsx

```tsx
import EditJobForm from '@/components/EditJobForm'
import { getSingleJobAction } from '@/utils/actions'

import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query'

async function JobDetailPage({ params }: { params: { id: string } }) {
  const queryClient = new QueryClient()

  await queryClient.prefetchQuery({
    queryKey: ['job', params.id],
    queryFn: () => getSingleJobAction(params.id),
  })

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <EditJobForm jobId={params.id} />
    </HydrationBoundary>
  )
}
export default JobDetailPage
```

#### Challenge - UpdateJobAction

1. **Define the updateJobAction function**

   - Define an asynchronous function named `updateJobAction` that takes a string `id` and an object `values` as parameters.
   - The `values` parameter should be of type `CreateAndEditJobType`.
   - This function should return a Promise that resolves to a `JobType` object or null.

2. **Authenticate the user**

   - Inside the `updateJobAction` function, call `authenticateAndRedirect` and store its return value in `userId`.

3. **Update the job in the database**

   - Use the `prisma.job.update` method to update the job in the database.
   - Pass an object to this method with `where` and `data` properties.
   - The `where` property should be an object with `id` and `clerkId` properties.
   - The `id` property should have `id` as its value and the `clerkId` property should have `userId` as its value.
   - The `data` property should be an object that spreads `values`.
   - Store the return value of this method in `job`.

4. **Handle errors**

   - Wrap the database operation in a try-catch block.
   - If an error occurs, return null.

5. **Return the updated job**

   - After the try-catch block, return `job`.

6. **Export the updateJobAction function**
   - Export `updateJobAction` so it can be used in other parts of your application.

#### UpdateJobAction

```ts
export async function updateJobAction(
  id: string,
  values: CreateAndEditJobType
): Promise<JobType | null> {
  const userId = authenticateAndRedirect()

  try {
    const job: JobType = await prisma.job.update({
      where: {
        id,
        clerkId: userId,
      },
      data: {
        ...values,
      },
    })
    return job
  } catch (error) {
    return null
  }
}
```

#### Challenge - EditJobForm

1. **Import necessary libraries and components**

   - Import `zodResolver` from `@hookform/resolvers/zod`.
   - Import `useForm` from `react-hook-form`.
   - Import `JobStatus`, `JobMode`, `createAndEditJobSchema`, and `CreateAndEditJobType` from your types file.
   - Import `Button` from your UI components directory.
   - Import `Form` from your UI components directory.
   - Import `CustomFormField` and `CustomFormSelect` from your local `FormComponents` file.
   - Import `useMutation`, `useQueryClient`, and `useQuery` from `react-query`.
   - Import `createJobAction`, `getSingleJobAction`, and `updateJobAction` from your actions file.
   - Import `useToast` from your UI components directory.
   - Import `useRouter` from `next/router`.

2. **Define the EditJobForm component**

   - Define a function component named `EditJobForm` that takes an object as a prop.
   - This object should have a `jobId` property of type string.

3. **Use hooks to get necessary data and functions**

   - Inside `EditJobForm`, use the `useQueryClient` hook to get the `queryClient` object.
   - Use the `useToast` hook to get the `toast` function.
   - Use the `useRouter` hook to get the router object.
   - Use the `useQuery` hook to fetch the job data.
   - Use the `useMutation` hook to get the `mutate` function and `isPending` state.

4. **Use the useForm hook to get form functions**

   - Use the `useForm` hook to get the form object.
   - Pass an object to this hook with `resolver` and `defaultValues` properties.

5. **Define the submit handler**

   - Define a function `onSubmit` that calls `mutate` with values.

6. **Create the component UI**

   - In the component's return statement, create the component UI using the `Form`, `CustomFormField`, `CustomFormSelect`, and `Button` components.

7. **Export the EditJobForm component**
   - After defining the `EditJobForm` component, export it so it can be used in other parts of your application.

#### EditJobForm

```tsx
'use client'

import { zodResolver } from '@hookform/resolvers/zod'
import { useForm } from 'react-hook-form'

import {
  JobStatus,
  JobMode,
  createAndEditJobSchema,
  CreateAndEditJobType,
} from '@/utils/types'

import { Button } from '@/components/ui/button'
import { Form } from '@/components/ui/form'

import { CustomFormField, CustomFormSelect } from './FormComponents'
import { useMutation, useQueryClient, useQuery } from '@tanstack/react-query'
import { getSingleJobAction, updateJobAction } from '@/utils/actions'
import { useToast } from '@/components/ui/use-toast'
import { useRouter } from 'next/navigation'
function EditJobForm({ jobId }: { jobId: string }) {
  const queryClient = useQueryClient()
  const { toast } = useToast()
  const router = useRouter()

  const { data } = useQuery({
    queryKey: ['job', jobId],
    queryFn: () => getSingleJobAction(jobId),
  })

  const { mutate, isPending } = useMutation({
    mutationFn: (values: CreateAndEditJobType) =>
      updateJobAction(jobId, values),
    onSuccess: (data) => {
      if (!data) {
        toast({
          description: 'there was an error',
        })
        return
      }
      toast({ description: 'job updated' })
      queryClient.invalidateQueries({ queryKey: ['jobs'] })
      queryClient.invalidateQueries({ queryKey: ['job', jobId] })
      queryClient.invalidateQueries({ queryKey: ['stats'] })
      router.push('/jobs')
      // form.reset();
    },
  })

  // 1. Define your form.
  const form = useForm<CreateAndEditJobType>({
    resolver: zodResolver(createAndEditJobSchema),
    defaultValues: {
      position: data?.position || '',
      company: data?.company || '',
      location: data?.location || '',
      status: (data?.status as JobStatus) || JobStatus.Pending,
      mode: (data?.mode as JobMode) || JobMode.FullTime,
    },
  })

  // 2. Define a submit handler.
  function onSubmit(values: CreateAndEditJobType) {
    // Do something with the form values.
    // ✅ This will be type-safe and validated.
    mutate(values)
  }

  return (
    <Form {...form}>
      <form
        onSubmit={form.handleSubmit(onSubmit)}
        className="bg-muted p-8 rounded"
      >
        <h2 className="capitalize font-semibold text-4xl mb-6">edit job</h2>
        <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3 items-start">
          {/* position */}
          <CustomFormField name="position" control={form.control} />
          {/* company */}
          <CustomFormField name="company" control={form.control} />
          {/* location */}
          <CustomFormField name="location" control={form.control} />

          {/* job status */}
          <CustomFormSelect
            name="status"
            control={form.control}
            labelText="job status"
            items={Object.values(JobStatus)}
          />
          {/* job  type */}
          <CustomFormSelect
            name="mode"
            control={form.control}
            labelText="job mode"
            items={Object.values(JobMode)}
          />

          <Button
            type="submit"
            className="self-end capitalize"
            disabled={isPending}
          >
            {isPending ? 'updating...' : 'edit job'}
          </Button>
        </div>
      </form>
    </Form>
  )
}
export default EditJobForm
```
