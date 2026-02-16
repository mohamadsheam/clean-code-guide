# Tips on Writing Clean Names

Names are everywhere in our code — variables, functions, classes, files. We name variables to hold values, functions to perform actions, classes to represent concepts, and files to organize our work. Good names make code readable; bad names make it confusing.

Here are few tips that we could follow:

## 1. Use Intent-Revealing Names

A name should answer three questions:

- **Why does it exist?**
- **What does it do?**
- **How is it used?**

If you need a comment to explain what a variable means, the name is not good enough.

**Bad example:**

```js
const d = new Date(); // what is d?
const x = users.filter(u => u.a === 1); // what is a? what does 1 mean?
```

**Better:**

```js
const currentDate = new Date();
const activeUsers = users.filter(user => user.isActive === true);
```

## 2. Avoid Magic Numbers and Strings

Hardcoded values scattered throughout code are hard to understand and dangerous to change.

**Bad example:**

```js
if (status === 2) {
  setTimeout(() => {
    sendNotification();
  }, 86400000);
}
```

**Better:**

```js
const USER_STATUS = {
  ACTIVE: 2,
  INACTIVE: 0
};

const ONE_DAY_IN_MS = 86_400_000;

if (status === USER_STATUS.ACTIVE) {
  setTimeout(() => {
    sendNotification();
  }, ONE_DAY_IN_MS);
}
```

## 3. Use Pronounceable and Searchable Names

If you cannot pronounce it, you cannot discuss it without sounding foolish. Also, avoid single-letter names that are hard to search for.

**Bad example:**

```js
const y = new Date().getFullYear();
const r = users.filter(u => u.s === 'active');
```

**Better:**

```js
const currentYear = new Date().getFullYear();
const activeUsers = users.filter(user => user.status === 'active');
```

## 4. Use Consistent Naming Conventions

Pick a convention and stick with it throughout the codebase:

| Type | Convention | Example |
|------|------------|---------|
| Variables & Functions | camelCase | `getUserById`, `isActive` |
| Classes & Types | PascalCase | `UserService`, `OrderItem` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `API_BASE_URL` |
| Files (classes) | PascalCase | `UserService.ts` |
| Files (utilities) | kebab-case | `date-utils.ts` |

## 5. Use Problem Domain Names

Use names from the problem domain, not computer science terms — unless the domain is programming itself.

**Bad example:**

```js
const list = getUsers();
const manager = new OrderManager();
const proc = processOrder(order);
```

**Better:**

```js
const users = getUsers();
const orderProcessor = new OrderProcessor();
const fulfillOrder = (order) => { ... };
```

## 6. Avoid Encodings

Do not encode type information into names — the language or IDE should handle that.

**Bad example:**

```js
const strName = "John";           // Hungarian notation - type in name
const usersArray = [];            // implementation detail in name
const _privateMethod = () => {};  // prefix to mark visibility
```

**Better:**

```js
const name = "John";
const users = [];
const calculateTotal = () => {};
```

## 7. Classes Should Have Noun or Noun Phrase Names

Class names should represent a thing, not an action.

**Bad example:**

```js
class ManageData { }
class Process { }
class Handler { }
```

**Better:**

```js
class UserManager { }
class DataProcessor { }
class EventHandler { }
```

## 8. Functions Should Have Verb or Verb Phrase Names

Function names should describe an action.

**Bad example:**

```js
function user() { }        // noun - what does it do?
function data() { }
```

**Better:**

```js
function getUser() { }     // verb - clear action
function validateInput() { }
function calculateTotal() { }
```

## 9. Use Boolean Names That Are Questions

Boolean variables and functions returning boolean should read like questions.

**Bad example:**

```js
const isActive = true;       // good
const isPremiumUser = false; // good
const flag = true;           // unclear
function checkValid() { }    // checkValid what?
```

**Better:**

```js
const isActive = true;
const isPremiumUser = false;
const hasPermission = true;
const canSubmit = true;
function isValidEmail(email) { }
function hasAccess(user, resource) { }
```

## 10. Add Meaningful Context

When a name alone is not clear, add context — either through the name itself or by wrapping in a class/module.

**Bad example:**

```js
const first = "John";
const last = "Doe";
const city = "New York";
const zip = "10001";
```

**Better:**

```js
const user = {
  firstName: "John",
  lastName: "Doe",
  address: {
    city: "New York",
    zipCode: "10001"
  }
};
```

Or use a class to provide context:

```js
class User {
  constructor(firstName, lastName, address) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.address = address;
  }

  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

## 11. Don't Be Afraid of Long Names

A long, descriptive name is better than a short, confusing one. Modern IDEs handle autocomplete, so length is not a problem.

**Bad example:**

```js
const getU = (id) => users.find(u => u.id === id);
```

**Better:**

```js
const getUserById = (userId) => users.find(user => user.id === userId);
```

## 12. One Concept Per Word

Pick one word per concept and use it consistently.

**Bad example:**

```js
function getUser() { }
function fetchProduct() { }
function retrieveOrder() { }
```

Three different words for the same concept — confusing.

**Better:**

```js
function getUser() { }
function getProduct() { }
function getOrder() { }
```

---

## Summary

| Tip | Key Takeaway |
|-----|---------------|
| Intent-Revealing | Names should explain purpose without comments |
| Avoid Magic Values | Use named constants |
| Searchable | Avoid single letters and obscure abbreviations |
| Consistent | Pick a convention and follow it |
| Problem Domain | Use names from the real world |
| No Encodings | Don't include types in names |
| Classes = Nouns | Classes represent things |
| Functions = Verbs | Functions represent actions |
| Booleans = Questions | Use `is`, `has`, `can`, `should` |
| Meaningful Context | Group related concepts together |
| Long Names OK | Clarity over brevity |
| One Word Per Concept | Be consistent |

Good naming is one of the hardest skills in programming — it requires thinking carefully about what code actually does. Invest time in naming, and your code will be cleaner, easier to read, and easier to maintain.
