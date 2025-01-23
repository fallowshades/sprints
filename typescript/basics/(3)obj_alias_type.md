## Interface - Methods

```ts
interface Book {
  readonly isbn: number
  title: string
  author: string
  genre?: string
  // method
  printAuthor(): void
  printTitle(message: string): string
}

const deepWork: Book = {
  isbn: 9781455586691,
  title: 'Deep Work',
  author: 'Cal Newport',
  genre: 'Self-help',
  printAuthor() {
    console.log(this.author)
  },
  printTitle(message) {
    return `${this.title} ${message}`
  },
}
deepWork.printAuthor()
const result = deepWork.printTitle('is an awesome book')
console.log(result)
```

## Interface - Methods (more options)

It's generally a good practice to match the structure of the interface and the implementing object or class as closely as possible. This makes the code easier to understand and maintain. So, if printAuthor is defined as a method in the Book interface, it would be more consistent to implement it as a method in the deepWork object.

```ts
interface Book {
  readonly isbn: number
  title: string
  author: string
  genre?: string
  // method
  printAuthor(): void
  printTitle(message: string): string
  printSomething: (someValue: number) => number
}

const deepWork: Book = {
  isbn: 9781455586691,
  title: 'Deep Work',
  author: 'Cal Newport',
  genre: 'Self-help',
  printAuthor() {
    console.log(this.author)
  },
  printTitle(message) {
    return `${this.title} ${message}`
  },
  // first option
  // printSomething: function (someValue) {
  //   return someValue;
  // },
  // second option
  printSomething: (someValue) => {
    // "this" gotcha
    console.log(deepWork.author)
    return someValue
  },
  // third option
  // printSomething(someValue) {
  //   return someValue;
  // },
  // alternative
  // printAuthor: () => {
  //   console.log(deepWork.author);
  // },
}
console.log(deepWork.printSomething(34))

deepWork.printAuthor()
const result = deepWork.printTitle('is an awesome book')
console.log(result)
```

## Challenge

- Start by defining an interface Computer using the interface keyword. This will serve as a blueprint for objects that will be of this type.
- Inside the interface, define the properties that the object should have. In this case, we have id, brand, ram, and storage.
- Use the readonly keyword before the id property to indicate that it cannot be changed once it's set.
- Use the ? after the storage property to indicate that this property is optional and may not exist on all objects of this type.
- Also inside the interface, define any methods that the object should have. In this case, we have upgradeRam, which is a function that takes a number and returns a number.
- Now that we have our interface, we can create an object that adheres to this interface. This object should have all the properties defined in the interface (except for optional ones, which are... optional), and the methods should be implemented.
- Finally, we can use our object. We can call its upgradeRam method to increase its RAM.

```ts
interface Computer {
  readonly id: number // cannot be changed once initialized
  brand: string
  ram: number
  upgradeRam(increase: number): number
  storage?: number // optional property
}

const laptop: Computer = {
  id: 1,
  brand: 'random brand',
  ram: 8, // in GB
  upgradeRam(amount: number) {
    this.ram += amount
    return this.ram
  },
}

laptop.storage = 256 // assigning value to optional property

console.log(laptop.upgradeRam(4)) // upgrades RAM by 4GB
console.log(laptop)
```

## Interface - Merging, Extend, TypeGuard

```ts
interface Person {
  name: string
  getDetails(): string
}

interface DogOwner {
  dogName: string
  getDogDetails(): string
}

// Merging (reopening) an interface in TypeScript is a process where you declare an interface with the same name more than once, and TypeScript will merge their members.

// Merging the interface
interface Person {
  age: number
}

// Usage
const person: Person = {
  name: 'John',
  age: 30,
  getDetails() {
    return `Name: ${this.name}, Age: ${this.age}`
  },
}

// Extending an interface in TypeScript is a way to create a new interface that inherits the properties and methods of an existing interface. You use the extends keyword to do this. When you extend an interface, the new interface will have all the members of the base interface, plus any new members that you add.

// Extending the interface
interface Employee extends Person {
  employeeId: number
}

const employee: Employee = {
  name: 'jane',
  age: 28,
  employeeId: 123,
  getDetails() {
    return `Name: ${this.name}, Age: ${this.age}, Employee ID: ${this.employeeId}`
  },
}

// Interface multiple inheritance
interface Manager extends Person, DogOwner {
  managePeople(): void
}

const manager: Manager = {
  name: 'Bob',
  age: 35,
  dogName: 'Rex',
  getDetails() {
    return `Name: ${this.name}, Age: ${this.age}`
  },
  getDogDetails() {
    return `Dog Name: ${this.dogName}`
  },
  managePeople() {
    console.log('Managing people...')
  },
}
```

