# take home away

##

### when and where


# take home away
##
### when and where

### UserInfo
- components/properties/UserInfo.tsx
```tsx
import Image from 'next/image';
type UserInfoProps = {
  profile: {
    profileImage: string;
    firstName: string;
  };
};
function UserInfo({ profile: { profileImage, firstName } }: UserInfoProps) {
  return (
    <article className='grid grid-cols-[auto,1fr] gap-4 mt-4'>
      <Image
        src={profileImage}
        alt={firstName}
        width={50}
        height={50}
        className='rounded-md w-12 h-12 object-cover'
      />
      <div>
        <p>
          Hosted by
          <span className='font-bold'> {firstName}</span>
        </p>
        <p className='text-muted-foreground font-light'>
          Superhost &middot; 2 years hosting
        </p>
      </div>
    </article>
  );
}
export default UserInfo;
```
- properties/[id]/page.tsx
```tsx
const firstName = property.profile.firstName;
const profileImage = property.profile.profileImage;
<UserInfo profile={{ firstName, profileImage }} />;
```
### Description
- components/properties/Title.tsx
```tsx
function Title({ text }: { text: string }) {
  return <h3 className='text-lg font-bold  mb-2'>{text}</h3>;
}
export default Title;
```
- components/properties/Description.tsx
```tsx
'use client';
import { useState } from 'react';
import { Button } from '@/components/ui/button';
import Title from './Title';
const Description = ({ description }: { description: string }) => {
  const [isFullDescriptionShown, setIsFullDescriptionShown] = useState(false);
  const words = description.split(' ');
  const isLongDescription = words.length > 100;
  const toggleDescription = () => {
    setIsFullDescriptionShown(!isFullDescriptionShown);
  };
  const displayedDescription =
    isLongDescription && !isFullDescriptionShown
      ? words.slice(0, 100).join(' ') + '...'
      : description;
  return (
    <article className='mt-4'>
      <Title text='Description' />
      <p className='text-muted-foreground font-light leading-loose'>
        {displayedDescription}
      </p>
      {isLongDescription && (
        <Button variant='link' className='pl-0' onClick={toggleDescription}>
          {isFullDescriptionShown ? 'Show less' : 'Show more'}
        </Button>
      )}
    </article>
  );
};
export default Description;
```
- properties/[id]/page.tsx
```tsx
<Separator className='mt-4' />
<Description description={property.description} />
```
## where
### Amenities
- components/properties/Amenities.tsx
```tsx
import { Amenity } from '@/utils/amenities';
import { LuFolderCheck } from 'react-icons/lu';
import Title from './Title';
function Amenities({ amenities }: { amenities: string }) {
  const amenitiesList: Amenity[] = JSON.parse(amenities as string);
  const noAmenities = amenitiesList.every((amenity) => !amenity.selected);
  if (noAmenities) {
    return null;
  }
  return (
    <div className='mt-4'>
      <Title text='What this place offers' />
      <div className='grid md:grid-cols-2 gap-x-4'>
        {amenitiesList.map((amenity) => {
          if (!amenity.selected) {
            return null;
          }
          return (
            <div key={amenity.name} className='flex items-center gap-x-4 mb-2 '>
              <LuFolderCheck className='h-6 w-6 text-primary' />
              <span className='font-light text-sm capitalize'>
                {amenity.name}
              </span>
            </div>
          );
        })}
      </div>
    </div>
  );
}
export default Amenities;
```
- properties/[id]/page.tsx
```tsx
<Amenities amenities={property.amenities} />
```
### PropertyMap
[ React Leaflet](https://react-leaflet.js.org/) 
Leaflet makes direct calls to the DOM when it is loaded, therefore React Leaflet is not compatible with server-side rendering.
```sh
npm install react react-dom leaflet react-leaflet
```
```sh
npm install -D @types/leaflet
```
- components/properties/PropertyMap.tsx
```tsx
'use client';
import { MapContainer, TileLayer, Marker, ZoomControl } from 'react-leaflet';
import 'leaflet/dist/leaflet.css';
import { icon } from 'leaflet';
const iconUrl =
  'https://unpkg.com/leaflet@1.9.3/dist/images/marker-icon-2x.png';
const markerIcon = icon({
  iconUrl: iconUrl,
  iconSize: [20, 30],
});
import { findCountryByCode } from '@/utils/countries';
import CountryFlagAndName from '../card/CountryFlagAndName';
import Title from './Title';
function PropertyMap({ countryCode }: { countryCode: string }) {
  const defaultLocation = [51.505, -0.09] as [number, number];
  const location = findCountryByCode(countryCode)?.location as [number, number];
  return (
    <div className='mt-4'>
      <div className='mb-4 '>
        <Title text='Where you will be staying' />
        <CountryFlagAndName countryCode={countryCode} />
      </div>
      <MapContainer
        scrollWheelZoom={false}
        zoomControl={false}
        className='h-[50vh] rounded-lg relative z-0'
        center={location || defaultLocation}
        zoom={7}
      >
        <TileLayer
          attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
          url='https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
        />
        <ZoomControl position='bottomright' />
        <Marker
          position={location || defaultLocation}
          icon={markerIcon}
        ></Marker>
      </MapContainer>
    </div>
  );
}
export default PropertyMap;
```
- properties/[id]/page.tsx
```tsx
const DynamicMap = dynamic(
  () => import('@/components/properties/PropertyMap'),
  {
    ssr: false,
    loading: () => <Skeleton className='h-[400px] w-full' />,
  }
);
return <DynamicMap countryCode={property.country} />;
```
Lazy Loading: Components wrapped with dynamic are lazy loaded. This means that the component code is not loaded until it is needed. For example, if you have a component that is only visible when a user clicks a button, you could use dynamic to ensure that the code for that component is not loaded until the button is clicked.
Server Side Rendering (SSR) Control: By default, Next.js pre-renders every page. This means that it generates HTML for each page in advance, instead of doing it all on the client-side. However, with dynamic, you can control this behavior. You can choose to disable SSR for specific modules, which can be useful for modules that have client-side dependencies.
### Deploy
```json
"scripts": {
  "dev": "next dev",
  "build": "npx prisma generate && next build",
  "start": "next start",
  "lint": "next lint"
},
```
- refactor NavSearch Component







<!--- Eraser file: https://app.eraser.io/workspace/SUukoDZZGbo0NF0fxbT4 --->
