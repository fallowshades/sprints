## Tuples

In TypeScript, a Tuple is a special type that allows you to create an array where the type of a fixed number of elements is known, but need not be the same - in other words it's an array with fixed length and ordered with fixed types. This is useful when you want to group different types of values together.

Tuples are useful when you want to return multiple values from a function.

By default, tuples in TypeScript are not read-only. This means you can modify the values of the elements in the tuple.However, TypeScript does provide a way to make tuples read-only using the readonly keyword.

```ts
let person: [string, number] = ['john', 25]
console.log(person[0]) // Outputs: john
console.log(person[1]) // Outputs: 25

let john: [string, number?] = ['john']

function getPerson(): [string, number] {
  return ['john', 25]
}

let randomPerson = getPerson()
console.log(randomPerson[0]) // Outputs: john
console.log(randomPerson[1])

// let susan: [string, number] = ['susan', 25];
// susan[0] = 'bob';
// susan.push('some random value');

let susan: readonly [string, number] = ['susan', 25]
// susan[0] = 'bob';
// susan.push('some random value');
console.log(susan)
```

## Enums

Enums in TypeScript allow us to define a set of named constants. Using enums can make it easier to document intent, or create a set of distinct cases.

```ts
enum ServerResponseStatus {
  Success = 200,
  Error = 'Error',
}

interface ServerResponse {
  result: ServerResponseStatus
  data: string[]
}

function getServerResponse(): ServerResponse {
  return {
    result: ServerResponseStatus.Success,
    data: ['first item', 'second item'],
  }
}

const response: ServerResponse = getServerResponse()
console.log(response)
```

## Enums - Gotcha : Reverse Mapping

In a numeric enum, TypeScript creates a reverse mapping from the numeric values to the enum member names. This means that if you assign a numeric value to an enum member, you can use that numeric value anywhere the enum type is expected.

In a string enum, TypeScript does not create a reverse mapping. This means that if you assign a string value to an enum member, you cannot use that string value anywhere the enum type is expected. You must use the enum member itself.

```ts
enum ServerResponseStatus {
  Success = 'Success',
  Error = 'Error',
}

Object.values(ServerResponseStatus).forEach((value) => {
  console.log(value)
})
```

```ts
enum ServerResponseStatus {
  Success = 200,
  Error = 500,
}

Object.values(ServerResponseStatus).forEach((value) => {
  if (typeof value === 'number') {
    console.log(value)
  }
})
```

```ts
enum NumericEnum {
  Member = 1,
}

enum StringEnum {
  Member = 'Value',
}

let numericEnumValue: NumericEnum = 1 // This is allowed
console.log(numericEnumValue) // 1

let stringEnumValue: StringEnum = 'Value' // This is not allowed
```

```ts
enum ServerResponseStatus {
  Success = 'Success',
  Error = 'Error',
}

function getServerResponse(): ServerResponse {
  return {
    // result: ServerResponseStatus.Success,
    // this will not fly with string enum but ok with number
    result: 'Success',
    data: ['first item', 'second item'],
  }
}
```

## Challenge

- Define an enum named UserRole with members Admin, Manager, and Employee.
- Define a type alias named User with properties id (number), name (string), role (UserRole), and contact (a tuple with two elements: email as string and phone as string).
- Define a function named createUser that takes a User object as its parameter and returns a User object.
- Call the createUser function with an object that matches the User type, store the result in a variable, and log the variable to the console.

```ts
// Define an enum named UserRole
enum UserRole {
  Admin,
  Manager,
  Employee,
}

// Define a type alias named User
type User = {
  id: number
  name: string
  role: UserRole
  contact: [string, string] // Tuple: [email, phone]
}

// Define a function named createUser
function createUser(user: User): User {
  return user
}

// Call the createUser function
const user: User = createUser({
  id: 1,
  name: 'John Doe',
  role: UserRole.Admin,
  contact: ['john.doe@example.com', '123-456-7890'],
})

console.log(user)
```

## Type Assertion

Type assertion in TypeScript is a way to tell the compiler what the type of an existing variable is. This is especially useful when you know more about the type of a variable than TypeScript does.

```ts
let someValue: any = 'This is a string'

// Using type assertion to treat 'someValue' as a string
let strLength: number = (someValue as string).length

type Bird = {
  name: string
}

// Assume we have a JSON string from an API or local file
let birdString = '{"name": "Eagle"}'
let dogString = '{"breed": "Poodle"}'

//

// Parse the JSON string into an object
let birdObject = JSON.parse(birdString)
let dogObject = JSON.parse(dogString)

// We're sure that the jsonObject is actually a Bird
let bird = birdObject as Bird
let dog = dogObject as Bird

console.log(bird.name)
console.log(dog.name)

enum Status {
  Pending = 'pending',
  Declined = 'declined',
}

type User = {
  name: string
  status: Status
}
// save Status.Pending in the DB as a string
// retrieve string from the DB
const statusValue = 'pending'

const user: User = { name: 'john', status: statusValue as Status }
```

## Type - 'unknown'

The unknown type in TypeScript is a type-safe counterpart of the any type. It's like saying that a variable could be anything, but we need to perform some type-checking before we can use it.

"error instanceof Error" checks if the error object is an instance of the Error class.

```ts
let unknownValue: unknown

// Assign different types of values to unknownValue
unknownValue = 'Hello World' // OK
unknownValue = [1, 2, 3] // OK
unknownValue = 42.3344556 // OK

// unknownValue.toFixed( ); // Error: Object is of type 'unknown'

// Now, let's try to use unknownValue
if (typeof unknownValue === 'number') {
  // TypeScript knows that unknownValue is a string in this block
  console.log(unknownValue.toFixed(2)) // OK
}

function runSomeCode() {
  const random = Math.random()
  if (random < 0.5) {
    throw new Error('Something went wrong')
  } else {
    throw 'some error'
  }
}

try {
  runSomeCode()
} catch (error) {
  if (error instanceof Error) {
    console.log(error.message)
  } else {
    console.log(error)
    console.log('there was an error....')
  }
}
```

## Type - "never"

In TypeScript, never is a type that represents the type of values that never occur.you can't assign any value to a variable of type never.
TypeScript will give a compile error if there are any unhandled cases, helping ensure that all cases are handled.

```ts
// let someValue: never = 0;

type Theme = 'light' | 'dark'

function checkTheme(theme: Theme) {
  if (theme === 'light') {
    console.log('light theme')
    return
  }
  if (theme === 'dark') {
    console.log('dark theme')
    return
  }
  theme
  // theme is of type never, because it can never have a value that is not 'light' or 'dark'.
}
```

```ts
enum Color {
  Red,
  Blue,
  // Green,
}

function getColorName(color: Color) {
  switch (color) {
    case Color.Red:
      return 'Red'
    case Color.Blue:
      return 'Blue'
    default:
      // at build time
      let unexpectedColor: never = color
      // at runtime
      throw new Error(`Unexpected color value: ${unexpectedColor}`)
  }
}

console.log(getColorName(Color.Red)) // Red
console.log(getColorName(Color.Blue)) // Blue
// console.log(getColorName(Color.Green)); // Green
```
