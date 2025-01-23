
## Charts

- components/charts/UsedLanguages.tsx
- components/charts/PopularRepos.tsx
- components/charts/ForkedRepos.tsx

UserProfile.tsx

```tsx
{
  repositories.totalCount > 0 && (
    <div className='grid md:grid-cols-2 gap-4'>
      <UsedLanguages repositories={repositories.nodes} />
      <PopularRepos repositories={repositories.nodes} />
      <ForkedRepos repositories={repositories.nodes} />
    </div>
  );
}
```

## Used Languages

components/charts/UsedLanguages.tsx

```tsx
import { type Repository } from '@/types';
import { Bar, BarChart, CartesianGrid, XAxis, YAxis } from 'recharts';
import {
  ChartConfig,
  ChartContainer,
  ChartTooltip,
  ChartTooltipContent,
} from '@/components/ui/chart';
import { calculatePopularLanguages } from '@/utils';

const UsedLanguages = ({ repositories }: { repositories: Repository[] }) => {
  // Calculate popular languages
  //  [{language: string, count: number}]
  const popularLanguages = calculatePopularLanguages(repositories);

  // Configuration for the chart's styling and labels
  // color sets the color of the bars

  const chartConfig = {
    language: {
      label: 'Language',
      color: '#2563eb',
    },
  } satisfies ChartConfig;
  return (
    <div>
      <h2 className='text-2xl font-semibold text-center mb-4'>
        Used Languages
      </h2>
      {/* ChartContainer handles responsive sizing and theme variables */}
      <ChartContainer config={chartConfig} className='h-100 w-full'>
        {/* BarChart is the main container for the bar chart visualization */}
        {/* accessibilityLayer adds ARIA labels for better screen reader support */}
        <BarChart accessibilityLayer data={popularLanguages}>
          {/* CartesianGrid adds horizontal guide lines */}
          <CartesianGrid vertical={false} />

          {/* XAxis configures the horizontal axis showing language names */}
          <XAxis
            dataKey='language'
            tickLine={false} // Removes tick marks
            tickMargin={10} // Adds spacing between labels and axis
          />

          {/* YAxis configures the vertical axis showing count values */}
          <YAxis dataKey='count' />

          {/* ChartTooltip shows details when hovering over bars */}
          <ChartTooltip content={<ChartTooltipContent />} />

          {/* Bar component defines how each data point is rendered */}
          {/* Uses CSS variable for color and adds rounded corners */}
          <Bar dataKey='count' fill='var(--color-language)' radius={4} />
        </BarChart>
      </ChartContainer>
    </div>
  );
};

export default UsedLanguages;
```

## Popular Repos

components/charts/PopularRepos.tsx

```tsx
import { type Repository } from '@/types';
import { Bar, BarChart, CartesianGrid, XAxis, YAxis } from 'recharts';
import {
  ChartConfig,
  ChartContainer,
  ChartTooltip,
  ChartTooltipContent,
} from '@/components/ui/chart';
import { calculateMostStarredRepos } from '@/utils';

const PopularRepos = ({ repositories }: { repositories: Repository[] }) => {
  // Calculate most starred repositories and return array of {repo: string, stars: number}
  const popularRepos = calculateMostStarredRepos(repositories);

  // Configuration for the chart's styling and labels
  const chartConfig = {
    repo: {
      label: 'Repository',
      color: '#e11c47', // Red color for the bars
    },
  } satisfies ChartConfig;

  return (
    <div>
      <h2 className='text-2xl font-semibold text-center mb-4'>Popular Repos</h2>
      {/* ChartContainer: Custom wrapper component that handles responsive sizing and theme */}
      <ChartContainer config={chartConfig} className='h-100 w-full'>
        {/* BarChart: Main chart component from recharts */}
        {/* accessibilityLayer adds ARIA labels for better screen reader support */}
        <BarChart accessibilityLayer data={popularRepos}>
          {/* CartesianGrid: Adds horizontal guide lines (vertical disabled) */}
          <CartesianGrid vertical={false} />

          {/* XAxis: Horizontal axis showing repository names */}
          {/* tickFormatter truncates long repository names to 10 characters */}
          <XAxis
            dataKey='repo'
            tickLine={false}
            tickMargin={10}
            tickFormatter={(value) => value.slice(0, 10)}
          />

          {/* YAxis: Vertical axis showing star counts */}
          <YAxis dataKey='stars' />

          {/* ChartTooltip: Custom tooltip component that appears on hover */}
          {/* ChartTooltipContent: Renders the actual content inside the tooltip */}
          <ChartTooltip content={<ChartTooltipContent />} />

          {/* Bar: The actual bar elements of the chart */}
          {/* fill uses CSS variable for consistent theming */}
          {/* radius adds rounded corners to the bars */}
          <Bar dataKey='stars' fill='var(--color-repo)' radius={4} />
        </BarChart>
      </ChartContainer>
    </div>
  );
};

export default PopularRepos;
```

