#

## QnA

[network-aware]

1. how to extend the formContainer for images

- button w form status and a wrapper inc img (tho the input is in the form container)

2. how is validation different compare to dynamic content

- need security (prop bc dynamic urls) and instans of encrypted formdata not the usual formdata (input=type file) (client server architecture require form to encrypt)

[dynamic-w-static_assets]

3. how to cache the image [https://supabase.com/docs/reference/javascript/storage-from-download]

- need a bucket to send the batch of image data + remote pattern to recive the delivery url

[internationalized-products]

1. how to create the objects dynamically according to model (coupling)

- const name = Prisma.PropertyScalarFieldEnum.price

2. how is redirect handles

- with zod schema

3. how is the mapping of {name:value} mapped for select and textArea

- const name = 'category'
- <TextAreaInput name='description' labelText='Description (10 - 1000 Words)' />

## summary refined product complete creation of user /prod

[network-aware]

- useFormStatus pending (+ premote pattern https://nextjs.org/docs/pages/api-reference/components/image#remotepatterns)
- image imput formData is andled with accept --> do with https
- special submit button
- custom validation

[dynamic-w-static_assets]

- refine file
- SUPABASE_URL=, SUPABASE_KEY= bucker for image data
  target image upload validate and full path
  - bucket store and return delivery url to access img

[internationalized-products]

- action prev state and formData like redux {...state, newUpdate:data}
- property create with formInput
- range, select, textarea defaults

## network aware p1

### ImageInput

components/form/ImageInput.tsx

```tsx
import { Label } from '../ui/label'
import { Input } from '../ui/input'

function ImageInput() {
  const name = 'image'
  return (
    <div className='mb-2'>
      <Label htmlFor={name} className='capitalize'>
        Image
      </Label>
      <Input
        id={name}
        name={name}
        type='file'
        required
        accept='image/*'
        className='max-w-xs'
      />
    </div>
  )
}
export default ImageInput
```

### SubmitButton

```tsx
type btnSize = 'default' | 'lg' | 'sm'

type SubmitButtonProps = {
  className?: string
  text?: string
  size?: btnSize
}

export function SubmitButton({
  className = '',
  text = 'submit',
  size = 'lg',
}: SubmitButtonProps) {
  const { pending } = useFormStatus()
  return (
    <Button
      type='submit'
      disabled={pending}
      className={`capitalize ${className}`}
      size={size}
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

### ImageInputContainer

components/form/ImageInputContainer.tsx

```tsx
'use client'
import { useState } from 'react'
import Image from 'next/image'
import { Button } from '../ui/button'
import FormContainer from './FormContainer'
import ImageInput from './ImageInput'
import { SubmitButton } from './Buttons'
import { type actionFunction } from '@/utils/types'
import { LuUser2 } from 'react-icons/lu'

type ImageInputContainerProps = {
  image: string
  name: string
  action: actionFunction
  text: string
  children?: React.ReactNode
}

function ImageInputContainer(props: ImageInputContainerProps) {
  const { image, name, action, text } = props
  const [isUpdateFormVisible, setUpdateFormVisible] = useState(false)

  const userIcon = (
    <LuUser2 className='w-24 h-24 bg-primary rounded-md text-white mb-4' />
  )
  return (
    <div>
      {image ? (
        <Image
          src={image}
          width={100}
          height={100}
          className='rounded-md object-cover mb-4 w-24 h-24'
          alt={name}
        />
      ) : (
        userIcon
      )}

      <Button
        variant='outline'
        size='sm'
        onClick={() => setUpdateFormVisible((prev) => !prev)}
      >
        {text}
      </Button>
      {isUpdateFormVisible && (
        <div className='max-w-lg mt-4'>
          <FormContainer action={action}>
            {props.children}
            <ImageInput />
            <SubmitButton size='sm' />
          </FormContainer>
        </div>
      )}
    </div>
  )
}
export default ImageInputContainer
```

### updateProfileImageAction

actions.ts

```ts
export const updateProfileImageAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  return { message: 'Profile image updated successfully' }
}
```

### Profile Page

```tsx
import {
  updateProfileAction,
  fetchProfile,
  updateProfileImageAction,
} from '@/utils/actions'

import ImageInputContainer from '@/components/form/ImageInputContainer'

/* image input container */
;<ImageInputContainer
  image={profile.profileImage}
  name={profile.username}
  action={updateProfileImageAction}
  text='Update Profile Image'
/>
```

### Remote Patterns

```mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'img.clerk.com',
      },
    ],
  },
}

