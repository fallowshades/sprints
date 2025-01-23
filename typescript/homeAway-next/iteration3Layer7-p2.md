
# take home away
##
### articles in * load pages
## linked articles
### Card Components
- components/card
    - CountryFlagAndName.tsx
    - FavoriteToggleButton.tsx
    - FavoriteToggleForm.tsx
    - LoadingCards.tsx
    - PropertyCard.tsx
    - PropertyRating.tsx
### PropertiesList
```tsx
import PropertyCard from '../card/PropertyCard';
import type { PropertyCardProps } from '@/utils/types';
function PropertiesList({ properties }: { properties: PropertyCardProps[] }) {
  return (
    <section className='mt-4 gap-8 grid sm:grid-cols-2  lg:grid-cols-3  xl:grid-cols-4'>
      {properties.map((property) => {
        return <PropertyCard key={property.id} property={property} />;
      })}
    </section>
  );
}
export default PropertiesList;
```
### formatCurrency
- utils/format.ts
```ts
export const formatCurrency = (amount: number | null) => {
  const value = amount || 0;
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(value);
};
```
### PropertyCard
```tsx
import Image from 'next/image';
import Link from 'next/link';
import CountryFlagAndName from './CountryFlagAndName';
import PropertyRating from './PropertyRating';
import FavoriteToggleButton from './FavoriteToggleButton';
import { PropertyCardProps } from '@/utils/types';
import { formatCurrency } from '@/utils/format';
function PropertyCard({ property }: { property: PropertyCardProps }) {
  const { name, image, price } = property;
  const { country, id: propertyId, tagline } = property;
  return (
    <article className='group relative'>
      <Link href={`/properties/${propertyId}`}>
        <div className='relative h-[300px] mb-2 overflow-hidden rounded-md'>
          <Image
            src={image}
            fill
            sizes='(max-width: 768px) 100vw, (max-width: 1200px) 50vw'
            alt={name}
            className='rounded-md object-cover transform group-hover:scale-110 transition-transform duration-500'
          />
        </div>
        <div className='flex justify-between items-center'>
          <h3 className='text-sm font-semibold mt-1'>
            {name.substring(0, 30)}
          </h3>
          {/* property rating */}
        </div>
        <p className='text-sm mt-1 text-muted-foreground '>
          {tagline.substring(0, 40)}
        </p>
        <div className='flex justify-between items-center mt-1'>
          <p className='text-sm mt-1 '>
            <span className='font-semibold'>{formatCurrency(price)} </span>
            night
          </p>
          {/* country and flag */}
        </div>
      </Link>
      <div className='absolute top-5 right-5 z-5'>
        {/* favorite toggle button */}
      </div>
    </article>
  );
}
export default PropertyCard;
```
### Property Rating
```tsx
import { FaStar } from 'react-icons/fa';
async function PropertyRating({
  propertyId,
  inPage,
}: {
  propertyId: string;
  inPage: boolean;
}) {
  // temp
  const rating = 4.7;
  const count = 100;
  const className = `flex gap-1 items-center ${inPage ? 'text-md' : 'text-xs'}`;
  const countText = count > 1 ? 'reviews' : 'review';
  const countValue = `(${count}) ${inPage ? countText : ''}`;
  return (
    <span className={className}>
      <FaStar className='w-3 h-3' />
      {rating} {countValue}
    </span>
  );
}
export default PropertyRating;
```
```tsx
<PropertyRating inPage={false} propertyId={propertyId} />
```
## handle id and conde
### FavoriteToggleButton
```tsx
import { FaHeart } from 'react-icons/fa';
import { Button } from '@/components/ui/button';
function FavoriteToggleButton({ propertyId }: { propertyId: string }) {
  return (
    <Button size='icon' variant='outline' className='p-2 cursor-pointer'>
      <FaHeart />
    </Button>
  );
}
export default FavoriteToggleButton;
```
```tsx
<div className='absolute top-5 right-5 z-5'>
  <FavoriteToggleButton propertyId={propertyId} />
</div>
```
### CountryFlagAndName
```tsx
import { findCountryByCode } from '@/utils/countries';
function CountryFlagAndName({ countryCode }: { countryCode: string }) {
  const validCountry = findCountryByCode(countryCode);
  const countryName =
    validCountry!.name.length > 20
      ? `${validCountry!.name.substring(0, 20)}...`
      : validCountry!.name;
  return (
    <span className='flex justify-between items-center gap-2 text-sm '>
      {validCountry?.flag} {countryName}
    </span>
  );
}
export default CountryFlagAndName;
```
```tsx
<CountryFlagAndName countryCode={country} />
```
## when to fetch identities for server side rendering
### Suspense
- app/loading.tsx - always an option
components/card/LoadingCards.tsx
```tsx
import { Skeleton } from '@/components/ui/skeleton';
function LoadingCards() {
  return (
    <section className='mt-4 gap-8 grid sm:grid-cols-2  lg:grid-cols-3  xl:grid-cols-4'>
      <SkeletonCard />
      <SkeletonCard />
      <SkeletonCard />
      <SkeletonCard />
    </section>
  );
}
export default LoadingCards;
export function SkeletonCard() {
  return (
    <div>
      <Skeleton className='h-[300px] rounded-md' />
      <Skeleton className='h-4 mt-2 w-3/4' />
      <Skeleton className='h-4 mt-2 w-1/2' />
    </div>
  );
}
```
app/page.tsx
- navigate to a different page, refresh and then navigate back to home page
- make sure you fetch in component not page
```tsx
import CategoriesList from '@/components/home/CategoriesList';
import PropertiesContainer from '@/components/home/PropertiesContainer';
import LoadingCards from '@/components/card/LoadingCards';
import { Suspense } from 'react';
function HomePage({
  searchParams,
}: {
  searchParams: { category?: string; search?: string };
}) {
  return (
    <section>
      <CategoriesList
        category={searchParams?.category}
        search={searchParams?.search}
      />
      <Suspense fallback={<LoadingCards />}>
        <PropertiesContainer
          category={searchParams?.category}
          search={searchParams?.search}
        />
      </Suspense>
    </section>
  );
}
export default HomePage;
```
### SearchInput
```sh
npm i use-debounce
```
components/navbar/NavSearch.tsx
```tsx
'use client';
import { Input } from '../ui/input';
import { useSearchParams, usePathname, useRouter } from 'next/navigation';
import { useDebouncedCallback } from 'use-debounce';
import { useState, useEffect } from 'react';
function NavSearch() {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();
  const [search, setSearch] = useState(
    searchParams.get('search')?.toString() || ''
  );
  const handleSearch = useDebouncedCallback((value: string) => {
    const params = new URLSearchParams(searchParams);
    if (value) {
      params.set('search', value);
    } else {
      params.delete('search');
    }
    replace(`${pathname}?${params.toString()}`);
  }, 300);
  useEffect(() => {
    if (!searchParams.get('search')) {
      setSearch('');
    }
  }, [searchParams.get('search')]);
  return (
    <Input
      type='search'
      placeholder='find a property...'
      className='max-w-xs dark:bg-muted '
      onChange={(e) => {
        setSearch(e.target.value);
        handleSearch(e.target.value);
      }}
      value={search}
    />
  );
}
export default NavSearch;
```
