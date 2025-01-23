# s2

## Search Form

```tsx
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { type FormEvent } from 'react';
import { useState } from 'react';

type SearchFormProps = {
  userName: string;
  setUserName: React.Dispatch<React.SetStateAction<string>>;
};

const SearchForm = ({ userName, setUserName }: SearchFormProps) => {
  const [text, setText] = useState(userName);

  const handleSearch = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    if (text === '') {
      console.log('Please enter a username');
      return;
    }
    setUserName(text);
  };

  return (
    <form
      onSubmit={handleSearch}
      className='flex items-center gap-x-2 w-full lg:w-1/3 mb-8'
    >
      <Label htmlFor='search' className='sr-only'>
        Search
      </Label>
      <Input
        type='text'
        id='search'
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder='Search Github User...'
        className='flex-grow bg-background'
      />
      <Button type='submit'>Search</Button>
    </form>
  );
};
export default SearchForm;
```

## Shadcn Toast

main.tsx

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.tsx';
// import Toaster component
import { Toaster } from '@/components/ui/toaster';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
    <Toaster />
  </StrictMode>
);
```

src/components/form/SearchForm.tsx

```tsx
import { useToast } from '@/hooks/use-toast';

const SearchForm = ({ userName, setUserName }: SearchFormProps) => {
  const { toast } = useToast();

  const handleSearch = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    if (text === '') {
      toast({
        description: 'Please enter a valid username',
      });
      return;
    }
    setUserName(text);
  };

  return <form>...</form>;
};
export default SearchForm;
```

## Graphql

GraphQL is a modern query language and runtime for APIs that allows clients to request specific data they need and nothing more. Unlike traditional REST APIs where you get fixed data from multiple endpoints, GraphQL provides a single endpoint where you can specify exactly what data you want to receive.

- **Schema**: The blueprint that defines all available data types and operations in your API
- **Query**: A request to read or fetch data (similar to GET in REST)
- **Mutation**: A request to create, update, or delete data (similar to POST/PUT/DELETE in REST)
- **Fields**: The individual pieces of data you can request (like user.name or post.title)
- **Arguments**: Parameters you can pass to fields to filter or modify the results (like limit: 10)
- **Types**: The different kinds of data objects available (like User, Post, Comment)
- **Nodes**: Objects in a GraphQL schema that have a unique identifier, typically representing entities in your data model (like a specific user or post)

[Practice API's](https://www.apollographql.com/blog/8-free-to-use-graphql-apis-for-your-projects-and-demos)

## Github GraphQL Explorer

[Github GraphQL Explorer](https://docs.github.com/en/graphql/overview/explorer)

## Github Personal Access Token

[Github](https://github.com/)

- profile
- settings
- developer settings
- personal access token
- generate new token
- create .env.local file
- add token to .env.local file

.env.local

```
VITE_GITHUB_TOKEN=YOUR_TOKEN_HERE
```

## Apollo Client

Apollo Client is a comprehensive state management library for JavaScript applications that helps you manage both local and remote data with GraphQL. It makes it easy to fetch, cache, and modify application data while automatically handling important concerns like tracking loading and error states. The library integrates especially well with React applications and provides features like automatic caching, optimistic UI updates, and error handling out of the box.

[Apollo Client](https://www.apollographql.com/docs/react/get-started/)

```bash
npm install @apollo/client graphql
```

- src/apolloClient.ts

```ts
// Core Apollo Client imports for GraphQL functionality
// ApolloClient: Main client class for making GraphQL requests
// InMemoryCache: Caching solution for storing query results
// HttpLink: Configures HTTP connection to GraphQL endpoint
// ApolloLink: Enables creation of middleware chain for request/response handling
import {
  ApolloClient,
  InMemoryCache,
  HttpLink,
  ApolloLink,
} from '@apollo/client';

// Error handling middleware for Apollo Client
// Provides detailed error information for both GraphQL and network errors
import { onError } from '@apollo/client/link/error';

// GitHub GraphQL API endpoint
const GITHUB_GRAPHQL_API = 'https://api.github.com/graphql';

// Configure error handling middleware
// This will intercept and log any GraphQL or network errors
const errorLink = onError(({ graphQLErrors, networkError }) => {
  // Handle GraphQL-specific errors (e.g., validation, resolver errors)
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, locations, path }) => {
      console.error(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`
      );
    });
  }

  // Handle network-level errors (e.g., connection issues)
  if (networkError) {
    console.error(`[Network error]: ${networkError}`);
  }
});

// Configure HTTP connection to GitHub's GraphQL API
// Including authentication token from environment variables
const httpLink = new HttpLink({
  uri: GITHUB_GRAPHQL_API,
  headers: {
    Authorization: `Bearer ${import.meta.env.VITE_GITHUB_TOKEN}`, // GitHub Personal Access Token
  },
});

// Create the Apollo Link chain
// Order matters: errorLink will run before httpLink
const link = ApolloLink.from([errorLink, httpLink]);

// Initialize Apollo Client with:
// - Configured link chain for network requests
// - In-memory cache for storing query results
const client = new ApolloClient({
  link,
  cache: new InMemoryCache(),
});

export default client;
```

src/main.tsx

```tsx
import { createRoot } from 'react-dom/client';
import App from './App.tsx';
import './index.css';
import { Toaster } from '@/components/ui/toaster';
// Apollo Provider
import { ApolloProvider } from '@apollo/client';
import client from './apolloClient';

createRoot(document.getElementById('root')!).render(
  <ApolloProvider client={client}>
    <App />
    <Toaster />
  </ApolloProvider>
);
```

## Query and Type

src/queries.ts

```ts
import { gql } from '@apollo/client';

export const GET_USER = gql`
  query ($login: String!) {
    user(login: $login) {
      name
      avatarUrl
      bio
      url
      repositories(first: 100) {
        totalCount
        nodes {
          name
          description
          stargazerCount
          forkCount
          url
          languages(first: 5) {
            edges {
              node {
                name
              }
              size
            }
          }
        }
      }
      followers {
        totalCount
      }
      following {
        totalCount
      }
      gists {
        totalCount
      }
    }
  }
`;
```

src/types.ts

```ts
export type LanguageEdge = {
  node: {
    name: string;
  };
  size: number;
};

export type Repository = {
  name: string;
  description: string;
  stargazerCount: number;
  forkCount: number;
  url: string;
  languages: {
    edges: LanguageEdge[];
  };
};

export type User = {
  name: string;
  avatarUrl: string;
  bio: string;
  url: string;
  repositories: {
    totalCount: number;
    nodes: Repository[];
  };
  followers: {
    totalCount: number;
  };
  following: {
    totalCount: number;
  };
  gists: {
    totalCount: number;
  };
};
export type UserData = {
  user: User;
};
```

## Query Hook

src/components/user/UserProfile.tsx

```tsx
import { useQuery } from '@apollo/client';
import { GET_USER } from '@/queries';
import { UserData } from '@/types';

type UserProfileProps = {
  userName: string;
};

const UserProfile = ({ userName }: UserProfileProps) => {
  const { loading, error, data } = useQuery<UserData>(GET_USER, {
    variables: { login: userName },
  });

  if (loading) return <div>Loading...</div>;
  if (error) return <h2 className='text-xl'>{error.message}</h2>;
  if (!data) return <h2 className='text-xl'>User Not Found.</h2>;

  const {
    avatarUrl,
    name,
    bio,
    url,
    repositories,
    followers,
    following,
    gists,
  } = data.user;

  return (
    <div>
      <h1>{bio}</h1>
    </div>
  );
};

export default UserProfile;
```