export default nextConfig
```

### imageSchema

schemas.ts

```ts
export const imageSchema = z.object({
  image: validateFile(),
})

function validateFile() {
  const maxUploadSize = 1024 * 1024
  const acceptedFileTypes = ['image/']
  return z
    .instanceof(File)
    .refine((file) => {
      return !file || file.size <= maxUploadSize
    }, `File size must be less than 1 MB`)
    .refine((file) => {
      return (
        !file || acceptedFileTypes.some((type) => file.type.startsWith(type))
      )
    }, 'File must be an image')
}
```

The .refine() method in Zod is used to add custom validation to a Zod schema. It takes two arguments:

A function that takes a value and returns a boolean. This function is the validation rule. If it returns true, the validation passes. If it returns false, the validation fails.
A string that is the error message to be returned when the validation fails.

### updateProfileImageAction

```ts
export const updateProfileImageAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  const user = await getAuthUser()
  try {
    const image = formData.get('image') as File
    const validatedFields = validateWithZodSchema(imageSchema, { image })

    return { message: 'Profile image updated successfully' }
  } catch (error) {
    return renderError(error)
  }
}
```

## dynamic-w-static_assets

### Create Bucket, Setup Policy and API Keys

```env
SUPABASE_URL=
SUPABASE_KEY=
```

### Setup Supabase

```sh
npm install @supabase/supabase-js
```

utils/supabase.ts

```ts
import { createClient } from '@supabase/supabase-js'

const bucket = 'home-away-draft'

// Create a single supabase client for interacting with your database
export const supabase = createClient(
  process.env.SUPABASE_URL as string,
  process.env.SUPABASE_KEY as string
)

export const uploadImage = async (image: File) => {
  const timestamp = Date.now()
  // const newName = `/users/${timestamp}-${image.name}`;
  const newName = `${timestamp}-${image.name}`

  const { data, error } = await supabase.storage
    .from(bucket)
    .upload(newName, image, {
      cacheControl: '3600',
    })
  if (!data) throw new Error('Image upload failed')
  return supabase.storage.from(bucket).getPublicUrl(newName).data.publicUrl
}
```

### updateProfileImageAction

```ts
export const updateProfileImageAction = async (
  prevState: any,
  formData: FormData
) => {
  const user = await getAuthUser()
  try {
    const image = formData.get('image') as File
    const validatedFields = validateWithZodSchema(imageSchema, { image })
    const fullPath = await uploadImage(validatedFields.image)

    await db.profile.update({
      where: {
        clerkId: user.id,
      },
      data: {
        profileImage: fullPath,
      },
    })
    revalidatePath('/profile')
    return { message: 'Profile image updated successfully' }
  } catch (error) {
    return renderError(error)
  }
}
```

### Remote Patterns

```mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'img.clerk.com',
      },
      {
        protocol: 'https',
        hostname: 'jxdujzgweuaphpgoowhu.supabase.co',
      },
    ],
  },
}

export default nextConfig
```

## internationalized-products (user imputs)

### Property Model

```prisma
model Profile {
  properties      Property[]
}

