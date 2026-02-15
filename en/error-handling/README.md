# Tips on Clean Error Handling

Error handling is one of the most overlooked aspects of clean code. Well-handled errors make our application robust, debuggable, and maintainable. Poor error handling, on the other hand, leads to cryptic failures, silent bugs, and frustrated users.

Here are some tips that we could follow:

## 1. Use Exceptions for Exceptional Cases

Not every error condition warrants an exception. Exceptions should be reserved for truly unexpected situations — things that shouldn't happen in normal program flow. Don't use exceptions for control flow.

**Bad example:**

```js
function findUser(id) {
  const user = database.find(id);
  if (user === null) {
    throw new Error("User not found");
  }
  return user;
}

// Using exceptions for expected flow
try {
  const user = findUser(123);
} catch (e) {
  // This happens regularly in normal operation
  console.log("No user found, creating new one");
}
```

**Better:**

```js
function findUserOrNull(id) {
  const user = database.find(id);
  return user || null;
}

// Clearer intent
const user = findUserOrNull(123);
if (!user) {
  console.log("No user found, creating new one");
  return;
}
```

Using exceptions for expected outcomes forces developers to use try-catch everywhere, which obscures real errors.

## 2. Provide Meaningful Error Messages

Error messages should help developers (and sometimes users) understand what went wrong, where, and why. A generic "An error occurred" message is useless for debugging.

**Bad example:**

```js
function processOrder(orderId) {
  if (!orderId) {
    throw new Error("Error");
  }
  // ...
}
```

**Better:**

```js
function processOrder(orderId) {
  if (!orderId) {
    throw new Error(
      `Cannot process order: orderId is required but was ${orderId}`
    );
  }
  // ...
}
```

Even better — use custom error classes with structured data:

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

function validateEmail(email) {
  if (!email || !email.includes("@")) {
    throw new ValidationError("Invalid email format", "email");
  }
}
```

## 3. Catch Specific Exceptions

Avoid bare `catch` blocks that catch everything. Different errors require different handling. Catch only what you can handle.

**Bad example:**

```js
try {
  processPayment(order);
} catch (e) {
  // Catching everything — even unexpected errors
  console.log("Payment failed");
}
```

**Better:**

```js
try {
  processPayment(order);
} catch (e) {
  if (e instanceof InsufficientFundsError) {
    notifyUser("Insufficient balance");
    showTopUpOption();
  } else if (e instanceof NetworkError) {
    retryPayment(order);
  } else {
    // Unexpected error — let it propagate
    throw e;
  }
}
```

## 4. Don't Swallow Exceptions

Never catch an exception and do nothing with it. Silent failures are among the hardest bugs to debug.

**Bad example:**

```js
try {
  sendWelcomeEmail(user);
} catch (e) {
  // Whoops — email failed, but we ignore it
}
```

**Better:**

```js
try {
  sendWelcomeEmail(user);
} catch (e) {
  // At minimum, log the error
  logger.error("Failed to send welcome email", {
    userId: user.id,
    error: e.message
  });
  // Then decide: retry? notify? fallback?
}
```

If you truly cannot handle an error, let it bubble up — don't catch it just to make it disappear.

## 5. Fail Fast on Invalid Inputs

Validate inputs at the boundary of our system (public APIs, entry points) and fail fast. Don't let invalid data propagate deep into our system.

**Bad example:**

```js
function calculateDiscount(user, product) {
  // No validation — assumes inputs are valid
  const basePrice = product.price;
  
  if (user.isPremium) {
    return basePrice * 0.8; // 20% discount
  }
  return basePrice;
}

