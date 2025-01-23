#

## QnA

[1]

1. how to setup array for relationship

- reference to the array in object that user and id owns

2. how and why rly is the review obj created like this?

- const numbers = Array.from({ length: 5 }, (\_, i) => {
  const value = i + 1
  return value.toString()
  }).reverse()

  3. how is incremental updates implementede

  - use client component with id and formAction

  4. how deep is the review obj

  - 1st layer comment, 2nd is user
    [2]

1. map the rad operation

1. comment incremental update as find ids from server action input

[3]

1. review wrapper can perform dynamics on two types of obj, which?

- user and product

[4]

- wrapper check user (buissiness rules)

## sum: mix bindings between [{prop, prop}] x [{prop, prop}]

### summary: complex combind, bind and aggregate

#### properties id and combined info behavior

[fill-up_card] (property+profile)(left-join --> populate --> select)

- select --> submit review (Array.from str)
- user create, findMany properties (redirect to id, + user info)
- review card compose obj --> rating, comment (map compine info)
- user fetch delete

[the-card]

- rating (array.from)
- comment (slice x toggle)
- fetch by relation (findMany, prevState delete review)

#### dynamics diff type of requests

[complex-bind]

- iconButton id enough to delete (network-aware + switch case post/get action) type
- bind pre state
- skeleton

#### property check owner before avg

- review (property extend query existance of, FindFirst <-- FindUnique)
- group by avg ()
- find first / find unique review

## fill up card

### Review Model

```prisma

model Review {
  id        String   @id @default(uuid())
  profile   Profile  @relation(fields: [profileId], references: [clerkId], onDelete: Cascade)
  profileId String
  property   Property  @relation(fields: [propertyId], references: [id], onDelete: Cascade)
  propertyId String
  rating    Int
  comment   String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
model Property {
 reviews Review[]
}
model Profile {
 reviews Review[]
}
```

DON'T FORGET !!!!

```sh
npx prisma db push
```

- restart server

### Reviews Setup

- create components/reviews

  - Comment.tsx
  - PropertyReviews.tsx
  - Rating.tsx
  - SubmitReview.tsx
  - ReviewCard.tsx

- create placeholder functions in actions.ts

```ts
export const createReviewAction = async () => {
  return { message: 'create review' }
}

export const fetchPropertyReviews = async () => {
  return { message: 'fetch reviews' }
}

export const fetchPropertyReviewsByUser = async () => {
  return { message: 'fetch user reviews' }
}

export const deleteReviewAction = async () => {
  return { message: 'delete  reviews' }
}
```

### RatingInput

- components/form/RatingInput.tsx

```tsx
import { Label } from '@/components/ui/label'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'

const RatingInput = ({
  name,
  labelText,
}: {
  name: string
  labelText?: string
}) => {
  const numbers = Array.from({ length: 5 }, (_, i) => {
    const value = i + 1
    return value.toString()
  }).reverse()

  return (
    <div className='mb-2 max-w-xs'>
      <Label htmlFor={name} className='capitalize'>
        {labelText || name}
      </Label>
      <Select defaultValue={numbers[0]} name={name} required>
        <SelectTrigger>
          <SelectValue />
        </SelectTrigger>
        <SelectContent>
          {numbers.map((number) => {
            return (
              <SelectItem key={number} value={number}>
                {number}
              </SelectItem>
            )
          })}
        </SelectContent>
      </Select>
    </div>
  )
}

export default RatingInput
```

---

### SubmitReview Component

- app/properties/[id]

```tsx
return (
  <section>
    <section></section>
    {/* after two column section */}
    <SubmitReview propertyId={property.id} />;
  </section>
)
```

- components/reviews/SubmitReview.tsx

