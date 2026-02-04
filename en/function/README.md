# Tips on writing clean function

Writing clean functions isn’t about following strict rules — it’s about making our code easy to read, reason about, and change. When each function has a clear purpose, our entire codebase becomes easier to maintain and evolve.

Here are few tips that we could follow:

## 1. Each function should have one clear responsibility

A function should do one thing, and do it well. Functions that do multiple things are hard to follow, understand, test, debug, and even reuse.When something breaks, it’s very hard to pinpoint the exact part that failed.

from the [clean code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) Uncle bob says-

>The first rule of functions is that they should be small.
>The second rule of functions is that they should be smaller than that.

**Small** doesn’t mean a fixed number of lines. Sometimes a function might be just 5 lines long, while other times it might take 50 lines to fulfill a single, well-defined responsibility. What matters is not the size, but the **clarity of purpose**.

**Bad example:**

```js
function processUser(user) {
  // validate user
  if (!user.email || !user.name) {
    throw new Error("Invalid user");
  }

  // save user
  database.save(user);

  // send welcome email
  emailService.sendWelcomeEmail(user.email);

  // log action
  console.log("User processed:", user.email);
}
```

This function:

- Validates data
- Saves to the database
- Sends an email
- Logs information

That’s four responsibilities in one function. Any change to validation, persistence, email logic, or logging risks breaking this function.

**Better:**

```js
function validateUser(user) {
  if (!user.email || !user.name) {
    throw new Error("Invalid user");
  }
}

function saveUser(user) {
  database.save(user);
}

function sendWelcomeEmail(user) {
  emailService.sendWelcomeEmail(user.email);
}

function logUserProcessed(user) {
  console.log("User processed:", user.email);
}

function processUser(user) {
  validateUser(user);
  saveUser(user);
  sendWelcomeEmail(user);
  logUserProcessed(user);
}
```

## 2. Use descriptive and intent-revealing names for your functions

A function name should immediately communicate its **purpose**, **effect**, and **usage**. Good names remove the need for comments and reduce cognitive overhead.

A name should reveal intent:

- why it exists?
- what it does?
- how it is used?

A very long name is also okay. its better than comments, abbreviations etc.

Avoid function like `process()`, `handle()` these force readers to examine the implementation to understand what the function do.

**Bad example:**

```js
function calc(x: number, y: number): number {
    return x * y * 0.1;
}

function process(user: User): void {
    // what actually this do ?
}

function handle(data: any): boolean {
    // handle what?
}
```

**Better**:

```js
const TAX_MULTIPLIER = 0.1;
function calculateTax(price: number , taxRate: number): number {
    return price * taxRate * TAX_MULTIPLIER;
}

function validateUserEmail(user: User) {
    // clear purpose
}

function isPaymentValid(paymentData: PaymentData): boolean {
    // name indicates what it checks
}
```

Tips to remember:
- Follow already established [naming conventions](https://belev.dev/6-tips-to-improve-naming-and-why-it-is-so-important)
- Action functions: use verbs (`calculate`, `validate`, `send`)
- Boolean functions: use question words (`is`, `has`, `can`)
- Consistency matters: `getUser`, `getOrder`, `getProduct` is clearer than mixing `fetch / retrieve`

## 3. Limit Function Parameters

More arguments are making the function harder to understand and to test as well because of all of the different arguments' combinations.

Try to keep the functions with up to 2-3 Parameters.

**Bad Example:**

```js
function createUser(name: string, email: string, password: string, role: string, isActive: boolean) { ... }
```

Problems:

- Hard to track parameter order
- Easy to pass incorrect arguments
- Harder to refactor safely

**Better:**

```js
interface UserData {
  name: string;
  email: string;
  password: string;
  role: string;
  isActive: boolean;
}

function createUser(userData: UserData) { ... }
```

Now:

- Easier to read, maintain, and extend
- Callers can pass a literal or construct an object beforehand
- Named properties improve clarity-
