## Modules - Global Scope "Gotcha"

If your TypeScript files aren't modules (i.e., they don't have any import or export statements), they're treated as scripts in the global scope. In this case, declaring the same variable in two different files would cause a conflict.

tutorial.ts

```ts
let name = 'shakeAdnBake'

const susan = 'susan'

export let something = 'something'
```

actions.ts

```ts
const susan = 'susan'

export const something = 'something'
```

tsconfig.json

```json
"moduleDetection": "force",
```

- output

tsconfig.json

```json
"module": "ESNext",
```

## Modules - Imports/Exports (including types)

```ts
export function sayHello(name: string): void {
  console.log(`Hello ${name}!`)
}

export let person = 'susan'

export type Student = {
  name: string
  age: number
}

const newStudent: Student = {
  name: 'peter',
  age: 24,
}

export default newStudent
```

```ts
import newStudent, { sayHello, person, type Student } from './actions'

sayHello('TypeScript')
console.log(person)
console.log(newStudent)

const anotherStudent: Student = {
  name: 'bob',
  age: 23,
}

console.log(anotherStudent)
```

## Modules - Javascript Files

When you set "allowJs": true in your tsconfig.json, TypeScript will process JavaScript files and can infer types to a certain extent based on the structure and usage of your JavaScript code.

However, TypeScript's ability to infer types from JavaScript is not as robust as when working with TypeScript files. For example, it might not be able to infer complex types or types that depend on runtime behavior.

- create example.js
- export someValue, import in tutorial

```ts
  "allowJs": true,
```

## Type Guarding

Type guarding is a term in TypeScript that refers to the ability to narrow down the type of an object within a certain scope. This is usually done using conditional statements that check the type of an object.

In the context of TypeScript, a type guard is some expression that performs a runtime check that guarantees the type in some scope.

## Challenge - "typeof" guard

- starter code

```ts
type ValueType = string | number | boolean

let value: ValueType
const random = Math.random()
value = random < 0.33 ? 'Hello' : random < 0.66 ? 123.456 : true
```

- Define the function checkValue that takes one parameter value of type ValueType.
- Inside the function, use an if statement to check if value is of type string. If it is, log value converted to lowercase and then return from the function.
- If value is not a string, use another if statement to check if value is of type number. If it is, log value formatted to two decimal places and then return from the function.
- If value is neither a string nor a number, it must be a boolean. Log the string "boolean: " followed by the boolean value.
- Finally, call the checkValue function with value as the argument.

```ts
function checkValue(value: ValueType) {
  if (typeof value === 'string') {
    console.log(value.toLowerCase())
    return
  }
  if (typeof value === 'number') {
    console.log(value.toFixed(2))
    return
  }
  console.log(`boolean: ${value}`)
}

checkValue(value)
```

## Challenge - Equality Narrowing

In TypeScript, equality narrowing is a form of type narrowing that occurs when you use equality checks like === or !== in your code

- starter code

```ts
type Dog = { type: 'dog'; name: string; bark: () => void }
type Cat = { type: 'cat'; name: string; meow: () => void }
type Animal = Dog | Cat
```

- Define a function named makeSound that takes one parameter animal of type Animal.
- Inside the function, use an if statement to check if animal.type is 'dog'.
- If animal.type is 'dog', TypeScript knows that animal is a Dog in this block. In this case, call the bark method of animal.
- If animal.type is not 'dog', TypeScript knows that animal is a Cat in the else block. In this case, call the meow method of animal.
- Now you can call the makeSound function with an Animal as the argument. The function will call the appropriate method (bark or meow) depending on the type of the animal.

```ts
function makeSound(animal: Animal) {
  if (animal.type === 'dog') {
    // TypeScript knows that `animal` is a Dog in this block
    animal.bark()
  } else {
    // TypeScript knows that `animal` is a Cat in this block
    animal.meow()
  }
}
```

## Challenge - check for property

The "in" operator in TypeScript is used to narrow down the type of a variable when used in a conditional statement.It checks if a property or method exists on an object. If it exists, TypeScript will narrow the type to the one that has this property.

- starter code

