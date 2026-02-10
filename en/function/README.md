# Tips on writing clean function

Writing clean functions isn't about following strict rules — it's about making our code easy to read, reason about, and change. When each function has a clear purpose, our entire codebase becomes easier to maintain and evolve.

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
- Named properties improve clarity

## 4. Avoid Side Effects

A function has a **side effect** when it modifies something outside its own scope — like changing a global variable, mutating an input parameter, or writing to a file. Side effects make code unpredictable and harder to debug.

**Bad Example:**

```js
let cartTotal = 0;

function addToCart(item) {
  cartTotal += item.price; // modifies external state
  items.push(item);        // mutates external array
}
```

Problems:

- The function secretly changes global state
- Hard to test in isolation
- Order of function calls matters unexpectedly

**Better:**

```js
function addToCart(cart, item) {
  return {
    items: [...cart.items, item],
    total: cart.total + item.price
  };
}

// Usage
const newCart = addToCart(currentCart, newItem);
```

Now the function is **pure** — same inputs always produce the same outputs, with no hidden changes.

## 5. Use Early Returns (Guard Clauses)

Deeply nested `if-else` blocks make code hard to follow. **Guard clauses** handle edge cases early and exit the function, keeping the main logic flat and readable.

**Bad Example:**

```js
function getDiscount(user) {
  if (user) {
    if (user.isActive) {
      if (user.isPremium) {
        return 0.2;
      } else {
        return 0.1;
      }
    } else {
      return 0;
    }
  } else {
    return 0;
  }
}
```

**Better:**

```js
function getDiscount(user) {
  if (!user) return 0;
  if (!user.isActive) return 0;
  if (user.isPremium) return 0.2;
  return 0.1;
}
```

Benefits:

- Reduces nesting and cognitive load
- Edge cases are handled upfront
- Main logic becomes immediately clear

## 6. Avoid Boolean Flag Arguments

Passing a boolean to toggle behavior inside a function is a code smell. It usually means the function is doing more than one thing.

**Bad Example:**

```js
function createFile(name, isTemp) {
  if (isTemp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}

// Caller has no idea what `true` means without checking the function
createFile("report.pdf", true);
```

**Better:**

```js
function createFile(name) {
  fs.create(name);
}

function createTempFile(name) {
  fs.create(`./temp/${name}`);
}

// Intent is now crystal clear
createTempFile("report.pdf");
```

## 7. Keep Functions at One Level of Abstraction

Mixing high-level logic with low-level details in the same function makes it hard to understand. Each function should operate at a **consistent level of abstraction**.

**Bad Example:**

```js
function renderPage(pageData) {
  const html = `<html><head>${pageData.title}</head>`; // low-level
  const user = getLoggedInUser();                      // high-level
  const content = parseMarkdown(pageData.body);        // mid-level
  return html + `<body>${content}</body></html>`;      // low-level
}
```

**Better:**

```js
function renderPage(pageData) {
  const user = getLoggedInUser();
  const content = parseMarkdown(pageData.body);
  return buildHtml(pageData.title, content);
}

function buildHtml(title, content) {
  return `<html><head>${title}</head><body>${content}</body></html>`;
}
```

Now `renderPage` reads like a story, and low-level HTML construction is delegated.

## 8. Functions Should Either Do Something or Answer Something

This is known as **Command-Query Separation (CQS)**. A function should either perform an action (command) or return data (query) — but not both.

**Bad Example:**

```js
function setAndGetPreviousValue(obj, key, newValue) {
  const oldValue = obj[key];
  obj[key] = newValue;
  return oldValue;
}
```

This function both mutates state and returns a value, making it confusing.

**Better:**

```js
function getValue(obj, key) {
  return obj[key];
}

function setValue(obj, key, value) {
  obj[key] = value;
}

// Usage is now explicit
const previousValue = getValue(config, "timeout");
setValue(config, "timeout", 5000);
```

## 9. Write Functions That Are Easy to Test

If a function is hard to test, it's usually a sign of poor design. Testable functions tend to be:

- Pure (no side effects)
- Small and focused
- Have explicit dependencies (passed as arguments, not hidden)

**Bad Example:**

```js
function calculateOrderTotal(orderId) {
  const order = database.getOrder(orderId); // hidden dependency
  const discount = getCurrentPromotion();    // hidden dependency
  return order.items.reduce((sum, item) => sum + item.price, 0) * (1 - discount);
}
```

Testing this requires mocking the database and promotion service.

**Better:**

```js
function calculateOrderTotal(items, discountRate) {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  return subtotal * (1 - discountRate);
}

// Now testing is straightforward
expect(calculateOrderTotal([{ price: 100 }, { price: 50 }], 0.1)).toBe(135);
```

---

## Summary

| Tip | Key Takeaway |
|-----|---------------|
| Single Responsibility | One function, one job |
| Descriptive Names | Names should reveal intent |
| Limit Parameters | Use objects for 3+ parameters |
| Avoid Side Effects | Prefer pure functions |
| Early Returns | Reduce nesting with guard clauses |
| No Boolean Flags | Split into separate functions |
| One Abstraction Level | Don't mix high and low-level code |
| Command-Query Separation | Do something OR return something |
| Testability | If it's hard to test, refactor it |

Clean functions are the foundation of clean code. Master these principles, and your codebase will thank you.
