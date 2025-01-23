#

##
[form-validation] 

- form w zod resolver + default
- onSubmit handler w shadCn https://ui.shadcn.com/docs/components/form
- type + enum validated
- select w name + ctrl type (status + mode)

- https://react-hook-form.com/get-started
- more reading (https://www.remix-validated-form.io/reference/field-array)

[pass-handler+parse-validation]
- obj.values client side (otherwise stringify)
- auth redirect and parse zod schema


[client-catche]
 - dehydrate fetch wrappers in provider wrapping app

## form-validation

### Shadcn/ui Forms

- install

```sh
npx shadcn-ui@latest add form input
```

### CreateJobForm Setup

- components/CreateJobForm
- render in add-job/page.tsx

```tsx
'use client'

import * as z from 'zod'
import { zodResolver } from '@hookform/resolvers/zod'
import { useForm } from 'react-hook-form'

import { Button } from '@/components/ui/button'
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'

const formSchema = z.object({
  username: z.string().min(2, {
    message: 'Username must be at least 2 characters.',
  }),
})

function CreateJobForm() {
  // 1. Define your form.
  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      username: '',
    },
  })

  // 2. Define a submit handler.
  function onSubmit(values: z.infer<typeof formSchema>) {
    // Do something with the form values.
    // ✅ This will be type-safe and validated.
    console.log(values)
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-8">
        <FormField
          control={form.control}
          name="username"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Username</FormLabel>
              <FormControl>
                <Input placeholder="shadcn" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">Submit</Button>
      </form>
    </Form>
  )
}
export default CreateJobForm
```

#### CreateJobForm - Details

1. **Imports:** Necessary modules and components are imported. This includes form handling and validation libraries, UI components, and the zod schema validation library.

2. **Form Schema:** A `formSchema` is defined using zod. This schema specifies that the `username` field is a string and must be at least 2 characters long.

3. **CreateJobForm Component:** This is the main component. It uses the `useForm` hook from `react-hook-form` to create a form instance which can be used to manage form state, handle form submission, and perform form validation. The form instance is configured with the zod schema as its resolver and a default value for the `username` field.

4. **Submit Handler:** A `onSubmit` function is defined. This function logs the form values when the form is submitted. The form values are type-checked and validated against the zod schema.

5. **Render:** The component returns a form with a single `username` field and a submit button. The `username` field is rendered using the `FormField` component, which is passed the form control and the field name. The `render` prop of `FormField` is used to render the actual input field and its associated label and message.

6. **Export:** The `CreateJobForm` component is exported as the default export of the module. This allows it to be imported in other files using the file path.

### Challenge - Create Types

1. **Create utils/types.ts:**

   - Create a new file named `types.ts` inside the `utils` directory.

2. **Define the `JobStatus` and `JobMode` enums:**

   - Define the `JobStatus` enum with the values 'applied', 'interview', 'offer', and 'rejected'.
   - Define the `JobMode` enum with the values 'fullTime', 'partTime', and 'internship'.

3. **Define the `createAndEditJobSchema` object:**

   - Use `z.object()` from the zod library to define a schema for creating and editing jobs.
   - The schema includes `position`, `company`, `location`, `status`, and `mode`. Each of these fields is a string with a minimum length of 2 characters, except for `status` and `mode` which are enums.

4. **Export the `createAndEditJobSchema` object:**

   - Export the `createAndEditJobSchema` object so it can be used in other files.

5. **Define and export the `CreateAndEditJobType` type:**
   - Use `z.infer<typeof createAndEditJobSchema>` to infer the type of the `createAndEditJobSchema` object.
   - Export the `CreateAndEditJobType` type so it can be used in other files.

Enums in TypeScript are a special type that allows you to define a set of named constants. They can be numeric or string-based.

### Types

- utils/types.ts

```ts
import * as z from 'zod'

export type JobType = {
  id: string
  createdAt: Date
  updatedAt: Date
  clerkId: string
  position: string
  company: string
  location: string
  status: string
  mode: string
}

export enum JobStatus {
  Pending = 'pending',
  Interview = 'interview',
  Declined = 'declined',
}

export enum JobMode {
  FullTime = 'full-time',
  PartTime = 'part-time',
  Internship = 'internship',
}
// Enums in TypeScript are a special type that allows you to define a set of named constants. They can be numeric or string-based.

export const createAndEditJobSchema = z.object({
  position: z.string().min(2, {
    message: 'position must be at least 2 characters.',
  }),
  company: z.string().min(2, {
    message: 'company must be at least 2 characters.',
  }),
  location: z.string().min(2, {
    message: 'location must be at least 2 characters.',
  }),
  status: z.nativeEnum(JobStatus),
  mode: z.nativeEnum(JobMode),
})

export type CreateAndEditJobType = z.infer<typeof createAndEditJobSchema>
```

### Explore Select Component

- install

```sh
npx shadcn-ui@latest add select
```

```tsx
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'

;<Select>
  <SelectTrigger className="w-[180px]">
    <SelectValue placeholder="Theme" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="light">Light</SelectItem>
    <SelectItem value="dark">Dark</SelectItem>
    <SelectItem value="system">System</SelectItem>
  </SelectContent>
</Select>
```

- [docs](https://ui.shadcn.com/docs/components/select)

### Challenge - FormComponents

1. **Import necessary libraries and components**

   - Import the `Control` type from `react-hook-form`.
   - Import the `Select`, `SelectContent`, `SelectItem`, `SelectTrigger`, and `SelectValue` components from your UI library.
   - Import the `FormControl`, `FormField`, `FormItem`, `FormLabel`, and `FormMessage` components from your UI library.
   - Import the `Input` component from your local UI components.

2. **Define the types for CustomFormField and CustomFormSelect components**

   - Define a type `CustomFormFieldProps` that includes `name` and `control` properties.
   - Define a type `CustomFormSelectProps` that includes `name`, `control`, `items`, and `labelText` properties.

3. **Define the CustomFormField component**

   - Define a new function component named `CustomFormField` that takes `CustomFormFieldProps` as props.

4. **Create the CustomFormField UI**

   - Inside the `CustomFormField` component, return a `FormField` component.
   - Pass `control` and `name` to the `FormField` component.
   - Inside the `FormField` component, render a `FormItem` that contains a `FormLabel`, a `FormControl` with an `Input`, and a `FormMessage`.

5. **Define the CustomFormSelect component**

   - Define a new function component named `CustomFormSelect` that takes `CustomFormSelectProps` as props.

6. **Create the CustomFormSelect UI**

   - Inside the `CustomFormSelect` component, return a `FormField` component.
   - Pass `control` and `name` to the `FormField` component.
   - Inside the `FormField` component, render a `FormItem` that contains a `FormLabel`, a `Select` with a `SelectTrigger` and `SelectContent`, and a `FormMessage`.
   - Inside the `SelectContent`, map over the `items` and return a `SelectItem` for each item.

7. **Export the components**
   - Export `CustomFormField` and `CustomFormSelect` so they can be used in other parts of your application.

### FormComponents

- components/FormComponents

```tsx
import { Control } from 'react-hook-form'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'
import {
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form'
import { Input } from './ui/input'

type CustomFormFieldProps = {
  name: string
  control: Control<any>
}

export function CustomFormField({ name, control }: CustomFormFieldProps) {
  return (
    <FormField
      control={control}
      name={name}
      render={({ field }) => (
        <FormItem>
          <FormLabel className="capitalize">{name}</FormLabel>
          <FormControl>
            <Input {...field} />
          </FormControl>
          <FormMessage />
        </FormItem>
      )}
    />
  )
}

type CustomFormSelectProps = {
  name: string
  control: Control<any>
  items: string[]
  labelText?: string
}

export function CustomFormSelect({
  name,
  control,
  items,
  labelText,
}: CustomFormSelectProps) {
  return (
    <FormField
      control={control}
      name={name}
      render={({ field }) => (
        <FormItem>
          <FormLabel className="capitalize">{labelText || name}</FormLabel>
          <Select onValueChange={field.onChange} defaultValue={field.value}>
            <FormControl>
              <SelectTrigger>
                <SelectValue />
              </SelectTrigger>
            </FormControl>
            <SelectContent>
              {items.map((item) => {
                return (
                  <SelectItem key={item} value={item}>
                    {item}
                  </SelectItem>
                )
              })}
            </SelectContent>
          </Select>

          <FormMessage />
        </FormItem>
      )}
    />
  )
}
export default CustomFormSelect
```

## pass handler and  perform middleware functionallity

### Challenge - CreateJobForm

1. **Import necessary libraries and components**

   - Import the `zodResolver` from `@hookform/resolvers/zod` for form validation.
   - Import the `useForm` hook from `react-hook-form` for form handling.
   - Import the necessary types and schemas for your form from `@/utils/types`.
   - Import the `Button` and `Form` components from `@/components/ui`.
   - Import the `CustomFormField` and `CustomFormSelect` components from `./FormComponents`.

2. **Define the CreateJobForm component**

   - Define a new function component named `CreateJobForm`.

3. **Initialize the form with useForm**

   - Inside the `CreateJobForm` component, use the `useForm` hook to initialize your form.
   - Pass the `CreateAndEditJobType` for your form data to `useForm`.
   - Use `zodResolver` with your `createAndEditJobSchema` for form validation.

4. **Define default values for the form**

   - Define default values for your form fields in the `useForm` hook.

5. **Define the form submission handler**

   - Inside the `CreateJobForm` component, define a function for handling form submission.
   - This function should take the form data as its parameter.

6. **Create the form UI**

   - In the component's return statement, create the form UI using the `Form` component.
   - Use your custom form field components to create the form fields.
   - Add a submit button to the form.

7. **Export the CreateJobForm component**
   - After defining the `CreateJobForm` component, export it so it can be used in other parts of your application.

### CreateJobForm

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

function CreateJobForm() {
  // 1. Define your form.
  const form = useForm<CreateAndEditJobType>({
    resolver: zodResolver(createAndEditJobSchema),
    defaultValues: {
      position: '',
      company: '',
      location: '',
      status: JobStatus.Pending,
      mode: JobMode.FullTime,
    },
  })

  function onSubmit(values: CreateAndEditJobType) {
    // Do something with the form values.
    // ✅ This will be type-safe and validated.
    console.log(values)
  }

  return (
    <Form {...form}>
      <form
        onSubmit={form.handleSubmit(onSubmit)}
        className="bg-muted p-8 rounded"
      >
        <h2 className="capitalize font-semibold text-4xl mb-6">add job</h2>
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

          <Button type="submit" className="self-end capitalize">
            create job
          </Button>
        </div>
      </form>
    </Form>
  )
}
export default CreateJobForm
```

### Create DB in Render

- create .env
- add to .gitignore
- copy external URL
  DATABASE_URL =

### Challenge - Setup Prisma

- setup new prisma instance
- setup connection file
- create Job model

```prisma
model Job {
  id        String      @id @default(uuid())
  clerkId   String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  position    String
  company   String
  location  String
  status      String
  mode     String
}
```

- push changes to render

### Setup Prisma

- setup new prisma instance

```sh
npx prisma init
```

- setup connection file

utils/db.ts

```ts
import { PrismaClient } from '@prisma/client'

const prismaClientSingleton = () => {
  return new PrismaClient()
}

type PrismaClientSingleton = ReturnType<typeof prismaClientSingleton>

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClientSingleton | undefined
}

const prisma = globalForPrisma.prisma ?? prismaClientSingleton()

export default prisma

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

- create Job model

schema.prisma

```prisma
/ This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Job {
  id        String      @id @default(uuid())
  clerkId   String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  position    String
  company   String
  location  String
  status      String
  mode     String
}

```

- push changes to render

```sh
npx prisma db push
```

### Challenge - CreateJobAction

1. **Import necessary libraries and modules**

   - Create utils/action.ts file
   - Import the prisma instance from your database configuration file.
   - Import the auth function from `@clerk/nextjs` for user authentication.
   - Import the necessary types and schemas from your types file.
   - Import the redirect function from `next/navigation` for redirection.
   - Import the `Prisma` namespace from `@prisma/client` for database operations.
   - Import `dayjs` for date and time manipulation.

2. **Define the authenticateAndRedirect function**

   - Define a function named `authenticateAndRedirect` that doesn't take any parameters.
   - Inside this function, call the auth function and destructure `userId` from its return value.
   - If `userId` is not defined, call the redirect function with `'/'` as the argument to redirect the user to the home page.
   - Return `userId`.

3. **Define the createJobAction function**

   - Define an asynchronous function named `createJobAction` that takes values of type `CreateAndEditJobType` as a parameter.
   - This function should return a Promise that resolves to `JobType` or null.

4. **Authenticate the user and validate the form values**

   - Inside the `createJobAction` function, call `authenticateAndRedirect` and store its return value in `userId`.
   - Call `createAndEditJobSchema.parse` with `values` as the argument to validate the form values.

5. **Create a new job in the database**

   - Use the `prisma.job.create` method to create a new job in the database.
   - Pass an object to this method with a `data` property.
   - The `data` property should be an object that spreads the `values` and adds a `clerkId` property with `userId` as its value.
   - Store the return value of this method in `job`.

6. **Handle errors**

   - Wrap the validation and database operation in a try-catch block.
   - If an error occurs, log the error to the console and return null.

7. **Return the new job**

   - After the try-catch block, return `job`.

8. **Export the createJobAction function**
   - Export `createJobAction` so it can be used in other parts of your application.

### CreateJobAction

- utils/actions.ts

```ts
'use server'

import prisma from './db'
import { auth } from '@clerk/nextjs'
import { JobType, CreateAndEditJobType, createAndEditJobSchema } from './types'
import { redirect } from 'next/navigation'
import { Prisma } from '@prisma/client'
import dayjs from 'dayjs'

function authenticateAndRedirect(): string {
  const { userId } = auth()
  if (!userId) {
    redirect('/')
  }
  return userId
}

export async function createJobAction(
  values: CreateAndEditJobType
): Promise<JobType | null> {
  // await new Promise((resolve) => setTimeout(resolve, 3000));
  const userId = authenticateAndRedirect()
  try {
    createAndEditJobSchema.parse(values)
    const job: JobType = await prisma.job.create({
      data: {
        ...values,

        clerkId: userId,
      },
    })
    return job
  } catch (error) {
    console.error(error)
    return null
  }
}
```

## client cache

### Explore Toast Component

- install

```sh
npx shadcn-ui@latest add toast

```

[docs](https://ui.shadcn.com/docs/components/toast)

### Challenge - Add React Query and Toaster

- add React Query and Toaster to providers.tsx
- wrap Home Page in React Query

### Add React Query and Toaster

- app/providers.tsx

```tsx
'use client'

import { ThemeProvider } from '@/components/theme-provider'
import { useState } from 'react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { Toaster } from '@/components/ui/toaster'

const Providers = ({ children }: { children: React.ReactNode }) => {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            // With SSR, we usually want to set some default staleTime
            // above 0 to avoid refetching immediately on the client
            staleTime: 60 * 1000 * 5,
          },
        },
      })
  )

  return (
    <ThemeProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      <Toaster />
      <QueryClientProvider client={queryClient}>
        {children}
        <ReactQueryDevtools initialIsOpen={false} />
      </QueryClientProvider>
    </ThemeProvider>
  )
}
export default Providers
```

- add-job/page

```tsx
import CreateJobForm from '@/components/CreateJobForm'
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query'

function AddJobPage() {
  const queryClient = new QueryClient()
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <CreateJobForm />
    </HydrationBoundary>
  )
}
export default AddJobPage
```
