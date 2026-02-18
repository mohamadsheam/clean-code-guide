# Tips on Writing Clean Tests

Writing tests is not just about verifying that code works — it's about documenting behavior, enabling safe refactoring, and serving as executable documentation for our codebase. Clean tests are just as important as clean production code.

Here are few tips that we could follow:

## 1. Test One Thing Per Test

Each test should verify a single behavior. When a test fails, we should immediately know what broke. Tests that check multiple things are harder to debug and maintain.

**Bad example:**

```js
test('user registration', () => {
  const user = registerUser('john@example.com', 'password123');
  
  expect(user.email).toBe('john@example.com');
  expect(user.password).toBe('password123'); // wait, passwords shouldn't be stored plainly!
  expect(user.isActive).toBe(true);
  expect(sendWelcomeEmail).toHaveBeenCalled(); // also testing email here
});
```

**Better:**

```js
test('registerUser creates user with correct email', () => {
  const user = registerUser('john@example.com', 'password123');
  expect(user.email).toBe('john@example.com');
});

test('registerUser creates active user by default', () => {
  const user = registerUser('john@example.com', 'password123');
  expect(user.isActive).toBe(true);
});

test('registerUser sends welcome email', () => {
  registerUser('john@example.com', 'password123');
  expect(sendWelcomeEmail).toHaveBeenCalledWith('john@example.com');
});
```

## 2. Use Descriptive Test Names

A test name should describe what behavior we're testing. When a test fails, the name alone should tell us what went wrong.

**Bad example:**

```js
test('test1', () => { ... });
test('user test', () => { ... });
test('should work', () => { ... });
```

**Better:**

```js
test('should return null when user not found', () => { ... });
test('should throw error when email is invalid', () => { ... });
test('should calculate discount correctly for premium users', () => { ... });
```

The name should follow the pattern: **should [expected behavior] when [condition]**

## 3. Follow the AAA Pattern (Arrange, Act, Assert)

Structure our tests consistently. This makes them easy to read and scan.

```js
test('should calculate total correctly', () => {
  // Arrange
  const cart = { items: [{ price: 100 }, { price: 50 }] };
  
  // Act
  const total = calculateTotal(cart);
  
  // Assert
  expect(total).toBe(150);
});
```

This clear separation helps:

- Quickly identify what is being set up
- What action is being tested
- What the expected outcome is

## 4. Keep Tests Independent

Tests should not rely on each other or depend on execution order. Each test should be able to run in isolation, in parallel, or in any order.

**Bad example:**

```js
let user;

test('creates user', () => {
  user = createUser('john');
  expect(user.id).toBe(1);
});

test('updates user', () => {
  user.name = 'jane'; // depends on previous test!
  expect(user.name).toBe('jane');
});
```

**Better:**

```js
test('creates user with correct id', () => {
  const user = createUser('john');
  expect(user.id).toBe(1);
});

test('updates user name', () => {
  const user = createUser('john');
  const updated = updateUserName(user.id, 'jane');
  expect(updated.name).toBe('jane');
});
```

## 5. Avoid Test Logic in Tests

Don't put complex logic inside our tests. If we find ourself writing loops, conditionals, or helper functions in tests, we might be testing too much or need to refactor.

**Bad example:**

```js
test('calculates order total correctly', () => {
  const items = [{ price: 10 }, { price: 20 }, { price: 30 }];
  let total = 0;
  
  for (const item of items) {
    if (item.price > 15) {
      total += item.price * 0.9; // 10% discount
    } else {
      total += item.price;
    }
  }
  
  expect(total).toBe(75); // 20 + 30*0.9 + 10 = 57... wait, math is hard here
});
```

**Better:**

```js
test('applies 10% discount for items over $15', () => {
  const items = [{ price: 20 }];
  const total = calculateTotal(items);
  expect(total).toBe(18); // 20 * 0.9
});

test('no discount for items $15 or less', () => {
  const items = [{ price: 10 }];
  const total = calculateTotal(items);
  expect(total).toBe(10);
});
```

## 6. Use Meaningful Assertions

Make our assertions specific. Don't just check for truthy values — check for the exact expected result.

**Bad example:**

```js
expect(user).toBeTruthy();
expect(order).toBeDefined();
expect(items.length).toBeGreaterThan(0);
```

**Better:**

```js
expect(user.id).toBe('usr_123');
expect(order.status).toBe('pending');
expect(items).toHaveLength(3);
```

## 7. Test Edge Cases

Don't just test the happy path. Good tests cover:

- Empty inputs
- Null/undefined values
- Boundary conditions
- Error cases

```js
test('should handle empty cart', () => {
  expect(calculateTotal({ items: [] })).toBe(0);
});

test('should handle null items', () => {
  expect(calculateTotal({ items: null })).toBe(0);
});

test('should throw when price is negative', () => {
  expect(() => calculateTotal({ items: [{ price: -10 }] }))
    .toThrow('Price cannot be negative');
});
```

## 8. Don't Test Implementation Details

Test behavior, not implementation. This makes tests more resilient to refactoring.

**Bad example:**

```js
test('user repository uses cache', () => {
  const user = getUser('john');
  expect(cache.get).toHaveBeenCalled(); // tied to implementation
});
```

**Better:**

```js
test('getUser returns cached user on subsequent calls', () => {
  const user1 = getUser('john');
  const user2 = getUser('john');
  expect(user1).toBe(user2); // same object = caching works
});
```

## 9. Keep Test Code Clean

Apply the same clean code principles to tests:

- Small, focused test functions
- Descriptive names
- No code duplication
- Extract helpers when needed

```js
// Extract repeated setup
function createTestUser(overrides = {}) {
  return {
    name: 'John',
    email: 'john@example.com',
    isActive: true,
    ...overrides
  };
}

test('should update user email', () => {
  const user = createTestUser();
  const updated = updateEmail(user.id, 'new@example.com');
  expect(updated.email).toBe('new@example.com');
});

test('should reject invalid email', () => {
  const user = createTestUser({ email: 'invalid' });
  expect(() => validateUser(user)).toThrow('Invalid email');
});
```

## 10. Tests Should Be Fast

Slow tests get ignored. Keep our tests fast:

- Mock slow dependencies (databases, APIs, file system)
- Use in-memory databases for testing
- Avoid unnecessary waits or sleeps
- Clean up after yourself

```js
test('should save user to database', async () => {
  const mockDb = jest.fn(() => Promise.resolve({ id: 1 }));
  
  await saveUser({ name: 'John' }, mockDb);
  
  expect(mockDb).toHaveBeenCalled();
});
```

---

## Summary

| Tip | Key Takeaway |
|-----|---------------|
| One Thing Per Test | Each test verifies single behavior |
| Descriptive Names | Name describes expected behavior |
| AAA Pattern | Arrange, Act, Assert |
| Independent Tests | No dependencies between tests |
| No Test Logic | Keep tests simple and focused |
| Meaningful Assertions | Check exact expected values |
| Test Edge Cases | Cover happy path and edge cases |
| Test Behavior | Don't test implementation details |
| Clean Test Code | Apply clean code principles to tests |
| Fast Tests | Keep tests quick to run |

Clean tests are the safety net that allows our codebase to evolve confidently. Invest time in writing good tests, and our future self will thank you.
