#

## QnA

[1]

1.

[2]

1. link from id nested route

[3]

##

### load and action

[query]

- aggregated boolean (spreed booking w property)(delete binding)
- map promise (action w type and link bind url)
- table page (head,bdy,cell)

### edit multiple

[default-value_mapped-w-arry-find-icon]

- map find and spread (target affected, edit formcontainer + submit)
- input w value instead of useState? (json.stringify amenities)

### reservations

[id-and-revalidate-path]

- image seperate update with redirect (update)
- fetch reservation of user (fetch)

## load and action

### LoadingTable

- create @/components/booking/LoadingTable.tsx

```tsx
import { Skeleton } from '../ui/skeleton'

function LoadingTable({ rows }: { rows?: number }) {
  const tableRows = Array.from({ length: rows || 5 }, (_, i) => {
    return (
      <div className='mb-4' key={i}>
        <Skeleton className='w-full h-8 rounded' />
      </div>
    )
  })
  return <>{tableRows}</>
}
export default LoadingTable
```

- create app/bookings/loading.tsx

```tsx
'use client'

import LoadingTable from '@/components/booking/LoadingTable'

function loading() {
  return <LoadingTable />
}

export default loading
```

### Fetch and Delete Rentals

- actions.ts

```ts
export const fetchRentals = async () => {
  const user = await getAuthUser()
  const rentals = await db.property.findMany({
    where: {
      profileId: user.id,
    },
    select: {
      id: true,
      name: true,
      price: true,
    },
  })

  const rentalsWithBookingSums = await Promise.all(
    rentals.map(async (rental) => {
      const totalNightsSum = await db.booking.aggregate({
        where: {
          propertyId: rental.id,
        },
        _sum: {
          totalNights: true,
        },
      })

      const orderTotalSum = await db.booking.aggregate({
        where: {
          propertyId: rental.id,
        },
        _sum: {
          orderTotal: true,
        },
      })

      return {
        ...rental,
        totalNightsSum: totalNightsSum._sum.totalNights,
        orderTotalSum: orderTotalSum._sum.orderTotal,
      }
    })
  )

  return rentalsWithBookingSums
}

export async function deleteRentalAction(prevState: { propertyId: string }) {
  const { propertyId } = prevState
  const user = await getAuthUser()

  try {
    await db.property.delete({
      where: {
        id: propertyId,
        profileId: user.id,
      },
    })

    revalidatePath('/rentals')
    return { message: 'Rental deleted successfully' }
  } catch (error) {
    return renderError(error)
  }
}
```

### Rentals Page

- create rentals/loading.tsx

```tsx
'use client'
import LoadingTable from '@/components/booking/LoadingTable'
function loading() {
  return <LoadingTable />
}
export default loading
```

```tsx
import EmptyList from '@/components/home/EmptyList'
import { fetchRentals, deleteRentalAction } from '@/utils/actions'
import Link from 'next/link'

import { formatCurrency } from '@/utils/format'
import {
  Table,
  TableBody,
  TableCaption,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'

import FormContainer from '@/components/form/FormContainer'
import { IconButton } from '@/components/form/Buttons'

async function RentalsPage() {
  const rentals = await fetchRentals()

  if (rentals.length === 0) {
    return (
      <EmptyList
        heading='No rentals to display.'
        message="Don't hesitate to create a rental."
      />
    )
  }

  return (
    <div className='mt-16'>
      <h4 className='mb-4 capitalize'>Active Properties : {rentals.length}</h4>
      <Table>
        <TableCaption>A list of all your properties.</TableCaption>
        <TableHeader>
          <TableRow>
            <TableHead>Property Name</TableHead>
            <TableHead>Nightly Rate </TableHead>
            <TableHead>Nights Booked</TableHead>
            <TableHead>Total Income</TableHead>
            <TableHead>Actions</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {rentals.map((rental) => {
            const { id: propertyId, name, price } = rental
            const { totalNightsSum, orderTotalSum } = rental
            return (
              <TableRow key={propertyId}>
                <TableCell>
                  <Link
                    href={`/properties/${propertyId}`}
                    className='underline text-muted-foreground tracking-wide'
                  >
                    {name}
                  </Link>
                </TableCell>
                <TableCell>{formatCurrency(price)}</TableCell>
                <TableCell>{totalNightsSum || 0}</TableCell>
                <TableCell>{formatCurrency(orderTotalSum)}</TableCell>

                <TableCell className='flex items-center gap-x-2'>
                  <Link href={`/rentals/${propertyId}/edit`}>
                    <IconButton actionType='edit'></IconButton>
                  </Link>
                  <DeleteRental propertyId={propertyId} />
                </TableCell>
              </TableRow>
            )
          })}
        </TableBody>
      </Table>
    </div>
  )
}

function DeleteRental({ propertyId }: { propertyId: string }) {
  const deleteRental = deleteRentalAction.bind(null, { propertyId })
  return (
    <FormContainer action={deleteRental}>
      <IconButton actionType='delete' />
    </FormContainer>
  )
}

export default RentalsPage
```

