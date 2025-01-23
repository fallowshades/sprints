
# take home away
##
### f()'al identities
## alternative tool to set params
### Favorites Model
```prisma
model Profile {
favorites    Favorite[]
}
model Property {
favorites    Favorite[]
}
model Favorite {
  id        String   @id @default(uuid())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  profile   Profile  @relation(fields: [profileId], references: [clerkId], onDelete: Cascade)
  profileId String
  property   Property  @relation(fields: [propertyId], references: [id], onDelete: Cascade)
  propertyId String
}
```
```sh
npx prisma db push
```
### CardSignInButton
components/form/Buttons.tsx
```tsx
import { SignInButton } from '@clerk/nextjs';
import { FaRegHeart, FaHeart } from 'react-icons/fa';
export const CardSignInButton = () => {
  return (
    <SignInButton mode='modal'>
      <Button
        type='button'
        size='icon'
        variant='outline'
        className='p-2 cursor-pointer'
        asChild
      >
        <FaRegHeart />
      </Button>
    </SignInButton>
  );
};
```
components/card/FavoriteToggleButton.tsx
```tsx
import { FaHeart } from 'react-icons/fa';
import { Button } from '@/components/ui/button';
import { auth } from '@clerk/nextjs/server';
import { CardSignInButton } from '../form/Buttons';
function FavoriteToggleButton({ propertyId }: { propertyId: string }) {
  const { userId } = auth();
  if (!userId) return <CardSignInButton />;
  return (
    <Button size='icon' variant='outline' className='p-2 cursor-pointer'>
      <FaHeart />
    </Button>
  );
}
export default FavoriteToggleButton;
```
### fetchFavorite
actions.ts
```ts
export const fetchFavoriteId = async ({
  propertyId,
}: {
  propertyId: string;
}) => {
  const user = await getAuthUser();
  const favorite = await db.favorite.findFirst({
    where: {
      propertyId,
      profileId: user.id,
    },
    select: {
      id: true,
    },
  });
  return favorite?.id || null;
};
export const toggleFavoriteAction = async () => {
  return { message: 'toggle favorite' };
};
```
### FavoriteToggleButton - Complete
```tsx
import { auth } from '@clerk/nextjs/server';
import { CardSignInButton } from '../form/Buttons';
import { fetchFavoriteId } from '@/utils/actions';
import FavoriteToggleForm from './FavoriteToggleForm';
async function FavoriteToggleButton({ propertyId }: { propertyId: string }) {
  const { userId } = auth();
  if (!userId) return <CardSignInButton />;
  const favoriteId = await fetchFavoriteId({ propertyId });
  return <FavoriteToggleForm favoriteId={favoriteId} propertyId={propertyId} />;
}
export default FavoriteToggleButton;
```
### CardSubmitButton
components/form/Buttons.tsx
```tsx
export const CardSubmitButton = ({ isFavorite }: { isFavorite: boolean }) => {
  const { pending } = useFormStatus();
  return (
    <Button
      type='submit'
      size='icon'
      variant='outline'
      className=' p-2 cursor-pointer'
    >
      {pending ? (
        <ReloadIcon className=' animate-spin' />
      ) : isFavorite ? (
        <FaHeart />
      ) : (
        <FaRegHeart />
      )}
    </Button>
  );
};
```
### FavoriteToggleForm
```tsx
'use client';
import { usePathname } from 'next/navigation';
import FormContainer from '../form/FormContainer';
import { toggleFavoriteAction } from '@/utils/actions';
import { CardSubmitButton } from '../form/Buttons';
type FavoriteToggleFormProps = {
  propertyId: string;
  favoriteId: string | null;
};
function FavoriteToggleForm({
  propertyId,
  favoriteId,
}: FavoriteToggleFormProps) {
  const pathname = usePathname();
  const toggleAction = toggleFavoriteAction.bind(null, {
    propertyId,
    favoriteId,
    pathname,
  });
  return (
    <FormContainer action={toggleAction}>
      <CardSubmitButton isFavorite={favoriteId ? true : false} />
    </FormContainer>
  );
}
export default FavoriteToggleForm;
```
### toggleFavoriteAction
actions.ts
```ts
export const toggleFavoriteAction = async (prevState: {
  propertyId: string;
  favoriteId: string | null;
  pathname: string;
}) => {
  const user = await getAuthUser();
  const { propertyId, favoriteId, pathname } = prevState;
  try {
    if (favoriteId) {
      await db.favorite.delete({
        where: {
          id: favoriteId,
        },
      });
    } else {
      await db.favorite.create({
        data: {
          propertyId,
          profileId: user.id,
        },
      });
    }
    revalidatePath(pathname);
    return { message: favoriteId ? 'Removed from Faves' : 'Added to Faves' };
  } catch (error) {
    return renderError(error);
  }
};
```
### fetchFavorites
actions.ts
```ts
export const fetchFavorites = async () => {
  const user = await getAuthUser();
  const favorites = await db.favorite.findMany({
    where: {
      profileId: user.id,
    },
    select: {
      property: {
        select: {
          id: true,
          name: true,
          tagline: true,
          price: true,
          country: true,
          image: true,
        },
      },
    },
  });
  return favorites.map((favorite) => favorite.property);
};
```
### Favorites Page
- favorites/loading.tsx
```tsx
'use client';
import LoadingCards from '@/components/card/LoadingCards';
function loading() {
  return <LoadingCards />;
}
export default loading;
```
- favorites/page.tsx
```tsx
import EmptyList from '@/components/home/EmptyList';
import PropertiesList from '@/components/home/PropertiesList';
import { fetchFavorites } from '@/utils/actions';
async function FavoritesPage() {
  const favorites = await fetchFavorites();
  if (favorites.length === 0) {
    return <EmptyList />;
  }
  return <PropertiesList properties={favorites} />;
}
export default FavoritesPage;
```
## when id page
### fetchPropertyDetails
```ts
export const fetchPropertyDetails = (id: string) => {
  return db.property.findUnique({
    where: {
      id,
    },
    include: {
      profile: true,
    },
  });
};
```
- properties/[id]/loading.tsx
```ts
'use client';
import { Skeleton } from '@/components/ui/skeleton';
function loading() {
  return <Skeleton className='h-[300px] md:h-[500px] w-full rounded' />;
}
export default loading;
```
- properties/[id]/page.tsx
```tsx
import { fetchPropertyDetails } from '@/utils/actions';
import { redirect } from 'next/navigation';
async function PropertyDetailsPage({ params }: { params: { id: string } }) {
  const property = await fetchPropertyDetails(params.id);
  if (!property) redirect('/');
  const { baths, bedrooms, beds, guests } = property;
  const details = { baths, bedrooms, beds, guests };
  return <div>PropertyDetailsPage</div>;
}
export default PropertyDetailsPage;
```
### BreadCrumbs
- components/properties/BreadCrumbs.tsx
```tsx
import {
  Breadcrumb,
  BreadcrumbItem,
  BreadcrumbLink,
  BreadcrumbList,
  BreadcrumbPage,
  BreadcrumbSeparator,
} from '@/components/ui/breadcrumb';
function BreadCrumbs({ name }: { name: string }) {
  return (
    <Breadcrumb>
      <BreadcrumbList>
        <BreadcrumbItem>
          <BreadcrumbLink href='/'>Home</BreadcrumbLink>
        </BreadcrumbItem>
        <BreadcrumbSeparator />
        <BreadcrumbItem>
          <BreadcrumbPage>{name}</BreadcrumbPage>
        </BreadcrumbItem>
      </BreadcrumbList>
    </Breadcrumb>
  );
}
export default BreadCrumbs;
```
- properties/[id]/page.tsx
```tsx
return (
  <section>
    <BreadCrumbs name={property.name} />
    <header className='flex justify-between items-center mt-4'>
      <h1 className='text-4xl font-bold '>{property.tagline}</h1>
      <div className='flex items-center gap-x-4'>
        {/* share button */}
        <FavoriteToggleButton propertyId={property.id} />
      </div>
    </header>
  </section>
);
```
## f()'s via dependencies
### ShareButton
[ React Share](https://www.npmjs.com/package/react-share) 
```sh
npm i react-share
```
- components/properties/ShareButton.tsx
```tsx
'use client';
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from '@/components/ui/popover';
import { Button } from '../ui/button';
import { LuShare2 } from 'react-icons/lu';
import {
  TwitterShareButton,
  EmailShareButton,
  LinkedinShareButton,
  TwitterIcon,
  EmailIcon,
  LinkedinIcon,
} from 'react-share';
function ShareButton({
  propertyId,
  name,
}: {
  propertyId: string;
  name: string;
}) {
  const url = process.env.NEXT_PUBLIC_WEBSITE_URL;
  const shareLink = `${url}/properties/${propertyId}`;
  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button variant='outline' size='icon' className='p-2'>
          <LuShare2 />
        </Button>
      </PopoverTrigger>
      <PopoverContent
        side='top'
        align='end'
        sideOffset={10}
        className='flex items-center gap-x-2 justify-center w-full'
      >
        <TwitterShareButton url={shareLink} title={name}>
          <TwitterIcon size={32} round />
        </TwitterShareButton>
        <LinkedinShareButton url={shareLink} title={name}>
          <LinkedinIcon size={32} round />
        </LinkedinShareButton>
        <EmailShareButton url={shareLink} subject={name}>
          <EmailIcon size={32} round />
        </EmailShareButton>
      </PopoverContent>
    </Popover>
  );
}
export default ShareButton;
```
- properties/[id]/page.tsx
```tsx
return (
  <div className='flex items-center gap-x-4'>
    <ShareButton name={property.name} propertyId={property.id} />
    <FavoriteToggleButton propertyId={property.id} />
  </div>
);
```
### ImageContainer
- components/properties/ImageContainer.tsx
```tsx
import Image from 'next/image';
function ImageContainer({
  mainImage,
  name,
}: {
  mainImage: string;
  name: string;
}) {
  return (
    <section className='h-[300px] md:h-[500px] relative mt-8'>
      <Image
        src={mainImage}
        fill
        sizes='100vw'
        alt={name}
        className='object-cover  rounded-md'
        priority
      />
    </section>
  );
}
export default ImageContainer;
```
- properties/[id]/page.tsx
```tsx
<ImageContainer mainImage={property.image} name={property.name} />
```
### Col Layout
- properties/[id]/page.tsx
```tsx
return (
  <section className='lg:grid lg:grid-cols-12 gap-x-12 mt-12'>
    <div className='lg:col-span-8'>
      <div className='flex gap-x-4 items-center'>
        <h1 className='text-xl font-bold'>{property.name}</h1>
        <PropertyRating inPage propertyId={property.id} />
      </div>
    </div>
    <div className='lg:col-span-4 flex flex-col items-center'>
      {/* calendar */}
    </div>
  </section>
);
```
### Calendar - Initial Setup
- components/properties/booking/BookingCalendar.tsx
```tsx
'use client';
import { useState } from 'react';
import { Calendar } from '@/components/ui/calendar';
import { DateRange } from 'react-day-picker';
export default function App() {
  const currentDate = new Date();
  const defaultSelected: DateRange = {
    from: undefined,
    to: undefined,
  };
  const [range, setRange] = useState<DateRange | undefined>(defaultSelected);
  return (
    <Calendar
      id='test'
      mode='range'
      defaultMonth={currentDate}
      selected={range}
      onSelect={setRange}
    />
  );
}
```
- properties/[id]/page.tsx
```tsx
<div className='lg:col-span-4 flex flex-col items-center'>
  {/* calendar */}
  <BookingCalendar />
</div>
```
### PropertyDetails
- utils/format.ts
```ts
export function formatQuantity(quantity: number, noun: string): string {
  return quantity === 1 ? `${quantity} ${noun}` : `${quantity} ${noun}s`;
}
```
- components/properties/PropertyDetails.tsx
```tsx
import { formatQuantity } from '@/utils/format';
type PropertyDetailsProps = {
  details: {
    bedrooms: number;
    baths: number;
    guests: number;
    beds: number;
  };
};
function PropertyDetails({
  details: { bedrooms, baths, guests, beds },
}: PropertyDetailsProps) {
  return (
    <p className='text-md font-light '>
      <span>{formatQuantity(bedrooms, 'bedroom')} &middot; </span>
      <span>{formatQuantity(baths, 'bath')} &middot; </span>
      <span>{formatQuantity(guests, 'guest')} &middot; </span>
      <span>{formatQuantity(beds, 'bed')}</span>
    </p>
  );
}
export default PropertyDetails;
```
- properties/[id]/page.tsx
```tsx
<PropertyDetails details={details} />
```