```tsx
'use client'
import { useState } from 'react'
import { SubmitButton } from '@/components/form/Buttons'
import FormContainer from '@/components/form/FormContainer'
import { Card } from '@/components/ui/card'
import RatingInput from '@/components/form/RatingInput'
import TextAreaInput from '@/components/form/TextAreaInput'
import { Button } from '@/components/ui/button'
import { createReviewAction } from '@/utils/actions'
function SubmitReview({ propertyId }: { propertyId: string }) {
  const [isReviewFormVisible, setIsReviewFormVisible] = useState(false)
  return (
    <div className='mt-8'>
      <Button onClick={() => setIsReviewFormVisible((prev) => !prev)}>
        Leave a Review
      </Button>
      {isReviewFormVisible && (
        <Card className='p-8 mt-8'>
          <FormContainer action={createReviewAction}>
            <input type='hidden' name='propertyId' value={propertyId} />
            <RatingInput name='rating' />
            <TextAreaInput
              name='comment'
              labelText='your thoughts on this property'
              defaultValue='Amazing place !!!'
            />
            <SubmitButton text='Submit' className='mt-4' />
          </FormContainer>
        </Card>
      )}
    </div>
  )
}

export default SubmitReview
```

- optional : set rows prop in TextArea.tsx

### Submit Review

- utils/schemas.ts

```ts
export const createReviewSchema = z.object({
  propertyId: z.string(),
  rating: z.coerce.number().int().min(1).max(5),
  comment: z.string().min(10).max(1000),
})
```

- action.ts

```ts
export async function createReviewAction(prevState: any, formData: FormData) {
  const user = await getAuthUser()
  try {
    const rawData = Object.fromEntries(formData)

    const validatedFields = validateWithZodSchema(createReviewSchema, rawData)
    await db.review.create({
      data: {
        ...validatedFields,
        profileId: user.id,
      },
    })
    revalidatePath(`/properties/${validatedFields.propertyId}`)
    return { message: 'Review submitted successfully' }
  } catch (error) {
    return renderError(error)
  }
}
```

### Fetch Property Reviews

- actions.ts

```ts
export async function fetchPropertyReviews(propertyId: string) {
  const reviews = await db.review.findMany({
    where: {
      propertyId,
    },
    select: {
      id: true,
      rating: true,
      comment: true,
      profile: {
        select: {
          firstName: true,
          profileImage: true,
        },
      },
    },
    orderBy: {
      createdAt: 'desc',
    },
  })
  return reviews
}
```

### Render Reviews

- app/properties/[id]

```tsx
return (
  <>
    {/* after two column section */}
    <SubmitReview propertyId={property.id} />
    <PropertyReviews propertyId={property.id} />
  </>
)
```

- components/reviews/PropertyReviews.tsx

```tsx
import { fetchPropertyReviews } from '@/utils/actions'
import Title from '@/components/properties/Title'

import ReviewCard from './ReviewCard'
async function PropertyReviews({ propertyId }: { propertyId: string }) {
  const reviews = await fetchPropertyReviews(propertyId)
  if (reviews.length < 1) return null
  return (
    <div className='mt-8'>
      <Title text='Reviews' />
      <div className='grid md:grid-cols-2 gap-8 mt-4 '>
        {reviews.map((review) => {
          const { comment, rating } = review
          const { firstName, profileImage } = review.profile
          const reviewInfo = {
            comment,
            rating,
            name: firstName,
            image: profileImage,
          }
          return <ReviewCard key={review.id} reviewInfo={reviewInfo} />
        })}
      </div>
    </div>
  )
}
export default PropertyReviews
```

### ReviewCard Component

```tsx
import { Card, CardContent, CardHeader } from '@/components/ui/card'
import Rating from './Rating'
import Comment from './Comment'
type ReviewCardProps = {
  reviewInfo: {
    comment: string
    rating: number
    name: string
    image: string
  }
  children?: React.ReactNode
}

function ReviewCard({ reviewInfo, children }: ReviewCardProps) {
  return (
    <Card className='relative'>
      <CardHeader>
        <div className='flex items-center'>
          <img
            src={reviewInfo.image}
            alt='profile'
            className='w-12 h-12 rounded-full object-cover'
          />
          <div className='ml-4'>
            <h3 className='text-sm font-bold capitalize mb-1'>
              {reviewInfo.name}
            </h3>
            <Rating rating={reviewInfo.rating} />
          </div>
        </div>
      </CardHeader>
      <CardContent>
        <Comment comment={reviewInfo.comment} />
      </CardContent>
      {/* delete button later */}
      <div className='absolute top-3 right-3'>{children}</div>
    </Card>
  )
}
export default ReviewCard
```