## Challenge - Part 1

- Define the Person interface Start by defining a Person interface with a name property of type string.
- Define the DogOwner interface Next, define a DogOwner interface that extends Person and adds a dogName property of type string.
- Define the Manager interface Then, define a Manager interface that extends Person and adds two methods: managePeople and delegateTasks. Both methods should have a return type of void.
- Define the getEmployee function Now, define a function called getEmployee that returns a Person, DogOwner, or Manager. Inside this function, generate a random number and use it to decide which type of object to return. If the number is less than 0.33, return a Person. If it's less than 0.66, return a DogOwner. Otherwise, return a Manager.
- Finally, create a variable called employee that can be a Person, DogOwner, or Manager, and assign it the return value of getEmployee. Then, log employee to the console.

```ts
interface Person {
  name: string
}

interface DogOwner extends Person {
  dogName: string
}

interface Manager extends Person {
  managePeople(): void
  delegateTasks(): void
}

const employee: Person | DogOwner | Manager = getEmployee()
console.log(employee)

function getEmployee(): Person | DogOwner | Manager {
  const random = Math.random()

  if (random < 0.33) {
    return {
      name: 'john',
    }
  } else if (random < 0.66) {
    return {
      name: 'sarah',
      dogName: 'Rex',
    }
  } else {
    return {
      name: 'bob',
      managePeople: () => console.log('Managing people...'),
      delegateTasks: () => console.log('Delegating tasks...'),
    }
  }
}
```

## Challenge - Part 2

A type predicate in TypeScript is a special kind of return type for a function that not only returns a boolean, but also asserts that the argument is of a specific type if the function returns true. It's typically used in user-defined type guard functions to narrow down the type of a variable within a certain scope. The syntax is arg is Type, where arg is the function argument and Type is the type you're checking for.

- Define the isManager function Define a function called isManager that takes an object of type Person | DogOwner | Manager and returns a boolean. This function should check if the managePeople method exists on the object, and return true if it does and false if it doesn't. The return type of this function should be a type predicate: obj is Manager.
- Run your code to see if it works as expected. If employee is a Manager, you should see the output of the delegateTasks method in the console. If employee is a Person or DogOwner, there should be no output.

```ts
// function isManager(obj: Person | DogOwner | Manager): boolean {
//   return 'managePeople' in obj;
// }

function isManager(obj: Person | DogOwner | Manager): obj is Manager {
  return 'managePeople' in obj
}

if (isManager(employee)) {
  employee.delegateTasks()
}
```

## Interface vs Type Alias

A type alias is a way to give a name to a type. It can represent primitive types, union types, intersection types, tuples, and any other types. Once defined, the alias can be used anywhere in place of the actual type.

```ts
type Person = {
  name: string
  age: number
}

let john: Person = { name: 'John', age: 30 }
```

Interface

An interface is a way to define a contract for a certain structure of an object.

```ts
interface Person {
  name: string
  age: number
}

let john: Person = { name: 'John', age: 30 }
```

Key Differences

- Type aliases can represent primitive types, union types, intersection types, tuples, etc., while interfaces are primarily used to represent the shape of an object.

```ts
// Type alias for a primitive type
type Score = number
type NumberOrString = number | string
// Type alias for literal types
type Direction = 'up' | 'down' | 'left' | 'right'

// Using the type aliases
let gameScore: Score = 100
let move: Direction = 'up'
```

- Interfaces can be merged using declaration merging. If you define an interface with the same name more than once, TypeScript will merge their definitions. Type aliases can't be merged in this way.

- Interfaces can be implemented by classes, while type aliases cannot.
- Type aliases can use computed properties, while interfaces cannot.

```ts
interface Person {
  name: string
  greet(): void
}

class Employee implements Person {
  name: string

  constructor(name: string) {
    this.name = name
  }

  greet() {
    console.log(`Hello, my name is ${this.name}`)
  }
}

let john = new Employee('John')
john.greet() // Outputs: Hello, my name is John
```

```ts
const propName = 'age'

type Animal = {
  [propName]: number
}

let tiger: Animal = { [propName]: 5 }
```