## Forked Repos

components/charts/ForkedRepos.tsx

```tsx
import { type Repository } from '@/types';
import { Bar, BarChart, CartesianGrid, XAxis, YAxis } from 'recharts';
import {
  ChartConfig,
  ChartContainer,
  ChartTooltip,
  ChartTooltipContent,
} from '@/components/ui/chart';
import { calculateMostForkedRepos } from '@/utils';

const ForkedRepos = ({ repositories }: { repositories: Repository[] }) => {
  // Calculate most forked repositories and return array of {repo: string, count: number}
  const mostForkedRepos = calculateMostForkedRepos(repositories);

  // Define chart configuration for styling and labels
  const chartConfig = {
    repo: {
      label: 'Repository',
      color: '#facd12',
    },
  } satisfies ChartConfig;

  return (
    <div>
      <h2 className='text-2xl font-semibold text-center mb-4'>Forked Repos</h2>
      {/* ChartContainer handles responsive sizing and theme variables */}
      <ChartContainer config={chartConfig} className='h-100 w-full'>
        {/* BarChart is the main container for the bar chart visualization */}
        {/* accessibilityLayer adds ARIA labels for better screen reader support */}
        <BarChart accessibilityLayer data={mostForkedRepos}>
          {/* CartesianGrid adds background gridlines, vertical lines disabled */}
          <CartesianGrid vertical={false} />

          {/* XAxis configures the horizontal axis */}
          <XAxis
            dataKey='repo' // Uses 'repo' property from data for labels
            tickLine={true} // Shows small lines at each tick mark
            tickMargin={10} // Space between tick line and label
            axisLine={false} // Hides the main axis line
            tickFormatter={(value) => value.slice(0, 10)} // Truncates long repo names
          />

          {/* YAxis configures the vertical axis, showing fork counts */}
          <YAxis dataKey='count' />

          {/* ChartTooltip shows details when hovering over bars */}
          <ChartTooltip content={<ChartTooltipContent />} />

          {/* Bar component defines the actual bars in the chart */}
          {/* Uses CSS variable for color and rounded corners (radius) */}
          <Bar dataKey='count' fill='var(--color-repo)' radius={4} />
        </BarChart>
      </ChartContainer>
    </div>
  );
};

export default ForkedRepos;
```

## Loading

src/components/user/Loading.tsx

```tsx
import { Skeleton } from '@/components/ui/skeleton';

/**
 * Loading component that displays placeholder content while data is being fetched
 * Uses shadcn/ui's Skeleton component to create loading animations
 */
const Loading = () => {
  return (
    <div>
      {/* Large header skeleton
          - h-[194px]: Fixed height of 194px
          - w-full: Full width on mobile
          - lg:w-1/2: Half width on large screens
          - mb-8: Bottom margin of 2rem */}
      <Skeleton className='h-[194px] w-full lg:w-1/2 mb-8 rounded ' />

      {/* Grid container for smaller skeletons
          - grid-cols-1: Single column on mobile
          - lg:grid-cols-2: 2 columns on large screens
          - xl:grid-cols-4: 4 columns on extra large screens
          - gap-2: Small gap between grid items */}
      <div className='grid grid-cols-1 lg:grid-cols-2 xl:grid-cols-4 gap-2 mb-8'>
        {/* Four identical skeleton items
            - h-[70px]: Fixed height of 70px
            - rounded: Rounded corners */}
        <Skeleton className=' h-[70px] rounded' />
        <Skeleton className=' h-[70px] rounded' />
        <Skeleton className=' h-[70px] rounded' />
        <Skeleton className=' h-[70px] rounded' />
      </div>
    </div>
  );
};

export default Loading;
```

UserProfile.tsx

```tsx
if (loading) return <Loading />;
```