## the card

### Rating

```tsx
import { FaStar, FaRegStar } from 'react-icons/fa'

function Rating({ rating }: { rating: number }) {
  // rating = 2
  // 1 <= 2 true
  // 2 <= 2 true
  // 3 <= 2 false
  // ....
  const stars = Array.from({ length: 5 }, (_, i) => i + 1 <= rating)

  return (
    <div className='flex items-center gap-x-1'>
      {stars.map((isFilled, i) => {
        const className = `w-3 h-3 ${
          isFilled ? 'text-primary' : 'text-gray-400'
        }`
        return isFilled ? (
          <FaStar className={className} key={i} />
        ) : (
          <FaRegStar className={className} key={i} />
        )
      })}
    </div>
  )
}

export default Rating
```

### Comment

```tsx
'use client'
import { useState } from 'react'
import { Button } from '@/components/ui/button'
function Comment({ comment }: { comment: string }) {
  const [isExpanded, setIsExpanded] = useState(false)

  const toggleExpanded = () => {
    setIsExpanded(!isExpanded)
  }
  const longComment = comment.length > 130
  const displayComment =
    longComment && !isExpanded ? `${comment.slice(0, 130)}...` : comment

  return (
    <div>
      <p className='text-sm'>{displayComment}</p>
      {longComment && (
        <Button
          variant='link'
          className='pl-0 text-muted-foreground'
          onClick={toggleExpanded}
        >
          {isExpanded ? 'Show Less' : 'Show More'}
        </Button>
      )}
    </div>
  )
}

export default Comment
```

### Fetch User's Reviews and Delete Review

```ts
export const fetchPropertyReviewsByUser = async () => {
  const user = await getAuthUser()
  const reviews = await db.review.findMany({
    where: {
      profileId: user.id,
    },
    select: {
      id: true,
      rating: true,
      comment: true,
      property: {
        select: {
          name: true,
          image: true,
        },
      },
    },
  })
  return reviews
}

export const deleteReviewAction = async (prevState: { reviewId: string }) => {
  const { reviewId } = prevState
  const user = await getAuthUser()

  try {
    await db.review.delete({
      where: {
        id: reviewId,
        profileId: user.id,
      },
    })

    revalidatePath('/reviews')
    return { message: 'Review deleted successfully' }
  } catch (error) {
    return renderError(error)
  }
}
```

## the dynamic action

### Icon Button

- components/form/Buttons.tsx

```tsx
import { LuTrash2, LuPenSquare } from 'react-icons/lu'

type actionType = 'edit' | 'delete'
export const IconButton = ({ actionType }: { actionType: actionType }) => {
  const { pending } = useFormStatus()

  const renderIcon = () => {
    switch (actionType) {
      case 'edit':
        return <LuPenSquare />
      case 'delete':
        return <LuTrash2 />
      default:
        const never: never = actionType
        throw new Error(`Invalid action type: ${never}`)
    }
  }

  return (
    <Button
      type='submit'
      size='icon'
      variant='link'
      className='p-2 cursor-pointer'
    >
      {pending ? <ReloadIcon className=' animate-spin' /> : renderIcon()}
    </Button>
  )
}
```

### Reviews Page

- app/reviews/page.tsx

```tsx
import EmptyList from '@/components/home/EmptyList'
import { deleteReviewAction, fetchPropertyReviewsByUser } from '@/utils/actions'
import ReviewCard from '@/components/reviews/ReviewCard'
import Title from '@/components/properties/Title'
import FormContainer from '@/components/form/FormContainer'
import { IconButton } from '@/components/form/Buttons'
async function ReviewsPage() {
  const reviews = await fetchPropertyReviewsByUser()
  if (reviews.length === 0) return <EmptyList />

  return (
    <>
      <Title text='Your Reviews' />
      <section className='grid md:grid-cols-2 gap-8 mt-4 '>
        {reviews.map((review) => {
          const { comment, rating } = review
          const { name, image } = review.property
          const reviewInfo = {
            comment,
            rating,
            name,
            image,
          }
          return (
            <ReviewCard key={review.id} reviewInfo={reviewInfo}>
              <DeleteReview reviewId={review.id} />
            </ReviewCard>
          )
        })}
      </section>
    </>
  )
}

const DeleteReview = ({ reviewId }: { reviewId: string }) => {
  const deleteReview = deleteReviewAction.bind(null, { reviewId })
  return (
    <FormContainer action={deleteReview}>
      <IconButton actionType='delete' />
    </FormContainer>
  )
}

export default ReviewsPage
```

