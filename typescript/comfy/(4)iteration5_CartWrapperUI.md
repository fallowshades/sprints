## SelectProductAmount

```tsx
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'

export enum Mode {
  SingleProduct = 'singleProduct',
  CartItem = 'cartItem',
}

type SelectProductAmountProps = {
  mode: Mode.SingleProduct
  amount: number
  setAmount: React.Dispatch<React.SetStateAction<number>>
}

type SelectCartItemAmountProps = {
  mode: Mode.CartItem
  amount: number
  setAmount: (value: number) => void
}

function SelectProductAmount({
  mode,
  amount,
  setAmount,
}: SelectProductAmountProps | SelectCartItemAmountProps) {
  const cartItem = mode === Mode.CartItem
  return (
    <>
      <h4 className='font-medium mb-2'>Amount :</h4>
      <Select
        defaultValue={amount.toString()}
        onValueChange={(value) => setAmount(Number(value))}
      >
        <SelectTrigger className={cartItem ? 'w-[75px]' : 'w-[150px]'}>
          <SelectValue placeholder={amount} />
        </SelectTrigger>
        <SelectContent>
          {Array.from({ length: cartItem ? amount + 10 : 10 }, (_, index) => {
            const selectValue = (index + 1).toString()
            return (
              <SelectItem key={index} value={selectValue}>
                {selectValue}
              </SelectItem>
            )
          })}
        </SelectContent>
      </Select>
    </>
  )
}
export default SelectProductAmount
```

SingleProduct.tsx

```tsx
import { SelectProductColor, SelectProductAmount } from '@/components'
import { Mode } from '@/components/SelectProductAmount'
;<SelectProductAmount
  mode={Mode.SingleProduct}
  amount={amount}
  setAmount={setAmount}
/>
```

## Shadcn Toast Component