// Error surfaces deep in the call stack
const discount = calculateDiscount(null, { price: 100 });
// Cannot tell where it started
```

**Better:**

```js
function calculateDiscount(user, product) {
  if (!user || typeof user.isPremium !== "boolean") {
    throw new TypeError("Invalid user: isPremium must be a boolean");
  }
  if (!product || typeof product.price !== "number") {
    throw new TypeError("Invalid product: price must be a number");
  }
  
  if (user.isPremium) {
    return product.price * 0.8;
  }
  return product.price;
}
```

The error now surfaces immediately at the source of the problem.

## 6. Wrap Third-Party Errors

When calling external libraries or services, wrap their errors in our own error types. This creates a stable API and adds context.

**Bad example:**

```js
try {
  const user = await externalAuthService.verifyToken(token);
} catch (e) {
  // Exposes internal implementation details
  throw e;
}
```

**Better:**

```js
class AuthenticationError extends Error {
  constructor(message, originalError) {
    super(message);
    this.name = "AuthenticationError";
    this.originalError = originalError;
  }
}

async function verifyUser(token) {
  try {
    return await externalAuthService.verifyToken(token);
  } catch (e) {
    throw new AuthenticationError(
      `Failed to verify token: ${e.message}`,
      e
    );
  }
}
```

Now callers don't need to know about the underlying auth service implementation.

## 7. Separate Error Handling from Business Logic

Keep your core logic focused on the happy path. Handle errors at the edges of our system.

**Bad example:**

```js
async function createUser(userData) {
  try {
    const exists = await database.findUserByEmail(userData.email);
    if (exists) {
      return { success: false, error: "Email exists" };
    }
    
    const user = await database.createUser(userData);
    try {
      await emailService.sendWelcome(user.email);
    } catch (e) {
      return { success: true, warning: "Email failed" };
    }
    
    return { success: true, user };
  } catch (e) {
    return { success: false, error: e.message };
  }
}
```

**Better:**

```js
async function createUser(userData) {
  const exists = await database.findUserByEmail(userData.email);
  if (exists) {
    throw new ConflictError("Email already registered");
  }
  
  return database.createUser(userData);
}

async function handleUserSignup(req, res) {
  try {
    const user = await createUser(req.body);
    await emailService.sendWelcome(user.email).catch(err => 
      logger.error("Welcome email failed", err)
    );
    res.status(201).json(user);
  } catch (e) {
    if (e instanceof ConflictError) {
      res.status(409).json({ error: e.message });
    } else {
      logger.error("Signup failed", e);
      res.status(500).json({ error: "Internal server error" });
    }
  }
}
```

The core function does one thing. Error handling happens in the controller layer.

## 8. Use a Consistent Error Handling Pattern

Establish a convention for how our application handles errors. Whether you use Result types, error first callbacks, or exceptions — be consistent.

**Example with Result type pattern:**

```js
function divide(a, b) {
  if (b === 0) {
    return { ok: false, error: "Cannot divide by zero" };
  }
  return { ok: true, value: a / b };
}

// Usage
const result = divide(10, 0);
if (!result.ok) {
  console.log(result.error);
  return;
}
console.log(result.value);
```

This pattern works well when you want explicit error handling and want to avoid exceptions.

## 9. Provide Recovery Paths

When something fails, give users (or the system) a way to recover. Don't just show an error and stop.

**Bad example:**

```js
function processOrder(order) {
  const payment = processPayment(order.payment);
  if (!payment.success) {
    throw new Error("Payment failed");
  }
  // ...
}
```

**Better:**

```js
function processOrder(order) {
  const payment = processPayment(order.payment);
  if (!payment.success) {
    return {
      success: false,
      error: "Payment failed",
      recovery: {
        action: "update_payment",
        availableMethods: ["card", "bank", "wallet"]
      }
    };
  }
  // ...
}
```

---

## Summary

| Tip | Key Takeaway |
|-----|--------------|
| Exceptions for Exceptions | Only throw when something unexpected happens |
| Meaningful Messages | Include context — what, where, why |
| Catch Specific | Handle what you can, rethrow what you can't |
| Don't Swallow | At minimum, log the error |
| Fail Fast | Validate inputs early |
| Wrap Errors | Add context to third-party errors |
| Separate Concerns | Business logic ≠ error handling |
| Be Consistent | Pick a pattern and stick to it |
| Provide Recovery | Don't just fail — offer next steps |

Good error handling doesn't just prevent crashes — it builds trust. When something goes wrong, clear errors help you fix problems fast and keep your users moving forward.
