#

## QnA

[1]

1. how is the object in the array interacted with

- set price, range --> booking

2. where is the defenition and the construction located

- utils/store, BookingWrapper
- dynamic lazy loading bc client side dependency batch

[2]

[3] derived -> confir, page

1. effect for related response from loading and refresh

- const unavailableDates = generateDisabledDates(blockedPeriods)

useEffect(() => {
const selectedRange = generateDateRange(range)

##

### fetch lazy loading

[id-into_useProperty]

- reference clerkId or id with rental and payment (wrapper of smaller units)
- vector checkin checkout (server types)
- state manage equation (client types)
- heavy lazy loaded (array batch w range type)("hydrate-ish" init w effect) + ssr

### blocked periods

[calc-portion]

- bounds and push to range (v type w Date str "manipulatable" data)
- split period unit v and calculate length
- type merged array with boolean for blocked
- relation of block
- domain

[calender]

- effect set interactive portion
- new derived total

### confirmBooking

[session-pushed]

- variable input (bind id + v units)
- variable output (find unique session for every single pushed/created obj)
- new disabled range today ->date
- (page with delete and clean up)

### fetch and delete

## fetch lazy loading

### Booking Model

- schema.prisma

```prisma
model Booking {
  id        String   @id @default(uuid())
  profile   Profile  @relation(fields: [profileId], references: [clerkId], onDelete: Cascade)
  profileId String
  property   Property  @relation(fields: [propertyId], references: [id], onDelete: Cascade)
  propertyId String
  orderTotal     Int
  totalNights    Int
  checkIn   DateTime
  checkOut  DateTime
  paymentStatus Boolean @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Profile {
  bookings Booking[]
}
model Property {
  bookings Booking[]
}

```

```bash
npx prisma db push
```

- restart server !!!

### Fetch Bookings

- actions.ts

```ts
export const fetchPropertyDetails = (id: string) => {
  return db.property.findUnique({
    where: {
      id,
    },
    include: {
      profile: true,
      bookings: {
        select: {
          checkIn: true,
          checkOut: true,
        },
      },
    },
  })
}
```

### Booking Types

- utils/types.ts

```ts
export type DateRangeSelect = {
  startDate: Date
  endDate: Date
  key: string
}

export type Booking = {
  checkIn: Date
  checkOut: Date
}
```

### Booking Components

- remove @/components/properties/BookingCalendar.tsx

- create @/components/booking
  - BookingCalendar.tsx
  - BookingContainer.tsx
  - BookingForm.tsx
  - BookingWrapper.tsx
  - ConfirmBooking.tsx

## lazy loading for code splitting

### Zustand

[Docs](https://docs.pmnd.rs/zustand/getting-started/introduction)

```sh
npm install zustand
```

### Setup Store

- utils/store.ts

```ts
import { create } from 'zustand'
import { Booking } from './types'
import { DateRange } from 'react-day-picker'
// Define the state's shape
type PropertyState = {
  propertyId: string
  price: number
  bookings: Booking[]
  range: DateRange | undefined
}

// Create the store
export const useProperty = create<PropertyState>(() => {
  return {
    propertyId: '',
    price: 0,
    bookings: [],
    range: undefined,
  }
})
```

### BookingWrapper

```tsx
'use client'

import { useProperty } from '@/utils/store'
import { Booking } from '@/utils/types'
import BookingCalendar from './BookingCalendar'
import BookingContainer from './BookingContainer'
import { useEffect } from 'react'

type BookingWrapperProps = {
  propertyId: string
  price: number
  bookings: Booking[]
}
export default function BookingWrapper({
  propertyId,
  price,
  bookings,
}: BookingWrapperProps) {
  useEffect(() => {
    useProperty.setState({
      propertyId,
      price,
      bookings,
    })
  }, [])
  return (
    <>
      <BookingCalendar />
      <BookingContainer />
    </>
  )
}
```

- properties/[id]/page.tsx

```tsx
const DynamicBookingWrapper = dynamic(
  () => import('@/components/booking/BookingWrapper'),
  {
    ssr: false,
    loading: () => <Skeleton className='h-[200px] w-full' />,
  }
)

return (
  <div className='lg:col-span-4 flex flex-col items-center'>
    {/* calendar */}
    <DynamicBookingWrapper
      propertyId={property.id}
      price={property.price}
      bookings={property.bookings}
    />
  </div>
)
```

---

## blocked periods

### Helper Functions

- utils/calendar.ts

```ts
import { DateRange } from 'react-day-picker'
import { Booking } from '@/utils/types'

export const defaultSelected: DateRange = {
  from: undefined,
  to: undefined,
}

export const generateBlockedPeriods = ({
  bookings,
  today,
}: {
  bookings: Booking[]
  today: Date
}) => {
  today.setHours(0, 0, 0, 0) // Set the time to 00:00:00.000

  const disabledDays: DateRange[] = [
    ...bookings.map((booking) => ({
      from: booking.checkIn,
      to: booking.checkOut,
    })),
    {
      from: new Date(0), // This is 01 January 1970 00:00:00 UTC.
      to: new Date(today.getTime() - 24 * 60 * 60 * 1000), // This is yesterday.
    },
  ]
  return disabledDays
}

export const generateDateRange = (range: DateRange | undefined): string[] => {
  if (!range || !range.from || !range.to) return []

  let currentDate = new Date(range.from)
  const endDate = new Date(range.to)
  const dateRange: string[] = []

  while (currentDate <= endDate) {
    const dateString = currentDate.toISOString().split('T')[0]
    dateRange.push(dateString)
    currentDate.setDate(currentDate.getDate() + 1)
  }

  return dateRange
}

export const generateDisabledDates = (
  disabledDays: DateRange[]
): { [key: string]: boolean } => {
  if (disabledDays.length === 0) return {}

  const disabledDates: { [key: string]: boolean } = {}
  const today = new Date()
  today.setHours(0, 0, 0, 0) // set time to 00:00:00 to compare only the date part

  disabledDays.forEach((range) => {
    if (!range.from || !range.to) return

    let currentDate = new Date(range.from)
    const endDate = new Date(range.to)

    while (currentDate <= endDate) {
      if (currentDate < today) {
        currentDate.setDate(currentDate.getDate() + 1)
        continue
      }
      const dateString = currentDate.toISOString().split('T')[0]
      disabledDates[dateString] = true
      currentDate.setDate(currentDate.getDate() + 1)
    }
  })

  return disabledDates
}

export function calculateDaysBetween({
  checkIn,
  checkOut,
}: {
  checkIn: Date
  checkOut: Date
}) {
  // Calculate the difference in milliseconds
  const diffInMs = Math.abs(checkOut.getTime() - checkIn.getTime())

  // Convert the difference in milliseconds to days
  const diffInDays = diffInMs / (1000 * 60 * 60 * 24)

  return diffInDays
}
```

### BoookingCalendar

```tsx
'use client'
import { Calendar } from '@/components/ui/calendar'
import { useEffect, useState } from 'react'
import { useToast } from '@/components/ui/use-toast'
import { DateRange } from 'react-day-picker'
import { useProperty } from '@/utils/store'

import {
  generateDisabledDates,
  generateDateRange,
  defaultSelected,
  generateBlockedPeriods,
} from '@/utils/calendar'

function BookingCalendar() {
  const currentDate = new Date()

  const [range, setRange] = useState<DateRange | undefined>(defaultSelected)

  useEffect(() => {
    useProperty.setState({ range })
  }, [range])

  return (
    <Calendar
      mode='range'
      defaultMonth={currentDate}
      selected={range}
      onSelect={setRange}
      className='mb-4'
    />
  )
}
export default BookingCalendar
```

### BookingContainer

```tsx
'use client'

import { useProperty } from '@/utils/store'
import ConfirmBooking from './ConfirmBooking'
import BookingForm from './BookingForm'
function BookingContainer() {
  const { range } = useProperty((state) => state)

  if (!range || !range.from || !range.to) return null
  if (range.to.getTime() === range.from.getTime()) return null
  return (
    <div className='w-full'>
      <BookingForm />
      <ConfirmBooking />
    </div>
  )
}

export default BookingContainer
```

### CalculateTotals

- utils/calculateTotals.ts

```ts
import { calculateDaysBetween } from '@/utils/calendar'

type BookingDetails = {
  checkIn: Date
  checkOut: Date
  price: number
}

export const calculateTotals = ({
  checkIn,
  checkOut,
  price,
}: BookingDetails) => {
  const totalNights = calculateDaysBetween({ checkIn, checkOut })
  const subTotal = totalNights * price
  const cleaning = 21
  const service = 40
  const tax = subTotal * 0.1
  const orderTotal = subTotal + cleaning + service + tax
  return { totalNights, subTotal, cleaning, service, tax, orderTotal }
}
```

## confirmbooking

### BookingForm

```tsx
import { calculateTotals } from '@/utils/calculateTotals'
import { Card, CardTitle } from '@/components/ui/card'
import { Separator } from '@/components/ui/separator'
import { useProperty } from '@/utils/store'
import { formatCurrency } from '@/utils/format'
function BookingForm() {
  const { range, price } = useProperty((state) => state)
  const checkIn = range?.from as Date
  const checkOut = range?.to as Date

  const { totalNights, subTotal, cleaning, service, tax, orderTotal } =
    calculateTotals({
      checkIn,
      checkOut,
      price,
    })
  return (
    <Card className='p-8 mb-4'>
      <CardTitle className='mb-8'>Summary </CardTitle>
      <FormRow label={`$${price} x ${totalNights} nights`} amount={subTotal} />
      <FormRow label='Cleaning Fee' amount={cleaning} />
      <FormRow label='Service Fee' amount={service} />
      <FormRow label='Tax' amount={tax} />
      <Separator className='mt-4' />
      <CardTitle className='mt-8'>
        <FormRow label='Booking Total' amount={orderTotal} />
      </CardTitle>
    </Card>
  )
}

function FormRow({ label, amount }: { label: string; amount: number }) {
  return (
    <p className='flex justify-between text-sm mb-2'>
      <span>{label}</span>
      <span>{formatCurrency(amount)}</span>
    </p>
  )
}

export default BookingForm
```

### ConfirmBooking

- action.ts

```ts
export const createBookingAction = async () => {
  return { message: 'create booking' }
}
```

```tsx
'use client'
import { SignInButton, useAuth } from '@clerk/nextjs'
import { Button } from '@/components/ui/button'
import { useProperty } from '@/utils/store'
import FormContainer from '@/components/form/FormContainer'
import { SubmitButton } from '@/components/form/Buttons'
import { createBookingAction } from '@/utils/actions'

function ConfirmBooking() {
  const { userId } = useAuth()
  const { propertyId, range } = useProperty((state) => state)
  const checkIn = range?.from as Date
  const checkOut = range?.to as Date
  if (!userId)
    return (
      <SignInButton mode='modal'>
        <Button type='button' className='w-full'>
          Sign In to Complete Booking
        </Button>
      </SignInButton>
    )

  const createBooking = createBookingAction.bind(null, {
    propertyId,
    checkIn,
    checkOut,
  })
  return (
    <section>
      <FormContainer action={createBooking}>
        <SubmitButton text='Reserve' className='w-full' />
      </FormContainer>
    </section>
  )
}
export default ConfirmBooking
```

### CreateBookingAction

```tsx
export const createBookingAction = async (prevState: {
  propertyId: string
  checkIn: Date
  checkOut: Date
}) => {
  const user = await getAuthUser()

  const { propertyId, checkIn, checkOut } = prevState
  const property = await db.property.findUnique({
    where: { id: propertyId },
    select: { price: true },
  })
  if (!property) {
    return { message: 'Property not found' }
  }
  const { orderTotal, totalNights } = calculateTotals({
    checkIn,
    checkOut,
    price: property.price,
  })

  try {
    const booking = await db.booking.create({
      data: {
        checkIn,
        checkOut,
        orderTotal,
        totalNights,
        profileId: user.id,
        propertyId,
      },
    })
  } catch (error) {
    return renderError(error)
  }
  redirect('/bookings')
}
```

### Blocked Periods/Dates

BookingCalendar.tsx

```tsx
function BookingCalendar() {
  const bookings = useProperty((state) => state.bookings)
  const blockedPeriods = generateBlockedPeriods({
    bookings,
    today: currentDate,
  })

  return (
    <Calendar
      mode='range'
      defaultMonth={currentDate}
      selected={range}
      onSelect={setRange}
      className='mb-4'
      // add disabled
      disabled={blockedPeriods}
    />
  )
}
export default BookingCalendar
```

### Unavailable Dates

BookingCalendar.tsx

```tsx
function BookingCalendar() {
  const { toast } = useToast()
  const unavailableDates = generateDisabledDates(blockedPeriods)

  useEffect(() => {
    const selectedRange = generateDateRange(range)
    const isDisabledDateIncluded = selectedRange.some((date) => {
      if (unavailableDates[date]) {
        setRange(defaultSelected)
        toast({
          description: 'Some dates are booked. Please select again.',
        })
        return true
      }
      return false
    })
    useProperty.setState({ range })
  }, [range])

  return (
    <Calendar
      mode='range'
      defaultMonth={currentDate}
      selected={range}
      onSelect={setRange}
      className='mb-4'
      // add disabled
      disabled={blockedPeriods}
    />
  )
}
export default BookingCalendar
```

### Fetch Bookings and Delete Booking

- actions.ts

```ts
export const fetchBookings = async () => {
  const user = await getAuthUser()
  const bookings = await db.booking.findMany({
    where: {
      profileId: user.id,
    },
    include: {
      property: {
        select: {
          id: true,
          name: true,
          country: true,
        },
      },
    },
    orderBy: {
      checkIn: 'desc',
    },
  })
  return bookings
}

export async function deleteBookingAction(prevState: { bookingId: string }) {
  const { bookingId } = prevState
  const user = await getAuthUser()

  try {
    const result = await db.booking.delete({
      where: {
        id: bookingId,
        profileId: user.id,
      },
    })

    revalidatePath('/bookings')
    return { message: 'Booking deleted successfully' }
  } catch (error) {
    return renderError(error)
  }
}
```

### Bookings Page

- utils/format.ts

```ts
export const formatDate = (date: Date) => {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date)
}
```

Bookings.tsx

```tsx
import EmptyList from '@/components/home/EmptyList'
import CountryFlagAndName from '@/components/card/CountryFlagAndName'
import Link from 'next/link'

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

import FormContainer from '@/components/form/FormContainer'
import { IconButton } from '@/components/form/Buttons'
import { fetchBookings, deleteBookingAction } from '@/utils/actions'

async function BookingsPage() {
  const bookings = await fetchBookings()
  if (bookings.length === 0) {
    return <EmptyList />
  }
  return (
    <div className='mt-16'>
      <h4 className='mb-4 capitalize'>total bookings : {bookings.length}</h4>
      <Table>
        <TableCaption>A list of your recent bookings.</TableCaption>
        <TableHeader>
          <TableRow>
            <TableHead>Property Name</TableHead>
            <TableHead>Country</TableHead>
            <TableHead>Nights</TableHead>
            <TableHead>Total</TableHead>
            <TableHead>Check In</TableHead>
            <TableHead>Check Out</TableHead>
            <TableHead>Actions</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {bookings.map((booking) => {
            const { id, orderTotal, totalNights, checkIn, checkOut } = booking
            const { id: propertyId, name, country } = booking.property
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
                <TableCell>
                  <DeleteBooking bookingId={id} />
                </TableCell>
              </TableRow>
            )
          })}
        </TableBody>
      </Table>
    </div>
  )
}

function DeleteBooking({ bookingId }: { bookingId: string }) {
  const deleteBooking = deleteBookingAction.bind(null, { bookingId })
  return (
    <FormContainer action={deleteBooking}>
      <IconButton actionType='delete' />
    </FormContainer>
  )
}

export default BookingsPage
```
