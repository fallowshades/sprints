#

## QnA

[Internationalization]

1. how is the select obj created

- const name = 'country'

2. how to handle checkboxes

- input, checkbox props, icon from obj

3. difference between action and load

- action has try catch and may redirect, load = findMany + select

[Rental-headBdyFooter]

1. how is search params already managed.

- router give as props (optional existance ?:)

2. how is params set

- hoisted with href link (concat: href={`/?category=${item.label}${searchTerm}`} + const searchTerm = search ? `&search=${search}` : '')

## id page combine dynamic static and user, All page query params (search, select)

[Internationalization]

- form client counter for batch wo type
- countries map and find
- json stringify naked obj on server --> client browser
- imput value obj.stringify -> checkboxes
  -- have iso standard, slide through
  -- map list of boolean

[Rental-headBdyFooter]

- optional params obj from root page --> construct query string
- search term coupled with amenities with link search term
- image and profile with head(search) body(container)
- item.icon componize emptyList, List

## pickable obj

### create id

#### Countries Input

```sh
npm i world-countries
```

- utils/countries.ts

```ts
import countries from 'world-countries'

export const formattedCountries = countries.map((item) => ({
  code: item.cca2,
  name: item.name.common,
  flag: item.flag,
  location: item.latlng,
  region: item.region,
}))
export const findCountryByCode = (code: string) =>
  formattedCountries.find((item) => item.code === code)
```

- components/form/CountriesInput.tsx

```tsx
import { Label } from '@/components/ui/label'
import { formattedCountries } from '@/utils/countries'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'

const name = 'country'
function CountriesInput({ defaultValue }: { defaultValue?: string }) {
  return (
    <div className='mb-2'>
      <Label htmlFor={name} className='capitalize'>
        country
      </Label>
      <Select
        defaultValue={defaultValue || formattedCountries[0].code}
        name={name}
        required
      >
        <SelectTrigger id={name}>
          <SelectValue />
        </SelectTrigger>
        <SelectContent>
          {formattedCountries.map((item) => {
            return (
              <SelectItem key={item.code} value={item.code}>
                <span className='flex items-center gap-2'>
                  {item.flag} {item.name}
                </span>
              </SelectItem>
            )
          })}
        </SelectContent>
      </Select>
    </div>
  )
}
export default CountriesInput
```

```tsx
<div className='grid sm:grid-cols-2 gap-8 mt-4'>
  <CountriesInput />
  <ImageInput />
</div>
```

#### Accommodation / Counter Input

- components/form/CounterInput.tsx

```tsx
'use client'
import { Card, CardHeader } from '@/components/ui/card'
import { LuMinus, LuPlus } from 'react-icons/lu'

import { Button } from '../ui/button'
import { useState } from 'react'

function CounterInput({
  detail,
  defaultValue,
}: {
  detail: string
  defaultValue?: number
}) {
  const [count, setCount] = useState(defaultValue || 0)
  const increaseCount = () => {
    setCount((prevCount) => prevCount + 1)
  }
  const decreaseCount = () => {
    setCount((prevCount) => {
      if (prevCount > 0) {
        return prevCount - 1
      }
      return prevCount
    })
  }
  return (
    <Card className='mb-4'>
      <input type='hidden' name={detail} value={count} />
      <CardHeader className='flex flex-col gapy-5'>
        <div className='flex items-center justify-between flex-wrap'>
          <div className='flex flex-col'>
            <h2 className='font-medium capitalize'>{detail}</h2>
            <p className='text-muted-foreground text-sm'>
              Specify the number of {detail}
            </p>
          </div>
          <div className='flex items-center gap-4'>
            <Button
              variant='outline'
              size='icon'
              type='button'
              onClick={decreaseCount}
            >
              <LuMinus className='w-5 h-5 text-primary' />
            </Button>
            <span className='text-xl font-bold w-5 text-center'>{count}</span>
            <Button
              variant='outline'
              size='icon'
              type='button'
              onClick={increaseCount}
            >
              <LuPlus className='w-5 h-5 text-primary' />
            </Button>
          </div>
        </div>
      </CardHeader>
    </Card>
  )
}

export default CounterInput
```

```tsx
return (
  <>
    <h3 className='text-lg mt-8 mb-4 font-medium'>Accommodation Details</h3>
    <CounterInput detail='guests' />
    <CounterInput detail='bedrooms' />
    <CounterInput detail='beds' />
    <CounterInput detail='baths' />
  </>
)
```

#### Amenities

- utils/amenities.ts