## edit id

### Fetch Rental Details

- actions.ts

```ts
export const fetchRentalDetails = async (propertyId: string) => {
  const user = await getAuthUser()

  return db.property.findUnique({
    where: {
      id: propertyId,
      profileId: user.id,
    },
  })
}

export const updatePropertyAction = async () => {
  return { message: 'update property action' }
}

export const updatePropertyImageAction = async () => {
  return { message: 'update property image' }
}
```

### Rentals Edit Page

- rentals/[id]/edit/page.tsx

```tsx
import {
  fetchRentalDetails,
  updatePropertyImageAction,
  updatePropertyAction,
} from '@/utils/actions'
import FormContainer from '@/components/form/FormContainer'
import FormInput from '@/components/form/FormInput'
import CategoriesInput from '@/components/form/CategoriesInput'
import PriceInput from '@/components/form/PriceInput'
import TextAreaInput from '@/components/form/TextAreaInput'
import CountriesInput from '@/components/form/CountriesInput'
import CounterInput from '@/components/form/CounterInput'
import AmenitiesInput from '@/components/form/AmenitiesInput'
import { SubmitButton } from '@/components/form/Buttons'
import { redirect } from 'next/navigation'
import { type Amenity } from '@/utils/amenities'
import ImageInputContainer from '@/components/form/ImageInputContainer'

async function EditRentalPage({ params }: { params: { id: string } }) {
  const property = await fetchRentalDetails(params.id)

  if (!property) redirect('/')

  const defaultAmenities: Amenity[] = JSON.parse(property.amenities)

  return (
    <section>
      <h1 className='text-2xl font-semibold mb-8 capitalize'>Edit Property</h1>
      <div className='border p-8 rounded-md '>
        <ImageInputContainer
          name={property.name}
          text='Update Image'
          action={updatePropertyImageAction}
          image={property.image}
        >
          <input type='hidden' name='id' value={property.id} />
        </ImageInputContainer>

        <FormContainer action={updatePropertyAction}>
          <input type='hidden' name='id' value={property.id} />
          <div className='grid md:grid-cols-2 gap-8 mb-4 mt-8'>
            <FormInput
              name='name'
              type='text'
              label='Name (20 limit)'
              defaultValue={property.name}
            />
            <FormInput
              name='tagline'
              type='text '
              label='Tagline (30 limit)'
              defaultValue={property.tagline}
            />
            <PriceInput defaultValue={property.price} />
            <CategoriesInput defaultValue={property.category} />
            <CountriesInput defaultValue={property.country} />
          </div>

          <TextAreaInput
            name='description'
            labelText='Description (10 - 100 Words)'
            defaultValue={property.description}
          />

          <h3 className='text-lg mt-8 mb-4 font-medium'>
            Accommodation Details
          </h3>
          <CounterInput detail='guests' defaultValue={property.guests} />
          <CounterInput detail='bedrooms' defaultValue={property.bedrooms} />
          <CounterInput detail='beds' defaultValue={property.beds} />
          <CounterInput detail='baths' defaultValue={property.baths} />
          <h3 className='text-lg mt-10 mb-6 font-medium'>Amenities</h3>
          <AmenitiesInput defaultValue={defaultAmenities} />
          <SubmitButton text='edit property' className='mt-12' />
        </FormContainer>
      </div>
    </section>
  )
}
export default EditRentalPage
```

### Amenities Input

```tsx
'use client'
import { useState } from 'react'
import { amenities, Amenity } from '@/utils/amenities'
import { Checkbox } from '@/components/ui/checkbox'

function AmenitiesInput({ defaultValue }: { defaultValue?: Amenity[] }) {
  const amenitiesWithIcons = defaultValue?.map(({ name, selected }) => ({
    name,
    selected,
    icon: amenities.find((amenity) => amenity.name === name)!.icon,
  }))
  const [selectedAmenities, setSelectedAmenities] = useState<Amenity[]>(
    amenitiesWithIcons || amenities
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
        {selectedAmenities.map((amenity) => {
          return (
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
                {amenity.name} <amenity.icon className='w-4 h-4' />
              </label>
            </div>
          )
        })}
      </div>
    </section>
  )
}
export default AmenitiesInput
```

