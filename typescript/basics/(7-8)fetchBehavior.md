## Challenge - Fetch Data

- setup type and provide correct return type

```ts
const url = 'https://www.scourse-api.com/react-tours-project'

// Define a type for the data you're fetching.
type Tour = {
  id: string
  name: string
  info: string
  image: string
  price: string
  // Add more fields as necessary.
}

async function fetchData(url: string): Promise<Tour[]> {
  try {
    const response = await fetch(url)

    // Check if the request was successful.
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data: Tour[] = await response.json()
    console.log(data)
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
tours.map((tour) => {
  console.log(tour.name)
})
```

## ZOD Library

- Tour Data "Gotcha"

```sh
npm install zod
```

- [Zod](https://zod.dev/)
- [Error Handling in Zod](https://zod.dev/ERROR_HANDLING)

```ts
import { z } from 'zod'
const url = 'https://www.course-api.com/react-tours-project'

const tourSchema = z.object({
  id: z.string(),
  name: z.string(),
  info: z.string(),
  image: z.string(),
  price: z.string(),
  somethign: z.string(),
})

// extract the inferred type
type Tour = z.infer<typeof tourSchema>

async function fetchData(url: string): Promise<Tour[]> {
  try {
    const response = await fetch(url)

    // Check if the request was successful.
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const rawData: Tour[] = await response.json()
    const result = tourSchema.array().safeParse(rawData)
    if (!result.success) {
      throw new Error(`Invalid data: ${result.error}`)
    }
    return result.data
  } catch (error) {
    const errMsg =
      error instanceof Error ? error.message : 'there was an error...'
    console.log(errMsg)

    // throw error;
    return []
  }
}

const tours = await fetchData(url)
tours.map((tour) => {
  console.log(tour.name)
})
```

## Typescript Declaration File

In TypeScript, .d.ts files, also known as declaration files,consist of type definitions, and are used to provide types for JavaScript code. They allow TypeScript to understand the shape and types of objects, functions, classes, etc., in JavaScript libraries, enabling type checking and autocompletion in TypeScript code that uses these libraries.

- create types.ts
- export RandomType

tsconfig.json

- `lib`: Set to `["ES2020", "DOM", "DOM.Iterable"]`. This specifies the library files to be included in the compilation.
  Specify a set of bundled library declaration files that describe the target runtime environment.

- libraries

[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)

- password hashing library

```sh
npm i bcryptjs
```

```sh
npm install --save-dev @types/bcryptjs
```

# tsconfig.json Configuration

[tsconfig](https://www.typescriptlang.org/tsconfig)

This project's TypeScript configuration is defined in the `tsconfig.json` file. Here's a breakdown of the configuration options:

- `include`: Set to `["src"]`. This tells TypeScript to only convert files in the `src` directory.

- `target`: Set to `ES2020`. This is the JavaScript version that the TypeScript code will be compiled to.

- `useDefineForClassFields`: Set to `true`. This enables the use of the `define` semantics for initializing class fields.

- `module`: Set to `ESNext`. This is the module system for the compiled code.

- `lib`: Set to `["ES2020", "DOM", "DOM.Iterable"]`. This specifies the library files to be included in the compilation.

- `skipLibCheck`: Set to `true`. This makes TypeScript skip type checking of declaration files (`*.d.ts`).

- `moduleResolution`: Set to `bundler`. This sets the strategy TypeScript uses to resolve modules.

- `allowImportingTsExtensions`: Set to `true`. This allows importing of TypeScript files from JavaScript files.

- `resolveJsonModule`: Set to `true`. This allows importing of `.json` modules from TypeScript files.

- `isolatedModules`: Set to `true`. This ensures that each file can be safely transpiled without relying on other import/export files.

- `noEmit`: Set to `true`. This tells TypeScript to not emit any output files (`*.js` and `*.d.ts` files) after compilation.

- `strict`: Set to `true`. This enables all strict type-checking options.

- `noUnusedLocals`: Set to `true`. This reports an error when local variables are declared but never used.

- `noUnusedParameters`: Set to `true`. This reports an error when function parameters are declared but never used.

- `noFallthroughCasesInSwitch`: Set to `true`. This reports an error for fall through cases in switch statements.

## Classes - Intro

Classes in JavaScript are a blueprint for creating objects. They encapsulate data with code to manipulate that data. Classes in JavaScript support inheritance and can be used to create more complex data structures.

A constructor in a class is a special method that gets called when you create a new instance of the class. It's often used to set the initial state of the object.

```ts
class Book {
  title: string
  author: string
  constructor(title: string, author: string) {
    this.title = title
    this.author = author
  }
}

const deepWork = new Book('deep work ', 'cal newport')
```

## Classes - Instance Property / Default Property

The checkedOut property in Book class is an instance property (or member variable). It's not specifically set in the constructor, so it could also be referred to as a default property or a property with a default value.

```ts
class Book {
  title: string
  author: string
  checkedOut: boolean = false
  constructor(title: string, author: string) {
    this.title = title
    this.author = author
  }
}

const deepWork = new Book('deep work ', 'cal newport')
deepWork.checkedOut = true
// deepWork.checkedOut = 'something else';
```

## Classes - ReadOnly Modifier

- readonly modifier

```ts
class Book {
  readonly title: string
  author: string
  checkedOut: boolean = false
  constructor(title: string, author: string) {
    this.title = title
    this.author = author
  }
}

const deepWork = new Book('deep work ', 'cal newport')

deepWork.title = 'something else'
```

## Classes - Private and Public Modifiers

- private and public modifiers

```ts
class Book {
  public readonly title: string
  public author: string
  private checkedOut: boolean = false
  constructor(title: string, author: string) {
    this.title = title
    this.author = author
  }
  public checkOut() {
    this.checkedOut = this.toggleCheckedOutStatus()
  }
  public isCheckedOut() {
    return this.checkedOut
  }
  private toggleCheckedOutStatus() {
    return !this.checkedOut
  }
}

const deepWork = new Book('Deep Work', 'Cal Newport')
deepWork.checkOut()
console.log(deepWork.isCheckedOut()) // true
// deepWork.toggleCheckedOutStatus(); // Error: Property 'toggleCheckedOutStatus' is private and only accessible within class 'Book'.
```

## Classes - Shorthand Syntax

In TypeScript, if you want to use the shorthand for creating and initializing class properties in the constructor, you need to use public, private, or protected access modifiers.

```ts
class Book {
  private checkedOut: boolean = false
  constructor(public readonly title: string, public author: string) {}
}
```

## Classes - Getters and Setters

Getters and setters are special methods in a class that allow you to control how a property is accessed and modified.They are used like properties, not methods, so you don't use parentheses to call them.

```ts
class Book {
  private checkedOut: boolean = false
  constructor(public readonly title: string, public author: string) {}
  get info() {
    return `${this.title} by ${this.author}`
  }

  private set checkOut(checkedOut: boolean) {
    this.checkedOut = checkedOut
  }
  get checkOut() {
    return this.checkedOut
  }
  public get someInfo() {
    this.checkOut = true
    return `${this.title} by ${this.author}`
  }
}

const deepWork = new Book('deep work', 'cal newport')
console.log(deepWork.info)
// deepWork.checkOut = true;
console.log(deepWork.someInfo)
console.log(deepWork.checkOut)
```

## Classes - Implement Interface

In TypeScript, an interface is a way to define a contract for a certain structure of an object. This contract can then be used by a class to ensure it adheres to the structure defined by the interface.

When a class implements an interface, it is essentially promising that it will provide all the properties and methods defined in the interface. If it does not, TypeScript will throw an error at compile time.

```ts
interface IPerson {
  name: string
  age: number
  greet(): void
}

class Person implements IPerson {
  constructor(public name: string, public age: number) {}

  greet() {
    console.log(`Hello, my name is ${this.name} and I'm ${this.age} years old.`)
  }
}

const hipster = new Person('shakeAndBake', 100)
hipster.greet()
```

## Tasks - Setup

- create tasks.html (root) and src/tasks.ts
- optional : change href in main.ts
- optional css
  - create tasks.css (copy from final or end of the README)
  - setup link in tasks.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Tasks</title>
    <link rel="stylesheet" href="src/tasks.css" />
  </head>
  <body>
    <main>
      <h2>Tasks</h2>
      <form class="form">
        <input type="text" class="form-input" />
        <button type="submit" class="btn">add task</button>
      </form>
      <ul class="list"></ul>
      <button class="test-btn">click me</button>
    </main>
    <script type="module" src="src/tasks.ts"></script>
  </body>
</html>
```

## Tasks - Part 2

```ts
const btn = document.querySelector('.btn')

btn?.addEventListener('click', () => {
  console.log('something')
})

if (btn) {
  // do something
}
```

The ! operator in TypeScript is officially known as the Non-null assertion operator. It is used to assert that its preceding expression is not null or undefined.

```ts
const btn = document.querySelector('.btn')!

btn.addEventListener('click', () => {
  console.log('something')
})
```

Element is the most general base class from which all element objects (i.e. objects that represent elements) in a Document inherit. It only has methods and properties common to all kinds of elements. More specific classes inherit from Element.

```ts
const btn = document.querySelector<HTMLButtonElement>('.selector')!

btn.disabled = true

const btn = document.querySelector('.selector')! as HTMLButtonElement

btn.disabled = true
```

## Tasks - Part 3

```ts
const taskForm = document.querySelector<HTMLFormElement>('.form')
const formInput = document.querySelector<HTMLInputElement>('.form-input')
const taskListElement = document.querySelector<HTMLUListElement>('.list')

// task type
type Task = {
  description: string
  isCompleted: boolean
}

const tasks: Task[] = []
```

## Tasks - Part 4

```ts
taskForm?.addEventListener('submit', (event) => {
  event.preventDefault()
  const taskDescription = formInput?.value
  if (taskDescription) {
    // add task to list
    // render tasks
    // update local storage

    formInput.value = ''
    return
  }
  alert('Please enter a task description')
})
```

- event gotcha

```ts
function createTask(event: SubmitEvent) {
  event.preventDefault()
  const taskDescription = formInput?.value
  if (taskDescription) {
    // add task to list
    // render tasks
    // update local storage

    formInput.value = ''
    return
  }
  alert('Please enter a task description')
}

taskForm?.addEventListener('submit', createTask)
```

## Tasks - Part 5

```ts
taskForm?.addEventListener('submit', (event) => {
  event.preventDefault()
  const taskDescription = formInput?.value
  if (taskDescription) {
    const task: Task = {
      description: taskDescription,
      isCompleted: false,
    }
    // add task to list
    addTask(task)
    // render tasks

    // update local storage

    formInput.value = ''
    return
  }
  alert('Please enter a task description')
})

function addTask(task: Task): void {
  tasks.push(task)
  // console.log(tasks);
}
```

## Tasks - Part 6

```ts
function renderTask(task: Task): void {
  const taskElement = document.createElement('li')
  taskElement.textContent = task.description
  taskListElement?.appendChild(taskElement)
}
```

```ts
// add task to list
addTask(task)
// render task
renderTask(task)
```

## Tasks - Part 7

```ts
// Retrieve tasks from localStorage
const tasks: Task[] = loadTasks()

// Load tasks from localStorage
function loadTasks(): Task[] {
  const storedTasks = localStorage.getItem('tasks')
  return storedTasks ? JSON.parse(storedTasks) : []
}

// tasks.forEach((task) => renderTask(task));
tasks.forEach(renderTask)

// Update tasks in localStorage
function updateStorage(): void {
  localStorage.setItem('tasks', JSON.stringify(tasks))
}
```

```ts
// add task to list
addTask(task)
// render task
renderTask(task)
// update local storage
updateStorage()
```

## Tasks - Part 8

```ts
function renderTask(task: Task): void {
  const taskElement = document.createElement('li')
  taskElement.textContent = task.description
  // checkbox
  const taskCheckbox = document.createElement('input')
  taskCheckbox.type = 'checkbox'
  taskCheckbox.checked = task.isCompleted

  taskElement.appendChild(taskCheckbox)
  taskListElement?.appendChild(taskElement)
}
```

## Tasks - Part 9

```ts
function renderTask(task: Task): void {
  const taskElement = document.createElement('li')
  taskElement.textContent = task.description
  // checkbox
  const taskCheckbox = document.createElement('input')
  taskCheckbox.type = 'checkbox'
  taskCheckbox.checked = task.isCompleted
  // toggle checkbox
  taskCheckbox.addEventListener('change', () => {
    task.isCompleted = !task.isCompleted
    updateStorage()
  })

  taskElement.appendChild(taskCheckbox)
  taskListElement?.appendChild(taskElement)
}
```

## Tasks - CSS

tasks.css

```css
/* ============= GLOBAL CSS =============== */

*,
::after,
::before {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  font-size: 100%;
} /*16px*/

:root {
  /* colors */
  --primary-100: #e2e0ff;
  --primary-200: #c1beff;
  --primary-300: #a29dff;
  --primary-400: #837dff;
  --primary-500: #645cff;
  --primary-600: #504acc;
  --primary-700: #3c3799;
  --primary-800: #282566;
  --primary-900: #141233;

  /* grey */
  --grey-50: #f8fafc;
  --grey-100: #f1f5f9;
  --grey-200: #e2e8f0;
  --grey-300: #cbd5e1;
  --grey-400: #94a3b8;
  --grey-500: #64748b;
  --grey-600: #475569;
  --grey-700: #334155;
  --grey-800: #1e293b;
  --grey-900: #0f172a;
  /* rest of the colors */
  --black: #222;
  --white: #fff;
  --red-light: #f8d7da;
  --red-dark: #842029;
  --green-light: #d1e7dd;
  --green-dark: #0f5132;

  --small-text: 0.875rem;
  --extra-small-text: 0.7em;
  /* rest of the vars */

  --border-radius: 0.25rem;
  --letter-spacing: 1px;
  --transition: 0.3s ease-in-out all;
  --max-width: 1120px;
  --fixed-width: 600px;
  --view-width: 90vw;
  /* box shadow*/
  --shadow-1: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
  --shadow-2: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  --shadow-3: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  --shadow-4: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  /* DARK MODE */
  --background-color: var(--grey-50);
  --text-color: var(--grey-900);
}

body {
  background: var(--background-color);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
    Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  font-weight: 400;
  line-height: 1;
  color: var(--text-color);
}
p {
  margin: 0;
}
h1,
h2,
h3,
h4,
h5 {
  margin: 0;
  font-weight: 400;
  line-height: 1;
  text-transform: capitalize;
  letter-spacing: var(--letter-spacing);
}

h1 {
  font-size: clamp(2rem, 5vw, 5rem); /* Large heading */
}

h2 {
  font-size: clamp(1.5rem, 3vw, 3rem); /* Medium heading */
}

h3 {
  font-size: clamp(1.25rem, 2.5vw, 2.5rem); /* Small heading */
}

h4 {
  font-size: clamp(1rem, 2vw, 2rem); /* Extra small heading */
}

h5 {
  font-size: clamp(0.875rem, 1.5vw, 1.5rem); /* Tiny heading */
}

small,
.text-small {
  font-size: var(--small-text);
}

a {
  text-decoration: none;
}
ul {
  list-style-type: none;
  padding: 0;
}

.img {
  width: 100%;
  display: block;
  object-fit: cover;
}
/* buttons */

.btn {
  cursor: pointer;
  color: var(--white);
  background: var(--primary-500);
  border: transparent;
  letter-spacing: var(--letter-spacing);
  box-shadow: var(--shadow-1);
  transition: var(--transition);
  text-transform: capitalize;
  display: inline-block;
}
.btn:hover {
  background: var(--primary-700);
  box-shadow: var(--shadow-3);
}
.btn-hipster {
  color: var(--primary-500);
  background: var(--primary-200);
}
.btn-hipster:hover {
  color: var(--primary-200);
  background: var(--primary-700);
}
.btn-block {
  width: 100%;
}

/* alerts */
.alert {
  padding: 0.375rem 0.75rem;
  margin-bottom: 1rem;
  border-color: transparent;
  border-radius: var(--border-radius);
}

.alert-danger {
  color: var(--red-dark);
  background: var(--red-light);
}
.alert-success {
  color: var(--green-dark);
  background: var(--green-light);
}
/* form */

.form {
  background: var(--white);
  border-radius: var(--border-radius);
  box-shadow: var(--shadow-2);
  padding: 2rem 2.5rem;
  margin-bottom: 2rem;
}

.form-input {
  width: 100%;
  padding: 0.375rem 0.75rem;
  border-top-left-radius: var(--border-radius);
  border-bottom-left-radius: var(--border-radius);
  background: var(--background-color);
  border: 1px solid var(--grey-200);
}

main {
  padding: 5rem 0;
  min-height: 100vh;
  width: 90vw;
  max-width: 500px;
  margin: 0 auto;
}

/* title */
main h2 {
  text-align: center;
  margin-bottom: 2rem;
}

.form {
  display: grid;
  grid-template-columns: 1fr 100px;
}

.form button {
  border-top-right-radius: var(--border-radius);
  border-bottom-right-radius: var(--border-radius);
}

.list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1.25rem;
  margin-bottom: 0.5rem;
  background: var(--white);
  border-radius: var(--border-radius);
  box-shadow: var(--shadow-1);
}
```
