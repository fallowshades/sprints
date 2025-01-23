his project stands as an in-depth guide to TypeScript, meticulously covering its fundamental basics and progressing to its more advanced concepts. It starts with basic setup instructions for creating a TypeScript project using Vite and progresses through a series of tutorials covering various TypeScript features and best practices. Key topics include type annotations, type inference, practical applications of type annotation, union types, handling of "any", "unknown", and "never" types, arrays, and objects fundamentals, challenges to reinforce learning, and functions with their complexities.

The project also delves into advanced TypeScript features such as generics, fetching data with TypeScript, working with the Zod library for data validation, understanding TypeScript declaration files, and class-based programming with TypeScript. Each tutorial is designed to provide hands-on experience with TypeScript, helping learners understand how to apply TypeScript features in real-world scenarios effectively.

Overall, the project is an in-depth TypeScript learning resource, ideal for developers who wish to gain a thorough understanding of TypeScript, from basic to advanced levels, through practical examples and challenges.

## Install

```sh
npm create vite@latest typescript -- --template vanilla-ts
```

## Setup

- create src/tutorial.ts
- import tutorial in src/main.ts

```ts
import './tutorial.ts'
```

- write code in tutorial

- create README.md
- copy from final

## Type Annotations

TypeScript Type Annotations allow developers to specify the types of variables, function parameters, return types, and object properties.

```ts
let awesomeName: string = 'shakeAndBake'
awesomeName = 'something'
awesomeName = awesomeName.toUpperCase()
// awesomeName = 20;

console.log(awesomeName)

let amount: number = 12
amount = 12 - 1
// amount = 'pants';

let isAwesome: boolean = true
isAwesome = false
// isAwesome = 'shakeAndBake';
```

## Type Inference

The typescript compiler can infer the type of the variable based on the literal value that is assigned when it is defined. Just make sure you are working in the typescript file 😄

```ts
let awesomeName = 'shakeAndBake'
awesomeName = 'something'
awesomeName = awesomeName.toUpperCase()
// awesomeName = 20;
```

## Challenge

- Create a variable of type string and try to invoke a string method on it.
- Create a variable of type number and try to perform a mathematical operation on it.
- Create a variable of type boolean and try to perform a logical operation on it.
- Try to assign a value of a different type to each of these variables and observe the TypeScript compiler's response.
- You can use type annotation or inference

```ts
// 1. String
let greeting: string = 'Hello, TypeScript!'
greeting = greeting.toUpperCase() // This should work fine

// 2. Number
let age: number = 25
age = age + 5 // This should work fine

// 3. Boolean
let isAdult: boolean = age >= 18
isAdult = !isAdult // This should work fine

// 4. Assigning different types
// Uncommenting any of these will result in a TypeScript error
// greeting = 10; // Error: Type 'number' is not assignable to type 'string'
// age = 'thirty'; // Error: Type 'string' is not assignable to type 'number'
// isAdult = 'yes'; // Error: Type 'string' is not assignable to type 'boolean'
```

## Setup Info

- even with error you can run the project but you won't be able to build it "npm run build"

## Union Type

In TypeScript, a Union Type allows a variable to hold a value of multiple, distinct types, specified using the | operator. It can also be used to specify that a variable can hold one of several specific values. More examples are coming up.

```ts
let tax: number | string = 10
tax = 100
tax = '$10'

// fancy name - literal value type
let requestStatus: 'pending' | 'success' | 'error' = 'pending'
requestStatus = 'success'
requestStatus = 'error'
// requestStatus = 'random';
```

## Type - "any"

In TypeScript, the "any" type is a powerful way to work with existing JavaScript, allowing you to opt-out of type-checking and let the values pass through compile-time checks. It means a variable declared with the any type can hold a value of any type. Later will also cover - "unknown" and "never"

```ts
let notSure: any = 4
notSure = 'maybe a string instead'
notSure = false // okay, definitely a boolean
```

## Practical Application of Type Annotation

