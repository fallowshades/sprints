# Search Github Users

This is a web app that allows you to search for users on Github.

## Technologies Used

- React
- Vite
- Tailwind CSS
- Shadcn UI
- Typescript
- Github API
- Appollo Client

## Setup

- create a new directory

```bash
npm create vite@latest . -- --template react-ts
```

```bash
npm install
```

```bash
npm run dev
```

## Tailwind CSS

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

index.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

App.tsx

```tsx
const App = () => {
  return <h1 className='text-2xl font-bold'>Search Github Users</h1>;
};
export default App;
```

- remove App.css
- change title in index.html

```html
<title>Search Github Users</title>
```

## Shadcn UI

tsconfig.json

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ],
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

tsconfig.app.json

```json
{
  "compilerOptions": {
    // rest of the options
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

```bash
npm i -D @types/node

```

vite.config.ts

```ts
import path from 'path';
import react from '@vitejs/plugin-react';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

- initialize shadcn

```bash
npx shadcn@latest init
```

- add components

```bash
npx shadcn@latest add button card chart input label skeleton toast
```

App.tsx

```tsx
import { Button } from '@/components/ui/button';
const App = () => {
  return (
    <div className='flex  items-center justify-center h-screen'>
      <div className='flex gap-4'>
        <Button>Click me</Button>
        <Button variant='outline' size='lg'>
          Click me
        </Button>
        <Button variant='destructive' size='sm'>
          Click me
        </Button>
      </div>
    </div>
  );
};
export default App;
```

## Structure

- setup local state in App.tsx
- create src/components/form/SearchForm.tsx
- create src/components/user/UserProfile.tsx
- render both components in App.tsx
- pass userName and setUserName to SearchForm
- pass userName to UserProfile

App.tsx

```tsx
const [userName, setUserName] = useState('quincylarson');
```

src/components/form/SearchForm.tsx

```tsx
type SearchFormProps = {
  userName: string;
  setUserName: React.Dispatch<React.SetStateAction<string>>;
};

const SearchForm = ({ userName, setUserName }: SearchFormProps) => {
  return <div>SearchForm</div>;
};
export default SearchForm;
```

src/components/user/UserProfile.tsx

```tsx
type UserProfileProps = {
  userName: string;
};

const UserProfile = ({ userName }: UserProfileProps) => {
  return <h1 className='text-2xl font-bold'>{userName}</h1>;
};
export default UserProfile;
```

src/App.tsx

```tsx
mport { useState } from 'react';
import SearchForm from './components/form/SearchForm';
import UserProfile from './components/user/UserProfile';

const App = () => {
  const [userName, setUserName] = useState('quincylarson');

  return (
    <main className='mx-auto max-w-6xl px-8 py-20'>
      <SearchForm userName={userName} setUserName={setUserName} />
      <UserProfile userName={userName} />
    </main>
  );
};
export default App;
```
