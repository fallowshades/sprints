# simple data relation w redirects and revalidate path

## QnA server component instead of thunc (syncronize state --> src of truth)

[fallback]

1. where is toast if not redux toolkit + thunc extra reducer enable it

- hook to provider context (wrapper)

2. if one implement a service should one always use the native option

- no sometimes exist compponent w options
- native [https://clerk.com/docs/components/unstyled/sign-out-button]
- options [https://clerk.com/docs/components/unstyled/sign-out-button]

3. how to make the client component network aware of the server component action processing dalay (pending/wait time)

- use a hook & trigger toast

4. how is validation performed to ensure buissiness rules

- parse properties fromEntries in obj

[cache]

1. what to concider when developing in contrast to production code

- hot module reload + cache resets

2. what managment is necessary to manage clek provider

- init. generate, migrate

[schema]

1. how does the server component consider session managment

- clerc has a backend SDK [https://clerk.com/docs/references/backend/user/update-user-metadata]

2. create vs update

- findUniq => current user
- parse raw data vs w zodSchema

3. how is the server component managing session

- backend sdk https://clerk.com/docs/references/backend/user/update-user-metadata

4. how can the object notation help manage diff actions (static img content / dynamic validation fields)

- Nullish coalescing and spread the dynamic parts

## summary combine formData and zod for naked obj validation and rolebase meta check

[fallback]

- create profile
- fromdata reflected in button and formContainer w pending state
- each entry is validated in body

  - [Toast](https://ui.shadcn.com/docs/components/toast)

[cache]

- next dev -> clear cache (see prisma instance) (dev hot module relode)

[dev-process]

- push config --> setup schemas
  . connection - DATABASE_URL : Transaction + Password + "?pgbouncer=true&connection_limit=1"
- DIRECT_URL : Session + Password
  (model testUser --> generated schema)

  - (Prisma Instance)[https://www.prisma.io/docs/guides/other/troubleshooting-orm/help-articles/nextjs-prisma-client-dev-practices#solution]

[schema]

- profile unique,default, update
- beside fields, clerkId, default email, image(static)

  - [Useful Info](https://supabase.com/partners/integrations/prisma)

[userstate-role_based]

await create metaData aswell --> page can await check privilage
default data and await getAuthUser

- [Prisma Docs push create crud records](https://www.prisma.io/docs/concepts/components/prisma-client/crud)

## falllback

### SignUp/SignIn and Customize Avatar (optional)

- customization
  - avatars

### Toast Component

[Toast](https://ui.shadcn.com/docs/components/toast)

providers.tsx

```tsx
'use client'
import { ThemeProvider } from './theme-provider'
import { Toaster } from '@/components/ui/toaster'

function Providers({ children }: { children: React.ReactNode }) {
  return (
    <>
      <Toaster />
      <ThemeProvider
        attribute='class'
        defaultTheme='system'
        enableSystem
        disableTransitionOnChange
      >
        {children}
      </ThemeProvider>
    </>
  )
}
export default Providers
```

### SignOutLink

- redirectUrl

```tsx
'use client'

import { SignOutButton } from '@clerk/nextjs'
import { useToast } from '../ui/use-toast'

function SignOutLink() {
  const { toast } = useToast()
  const handleLogout = () => {
    toast({ description: 'You have been signed out.' })
  }
  return (
    <SignOutButton redirectUrl='/'>
      <button className='w-full text-left' onClick={handleLogout}>
        Logout
      </button>
    </SignOutButton>
  )
}
export default SignOutLink
```

### LinksDropdown - Complete

```tsx
return (
  <DropdownMenuContent>
    <SignedOut>...</SignedOut>
    <SignedIn>....</SignedIn>
  </DropdownMenuContent>
)
```

```tsx
return (
  <DropdownMenuContent className='w-52' align='start' sideOffset={10}>
    <SignedOut>
      <DropdownMenuItem>
        <SignInButton mode='modal'>
          <button className='w-full text-left'>Login</button>
        </SignInButton>
      </DropdownMenuItem>
      <DropdownMenuSeparator />
      <DropdownMenuItem>
        <SignUpButton mode='modal'>
          <button className='w-full text-left'>Register</button>
        </SignUpButton>
      </DropdownMenuItem>
    </SignedOut>
    <SignedIn>
      {links.map((link) => {
        return (
          <DropdownMenuItem key={link.href}>
            <Link href={link.href} className='capitalize w-full'>
              {link.label}
            </Link>
          </DropdownMenuItem>
        )
      })}
      <DropdownMenuSeparator />
      <DropdownMenuItem>
        <SignOutLink />
      </DropdownMenuItem>
    </SignedIn>
  </DropdownMenuContent>
)
```

### Direct User

.env.local

```bash
NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL=/profile/create
NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL=/profile/create
```

### Create Profile

- profile
  - create

```tsx
import { Label } from '@/components/ui/label'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
const createProfileAction = async (formData: FormData) => {
  'use server'
  const firstName = formData.get('firstName') as string
  console.log(firstName)
}

function CreateProfile() {
  return (
    <section>
      <h1 className='text-2xl font-semibold mb-8 capitalize'>new user</h1>
      <div className='border p-8 rounded-md max-w-lg'>
        <form action={createProfileAction}>
          <div className='mb-2'>
            <Label htmlFor='firstName'>First Name</Label>
            <Input id='firstName' name='firstName' type='text' />
          </div>
          <Button type='submit' size='lg'>
            Create Profile
          </Button>
        </form>
      </div>
    </section>
  )
}
export default CreateProfile
```

### FormInput

- components/form/FormInput.tsx

```tsx
import { Label } from '../ui/label'
import { Input } from '../ui/input'

type FormInputProps = {
  name: string
  type: string
  label?: string
  defaultValue?: string
  placeholder?: string
}

function FormInput({
  label,
  name,
  type,
  defaultValue,
  placeholder,
}: FormInputProps) {
  return (
    <div className='mb-2'>
      <Label htmlFor={name} className='capitalize'>
        {label || name}
      </Label>
      <Input
        id={name}
        name={name}
        type={type}
        defaultValue={defaultValue}
        placeholder={placeholder}
        required
      />
    </div>
  )
}

export default FormInput
```

### Default Submit Button

- components/form/Buttons.tsx

```tsx
'use client'
import { ReloadIcon } from '@radix-ui/react-icons'
import { useFormStatus } from 'react-dom'
import { Button } from '@/components/ui/button'

type SubmitButtonProps = {
  className?: string
  text?: string
}

export function SubmitButton({
  className = '',
  text = 'submit',
}: SubmitButtonProps) {
  const { pending } = useFormStatus()
  return (
    <Button
      type='submit'
      disabled={pending}
      className={`capitalize ${className}`}
      size='lg'
    >
      {pending ? (
        <>
          <ReloadIcon className='mr-2 h-4 w-4 animate-spin' />
          Please wait...
        </>
      ) : (
        text
      )}
    </Button>
  )
}
```

### FormContainer

- create components/form/FormContainer.tsx

```tsx
'use client'

import { useFormState } from 'react-dom'
import { useEffect } from 'react'
import { useToast } from '@/components/ui/use-toast'
import { actionFunction } from '@/utils/types'

const initialState = {
  message: '',
}

function FormContainer({
  action,
  children,
}: {
  action: actionFunction
  children: React.ReactNode
}) {
  const [state, formAction] = useFormState(action, initialState)
  const { toast } = useToast()
  useEffect(() => {
    if (state.message) {
      toast({ description: state.message })
    }
  }, [state])
  return <form action={formAction}>{children}</form>
}
export default FormContainer
```

- create utils/types.ts

```ts
export type actionFunction = (
  prevState: any,
  formData: FormData
) => Promise<{ message: string }>
```

### Create Profile - Refactor

```tsx
import FormInput from '@/components/form/FormInput'
import { SubmitButton } from '@/components/form/Buttons'
import FormContainer from '@/components/form/FormContainer'

const createProfileAction = async (prevState: any, formData: FormData) => {
  'use server'
  const firstName = formData.get('firstName') as string
  if (firstName !== 'shakeAndBake') return { message: 'there was an error...' }
  return { message: 'Profile Created' }
}

function CreateProfile() {
  return (
    <section>
      <h1 className='text-2xl font-semibold mb-8 capitalize'>new user</h1>
      <div className='border p-8 rounded-md max-w-lg'>
        <FormContainer action={createProfileAction}>
          <div className='grid gap-4 mt-4 '>
            <FormInput type='text' name='firstName' label='First Name' />
            <FormInput type='text' name='lastName' label='Last Name' />
            <FormInput type='text' name='username' label='Username' />
          </div>
          <SubmitButton text='Create Profile' className='mt-8' />
        </FormContainer>
      </div>
    </section>
  )
}
export default CreateProfile
```

### Zod

Zod is a JavaScript library for building schemas and validating data, providing type safety and error handling.

```sh
npm install zod
```

- create utils/schemas.ts

```ts
import * as z from 'zod'
import { ZodSchema } from 'zod'

export const profileSchema = z.object({
  // firstName: z.string().max(5, { message: 'max length is 5' }),
  firstName: z.string(),
  lastName: z.string(),
  username: z.string(),
})
```

- create utils/actions.ts
- import in profile/create page.tsx

```ts
'use server'

import { profileSchema } from './schemas'

export const createProfileAction = async (
  prevState: any,
  formData: FormData
) => {
  try {
    const rawData = Object.fromEntries(formData)
    const validatedFields = profileSchema.parse(rawData)
    console.log(validatedFields)
    return { message: 'Profile Created' }
  } catch (error) {
    console.log(error)
    return { message: 'there was an error...' }
  }
}
```

## catche

### Supabase

- create account and organization
- create project
- setup password in .env (optional)
- add .env to .gitignore !!!
- it will take few minutes

### Prisma

- install prisma vs-code extension

Prisma ORM is a database toolkit that simplifies database access in web applications. It allows developers to interact with databases using a type-safe and auto-generated API, making database operations easier and more secure.

- Prisma server: A standalone infrastructure component sitting on top of your database.
- Prisma client: An auto-generated library that connects to the Prisma server and lets you read, write and stream data in your database. It is used for data access in your applications.

```sh
npm install prisma --save-dev
npm install @prisma/client
```

```sh
npx prisma init
```

### Setup Instance

In development, the command next dev clears Node.js cache on run. This in turn initializes a new PrismaClient instance each time due to hot reloading that creates a connection to the database. This can quickly exhaust the database connections as each PrismaClient instance holds its own connection pool.

(Prisma Instance)[https://www.prisma.io/docs/guides/other/troubleshooting-orm/help-articles/nextjs-prisma-client-dev-practices#solution]

- create utils/db.ts

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

### Connect Supabase with Prisma

[Useful Info](https://supabase.com/partners/integrations/prisma)

- add to .env

```bash
DATABASE_URL=""
DIRECT_URL=""
```

- DATABASE_URL : Transaction + Password + "?pgbouncer=true&connection_limit=1"
- DIRECT_URL : Session + Password

```prisma
datasource db {
  provider          = "postgresql"
  url               = env("DATABASE_URL")
  directUrl         = env("DIRECT_URL")
}

generator client {
  provider = "prisma-client-js"
}

model TestProfile {
id  String @id @default(uuid())
name String
}

```

- npx prisma migrate dev --name init
- npx prisma db push

npx prisma migrate dev --name init creates a new migration for your database schema
changes and applies it, while npx prisma db push directly updates the database schema without creating a migration. In the context of databases, a migration is set of operations, that modify the database schema, helping it evolve over time while preserving existing data.

```bash
npx prisma db push
```

```bash
npx prisma studio
```

## Optional - Prisma Crud

[Prisma Docs](https://www.prisma.io/docs/concepts/components/prisma-client/crud)

- Create Single Record

```js
const task = await prisma.task.create({
  data: {
    content: 'some task',
  },
})
```

- Get All Records

```js
const tasks = await prisma.task.findMany()
```

- Get record by ID or unique identifier

```js
// By unique identifier
const user = await prisma.user.findUnique({
  where: {
    email: 'elsa@prisma.io',
  },
})

// By ID
const task = await prisma.task.findUnique({
  where: {
    id: id,
  },
})
```

- Update Record

```js
const updateTask = await prisma.task.update({
  where: {
    id: id,
  },
  data: {
    content: 'updated task',
  },
})
```

- Update or create records

```js
const upsertTask = await prisma.task.upsert({
  where: {
    id: id,
  },
  update: {
    content: 'some value',
  },
  create: {
    content: 'some value',
  },
})
```

- Delete a single record

```js
const deleteTask = await prisma.task.delete({
  where: {
    id: id,
  },
})
```

### Profile Model

```prisma
model Profile {
  id           String     @id @default(uuid())
  clerkId      String     @unique
  firstName    String
  lastName     String
  username     String
  email        String
  profileImage String
  createdAt    DateTime   @default(now())
  updatedAt    DateTime   @updatedAt

}
```

```bash
npx prisma db push
```

```bash
npx prisma studio
```

## array schema buissuness rules

### CreateProfile Action - Complete

[Clerk User Metadata](https://clerk.com/docs/users/metadata)

```ts
import db from './db'
import { auth, clerkClient, currentUser } from '@clerk/nextjs/server'
import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export const createProfileAction = async (
  prevState: any,
  formData: FormData
) => {
  try {
    const user = await currentUser()
    if (!user) throw new Error('Please login to create a profile')

    const rawData = Object.fromEntries(formData)
    const validatedFields = profileSchema.parse(rawData)

    await db.profile.create({
      data: {
        clerkId: user.id,
        email: user.emailAddresses[0].emailAddress,
        profileImage: user.imageUrl ?? '',
        ...validatedFields,
      },
    })
    await clerkClient.users.updateUserMetadata(user.id, {
      privateMetadata: {
        hasProfile: true,
      },
    })
  } catch (error) {
    return {
      message: error instanceof Error ? error.message : 'An error occurred',
    }
  }
  redirect('/')
}
```

### FetchProfileImage

actions.ts

```ts
export const fetchProfileImage = async () => {
  const user = await currentUser()
  if (!user) return null

  const profile = await db.profile.findUnique({
    where: {
      clerkId: user.id,
    },
    select: {
      profileImage: true,
    },
  })
  return profile?.profileImage
}
```

- components/navbar/UserIcon.tsx

```tsx
import { LuUser2 } from 'react-icons/lu'
import { fetchProfileImage } from '@/utils/actions'

async function UserIcon() {
  const profileImage = await fetchProfileImage()

  if (profileImage)
    return (
      <img src={profileImage} className='w-6 h-6 rounded-full object-cover' />
    )
  return <LuUser2 className='w-6 h-6 bg-primary rounded-full text-white' />
}
export default UserIcon
```

### Modify Create Profile

```tsx
import { currentUser } from '@clerk/nextjs/server';

import { redirect } from 'next/navigation';
async function CreateProfile() {
  const user = await currentUser();
  if (user?.privateMetadata?.hasProfile) redirect('/');
  ....
}
```

### Update Profile

actions.ts

```ts
const getAuthUser = async () => {
  const user = await currentUser()
  if (!user) {
    throw new Error('You must be logged in to access this route')
  }
  if (!user.privateMetadata.hasProfile) redirect('/profile/create')
  return user
}
```

```ts
export const fetchProfile = async () => {
  const user = await getAuthUser()

  const profile = await db.profile.findUnique({
    where: {
      clerkId: user.id,
    },
  })
  if (!profile) return redirect('/profile/create')
  return profile
}
```

```ts
export const updateProfileAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  return { message: 'update profile action' }
}
```

app/profile/page.tsx

```tsx
import FormContainer from '@/components/form/FormContainer'
import { updateProfileAction, fetchProfile } from '@/utils/actions'
import FormInput from '@/components/form/FormInput'
import { SubmitButton } from '@/components/form/Buttons'

async function ProfilePage() {
  const profile = await fetchProfile()

  return (
    <section>
      <h1 className='text-2xl font-semibold mb-8 capitalize'>user profile</h1>
      <div className='border p-8 rounded-md'>
        {/* image input container */}

        <FormContainer action={updateProfileAction}>
          <div className='grid gap-4 md:grid-cols-2 mt-4 '>
            <FormInput
              type='text'
              name='firstName'
              label='First Name'
              defaultValue={profile.firstName}
            />
            <FormInput
              type='text'
              name='lastName'
              label='Last Name'
              defaultValue={profile.lastName}
            />
            <FormInput
              type='text'
              name='username'
              label='Username'
              defaultValue={profile.username}
            />
          </div>
          <SubmitButton text='Update Profile' className='mt-8' />
        </FormContainer>
      </div>
    </section>
  )
}
export default ProfilePage
```

actions.ts

```ts
export const updateProfileAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  const user = await getAuthUser()
  try {
    const rawData = Object.fromEntries(formData)

    const validatedFields = profileSchema.parse(rawData)

    await db.profile.update({
      where: {
        clerkId: user.id,
      },
      data: validatedFields,
    })
    revalidatePath('/profile')
    return { message: 'Profile updated successfully' }
  } catch (error) {
    return {
      message: error instanceof Error ? error.message : 'An error occurred',
    }
  }
}
```

### Alternative Error Handling

actions.ts

```ts
const renderError = (error: unknown): { message: string } => {
  console.log(error)
  return {
    message: error instanceof Error ? error.message : 'An error occurred',
  }
}
```

```ts
export const updateProfileAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  const user = await getAuthUser()
  try {
    const rawData = Object.fromEntries(formData)

    const validatedFields = profileSchema.safeParse(rawData)
    if (!validatedFields.success) {
      const errors = validatedFields.error.errors.map((error) => error.message)
      throw new Error(errors.join(','))
    }

    await db.profile.update({
      where: {
        clerkId: user.id,
      },
      data: validatedFields.data,
    })
    revalidatePath('/profile')
    return { message: 'Profile updated successfully' }
  } catch (error) {
    return renderError(error)
  }
}
```

### ValidateWithZodSchema

schemas.ts

```ts
export function validateWithZodSchema<T>(
  schema: ZodSchema<T>,
  data: unknown
): T {
  const result = schema.safeParse(data)
  if (!result.success) {
    const errors = result.error.errors.map((error) => error.message)

    throw new Error(errors.join(', '))
  }
  return result.data
}
```

actions.ts

```ts
// createProfileAction

const validatedFields = validateWithZodSchema(profileSchema, rawData)

// updateProfileAction
const validatedFields = validateWithZodSchema(profileSchema, rawData)

await db.profile.update({
  where: {
    clerkId: user.id,
  },
  data: validatedFields,
})
```