```ts
const books = ['1984', 'Brave New World', 'Fahrenheit 451']

let foundBook: string | undefined

for (let book of books) {
  if (book === '1984') {
    foundBook = book
    foundBook = foundBook.toUpperCase()
    break
  }
}

console.log(foundBook?.length)
```

The reason to explicitly type foundBook as string | undefined is to make it clear to anyone reading the code (including your future self) that foundBook might be undefined at runtime. This is a good practice in TypeScript because it helps prevent bugs related to undefined values.

## Challenge

- Create a variable orderStatus of type 'processing' | 'shipped' | 'delivered' and assign it the value 'processing'. Then, try to assign it the values 'shipped' and 'delivered'.
- Create a variable discount of type number | string and assign it the value 20. Then, try to assign it the value '20%'.

```ts
// 1. Order Status
let orderStatus: 'processing' | 'shipped' | 'delivered' = 'processing'
orderStatus = 'shipped'
orderStatus = 'delivered'
// orderStatus = 'cancelled'; // This will result in a TypeScript error

// 2. Discount
let discount: number | string = 20
discount = '20%'
// discount = true; // This will result in a TypeScript error
```

## Arrays - Fundamentals

In TypeScript, arrays are used to store multiple values in a single variable. You can define the type of elements that an array can hold using type annotations.

```ts
let prices: number[] = [100, 75, 42]

let fruit: string[] = ['apple', 'orange']
// fruit.push(1);
// let people: string[] = ['bobo', 'peter', 1];
//
// be careful "[]" means literally empty array
// let randomValues: [] = [1];

// be careful with inferred array types
// let names = ['peter', 'susan'];
// let names = ['peter', 'susan', 1];
let array: (string | boolean)[] = ['apple', true, 'orange', false]
```

## Challenge

- Create an array temperatures of type number[] and assign it some values. Then, try to add a string value to it.
- Create an array colors of type string[] and assign it some values. Then, try to add a boolean value to it.
- Create an array mixedArray of type (number | string)[] and assign it some values. Then, try to add a boolean value to it.

```ts
// 1. Temperatures
let temperatures: number[] = [20, 25, 30]
// temperatures.push('hot'); // This will result in a TypeScript error

// 2. Colors
let colors: string[] = ['red', 'green', 'blue']
// colors.push(true); // This will result in a TypeScript error

// 3. Mixed Array
let mixedArray: (number | string)[] = [1, 'two', 3]
// mixedArray.push(true); // This will result in a TypeScript error
```

## Objects - Fundamentals

In TypeScript, an object is a collection of key-value pairs with specified types for each key, providing static type checking for properties.

```ts
let car: { brand: string; year: number } = { brand: 'toyota', year: 2020 }
car.brand = 'ford'
// car.color = 'blue';

let car1: { brand: string; year: number } = { brand: 'audi', year: 2021 }
// let car2: { brand: string; year: number } = { brand: 'audi' };

let book = { title: 'book', cost: 20 }
let pen = { title: 'pen', cost: 5 }
let notebook = { title: 'notebook' }

let items: { readonly title: string; cost?: number }[] = [book, pen, notebook]

items[0].title = 'new book' // Error: Cannot assign to 'title' because it is a read-only property
```

## Challenge

- Create an object bike of type { brand: string, year: number } and assign it some values. Then, try to assign a string to the year property.
- Create an object laptop of type { brand: string, year: number } and try to assign an object with missing year property to it.
- Create an array products of type { title: string, price?: number }[] and assign it some values. Then, try to add an object with a price property of type string to it.

```ts
// 1. Bike
let bike: { brand: string; year: number } = { brand: 'Yamaha', year: 2010 }
// bike.year = 'old'; // This will result in a TypeScript error

// 2. Laptop
let laptop: { brand: string; year: number } = { brand: 'Dell', year: 2020 }
// let laptop2: { brand: string, year: number } = { brand: 'HP' }; // This will result in a TypeScript error

// 3. Products
let product1 = { title: 'Shirt', price: 20 }
let product2 = { title: 'Pants' }
let products: { title: string; price?: number }[] = [product1, product2]
// products.push({ title: 'Shoes', price: 'expensive' }); // This will result in a TypeScript error
```