## reservations

### Update Property Action

```ts
export const updatePropertyAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  const user = await getAuthUser()
  const propertyId = formData.get('id') as string

  try {
    const rawData = Object.fromEntries(formData)
    const validatedFields = validateWithZodSchema(propertySchema, rawData)
    await db.property.update({
      where: {
        id: propertyId,
        profileId: user.id,
      },
      data: {
        ...validatedFields,
      },
    })

    revalidatePath(`/rentals/${propertyId}/edit`)
    return { message: 'Update Successful' }
  } catch (error) {
    return renderError(error)
  }
}
```

### Update Property Image Action

```ts
export const updatePropertyImageAction = async (
  prevState: any,
  formData: FormData
): Promise<{ message: string }> => {
  const user = await getAuthUser()
  const propertyId = formData.get('id') as string

  try {
    const image = formData.get('image') as File
    const validatedFields = validateWithZodSchema(imageSchema, { image })
    const fullPath = await uploadImage(validatedFields.image)

    await db.property.update({
      where: {
        id: propertyId,
        profileId: user.id,
      },
      data: {
        image: fullPath,
      },
    })
    revalidatePath(`/rentals/${propertyId}/edit`)
    return { message: 'Property Image Updated Successful' }
  } catch (error) {
    return renderError(error)
  }
}
```

### Reservations

- in app/reservations create page.tsx and loading.tsx

```tsx
'use client'

import LoadingTable from '@/components/booking/LoadingTable'

function loading() {
  return <LoadingTable />
}
export default loading
```

- add to links

utils/links.ts

```ts
export const links: NavLink[] = [
  { href: '/', label: 'home' },
  { href: '/favorites ', label: 'favorites' },
  { href: '/bookings ', label: 'bookings' },
  { href: '/reviews ', label: 'reviews' },
  { href: '/reservations ', label: 'reservations' },
  { href: '/rentals/create ', label: 'create rental' },
  { href: '/rentals', label: 'my rentals' },
  { href: '/profile ', label: 'profile' },
]
```

### Fetch Reservations

```ts
export const fetchReservations = async () => {
  const user = await getAuthUser()

  const reservations = await db.booking.findMany({
    where: {
      property: {
        profileId: user.id,
      },
    },

    orderBy: {
      createdAt: 'desc', // or 'asc' for ascending order
    },

    include: {
      property: {
        select: {
          id: true,
          name: true,
          price: true,
          country: true,
        },
      }, // include property details in the result
    },
  })
  return reservations
}
```

### Reservations Page

```tsx
import { fetchReservations } from '@/utils/actions'
import Link from 'next/link'
import EmptyList from '@/components/home/EmptyList'
import CountryFlagAndName from '@/components/card/CountryFlagAndName'

import { formatDate, formatCurrency } from '@/utils/format'
import {
  Table,
  TableBody,
  TableCaption,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'

async function ReservationsPage() {
  const reservations = await fetchReservations()

  if (reservations.length === 0) {
    return <EmptyList />
  }

  return (
    <div className='mt-16'>
      <h4 className='mb-4 capitalize'>
        total reservations : {reservations.length}
      </h4>
      <Table>
        <TableCaption>A list of your recent reservations.</TableCaption>
        <TableHeader>
          <TableRow>
            <TableHead>Property Name</TableHead>
            <TableHead>Country</TableHead>
            <TableHead>Nights</TableHead>
            <TableHead>Total</TableHead>
            <TableHead>Check In</TableHead>
            <TableHead>Check Out</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {reservations.map((item) => {
            const { id, orderTotal, totalNights, checkIn, checkOut } = item
            const { id: propertyId, name, country } = item.property
            const startDate = formatDate(checkIn)
            const endDate = formatDate(checkOut)
            return (
              <TableRow key={id}>
                <TableCell>
                  <Link
                    href={`/properties/${propertyId}`}
                    className='underline text-muted-foreground tracking-wide'
                  >
                    {name}
                  </Link>
                </TableCell>
                <TableCell>
                  <CountryFlagAndName countryCode={country} />
                </TableCell>
                <TableCell>{totalNights}</TableCell>
                <TableCell>{formatCurrency(orderTotal)}</TableCell>
                <TableCell>{startDate}</TableCell>
                <TableCell>{endDate}</TableCell>
              </TableRow>
            )
          })}
        </TableBody>
      </Table>
    </div>
  )
}
export default ReservationsPage
```