model Property {
  id          String     @id @default(uuid())
  name        String
  tagline     String
  category    String
  image       String
  country     String
  description String
  price       Int
  guests      Int
  bedrooms    Int
  beds        Int
  baths       Int
  amenities   String
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  profile     Profile    @relation(fields: [profileId], references: [clerkId], onDelete: Cascade)
  profileId   String
}
```

### Property Schema

- yes, no image 😜

schemas.ts

```ts
export const propertySchema = z.object({
  name: z
    .string()
    .min(2, {
      message: 'name must be at least 2 characters.',
    })
    .max(100, {
      message: 'name must be less than 100 characters.',
    }),
  tagline: z
    .string()
    .min(2, {
      message: 'tagline must be at least 2 characters.',
    })
    .max(100, {
      message: 'tagline must be less than 100 characters.',
    }),
  price: z.coerce.number().int().min(0, {
    message: 'price must be a positive number.',
  }),
  category: z.string(),
  description: z.string().refine(
    (description) => {
      const wordCount = description.split(' ').length
      return wordCount >= 10 && wordCount <= 1000
    },
    {
      message: 'description must be between 10 and 1000 words.',
    }
  ),
  country: z.string(),
  guests: z.coerce.number().int().min(0, {
    message: 'guest amount must be a positive number.',
  }),
  bedrooms: z.coerce.number().int().min(0, {
    message: 'bedrooms amount must be a positive number.',
  }),
  beds: z.coerce.number().int().min(0, {
    message: 'beds amount must be a positive number.',
  }),
  baths: z.coerce.number().int().min(0, {
    message: 'bahts amount must be a positive number.',
  }),
  amenities: z.string(),
})
```

### createPropertyAction

actions.ts

```ts
export const createPropertyAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  const user = await getAuthUser()
  try {
    const rawData = Object.fromEntries(formData)
    const validatedFields = validateWithZodSchema(propertySchema, rawData)
  } catch (error) {
    return renderError(error)
  }
  redirect('/')
}
```

### Create Rental Page

- app/rentals/create/page.tsx

```tsx
import FormInput from '@/components/form/FormInput'
import FormContainer from '@/components/form/FormContainer'
import { createPropertyAction } from '@/utils/actions'
import { SubmitButton } from '@/components/form/Buttons'

function CreateProperty() {
  return (
    <section>
      <h1 className='text-2xl font-semibold mb-8 capitalize'>
        create property
      </h1>
      <div className='border p-8 rounded-md'>
        <h3 className='text-lg mb-4 font-medium'>General Info</h3>
        <FormContainer action={createPropertyAction}>
          <div className='grid md:grid-cols-2 gap-8 mb-4'>
            <FormInput
              name='name'
              type='text'
              label='Name (20 limit)'
              defaultValue='Cabin in Latvia'
            />
            <FormInput
              name='tagline'
              type='text '
              label='Tagline (30 limit)'
              defaultValue='Dream Getaway Awaits You Here!'
            />
            {/* price */}
            {/* categories */}
          </div>
          {/* text area / description */}
          <SubmitButton text='create rental' className='mt-12' />
        </FormContainer>
      </div>
    </section>
  )
}
export default CreateProperty
```

### Price Input

- components/form/PriceInput.tsx

```ts
import { Label } from '../ui/label'
import { Input } from '../ui/input'
import { Prisma } from '@prisma/client'

const name = Prisma.PropertyScalarFieldEnum.price
// const name = 'price';
type FormInputNumberProps = {
  defaultValue?: number
}

function PriceInput({ defaultValue }: FormInputNumberProps) {
  return (
    <div className='mb-2'>
      <Label htmlFor='price' className='capitalize'>
        Price ($)
      </Label>
      <Input
        id={name}
        type='number'
        name={name}
        min={0}
        defaultValue={defaultValue || 100}
        required
      />
    </div>
  )
}
export default PriceInput
```

```tsx
/* price */
<PriceInput />
```

### Categories Data

- utils/categories.ts

```ts
import { IconType } from 'react-icons'
import { MdCabin } from 'react-icons/md'

