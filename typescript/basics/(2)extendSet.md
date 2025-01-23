## Functions - Using Union Types as Function Parameters

### Challenge

Your task is to create a function named processInput that accepts a parameter of a union type string | number. The function should behave as follows:

- If the input is of type number, the function should multiply the number by 2 and log the result to the console.
- If the input is of type string, the function should convert the string to uppercase and log the result to the console.

```ts
function processInput(input: string | number) {
  if (typeof input === 'number') {
    console.log(input * 2)
  } else {
    console.log(input.toUpperCase())
  }
}

processInput(10) // Output: 20
processInput('Hello') // Output: HELLO
```

In this example, the processInput function takes a parameter input that can be either a string or a number. Inside the function, we use a type guard (typeof input === 'number') to check the type of input at runtime. If input is a number, we double it. If input is a string, we convert it to uppercase.

## Functions - Using Objects as Function Parameters

```ts
function createEmployee({ id }: { id: number }): {
  id: number
  isActive: boolean
} {
  return { id, isActive: id % 2 === 0 }
}

const first = createEmployee({ id: 1 })
const second = createEmployee({ id: 2 })
console.log(first, second)

// alternative
function createStudent(student: { id: number; name: string }) {
  console.log(`Welcome to the course ${student.name.toUpperCase()}!!!`)
}

const newStudent = {
  id: 5,
  name: 'anna',
}

createStudent(newStudent)
```

## Gotcha - Excess Property Checks

```ts
function createStudent(student: { id: number; name: string }) {
  console.log(`Welcome to the course ${student.name.toUpperCase()}!!!`)
}

const newStudent = {
  id: 5,
  name: 'anna',
  email: 'anna@gmail.com',
}

createStudent(newStudent)
createStudent({ id: 1, name: 'bob', email: 'bob@gmail.com' })
```

TypeScript only performs excess property checks on object literals where they're used, not on references to them.

In TypeScript, when you pass an object literal (like { id: 1, name: 'bob', email: 'bob@gmail.com' }) directly to a function or assign it to a variable with a specified type, TypeScript checks that the object only contains known properties. This is done to catch common errors.

However, when you pass newStudent to createStudent, TypeScript doesn't complain about the email property. This is because newStudent is not an object literal at the point where it's passed to createStudent.

## Challenge

Your task is to create a function named processData that accepts two parameters:

- The first parameter, input, should be a union type that can be either a string or a number.
- The second parameter, config, should be an object with a reverse property of type boolean, by default it "reverse" should be false

The function should behave as follows:

- If input is of type number, the function should return the square of the number.
- If input is of type string, the function should return the string in uppercase.
- If the reverse property on the config object is true, and input is a string, the function should return the reversed string in uppercase.

```ts
function processData(
  input: string | number,
  config: { reverse: boolean } = { reverse: false }
): string | number {
  if (typeof input === 'number') {
    return input * input
  } else {
    return config.reverse
      ? input.toUpperCase().split('').reverse().join('')
      : input.toUpperCase()
  }
}

console.log(processData(10)) // Output: 100
console.log(processData('Hello')) // Output: HELLO
console.log(processData('Hello', { reverse: true })) // Output: OLLEH
```

## Type Alias

A type alias in TypeScript is a new name or shorthand for an existing type, making it easier to reuse complex types. However, it's important to note that it doesn't create a new unique type - it's just an alias.All the same rules apply to the aliased type, including the ability to mark properties as optional or readonly.

```ts
const john: { id: number; name: string; isActive: boolean } = {
  id: 1,
  name: 'john',
  isActive: true,
}
const susan: { id: number; name: string; isActive: boolean } = {
  id: 1,
  name: 'susan',
  isActive: false,
}

function createUser(user: { id: number; name: string; isActive: boolean }): {
  id: number
  name: string
  isActive: boolean
} {
  console.log(`Hello there ${user.name.toUpperCase()} !!!`)

  return user
}
```

```ts
type User = { id: number; name: string; isActive: boolean }

const john: User = {
  id: 1,
  name: 'john',
  isActive: true,
}
const susan: User = {
  id: 1,
  name: 'susan',
  isActive: false,
}

function createUser(user: User): User {
  console.log(`Hello there ${user.name.toUpperCase()} !!!`)
  return user
}

type StringOrNumber = string | number // Type alias for string | number

let value: StringOrNumber
value = 'hello' // This is valid
value = 123 // This is also valid

type Theme = 'light' | 'dark' // Type alias for theme

let theme: Theme
theme = 'light' // This is valid
theme = 'dark' // This is also valid

// Function that accepts the Theme type alias
function setTheme(t: Theme) {
  theme = t
}

setTheme('dark') // This will set the theme to 'dark'
```

## Challenge

- Define the Employee type: Create a type Employee with properties id (number), name (string), and department (string).

- Define the Manager type: Create a type Manager with properties id (number), name (string), and employees (an array of Employee).

- Create a Union Type: Define a type Staff that is a union of Employee and Manager.

- Create the printStaffDetails function: This function should accept a parameter of type Staff. Inside the function, use a type guard to check if the 'employees' property exists in the passed object. If it does, print a message indicating that the person is a manager and the number of employees they manage. If it doesn't, print a message indicating that the person is an employee and the department they belong to.

- Create Employee and Manager objects: Create two Employee objects. One named alice and second named steve. Also create a Manager object named bob who manages alice and steve.

- Test the function: Call the printStaffDetails function with alice and bob as arguments and verify the output.

```ts
type Employee = { id: number; name: string; department: string }
type Manager = { id: number; name: string; employees: Employee[] }

type Staff = Employee | Manager

function printStaffDetails(staff: Staff) {
  if ('employees' in staff) {
    console.log(
      `${staff.name} is a manager of ${staff.employees.length} employees.`
    )
  } else {
    console.log(
      `${staff.name} is an employee in the ${staff.department} department.`
    )
  }
}

const alice: Employee = { id: 1, name: 'Alice', department: 'Sales' }
const steve: Employee = { id: 1, name: 'Steve', department: 'HR' }
const bob: Manager = { id: 2, name: 'Bob', employees: [alice, steve] }

printStaffDetails(alice) // Outputs: Alice is an employee in the Sales department.
printStaffDetails(bob)
```

## Intersection Types

In TypeScript, an intersection type (TypeA & TypeB) is a way of combining multiple types into one. This means that an object of an intersection type will have all the properties of TypeA and all the properties of TypeB. It's a way of creating a new type that merges the properties of existing types.

```ts
type Book = { id: number; name: string; price: number }
type DiscountedBook = Book & { discount: number }
const book1: Book = {
  id: 2,
  name: 'How to Cook a Dragon',
  price: 15,
}

const book2: Book = {
  id: 3,
  name: 'The Secret Life of Unicorns',
  price: 18,
}

const discountedBook: DiscountedBook = {
  id: 4,
  name: 'Gnomes vs. Goblins: The Ultimate Guide',
  price: 25,
  discount: 0.15,
}
```

## Type Alias - Computed Properties

Computed properties in JavaScript are a feature that allows you to dynamically create property keys on objects. This is done by wrapping an expression in square brackets [] that computes the property name when creating an object.

```ts
const propName = 'age'

type Animal = {
  [propName]: number
}

let tiger: Animal = { [propName]: 5 }
```