```ts
type Dog = { type: 'dog'; name: string; bark: () => void }
type Cat = { type: 'cat'; name: string; meow: () => void }
type Animal = Dog | Cat
```

- Define a function named makeSound that takes one parameter animal of type Animal.
- Inside the function, use an if statement with the in operator to check if the bark method exists on the animal object.
- If the bark method exists on animal, TypeScript knows that animal is a Dog in this block. In this case, call the bark method of animal.
- If the bark method does not exist on animal, TypeScript knows that animal is a Cat in the else block. In this case, call the meow method of animal.
- Now you can call the makeSound function with an Animal as the argument. The function will call the appropriate method (bark or meow) depending on the type of the animal.

```ts
function makeSound(animal: Animal) {
  if ('bark' in animal) {
    // TypeScript knows that `animal` is a Dog in this block
    animal.bark()
  } else {
    // TypeScript knows that `animal` is a Cat in this block
    animal.meow()
  }
}
```

## Challenge - "Truthy"/"Falsy" guard

In TypeScript, "Truthy"/"Falsy" guard is a simple check for a truthy or falsy value

- Define a function named printLength that takes one parameter str which can be of type string, null, or undefined.
- Inside the function, use an if statement to check if str is truthy. In JavaScript and TypeScript, a truthy value is a value that is considered true when encountered in a Boolean context. All values are truthy unless they are defined as falsy (i.e., except for false, 0, -0, 0n, "", null, undefined, and NaN).
- If str is truthy, it means it's a string (since null and undefined are falsy). In this case, log the length of str using the length property of the string.
- If str is not truthy (i.e., it's either null or undefined), log the string 'No string provided'.

- Now you can call the printLength function with a string, null, or undefined as the argument. The function will print the length of the string if a string is provided, or 'No string provided' otherwise.

```ts
function printLength(str: string | null | undefined) {
  if (str) {
    // In this block, TypeScript knows that `str` is a string
    // because `null` and `undefined` are falsy values.
    console.log(str.length)
  } else {
    console.log('No string provided')
  }
}

printLength('Hello') // Outputs: 5
printLength(null) // Outputs: No string provided
printLength(undefined) // Outputs: No string provided
```

## Challenge - "instanceof" type guard

The instanceof type guard is a way in TypeScript to check the specific class or constructor function of an object at runtime. It returns true if the object is an instance of the class or created by the constructor function, and false otherwise.

```ts
try {
  // Some code that may throw an error
  throw new Error('This is an error')
} catch (error) {
  if (error instanceof Error) {
    console.log('Caught an Error object: ' + error.message)
  } else {
    console.log('Caught an unknown error')
  }
}
```

- Start by defining the function using the function keyword followed by the function name, in this case checkInput.
- Define the function's parameter. The function takes one parameter, input, which can be of type Date or string. This is denoted by input: Date | string.
- Inside the function, use an if statement to check if the input is an instance of Date. This is done using the instanceof operator.
- If the input is an instance of Date, return the year part of the date as a string. This is done by calling the getFullYear method on the input and then converting it to a string using the toString method.
- If the input is not an instance of Date (which means it must be a string), return the input as it is.
- After defining the function, you can use it by calling it with either a Date or a string as the argument. The function will return the year part of the date if a Date is passed, or the original string if a string is passed.
- You can store the return value of the function in a variable and then log it to the console to see the result.

```ts
function checkInput(input: Date | string): string {
  if (input instanceof Date) {
    return input.getFullYear().toString()
  }
  return input
}

const year = checkInput(new Date())
const random = checkInput('2020-05-05')
console.log(year)
console.log(random)
```

## Challenge - Type Predicate

A type predicate is a function whose return type is a special kind of type that can be used to narrow down types within conditional blocks.

- starter code

```ts
type Student = {
  name: string
  study: () => void
}

type User = {
  name: string
  login: () => void
}

type Person = Student | User

const randomPerson = (): Person => {
  return Math.random() > 0.5
    ? { name: 'john', study: () => console.log('Studying') }
    : { name: 'mary', login: () => console.log('Logging in') }
}

const person = randomPerson()
```

- Define the Person and Student types. Student should have a study method and Person should have a login method.
- Create a function named isStudent that takes a parameter person of type Person.
- In the function signature, specify a return type that is a type predicate: person is Student.
- In the function body, use a type assertion to treat person as a Student, and check if the study - method is not undefined. This will return true if person is a Student, and false otherwise.
- Use the isStudent function in an if statement with person as the argument.
- In the if block (where isStudent(person) is true), call the study method on person. TypeScript knows that person is a Student in this block, so this is safe.
- In the else block (where isStudent(person) is false), call the login method on person. This is safe because if person is not a Student, it must be a Person, and all Person objects have a login method.

```ts
function isStudent(person: Person): person is Student {
  // return 'study' in person;
  return (person as Student).study !== undefined
}

// Usage

if (isStudent(person)) {
  person.study() // This is safe because TypeScript knows that 'person' is a Student.
} else {
  person.login()
}
```

## Optional - type "never" gotcha

```ts
type Student = {
  name: string
  study: () => void
}

type User = {
  name: string
  login: () => void
}

type Person = Student | User

const person: Person = {
  name: 'anna',
  study: () => console.log('Studying'),
  // login: () => console.log('Logging in'),
}
// person;
function isStudent(person: Person): person is Student {
  // return 'study' in person;
  return (person as Student).study !== undefined
}

// Usage

if (isStudent(person)) {
  person.study() // This is safe because TypeScript knows that 'person' is a Student.
} else {
  // in this case person is type "never"
  console.log(person)
}
```

## Challenge - Discriminated Unions and exhaustive check using the never type

A discriminated union in TypeScript is a type that can be one of several different types, each identified by a unique literal property (the discriminator), allowing for type-safe handling of each possible variant.

- starter code

```ts
type IncrementAction = {
  amount: number
  timestamp: number
  user: string
}

type DecrementAction = {
  amount: number
  timestamp: number
  user: string
}

type Action = IncrementAction | DecrementAction
```

- Write a reducer function that takes the current state and an action, and returns the new state. The reducer function should use a switch statement or if-else chain on the type property of the action to handle each action type differently.

- In the default case of the switch statement or the final else clause, perform an exhaustive check by assigning the action to a variable of type never. If there are any action types that haven't been handled, TypeScript will give a compile error.

- Implement the logic for each action type in the reducer function. This typically involves returning a new state based on the current state and the properties of the action.

- Use the reducer function in your application to handle actions and update the state.

```ts
type IncrementAction = {
  type: 'increment'
  amount: number
  timestamp: number
  user: string
}

type DecrementAction = {
  type: 'decrement'
  amount: number
  timestamp: number
  user: string
}

type Action = IncrementAction | DecrementAction

function reducer(state: number, action: Action): number {
  switch (action.type) {
    case 'increment':
      return state + action.amount
    case 'decrement':
      return state - action.amount

    default:
      const unexpectedAction: never = action
      throw new Error(`Unexpected action: ${unexpectedAction}`)
  }
}

const newState = reducer(15, {
  user: 'john',
  type: 'increment',
  amount: 5,
  timestamp: 123456,
})
```

## Generics - Fundamentals

Generics in TypeScript are a way to create reusable code components that work with a variety of types as opposed to a single one.

In other words, generics allow you to write a function or a class that can work with any data type. You can think of generics as a kind of variable for types.

```ts
// In TypeScript, you can declare an array using two syntaxes:

// let array1: string[] = ['Apple', 'Banana', 'Mango'];
// let array2: number[] = [1, 2, 3];
// let array3: boolean[] = [true, false, true];

let array1: Array<string> = ['Apple', 'Banana', 'Mango']
let array2: Array<number> = [1, 2, 3]
let array3: Array<boolean> = [true, false, true]
```

## Generics - Create Generic Function and Interface

```ts
//
function createString(arg: string): string {
  return arg
}
function createNumber(arg: number): number {
  return arg
}

// Define a generic function
function genericFunction<T>(arg: T): T {
  return arg
}

const someStringValue = genericFunction<string>('Hello World')
const someNumberValue = genericFunction<number>(2)

// Define a generic interface
interface GenericInterface<T> {
  value: T
  getValue: () => T
}

const genericString: GenericInterface<string> = {
  value: 'Hello World',
  getValue() {
    return this.value
  },
}
```

## Generics - Promise Example

```ts
// A Promise in JavaScript (and thus TypeScript) is an object representing the eventual completion or failure of an asynchronous operation.

async function someFunc(): Promise<string> {
  return 'Hello World'
}

const result = someFunc()
```

## Generics - Generate Array

```ts
// generate an array of strings
function generateStringArray(length: number, value: string): string[] {
  let result: string[] = []
  result = Array(length).fill(value)
  return result
}

console.log(generateStringArray(3, 'hello'))

function createArray<T>(length: number, value: T): Array<T> {
  let result: T[] = []
  result = Array(length).fill(value)
  return result
}

let arrayStrings = createArray<string>(3, 'hello') // ["hello", "hello", "hello"]
let arrayNumbers = createArray<number>(4, 100) // [100, 100, 100, 100]

console.log(arrayStrings)
console.log(arrayNumbers)
```

## Generics - Part 5

```ts
function pair<T, U>(param1: T, param2: U): [T, U] {
  return [param1, param2]
}

// Usage
let result = pair<number, string>(123, 'Hello')
console.log(result) // Output: [123, "Hello"]
```

## Generics - Inferred Type and Type Constraints

```ts
function pair<T, U>(param1: T, param2: U): [T, U] {
  return [param1, param2]
}

// Usage
let result = pair(123, 'Hello')

//  const [name,setName] = useState('')
//  const [products,setProducts] = useState<Product[]>([])

// type constraint on the generic type T, generic type can be either a number or a string.

function processValue<T extends number | string>(value: T): T {
  console.log(value)
}

processValue('hello')
processValue(12)
processValue(true)
```

## Generics - Type Constraints 2

```ts
type Car = {
  brand: string
  model: string
}

const car: Car = {
  brand: 'ford',
  model: 'mustang',
}

type Product = {
  name: string
  price: number
}

const product: Product = {
  name: 'shoes',
  price: 1.99,
}

type Student = {
  name: string
  age: number
}

const student: Student = {
  name: 'peter',
  age: 20,
}

// T extends Student is a type constraint on the generic type T. It means that the type T can be any type, but it must be a subtype of Student or Student itself. In other words, T must have at least the same properties and methods that Student has.

// function printName<T extends Student>(input: T): void {
//   console.log(input.name);
// }

// printName(student);

// function printName<T extends Student | Product>(input: T): void {
//   console.log(input.name);
// }

// printName(product);

// The extends { name: string } part is a type constraint on T. It means that T can be any type, but it must be an object that has at least a name property of type string.
// In other words, T must have at least the same properties and methods that { name: string } has.
function printName<T extends { name: string }>(input: T): void {
  console.log(input.name)
}

printName(student)
printName(product)
```

## Generics - Default Value

```ts
interface StoreData<T = any> {
  data: T[]
}

const storeNumbers: StoreData<number> = {
  data: [1, 2, 3, 4],
}

const randomStuff: StoreData = {
  data: ['random', 1],
}
```

```ts
// data is located in the data property

const { data } = axios.get(someUrl)

axios.get<{ name: string }[]>(someUrl)

export class Axios {
  get<T = any, R = AxiosResponse<T>, D = any>(
    url: string,
    config?: AxiosRequestConfig<D>
  ): Promise<R>
}

export interface AxiosResponse<T = any, D = any> {
  data: T
  status: number
  statusText: string
  headers: RawAxiosResponseHeaders | AxiosResponseHeaders
  config: InternalAxiosRequestConfig<D>
  request?: any
}
```

## Fetch Data

- typically axios and React Query (or both 🚀🚀🚀)
- we won't set any state values

```ts
const url = 'https://www.course-api.com/react-tours-project'

async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    // Check if the request was successful.
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()
    return data
  } catch (error) {
    const errMsg =
      error instanceof Error ? error.message : 'there was an error...'
    console.error(errMsg)
    // throw error;
    return []
  }
}

const tours = await fetchData(url)
tours.map((tour: any) => {
  console.log(tour.name)
})

// return empty array
// throw error in catch block
// we are not setting state values in this function
```