import { TbCaravan, TbTent, TbBuildingCottage } from 'react-icons/tb'

import { GiWoodCabin, GiMushroomHouse } from 'react-icons/gi'
import { PiWarehouse, PiLighthouse, PiVan } from 'react-icons/pi'

import { GoContainer } from 'react-icons/go'

type Category = {
  label: CategoryLabel
  icon: IconType
}

export type CategoryLabel =
  | 'cabin'
  | 'tent'
  | 'airstream'
  | 'cottage'
  | 'container'
  | 'caravan'
  | 'tiny'
  | 'magic'
  | 'warehouse'
  | 'lodge'

export const categories: Category[] = [
  {
    label: 'cabin',
    icon: MdCabin,
  },
  {
    label: 'airstream',
    icon: PiVan,
  },
  {
    label: 'tent',
    icon: TbTent,
  },
  {
    label: 'warehouse',
    icon: PiWarehouse,
  },
  {
    label: 'cottage',
    icon: TbBuildingCottage,
  },
  {
    label: 'magic',
    icon: GiMushroomHouse,
  },
  {
    label: 'container',
    icon: GoContainer,
  },
  {
    label: 'caravan',
    icon: TbCaravan,
  },

  {
    label: 'tiny',
    icon: PiLighthouse,
  },
  {
    label: 'lodge',
    icon: GiWoodCabin,
  },
]
```

### Categories Input

```tsx
import { Label } from '@/components/ui/label'
import { categories } from '@/utils/categories'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'

const name = 'category'
function CategoriesInput({ defaultValue }: { defaultValue?: string }) {
  return (
    <div className='mb-2'>
      <Label htmlFor={name} className='capitalize'>
        Categories
      </Label>
      <Select
        defaultValue={defaultValue || categories[0].label}
        name={name}
        required
      >
        <SelectTrigger id={name}>
          <SelectValue />
        </SelectTrigger>
        <SelectContent>
          {categories.map((item) => {
            return (
              <SelectItem key={item.label} value={item.label}>
                <span className='flex items-center gap-2'>
                  <item.icon /> {item.label}
                </span>
              </SelectItem>
            )
          })}
        </SelectContent>
      </Select>
    </div>
  )
}
export default CategoriesInput
```

```tsx
/* categories */
<CategoriesInput />
```

### TextArea Input

- components/form/TextAreaInput.tsx

```tsx
import { Label } from '@/components/ui/label'
import { Textarea } from '@/components/ui/textarea'

type TextAreaInputProps = {
  name: string
  labelText?: string
  defaultValue?: string
}

function TextAreaInput({ name, labelText, defaultValue }: TextAreaInputProps) {
  return (
    <div className='mb-2'>
      <Label htmlFor={name} className='capitalize'>
        {labelText || name}
      </Label>
      <Textarea
        id={name}
        name={name}
        defaultValue={defaultValue || tempDefaultDescription}
        rows={5}
        required
        className='leading-loose'
      />
    </div>
  )
}

const tempDefaultDescription =
  'Glamping Tuscan Style in an Aframe Cabin Tent, nestled in a beautiful olive orchard. AC, heat, Queen Bed, TV, Wi-Fi and an amazing view. Close to Weeki Wachee River State Park, mermaids, manatees, Chassahwitzka River and on the SC Bike Path. Kayaks available for rivers. Bathhouse, fire pit, Kitchenette, fresh eggs. Relax & enjoy fresh country air. No pets please. Ducks, hens and roosters roam the grounds. We have a Pot Cake Rescue from Bimini, Retriever and Pom dog. The space is inspiring and relaxing. Enjoy the beauty of the orchard. Spring trees are in blossom and harvested in Fall. We have a farm store where we sell our farm to table products'
export default TextAreaInput
```

```tsx
/* text area / description */
<TextAreaInput name='description' labelText='Description (10 - 1000 Words)' />
```
