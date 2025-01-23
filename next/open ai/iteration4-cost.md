### Max_Tokens

actions.js

```js
export const generateChatResponse = async (chatMessages) => {
  try {
    const response = await openai.chat.completions.create({
      max_tokens: 100,
    })

    return response.choices[0].message
  } catch (error) {
    console.log(error)
    return null
  }
}
```

### Clerk Logic

- remove ability to delete account

  - Email, Phone, Username
    - Allow users to delete their accounts (false)

- block disposable emails

  - Restrictions
    - Block sign-ups that use disposable email addresses

- remove all users

### Token Model

```prisma
model Token {
  clerkId String @id
  tokens Int @default (1000)
}
```

- migrate

### Actions

actions.js

```js
export const fetchUserTokensById = async (clerkId) => {
  const result = await prisma.token.findUnique({
    where: {
      clerkId,
    },
  })

  return result?.tokens
}

export const generateUserTokensForId = async (clerkId) => {
  const result = await prisma.token.create({
    data: {
      clerkId,
    },
  })
  return result?.tokens
}

export const fetchOrGenerateTokens = async (clerkId) => {
  const result = await fetchUserTokensById(clerkId)
  if (result) {
    return result.tokens
  }
  return (await generateUserTokensForId(clerkId)).tokens
}

export const subtractTokens = async (clerkId, tokens) => {
  const result = await prisma.token.update({
    where: {
      clerkId,
    },
    data: {
      tokens: {
        decrement: tokens,
      },
    },
  })
  revalidatePath('/profile')
  // Return the new token value
  return result.tokens
}
```

### Generate and Show Tokens

components/MemberProfile.jsx

```js
import { fetchOrGenerateTokens } from '@/utils/actions'
import { UserButton, auth, currentUser } from '@clerk/nextjs'

const MemberProfile = async () => {
  const user = await currentUser()
  const { userId } = auth()
  await fetchOrGenerateTokens(userId)
  return (
    <div className="px-4 flex items-center gap-2">
      <UserButton afterSignOutUrl="/" />
      <p>{user.emailAddresses[0].emailAddress}</p>
    </div>
  )
}
export default MemberProfile
```

profile/page.js

```js
import { fetchUserTokensById } from '@/utils/actions'
import { UserProfile, auth } from '@clerk/nextjs'

const ProfilePage = async () => {
  const { userId } = auth()
  const currentTokens = await fetchUserTokensById(userId)
  return (
    <div>
      <h2 className="mb-8 ml-8 text-xl font-extrabold">
        Token Amount : {currentTokens}
      </h2>
      <UserProfile />
    </div>
  )
}
export default ProfilePage
```

### Tours

actions.js

```js
export const generateTourResponse = () => {
  return { tour: tourData.tour, tokens: response.usage.total_tokens }
}
```

components/NewTour.jsx

```js
'use client';

import {
  fetchUserTokensById,
  subtractTokens,
} from '@/utils/actions';
import { useAuth } from '@clerk/nextjs';
const NewTour = () => {


  const { userId } = useAuth();

  const {
    mutate,
    isPending,
    data: tour,
  } = useMutation({
    mutationFn: async (destination) => {
      const existingTour = await getExistingTour(destination);
      if (existingTour) return existingTour;

      const currentTokens = await fetchUserTokensById(userId);

      if (currentTokens < 300) {
        toast.error('Token balance too low....');
        return;
      }

      const newTour = await generateTourResponse(destination);
      if (!newTour) {
        toast.error('No matching city found...');
        return null;
      }

      const response = await createNewTour(newTour.tour);
      queryClient.invalidateQueries({ queryKey: ['tours'] });
      const newTokens = await subtractTokens(userId, newTour.tokens);
      toast.success(`${newTokens} tokens remaining...`);
      return newTour.tour;
    },
  });
  ....
}
```

### Chat

actions.js

```js
const generateChatResponse = () => {
  return {
    message: response.choices[0].message,
    tokens: response.usage.total_tokens,
  }
}
```

components/Chat.jsx

```js
'use client';

import { fetchUserTokensById, subtractTokens } from '@/utils/actions';

import { useAuth } from '@clerk/nextjs';
const Chat = () => {
  const { userId } = useAuth();

  const { mutate, isPending } = useMutation({
    mutationFn: async (query) => {
      const currentTokens = await fetchUserTokensById(userId);

      if (currentTokens < 100) {
        toast.error('Token balance too low....');
        return;
      }

      const response = await generateChatResponse([...messages, query]);

      if (!response) {
        toast.error('Something went wrong...');
        return;
      }
      setMessages((prev) => [...prev, response.message]);
      const newTokens = await subtractTokens(userId, response.tokens);
      toast.success(`${newTokens} tokens remaining...`);
    },
  });
...
};
export default Chat;
```

## Deploy

package.json

```js

"scripts": {
    "build": "npx prisma generate && next build",
  },
```

- shorter prompt
  "stops":["stop 1","stop 2", "stop 3"]

- planetscale
- github repo
- vercel