```ts
import { IconType } from 'react-icons'
export type Amenity = {
  name: string
  icon: IconType
  selected: boolean
}
import {
  FiCloud,
  FiTruck,
  FiZap,
  FiWind,
  FiSun,
  FiCoffee,
  FiFeather,
  FiAirplay,
  FiTrello,
  FiBox,
  FiAnchor,
  FiDroplet,
  FiMapPin,
  FiSunrise,
  FiSunset,
  FiMusic,
  FiHeadphones,
  FiRadio,
  FiFilm,
  FiTv,
} from 'react-icons/fi'

export const amenities: Amenity[] = [
  { name: 'unlimited cloud storage', icon: FiCloud, selected: false },
  { name: 'VIP parking for squirrels', icon: FiTruck, selected: false },
  { name: 'self-lighting fire pit', icon: FiZap, selected: false },
  {
    name: 'bbq grill with a masterchef diploma',
    icon: FiWind,
    selected: false,
  },
  { name: 'outdoor furniture (tree stumps)', icon: FiSun, selected: false },
  { name: 'private bathroom (bushes nearby)', icon: FiCoffee, selected: false },
  { name: 'hot shower (sun required)', icon: FiFeather, selected: false },
  { name: 'kitchenette (aka fire pit)', icon: FiAirplay, selected: false },
  { name: 'natural heating (bring a coat)', icon: FiTrello, selected: false },
  {
    name: 'air conditioning (breeze from the west)',
    icon: FiBox,
    selected: false,
  },
  { name: 'bed linens (leaves)', icon: FiAnchor, selected: false },
  { name: 'towels (more leaves)', icon: FiDroplet, selected: false },
  {
    name: 'picnic table (yet another tree stump)',
    icon: FiMapPin,
    selected: false,
  },
  { name: 'hammock (two trees and a rope)', icon: FiSunrise, selected: false },
  { name: 'solar power (daylight)', icon: FiSunset, selected: false },
  { name: 'water supply (river a mile away)', icon: FiMusic, selected: false },
  {
    name: 'cooking utensils (sticks and stones)',
    icon: FiHeadphones,
    selected: false,
  },
  { name: 'cool box (hole in the ground)', icon: FiRadio, selected: false },
  { name: 'lanterns (fireflies)', icon: FiFilm, selected: false },
  { name: 'first aid kit (hope and prayers)', icon: FiTv, selected: false },
]

export const conservativeAmenities: Amenity[] = [
  { name: 'cloud storage', icon: FiCloud, selected: false },
  { name: 'parking', icon: FiTruck, selected: false },
  { name: 'fire pit', icon: FiZap, selected: false },
  { name: 'bbq grill', icon: FiWind, selected: false },
  { name: 'outdoor furniture', icon: FiSun, selected: false },
  { name: 'private bathroom', icon: FiCoffee, selected: false },
  { name: 'hot shower', icon: FiFeather, selected: false },
  { name: 'kitchenette', icon: FiAirplay, selected: false },
  { name: 'heating', icon: FiTrello, selected: false },
  { name: 'air conditioning', icon: FiBox, selected: false },
  { name: 'bed linens', icon: FiAnchor, selected: false },
  { name: 'towels', icon: FiDroplet, selected: false },
  { name: 'picnic table', icon: FiMapPin, selected: false },
  { name: 'hammock', icon: FiSunrise, selected: false },
  { name: 'solar power', icon: FiSunset, selected: false },
  { name: 'water supply', icon: FiMusic, selected: false },
  { name: 'cooking utensils', icon: FiHeadphones, selected: false },
  { name: 'cool box', icon: FiRadio, selected: false },
  { name: 'lanterns', icon: FiFilm, selected: false },
  { name: 'first aid kit', icon: FiTv, selected: false },
]
```

- components/form/AmenitiesInput.tsx

```tsx
'use client'
import { useState } from 'react'
import { amenities, Amenity } from '@/utils/amenities'
import { Checkbox } from '@/components/ui/checkbox'

function AmenitiesInput({ defaultValue }: { defaultValue?: Amenity[] }) {
  const [selectedAmenities, setSelectedAmenities] = useState<Amenity[]>(
    defaultValue || amenities
  )

  const handleChange = (amenity: Amenity) => {
    setSelectedAmenities((prev) => {
      return prev.map((a) => {
        if (a.name === amenity.name) {
          return { ...a, selected: !a.selected }
        }
        return a
      })
    })
  }

  return (
    <section>
      <input
        type='hidden'
        name='amenities'
        value={JSON.stringify(selectedAmenities)}
      />
      <div className='grid grid-cols-2 gap-4'>
        {selectedAmenities.map((amenity) => (
          <div key={amenity.name} className='flex items-center space-x-2'>
            <Checkbox
              id={amenity.name}
              checked={amenity.selected}
              onCheckedChange={() => handleChange(amenity)}
            />
            <label
              htmlFor={amenity.name}
              className='text-sm font-medium leading-none capitalize flex gap-x-2 items-center'
            >
              {amenity.name}
              <amenity.icon className='w-4 h-4' />
            </label>
          </div>
        ))}
      </div>
    </section>
  )
}
export default AmenitiesInput
```

```tsx
return (
  <>
    <h3 className='text-lg mt-10 mb-6 font-medium'>Amenities</h3>
    <AmenitiesInput />
  </>
)
```

#### createRentalAction

```tsx
export const createPropertyAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  const user = await getAuthUser()
  try {
    const rawData = Object.fromEntries(formData)
    const file = formData.get('image') as File

    const validatedFields = validateWithZodSchema(propertySchema, rawData)
    const validatedFile = validateWithZodSchema(imageSchema, { image: file })
    const fullPath = await uploadImage(validatedFile.image)

    await db.property.create({
      data: {
        ...validatedFields,
        image: fullPath,
        profileId: user.id,
      },
    })
  } catch (error) {
    return renderError(error)
  }
  redirect('/')
}
```