## Functions - Fundamentals

In TypeScript, functions can have typed parameters and return values, which provides static type checking and autocompletion support.

```ts
function sayHi(name: string) {
  console.log(`Hello there ${name.toUpperCase()}!!!`)
}

sayHi('john')
// sayHi(3)
// sayHi('peter', 'random');

function calculateDiscount(price: number): number {
  // price.toUpperCase();
  const hasDiscount = true
  if (hasDiscount) {
    return price
    // return 'Discount Applied';
  }
  return price * 0.9
}

const finalPrice = calculateDiscount(200)
console.log(finalPrice)

// "any" example
function addThree(number: any) {
  let anotherNumber: number = 3
  return number + anotherNumber
}
const result = addThree(2)
const someValue = result

// run time error
someValue.myMethod()
```

## Challenge

- Create a new array of names.
- Write a new function that checks if a name is in your array. This function should take a name as a parameter and return a boolean.
- Use this function to check if various names are in your array and log the results.

```ts
const names: string[] = ['John', 'Jane', 'Jim', 'Jill']

function isNameInList(name: string): boolean {
  return names.includes(name)
}

let nameToCheck: string = 'Jane'
if (isNameInList(nameToCheck)) {
  console.log(`${nameToCheck} is in the list.`)
} else {
  console.log(`${nameToCheck} is not in the list.`)
}
```

## Functions - Optional and Default Parameters

In TypeScript, a default parameter value is an alternative to an optional parameter. When you provide a default value for a parameter, you're essentially making it optional because you're specifying a value that the function will use if no argument is provided for that parameter.

However, there's a key difference between a parameter with a default value and an optional parameter. If a parameter has a default value, and you call the function without providing an argument for that parameter, the function will use the default value. But if a parameter is optional (indicated with a ?), and you call the function without providing an argument for that parameter, the value of the parameter inside the function will be undefined.

- a function with optional parameters must work when they are not supplied

```ts
function calculatePrice(price: number, discount?: number) {
  return price - (discount || 0)
}

let priceAfterDiscount = calculatePrice(100, 20)
console.log(priceAfterDiscount) // Output: 80

let priceWithoutDiscount = calculatePrice(300)
console.log(priceWithoutDiscount) // Output: 300

function calculateScore(initialScore: number, penaltyPoints: number = 0) {
  return initialScore - penaltyPoints
}

let scoreAfterPenalty = calculateScore(100, 20)
console.log(scoreAfterPenalty) // Output: 80

let scoreWithoutPenalty = calculateScore(300)
console.log(scoreWithoutPenalty) // Output: 300
```

## Function - rest parameter

In JavaScript, a rest parameter is denoted by three dots (...) before the parameter's name and allows a function to accept any number of arguments. These arguments are collected into an array, which can be accessed within the function.

```ts
function sum(message: string, ...numbers: number[]): string {
  const doubled = numbers.map((num) => num * 2)
  console.log(doubled)

  let total = numbers.reduce((previous, current) => {
    return previous + current
  }, 0)
  return `${message} ${total}`
}

let result = sum('The total is:', 1, 2, 3, 4, 5) // result will be "The total is: 15"
```

## Functions - "void" return type

In TypeScript, void is a special type that represents the absence of a value. When used as a function return type, void indicates that the function does not return a value.

```ts
function logMessage(message: string): void {
  console.log(message)
}

logMessage('Hello, TypeScript!') // Output: Hello, TypeScript!
```

It's important to note that in TypeScript, a function that is declared with a void return type can still return a value, but the value will be ignored.For example, the following code is valid TypeScript:

```ts
function logMessage(message: string): void {
  console.log(message)
  return 'This value is ignored'
}

logMessage('Hello, TypeScript!') // Output: Hello, TypeScript!
```
