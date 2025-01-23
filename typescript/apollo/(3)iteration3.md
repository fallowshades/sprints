
## User Card

src/components/user/UserCard.tsx

```tsx
import { Button } from '@/components/ui/button';
import {
  Card,
  CardTitle,
  CardDescription,
  CardHeader,
} from '@/components/ui/card';

type UserCardProps = {
  avatarUrl: string;
  name: string;
  bio: string;
  url: string;
};
const UserCard = ({ avatarUrl, name, bio, url }: UserCardProps) => {
  return (
    <Card className='w-full lg:w-1/2 mb-8'>
      <CardHeader className='flex-row gap-x-8 items-center'>
        <img
          src={avatarUrl}
          alt={name}
          className='w-36 h-36  rounded object-cover'
        />
        <div className='flex flex-col gap-y-2'>
          <CardTitle>{name || 'Coding Addict'}</CardTitle>
          <CardDescription>
            {bio || 'Passionate about coding and technology.'}
          </CardDescription>
          <Button asChild size='sm' className='w-1/2 mt-2'>
            <a href={url} target='_blank' rel='noreferrer'>
              Follow
            </a>
          </Button>
        </div>
      </CardHeader>
    </Card>
  );
};
export default UserCard;
```

- UserProfile.tsx

```tsx
return (
  <div>
    <UserCard avatarUrl={avatarUrl} name={name} bio={bio} url={url} />
  </div>
);
```

## Stats Card

```tsx
import { Card, CardTitle, CardDescription } from '../ui/card';

type StatsCardProps = {
  title: string;
  count: number;
};

function StatsCard({ title, count }: StatsCardProps) {
  return (
    <Card>
      <div className='flex flex-row justify-between items-center p-6'>
        <CardTitle>{title}</CardTitle>
        <CardDescription>{count}</CardDescription>
      </div>
    </Card>
  );
}

export default StatsCard;
```

## Stats Container

```tsx
import StatsCard from './StatsCard';

type StatsContainerProps = {
  totalRepos: number;
  followers: number;
  following: number;
  gists: number;
};

const StatsContainer = (props: StatsContainerProps) => {
  const { totalRepos, followers, following, gists } = props;

  return (
    <div className='grid grid-cols-1 md:grid-cols-2 xl:grid-cols-4 gap-2 mb-8 '>
      <StatsCard title='Total Repositories' count={totalRepos} />
      <StatsCard title='Followers' count={followers} />
      <StatsCard title='Following' count={following} />
      <StatsCard title='Gists' count={gists} />
    </div>
  );
};
export default StatsContainer;
```

UserProfile.tsx

```tsx
return (
  <div>
    <UserCard avatarUrl={avatarUrl} name={name} bio={bio} url={url} />
    <StatsContainer
      totalRepos={repositories.totalCount}
      followers={followers.totalCount}
      following={following.totalCount}
      gists={gists.totalCount}
    />
  </div>
);
```

## Util Functions

And once we are done with the Stats container, we can start working on the charts, but since charts will need very specific data, first we will need to create some util functions to help us generate such data.

src/utils.ts

```ts
import { Repository } from './types';

/**
 * Calculates the top 5 most forked repositories
 * @param repositories Array of repository data from GitHub API
 * @returns Array of objects containing repository names and their fork counts
 * Example return: [{ repo: "react", count: 1000 }, { repo: "vue", count: 500 }]
 */
export const calculateMostForkedRepos = (
  repositories: Repository[]
): { repo: string; count: number }[] => {
  if (repositories.length === 0) {
    return [];
  }

  // Transform repository data into simplified objects containing only name and fork count
  const forkedRepos = repositories
    .map((repo) => ({
      repo: repo.name, // Extract repository name
      count: repo.forkCount, // Extract number of forks
    }))
    .sort((a, b) => b.count - a.count) // Sort by fork count in descending order
    .slice(0, 5); // Take only the top 5 repositories

  return forkedRepos;
};

/**
 * Calculates the top 5 most starred repositories
 * @param repositories Array of repository data from GitHub API
 * @returns Array of objects containing repository names and their star counts
 * Example return: [{ repo: "tensorflow", stars: 5000 }, { repo: "linux", stars: 4000 }]
 */
export const calculateMostStarredRepos = (
  repositories: Repository[]
): { repo: string; stars: number }[] => {
  if (repositories.length === 0) {
    return [];
  }

  // Transform repository data into simplified objects containing only name and star count
  const starredRepos = repositories
    .map((repo) => ({
      repo: repo.name, // Extract repository name
      stars: repo.stargazerCount, // Extract number of stars (stargazers)
    }))
    .sort((a, b) => b.stars - a.stars) // Sort by star count in descending order
    .slice(0, 5); // Take only the top 5 repositories

  return starredRepos;
};

/**
 * Calculates the top 5 most used programming languages across all repositories
 * @param repositories Array of repository data from GitHub API
 * @returns Array of objects containing language names and their occurrence count
 * Example return: [{ language: "JavaScript", count: 10 }, { language: "Python", count: 7 }]
 */

export const calculatePopularLanguages = (
  repositories: Repository[]
): { language: string; count: number }[] => {
  // Return empty array if no repositories are provided
  if (repositories.length === 0) {
    return [];
  }

  // Initialize a map to track how many times each language appears
  // Example: { "JavaScript": 5, "Python": 3, "TypeScript": 2 }
  const languageMap: { [key: string]: number } = {};

  repositories.forEach((repo) => {
    // Skip repositories with no languages
    if (repo.languages.edges.length === 0) {
      return;
    }

    // Iterate through each language in the repository
    // languages.edges comes from GitHub's GraphQL API structure
    repo.languages.edges.forEach((language) => {
      const { name } = language.node;
      // Increment the count for this language, initializing to 1 if it's the first occurrence
      languageMap[name] = (languageMap[name] || 0) + 1;
    });
  });

  // If no languages were found in any repository, return empty array
  if (Object.keys(languageMap).length === 0) {
    return [];
  }

  // Convert the language map into an array of objects and sort them
  return (
    Object.entries(languageMap)
      // Convert entries into array of [language, count] pairs
      .sort(([, a], [, b]) => b - a) // Sort by count in descending order
      .slice(0, 5) // Take only the top 5 languages
      .map(([language, count]) => ({ language, count }))
  ); // Transform into required object format
};
```