[Toast](https://ui.shadcn.com/docs/components/toast)

```sh
npx shadcn-ui@latest add toast

```

main.tsx

```tsx
import { Toaster } from '@/components/ui/toaster'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <Provider store={store}>
    <Toaster />
    <App />
  </Provider>
)
```

Component

```tsx
import { useToast } from '@/components/ui/use-toast'

export const ToastDemo = () => {
  const { toast } = useToast()

  return (
    <Button
      onClick={() => {
        toast({
          title: 'Scheduled: Catch up',
          description: 'Friday, February 10, 2023 at 5:57 PM',
        })
      }}
    >
      Show Toast
    </Button>
  )
}
```

loaders/actions, features

```ts
import { toast } from '@/components/ui/use-toast'

toast({ description: 'Item added to cart' })
```

## CartItem and CartState Types

utils/types.ts

```ts
export type CartItem = {
  cartID: string
  productID: number
  image: string
  title: string
  price: string
  amount: number
  productColor: string
  company: string
}

export type CartState = {
  cartItems: CartItem[]
  numItemsInCart: number
  cartTotal: number
  shipping: number
  tax: number
  orderTotal: number
}
```

## CartSlice

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'
import { type CartItem, type CartState } from '@/utils'
import { toast } from '@/components/ui/use-toast'

const defaultState: CartState = {
  cartItems: [],
  numItemsInCart: 0,
  cartTotal: 0,
  shipping: 500,
  tax: 0,
  orderTotal: 0,
}

const getCartFromLocalStorage = (): CartState => {
  const cart = localStorage.getItem('cart')
  return cart ? JSON.parse(cart) : defaultState
}

const cartSlice = createSlice({
  name: 'cart',
  initialState: getCartFromLocalStorage(),
  reducers: {
    addItem: (state, action: PayloadAction<CartItem>) => {
      const newCartItem = action.payload
      const item = state.cartItems.find((i) => i.cartID === newCartItem.cartID)
      if (item) {
        item.amount += newCartItem.amount
      } else {
        state.cartItems.push(newCartItem)
      }
      state.numItemsInCart += newCartItem.amount
      state.cartTotal += Number(newCartItem.price) * newCartItem.amount
      // state.tax = 0.1 * state.cartTotal;
      // state.orderTotal = state.cartTotal + state.shipping + state.tax;
      // localStorage.setItem('cart', JSON.stringify(state));
      cartSlice.caseReducers.calculateTotals(state)
      toast({ description: 'Item added to cart' })
    },
    clearCart: () => {
      localStorage.setItem('cart', JSON.stringify(defaultState))
      return defaultState
    },
    removeItem: (state, action: PayloadAction<string>) => {
      const cartID = action.payload
      const cartItem = state.cartItems.find((i) => i.cartID === cartID)
      if (!cartItem) return
      state.cartItems = state.cartItems.filter((i) => i.cartID !== cartID)
      state.numItemsInCart -= cartItem.amount
      state.cartTotal -= Number(cartItem.price) * cartItem.amount
      cartSlice.caseReducers.calculateTotals(state)
      toast({ description: 'Item removed from the cart' })
    },
    editItem: (
      state,
      action: PayloadAction<{ cartID: string; amount: number }>
    ) => {
      const { cartID, amount } = action.payload
      const cartItem = state.cartItems.find((i) => i.cartID === cartID)
      if (!cartItem) return

      state.numItemsInCart += amount - cartItem.amount
      state.cartTotal += Number(cartItem.price) * (amount - cartItem.amount)
      cartItem.amount = amount
      cartSlice.caseReducers.calculateTotals(state)
      toast({ description: 'Amount updated' })
    },
    calculateTotals: (state) => {
      state.tax = 0.1 * state.cartTotal
      state.orderTotal = state.cartTotal + state.shipping + state.tax
      localStorage.setItem('cart', JSON.stringify(state))
    },
  },
})

export const { addItem, clearCart, removeItem, editItem } = cartSlice.actions
export default cartSlice.reducer
```

## SingleProduct - AddItem

```tsx
import { type CartItem } from '@/utils'
import { useAppDispatch } from '@/hooks'
import { addItem } from '@/features/cart/cartSlice'

const dispatch = useAppDispatch()

const cartProduct: CartItem = {
  cartID: product.id + productColor,
  productID: product.id,
  image,
  title,
  price,
  amount,
  productColor,
  company,
}

const addToCart = () => {
  dispatch(addItem(cartProduct))
}
```

## CartPage - Setup

- create CartTotals, CartItemsList, CartItemColumns
- import/export

Cart.tsx

```tsx
import { useAppSelector } from '@/hooks'
import { CartItemsList, SectionTitle, CartTotals } from '@/components'
import { Link } from 'react-router-dom'
import { Button } from '@/components/ui/button'

const Cart = () => {
  // temp
  const user = null

  const numItemsInCart = useAppSelector(
    (state) => state.cartState.numItemsInCart
  )

  if (numItemsInCart === 0) {
    return <SectionTitle text='Empty cart ☹️' />
  }
  return (
    <>
      <SectionTitle text='Shopping Cart' />
      <div className='mt-8 grid gap-8  lg:grid-cols-12'>
        <div className='lg:col-span-8'>
          <CartItemsList />
        </div>
        <div className='lg:col-span-4 lg:pl-4'>
          <CartTotals />
          {user ? (
            <Button asChild className='mt-8 w-full'>
              <Link to='/checkout'>Proceed to checkout</Link>
            </Button>
          ) : (
            <Button asChild className='mt-8 w-full'>
              <Link to='/login'>Please Login</Link>
            </Button>
          )}
        </div>
      </div>
    </>
  )
}
export default Cart
```

## CartTotals

```tsx
import { useAppSelector } from '@/hooks'
import { formatAsDollars } from '@/utils'
import { Card, CardTitle } from '@/components/ui/card'
import { Separator } from './ui/separator'

const CartTotals = () => {
  const { cartTotal, shipping, tax, orderTotal } = useAppSelector(
    (state) => state.cartState
  )

  return (
    <Card className='p-8 bg-muted'>
      <CartTotalRow label='Subtotal' amount={cartTotal} />
      <CartTotalRow label='Shipping' amount={shipping} />
      <CartTotalRow label='Tax' amount={tax} />
      <CardTitle className='mt-8'>
        <CartTotalRow label='Order Total' amount={orderTotal} lastRow />
      </CardTitle>
    </Card>
  )
}

function CartTotalRow({
  label,
  amount,
  lastRow,
}: {
  label: string
  amount: number
  lastRow?: boolean
}) {
  return (
    <>
      <p className='flex justify-between text-sm'>
        <span>{label}</span>
        <span>{formatAsDollars(amount)}</span>
      </p>
      {lastRow ? null : <Separator className='my-2' />}
    </>
  )
}

export default CartTotals
```

## CartItemsList

```tsx
import { useAppSelector } from '@/hooks'
import { Card } from './ui/card'

import {
  FirstColumn,
  SecondColumn,
  ThirdColumn,
  FourthColumn,
} from './CartItemColumns'
const CartItemsList = () => {
  const cartItems = useAppSelector((state) => state.cartState.cartItems)

  return (
    <div>
      {cartItems.map((cartItem) => {
        const { cartID, title, price, image, amount, company, productColor } =
          cartItem
        return (
          <Card
            key={cartID}
            className='flex flex-col gap-y-4 sm:flex-row flex-wrap p-6 mb-8'
          >
            Cart Item
          </Card>
        )
      })}
    </div>
  )
}

export default CartItemsList
```

## CartItemColumns

```tsx
import { formatAsDollars } from '@/utils'
import { useAppDispatch } from '@/hooks'
import { Button } from './ui/button'
import { editItem, removeItem } from '@/features/cart/cartSlice'
import SelectProductAmount from './SelectProductAmount'
import { Mode } from './SelectProductAmount'
export const ThirdColumn = ({
  amount,
  cartID,
}: {
  amount: number
  cartID: string
}) => {
  const dispatch = useAppDispatch()

  const removeItemFromTheCart = () => {
    dispatch(removeItem(cartID))
  }

  const setAmount = (value: number) => {
    dispatch(editItem({ cartID, amount: value }))
  }

  return (
    <div>
      <SelectProductAmount
        amount={amount}
        setAmount={setAmount}
        mode={Mode.CartItem}
      />
      <Button variant='link' className='-ml-4' onClick={removeItemFromTheCart}>
        remove
      </Button>
    </div>
  )
}

export const FirstColumn = ({
  image,
  title,
}: {
  image: string
  title: string
}) => {
  return (
    <img
      src={image}
      alt={title}
      className='h-24 w-24 rounded-lg sm:h-32 sm:w-32 object-cover'
    />
  )
}

export const SecondColumn = ({
  title,
  company,
  productColor,
}: {
  title: string
  company: string
  productColor: string
}) => {
  return (
    <div className='sm:ml-4 md:ml-12 sm:w-48'>
      <h3 className='capitalize font-medium'>{title}</h3>
      <h4 className='mt-2 capitalize text-sm'>{company}</h4>
      <p className='mt-4 text-sm capitalize flex items-center gap-x-2'>
        color :
        <span
          style={{
            width: '15px',
            height: '15px',
            borderRadius: '50%',
            backgroundColor: productColor,
          }}
        ></span>
      </p>
    </div>
  )
}

export const FourthColumn = ({ price }: { price: string }) => {
  return <p className='font-medium sm:ml-auto'>{formatAsDollars(price)}</p>
}
```

CartItemsList.tsx

```tsx
<FirstColumn image={image} title={title} />
<SecondColumn title={title} company={company} productColor={productColor} />
<ThirdColumn amount={amount} cartID={cartID} />
<FourthColumn price={price} />
```

## UserSlice

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'
import { toast } from '@/components/ui/use-toast'

export type User = {
  username: string
  jwt: string
}

type UserState = {
  user: User | null
}

const getUserFromLocalStorage = (): User | null => {
  const user = localStorage.getItem('user')
  if (!user) return null
  return JSON.parse(user)
}

const initialState: UserState = {
  user: getUserFromLocalStorage(),
}

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    loginUser: (state, action: PayloadAction<User>) => {
      const user = action.payload
      state.user = user
      localStorage.setItem('user', JSON.stringify(user))

      if (user.username === 'demo user') {
        toast({ description: 'Welcome Guest User' })
        return
      }
      toast({ description: 'Login successful' })
    },
    logoutUser: (state) => {
      state.user = null
      // localStorage.clear()
      localStorage.removeItem('user')
    },
  },
})

export const { loginUser, logoutUser } = userSlice.actions

export default userSlice.reducer
```