### head body and footer structure

#### fetchProperties

utils/types.ts

```ts
export type PropertyCardProps = {
  image: string
  id: string
  name: string
  tagline: string
  country: string
  price: number
}
```

actions.ts

```ts
export const fetchProperties = async ({
  search = '',
  category,
}: {
  search?: string
  category?: string
}) => {
  const properties = await db.property.findMany({
    where: {
      category,
      OR: [
        { name: { contains: search, mode: 'insensitive' } },
        { tagline: { contains: search, mode: 'insensitive' } },
      ],
    },
    select: {
      id: true,
      name: true,
      tagline: true,
      country: true,
      image: true,
      price: true,
    },
  })
  return properties
}
```

## map inputs from arr and in obj

#### Home Page

- create in components/home
  - CategoriesList.tsx
  - EmptyList.tsx
  - PropertiesContainer.tsx
  - PropertiesList.tsx

```tsx
import CategoriesList from '@/components/home/CategoriesList'
import PropertiesContainer from '@/components/home/PropertiesContainer'

function HomePage() {
  return (
    <section>
      <CategoriesList />
      <PropertiesContainer />
    </section>
  )
}
export default HomePage
```

#### Search Params

```tsx
import CategoriesList from '@/components/home/CategoriesList'
import PropertiesContainer from '@/components/home/PropertiesContainer'

function HomePage({
  searchParams,
}: {
  searchParams: { category?: string; search?: string }
}) {
  // console.log(searchParams);

  return (
    <section>
      <CategoriesList
        category={searchParams?.category}
        search={searchParams?.search}
      />
      <PropertiesContainer
        category={searchParams?.category}
        search={searchParams?.search}
      />
    </section>
  )
}
export default HomePage
```

#### CategoriesList

```tsx
import { categories } from '@/utils/categories'
import { ScrollArea, ScrollBar } from '../ui/scroll-area'
import Link from 'next/link'

function CategoriesList({
  category,
  search,
}: {
  category?: string
  search?: string
}) {
  const searchTerm = search ? `&search=${search}` : ''
  return (
    <section>
      <ScrollArea className='py-6'>
        <div className='flex gap-x-4'>
          {categories.map((item) => {
            const isActive = item.label === category
            return (
              <Link
                key={item.label}
                href={`/?category=${item.label}${searchTerm}`}
              >
                <article
                  className={`p-3 flex flex-col items-center cursor-pointer duration-300  hover:text-primary w-[100px] ${
                    isActive ? 'text-primary' : ''
                  }`}
                >
                  <item.icon className='w-8 h-8 ' />
                  <p className='capitalize text-sm mt-1'>{item.label}</p>
                </article>
              </Link>
            )
          })}
        </div>
        <ScrollBar orientation='horizontal' />
      </ScrollArea>
    </section>
  )
}
export default CategoriesList
```

#### EmptyList

```tsx
import { Button } from '../ui/button'
import Link from 'next/link'

function EmptyList({
  heading = 'No items in the list.',
  message = 'Keep exploring our properties.',
  btnText = 'back home',
}: {
  heading?: string
  message?: string
  btnText?: string
}) {
  return (
    <div className='mt-4'>
      <h2 className='text-xl font-bold '>{heading}</h2>
      <p className='text-lg'>{message}</p>
      <Button asChild className='mt-4 capitalize' size='lg'>
        <Link href='/'>{btnText}</Link>
      </Button>
    </div>
  )
}
export default EmptyList
```

#### PropertiesContainer

```tsx
import { fetchProperties } from '@/utils/actions'
import PropertiesList from './PropertiesList'
import EmptyList from './EmptyList'
import type { PropertyCardProps } from '@/utils/types'

async function PropertiesContainer({
  category,
  search,
}: {
  category?: string
  search?: string
}) {
  const properties: PropertyCardProps[] = await fetchProperties({
    category,
    search,
  })

  if (properties.length === 0) {
    return (
      <EmptyList
        heading='No results.'
        message='Try changing or removing some of your filters.'
        btnText='Clear Filters'
      />
    )
  }

  return <PropertiesList properties={properties} />
}
export default PropertiesContainer
```

#### Card Components

- components/card
  - CountryFlagAndName.tsx
  - FavoriteToggleButton.tsx
  - FavoriteToggleForm.tsx
  - LoadingCards.tsx
  - PropertyCard.tsx
  - PropertyRating.tsx

#### PropertiesList

```tsx
import PropertyCard from '../card/PropertyCard'
import type { PropertyCardProps } from '@/utils/types'

function PropertiesList({ properties }: { properties: PropertyCardProps[] }) {
  return (
    <section className='mt-4 gap-8 grid sm:grid-cols-2  lg:grid-cols-3  xl:grid-cols-4'>
      {properties.map((property) => {
        return <PropertyCard key={property.id} property={property} />
      })}
    </section>
  )
}
export default PropertiesList
```