- loading.tsx

```tsx
'use client'

import { Card, CardContent, CardHeader } from '@/components/ui/card'
import { Skeleton } from '@/components/ui/skeleton'
function loading() {
  return (
    <section className='grid md:grid-cols-2 gap-8 mt-4 '>
      <ReviewLoadingCard />
      <ReviewLoadingCard />
    </section>
  )
}

const ReviewLoadingCard = () => {
  return (
    <Card>
      <CardHeader>
        <div className='flex items-center'>
          <Skeleton className='w-12 h-12 rounded-full' />
          <div className='ml-4'>
            <Skeleton className='w-[150px] h-4 mb-2' />
            <Skeleton className='w-[100px] h-4' />
          </div>
        </div>
      </CardHeader>
    </Card>
  )
}

export default loading
```

## find and group logic a/b

### Allow Review

- actions.ts

```ts
export const findExistingReview = async (
  userId: string,
  propertyId: string
) => {
  return db.review.findFirst({
    where: {
      profileId: userId,
      propertyId: propertyId,
    },
  })
}
```

- app/properties/[id]

```tsx
import { findExistingReview } from '@/utils/actions'
import { auth } from '@clerk/nextjs/server'

async function PropertyDetailsPage({ params }: { params: { id: string } }) {
  const { userId } = auth()
  const isNotOwner = property.profile.clerkId !== userId
  const reviewDoesNotExist =
    userId && isNotOwner && !(await findExistingReview(userId, property.id))

  return <>{reviewDoesNotExist && <SubmitReview propertyId={property.id} />}</>
}
```

Prisma's findUnique and findFirst methods are used to retrieve a single record from the database, but they have some differences in their behavior:

- findUnique: This method is used when you want to retrieve a single record that matches a unique constraint or a primary key. If no record is found, it returns null.

- findFirst: This method is used when you want to retrieve a single record that matches a non-unique constraint. It can also be used with ordering and filtering. If no record is found, it returns null.

In summary, use findUnique when you're sure the field you're querying by is unique, and use findFirst when you're querying by a non-unique field or need more complex queries with ordering and filtering.

```ts
const user = await prisma.user.findUnique({
  where: {
    email: 'alice@prisma.io',
  },
})

const user = await prisma.user.findFirst({
  where: {
    email: {
      contains: 'prisma.io',
    },
  },
  orderBy: {
    name: 'asc',
  },
})
```

### PropertyRating - Complete

- actions

```ts
export async function fetchPropertyRating(propertyId: string) {
  const result = await db.review.groupBy({
    by: ['propertyId'],
    _avg: {
      rating: true,
    },
    _count: {
      rating: true,
    },
    where: {
      propertyId,
    },
  })

  // empty array if no reviews
  return {
    rating: result[0]?._avg.rating?.toFixed(1) ?? 0,
    count: result[0]?._count.rating ?? 0,
  }
}
```

- components/card/PropertyRating.tsx

```tsx
import { fetchPropertyRating } from '@/utils/actions'
import { FaStar } from 'react-icons/fa'

async function PropertyRating({
  propertyId,
  inPage,
}: {
  propertyId: string
  inPage: boolean
}) {
  const { rating, count } = await fetchPropertyRating(propertyId)
  if (count === 0) return null
  const className = `flex gap-1 items-center ${inPage ? 'text-md' : 'text-xs'}`
  const countText = count === 1 ? 'review' : 'reviews'
  const countValue = `(${count}) ${inPage ? countText : ''}`
  return (
    <span className={className}>
      <FaStar className='w-3 h-3' />
      {rating} {countValue}
    </span>
  )
}

export default PropertyRating
```
