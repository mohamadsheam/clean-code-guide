# Tips on Writing Clean Comments

Comments should explain **why** we do something, not **what** we do. Well-written comments clarify intent, prevent misunderstandings, and document business logic that isn't obvious from the code itself.

Here are key principles for writing effective comments:

## 1. Don't Comment Bad Code - Rewrite It

The best comment is no comment. If you need comments to explain what your code does, your code is unclear.

**Bad Example:**

```js
// Check if user is eligible for discount
if (user.age >= 65 && user.membershipYears >= 5) {
  // Apply 20% senior discount
  user.price = user.price * 0.8;
}
```

**Better:**

```js
if (isEligibleForSeniorDiscount(user)) {
  applySeniorDiscount(user);
}

function isEligibleForSeniorDiscount(user) {
  return user.age >= 65 && user.membershipYears >= 5;
}

function applySeniorDiscount(user) {
  user.price = user.price * 0.8;
}
```

## 2. Explain Why, Not What

Comments should reveal intent, decisions, and business rules that aren't obvious from the code.

**Bad Example:**

```js
// Increment counter by 1
counter++;
```

**Better:**

```js
// Track failed login attempts for security monitoring
failedLoginAttempts++;
```

**Good Example:**

```js
// Skip weekends in business day calculations
// Regulatory requirement: banking days exclude Sat/Sun
if (date.getDay() === 0 || date.getDay() === 6) {
  continue;
}
```

## 3. Keep Comments Up-to-Date

Outdated comments are worse than no comments because they mislead.

**Bad Example:**

```js
// TODO: Remove this after migration to v2 (deprecated)
const API_URL = "https://api.example.com/v1";
```

**Better:**

```js
// Legacy v1 API - maintain for backwards compatibility until 2024-12-31
// See MIGRATION.md for v2 API details
const LEGACY_API_URL = "https://api.example.com/v1";
```

## 4. Use Comments to Document Important Decisions

Capture the reasoning behind non-obvious technical or business decisions.

```js
// Using React's useCallback to prevent unnecessary re-renders
// in the child component that receives this function as prop
const handleSubmit = useCallback((data) => {
  processSubmission(data);
}, [data]);

// Chose SQLite over PostgreSQL for this embedded application
// because: no external database server required, smaller footprint,
// and we don't need concurrent writes for single-user usage
const db = new Database('./app.db');
```

## 5. Comment Warnings and Consequences

Alert other developers about dangerous operations or side effects.

```js
// ⚠️  DESTRUCTIVE OPERATION: This will permanently delete user data
// Backup should be verified before calling this function
function deleteUserAccount(userId) {
  cascadeDeleteUserData(userId);
  auditLog.userDeleted(userId);
}

// Performance note: This query is intentionally NOT optimized
// because it runs only once during yearly reporting
// DO NOT use in time-critical paths
function generateAnnualReport(year) {
  return db.query(/* complex, slow query */);
}
```

## 6. Don't Comment Out Code - Delete It

Version control preserves code history. Commented code creates confusion.

**Bad Example:**

```js
function processOrder(order) {
  // if (order.isExpress) {
  //   processExpressShipping(order);
  // }
  processStandardShipping(order);
}
```

**Better:**

```js
function processOrder(order) {
  processStandardShipping(order);
}
```

## 7. Write Comments for Future Maintainers

Assume the person reading your code doesn't have the context you have now.

```js
// This edge case occurs when users submit forms with identical timestamps
// Due to database precision, both records might have the same created_at
// We use the record's primary key as a tiebreaker for consistent ordering
function sortSubmissions(submissions) {
  return submissions.sort((a, b) => {
    if (a.created_at === b.created_at) {
      return a.id - b.id; // Tiebreaker: lower ID first
    }
    return new Date(a.created_at) - new Date(b.created_at);
  });
}
```

## 8. Use Standard Comment Formats

Follow consistent comment patterns for different types of information.

```js
// JSDoc format for function documentation
/**
 * Calculates the total price including tax and discounts
 * @param {number} basePrice - Original price before modifications
 * @param {number} taxRate - Tax rate as decimal (0.1 = 10%)
 * @param {Discount|null} discount - Optional discount object
 * @returns {number} Final price after tax and discounts
 */
function calculateFinalPrice(basePrice, taxRate, discount = null) {
  let price = basePrice;
  
  if (discount) {
    // Apply percentage discount if available
    price = price * (1 - discount.percentage);
  }
  
  return price * (1 + taxRate);
}

// Inline comments for complex logic
const discount = user.isPremium 
  ? PREMIUM_DISCOUNT    // 15% for premium users
  : STANDARD_DISCOUNT;  // 5% for regular users

// Section comments for code organization
// ==========================================
// Database Configuration
// ==========================================
const dbConfig = {
  host: process.env.DB_HOST,
  // ... other config
};
```

## 9. Avoid Redundant Comments

Don't repeat what the code already clearly states.

**Bad Example:**

```js
// Initialize user object with name and email
const user = {
  name: "John",
  email: "john@example.com"
};

// Check if user exists
if (user) {
  // Display welcome message
  console.log("Welcome!");
}
```

**Better:**

```js
const user = {
  name: "John",
  email: "john@example.com"
};

if (user) {
  console.log("Welcome!");
}
```

## 10. Use TODO and FIXME Comments Wisely

Track work that needs attention, but be specific about what's needed.

```js
// TODO: Implement input validation for email format
// Add regex validation before user registration
function registerUser(email, password) {
  // Implementation pending validation logic
}

// FIXME: This fails when orders contain zero-quantity items
// Reproduces when users have empty items in cart
function calculateOrderTotal(items) {
  return items.reduce((total, item) => total + (item.price * item.quantity), 0);
}

// NOTE: Temporary workaround for third-party API rate limiting
// Remove after implementing proper retry mechanism (ticket #1234)
setTimeout(() => fetchUserData(), 1000);
```

---

## Summary

| Principle | Key Takeaway |
|-----------|---------------|
| Don't comment bad code | Rewrite unclear code instead |
| Explain why, not what | Reveal intent and decisions |
| Keep comments current | Remove or update outdated comments |
| Document decisions | Capture reasoning for future developers |
| Warn about consequences | Highlight dangerous or performance-critical code |
| Don't comment out code | Delete code, version control preserves history |
| Write for maintainers | Assume no context from the future reader |
| Use standard formats | JSDoc, inline, and section comments consistently |
| Avoid redundancy | Don't restate what the code clearly shows |
| Use TODO/FIXME wisely | Be specific about what needs attention |
