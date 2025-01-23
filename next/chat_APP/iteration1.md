#### the page

nextjs-realtime-chat/src/app/(dashboard)/dashboard/chat/[chatId]
/page.tsx

- determine which chat

```tsx
const page = async ({ params }: PageProps) => {

     const { chatId } = params
  const session = await getServerSession(authOptions)
  if (!session) notFound()

  const { user } = session

  const [userId1, userId2] = chatId.split('--')

  if (user.id !== userId1 && user.id !== userId2) {
    notFound()
  }

  const chatPartnerId = user.id === userId1 ? userId2 : userId1


  if (user.id !== userId1 && user.id !== userId2) {
    notFound()
  }

  return(
    ...
  )

}
export default page
```

- action handler

[sorted-comb] https://redis.io/docs/latest/commands/zrange/

```tsx
async function getChatMessages(chatId: string) {
  try {
    const results: string[] = await fetchRedis(
      'zrange',
      `chat:${chatId}:messages`,
      0,
      -1
    )

    const dbMessages = results.map((message) => JSON.parse(message) as Message)

    const reversedDbMessages = dbMessages.reverse()

    const messages = messageArrayValidator.parse(reversedDbMessages)

    return messages
  } catch (error) {
    notFound()
  }
```

#### the model --> friends

```tsx
interface User {
  ...
  id: string
}

interface Chat {
  id: string
  messages: Message[]
}

interface Message {
  id: string
  senderId: string
  receiverId: string
  text: string
  timestamp: number
}

interface FriendRequest {
  id: string
  senderId: string
  receiverId: string
}
```

#### valid sidebar

nextjs-realtime-chat/src/lib/validations
/message.ts

```tsx
import { z } from 'zod'

export const messageValidator = z.object({
  id: z.string(),
  senderId: z.string(),
  text: z.string(),
  timestamp: z.number(),
})

export const messageArrayValidator = z.array(messageValidator)

export type Message = z.infer<typeof messageValidator>
```

[chatid]

```tsx
async function getChatMessages(chatId: string) {
  try {
    const results: string[] = await fetchRedis(
      'zrange',
      `chat:${chatId}:messages`,
      0,
      -1
    )

    const dbMessages = results.map((message) => JSON.parse(message) as Message)

    const reversedDbMessages = dbMessages.reverse()

    const messages = messageArrayValidator.parse(reversedDbMessages)

    return messages
  } catch (error) {
    notFound()
  }
}
```

Layout.tsx

- flat structure get friends also

```tsx
const session = await getServerSession(authOptions)
if (!session) notFound()

const friends = await getFriendsByUserId(session.user.id)
console.log('friends', friends)
```

nextjs-realtime-chat/src/helpers
/get-friends-by-user-id.ts

- promise array of fetches

```tsx
import { fetchRedis } from './redis'

export const getFriendsByUserId = async (userId: string) => {
  // retrieve friends for current user
  console.log('userid', userId)
  const friendIds = (await fetchRedis(
    'smembers',
    `user:${userId}:friends`
  )) as string[]
  console.log('friend ids', friendIds)

  const friends = await Promise.all(
    friendIds.map(async (friendId) => {
      const friend = (await fetchRedis('get', `user:${friendId}`)) as string
      const parsedFriend = JSON.parse(friend) as User
      return parsedFriend
    })
  )

  return friends
}
```

Breadcrumbsnextjs-realtime-chat/src/components
/SidebarChatList.tsx

```tsx

  return (
    <ul role='list' className='max-h-[25rem] overflow-y-auto -mx-2 space-y-1'>
      {activeChats.sort().map((friend) => {

      }

      </us>)})
```

#### logic and presentation

src/lib/utils.ts

```tsx
export function chatHrefConstructor(id1: string, id2: string) {
  const sortedIds = [id1, id2].sort()
  return `${sortedIds[0]}--${sortedIds[1]}`
}
```

```tsx
return(
    ...
         <li>
                  <FriendRequestSidebarOptions
                    sessionId={session.user.id}
                    initialUnseenRequestCount={unseenRequestCount}
                  />
                </li>
              </ul>
)
```

nextjs-realtime-chat/src/components
/SidebarChatList.tsx

- shared sidebar component reflect state in particular page

```tsx
const SidebarChatList: FC<SidebarChatListProps> = ({ friends, sessionId }) => {
  const router = useRouter()
  const pathname = usePathname()
//   const [unseenMessages, setUnseenMessages] = useState<Message[]>([])
//   const [activeChats, setActiveChats] = useState<User[]>(friends)


 return (
          <li key={friend.id}>
            <a
              href={`/dashboard/chat/${chatHrefConstructor(
                sessionId,
                friend.id
              )}`}
              className='text-gray-700 hover:text-indigo-600 hover:bg-gray-50 group flex items-center gap-x-3 rounded-md p-2 text-sm leading-6 font-semibold'>
              {friend.name}
              {unseenMessagesCount > 0 ? (
                <div className='bg-indigo-600 font-medium text-xs text-white w-4 h-4 rounded-full flex justify-center items-center'>
                  {unseenMessagesCount}
                </div>
              ) : null}
            </a>
          </li>
        )
      })}
    </ul>
```

- above

```tsx
 <nav className='flex flex-1 flex-col'>
          <ul role='list' className='flex flex-1 flex-col gap-y-7'>
            <li>
              <SidebarChatList sessionId={session.user.id} friends={friends} />
            </li>
            <li>
              <div className='text-xs font-semibold leading-6 text-gray-400'>
                Overview
              </div>
```

- prepare {unSeenMessagesCount > 0 ? ():null} --> afterwards

```tsx
  return (
    <ul role='list' className='max-h-[25rem] overflow-y-auto -mx-2 space-y-1'>
      {activeChats.sort().map((friend) => {
        const unseenMessagesCount = unseenMessages.filter((unseenMsg) => {
          return unseenMsg.senderId === friend.id
        }).length

```
