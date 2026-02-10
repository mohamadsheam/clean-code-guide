# Tips on Writing Clean Classes

Clean classes are the backbone of maintainable object-oriented code. A well-designed class is easy to understand, test, and extend. Here are the key principles to follow.

## 1. Single Responsibility Principle (SRP)

A class should have **one reason to change**. If you can think of multiple reasons why a class might need modification, it's doing too much.

**Bad Example:**

```php
class User
{
    public function __construct(
        private string $name,
        private string $email
    ) {}

    public function save(): void
    {
        // SQL logic to save user to database
        $pdo = new PDO('mysql:host=localhost;dbname=app', 'root', '');
        $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (?, ?)');
        $stmt->execute([$this->name, $this->email]);
    }

    public function sendWelcomeEmail(): void
    {
        // Email sending logic
        mail($this->email, 'Welcome!', 'Thanks for joining us.');
    }

    public function generateReport(): string
    {
        // Report generation logic
        return "User Report: {$this->name}, {$this->email}";
    }
}
```

This class has **four responsibilities**: representing user data, database persistence, email sending, and report generation.

**Better:**

```php
class User
{
    public function __construct(
        private string $name,
        private string $email
    ) {}

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): string
    {
        return $this->email;
    }
}

class UserRepository
{
    public function __construct(private PDO $pdo) {}

    public function save(User $user): void
    {
        $stmt = $this->pdo->prepare('INSERT INTO users (name, email) VALUES (?, ?)');
        $stmt->execute([$user->getName(), $user->getEmail()]);
    }
}

class WelcomeEmailService
{
    public function send(User $user): void
    {
        mail($user->getEmail(), 'Welcome!', 'Thanks for joining us.');
    }
}

class UserReportGenerator
{
    public function generate(User $user): string
    {
        return "User Report: {$user->getName()}, {$user->getEmail()}";
    }
}
```

Now each class has a single, clear purpose.

## 2. Keep Classes Small and Focused

Small classes are easier to understand, test, and maintain. If a class has grown too large, it's usually violating SRP.

**Signs your class is too big:**

- You need to scroll extensively to read it
- It has many private helper methods
- It's hard to name without using "And" or "Or"
- It has multiple groups of methods that don't interact with each other

**Bad Example:**

```php
class OrderManager
{
    public function createOrder() { /* ... */ }
    public function updateOrder() { /* ... */ }
    public function deleteOrder() { /* ... */ }
    public function calculateTax() { /* ... */ }
    public function applyDiscount() { /* ... */ }
    public function generateInvoice() { /* ... */ }
    public function sendConfirmationEmail() { /* ... */ }
    public function updateInventory() { /* ... */ }
    public function processPayment() { /* ... */ }
    public function refundPayment() { /* ... */ }
    // ... 20 more methods
}
```

**Better:**

```php
class Order { /* order data and state */ }
class OrderRepository { /* CRUD operations */ }
class TaxCalculator { /* tax logic */ }
class DiscountService { /* discount logic */ }
class InvoiceGenerator { /* invoice creation */ }
class OrderNotificationService { /* email notifications */ }
class InventoryService { /* stock management */ }
class PaymentProcessor { /* payment handling */ }
```

## 3. Favor Composition Over Inheritance

Inheritance creates tight coupling between classes. Composition gives you more flexibility and makes your code easier to change.

**Bad Example:**

```php
class Animal
{
    public function eat(): void { /* ... */ }
    public function sleep(): void { /* ... */ }
    public function fly(): void { /* ... */ }  // Not all animals fly!
}

class Dog extends Animal
{
    // Dog inherits fly() but dogs can't fly
    // We have to override it or leave it broken
    public function fly(): void
    {
        throw new Exception("Dogs can't fly!");
    }
}
```

**Better:**

```php
interface Eater
{
    public function eat(): void;
}

interface Sleeper
{
    public function sleep(): void;
}

interface Flyer
{
    public function fly(): void;
}

class Dog implements Eater, Sleeper
{
    public function eat(): void { /* ... */ }
    public function sleep(): void { /* ... */ }
    // No fly() method - dogs simply don't have this capability
}

class Bird implements Eater, Sleeper, Flyer
{
    public function eat(): void { /* ... */ }
    public function sleep(): void { /* ... */ }
    public function fly(): void { /* ... */ }
}
```

## 4. Depend on Abstractions, Not Concretions

Classes should depend on interfaces or abstract classes rather than concrete implementations. This makes your code more flexible and testable.

**Bad Example:**

```php
class OrderService
{
    private MySQLDatabase $database;
    private SmtpMailer $mailer;

    public function __construct()
    {
        $this->database = new MySQLDatabase();
        $this->mailer = new SmtpMailer();
    }

    public function placeOrder(Order $order): void
    {
        $this->database->save($order);
        $this->mailer->send($order->getCustomerEmail(), 'Order placed!');
    }
}
```

Problems:

- Can't swap MySQL for PostgreSQL without modifying this class
- Can't test without a real database and mail server
- Tightly coupled to specific implementations

**Better:**

```php
interface DatabaseInterface
{
    public function save(object $entity): void;
}

interface MailerInterface
{
    public function send(string $to, string $message): void;
}

class OrderService
{
    public function __construct(
        private DatabaseInterface $database,
        private MailerInterface $mailer
    ) {}

    public function placeOrder(Order $order): void
    {
        $this->database->save($order);
        $this->mailer->send($order->getCustomerEmail(), 'Order placed!');
    }
}

// Now you can inject any implementation
$service = new OrderService(
    new MySQLDatabase(),
    new SmtpMailer()
);

// Or use mocks for testing
$service = new OrderService(
    new InMemoryDatabase(),
    new FakeMailer()
);
```

## 5. Use Constructor Injection for Dependencies

Dependencies should be passed through the constructor, not created inside the class or fetched from global state.

**Bad Example:**

```php
class InvoiceGenerator
{
    public function generate(Order $order): Invoice
    {
        $taxCalculator = new TaxCalculator();  // Hidden dependency
        $config = Config::getInstance();        // Global state
        $logger = Logger::getLogger();          // Static call

        $tax = $taxCalculator->calculate($order);
        // ...
    }
}
```

**Better:**

```php
class InvoiceGenerator
{
    public function __construct(
        private TaxCalculator $taxCalculator,
        private Config $config,
        private LoggerInterface $logger
    ) {}

    public function generate(Order $order): Invoice
    {
        $tax = $this->taxCalculator->calculate($order);
        // ... all dependencies are explicit and testable
    }
}
```

Benefits:

- Dependencies are explicit and visible
- Easy to mock for testing
- Class is honest about what it needs

## 6. Encapsulate What Varies

Hide the parts of your code that are likely to change behind stable interfaces. This protects the rest of your system from ripple effects.

**Bad Example:**

```php
class ShippingCostCalculator
{
    public function calculate(Order $order): float
    {
        if ($order->getShippingMethod() === 'standard') {
            return $order->getWeight() * 1.5;
        } elseif ($order->getShippingMethod() === 'express') {
            return $order->getWeight() * 3.0 + 5.0;
        } elseif ($order->getShippingMethod() === 'overnight') {
            return $order->getWeight() * 5.0 + 15.0;
        }
        // Adding new shipping methods requires modifying this class
    }
}
```

**Better:**

```php
interface ShippingStrategy
{
    public function calculate(Order $order): float;
}

class StandardShipping implements ShippingStrategy
{
    public function calculate(Order $order): float
    {
        return $order->getWeight() * 1.5;
    }
}

class ExpressShipping implements ShippingStrategy
{
    public function calculate(Order $order): float
    {
        return $order->getWeight() * 3.0 + 5.0;
    }
}

class OvernightShipping implements ShippingStrategy
{
    public function calculate(Order $order): float
    {
        return $order->getWeight() * 5.0 + 15.0;
    }
}

class ShippingCostCalculator
{
    public function __construct(private ShippingStrategy $strategy) {}

    public function calculate(Order $order): float
    {
        return $this->strategy->calculate($order);
    }
}
```

Now adding a new shipping method doesn't require changing existing code.

## 7. Make Classes Easy to Test

A class that's hard to test is usually poorly designed. Testable classes have:

- Dependencies injected (not created internally)
- No static method calls or global state
- Single responsibility
- Clear inputs and outputs

**Bad Example:**

```php
class ReportService
{
    public function generateMonthlyReport(): string
    {
        $date = new DateTime();  // Untestable - always "now"
        $data = Database::query('SELECT * FROM sales');  // Static call
        file_put_contents('/var/reports/monthly.pdf', $data);  // Side effect
        return '/var/reports/monthly.pdf';
    }
}
```

**Better:**

```php
interface Clock
{
    public function now(): DateTimeInterface;
}

interface SalesRepository
{
    public function getSalesForMonth(DateTimeInterface $month): array;
}

interface FileStorage
{
    public function write(string $path, string $content): void;
}

class ReportService
{
    public function __construct(
        private Clock $clock,
        private SalesRepository $sales,
        private FileStorage $storage
    ) {}

    public function generateMonthlyReport(): string
    {
        $date = $this->clock->now();
        $data = $this->sales->getSalesForMonth($date);
        $path = "/var/reports/monthly-{$date->format('Y-m')}.pdf";
        $this->storage->write($path, $this->formatReport($data));
        return $path;
    }

    private function formatReport(array $data): string { /* ... */ }
}
```

Now you can inject a fake clock, in-memory repository, and mock storage for testing.

## 8. Avoid God Classes

A "God Class" knows too much or does too much. It becomes a maintenance nightmare where every change risks breaking something.

**Warning signs:**

- Class name contains "Manager", "Handler", "Processor", "Helper", or "Utility"
- More than ~200 lines of code
- More than ~10 public methods
- Dependencies on many other classes
- Used by almost every other class in the system

**Solution:** Break the God Class into smaller, focused classes. Use the SRP as your guide.

## 9. Use Value Objects for Related Data

Instead of passing around primitive types, group related data into small, immutable value objects.

**Bad Example:**

```php
class Order
{
    public function setShippingAddress(
        string $street,
        string $city,
        string $state,
        string $zipCode,
        string $country
    ): void {
        // ...
    }
}
```

**Better:**

```php
class Address
{
    public function __construct(
        public readonly string $street,
        public readonly string $city,
        public readonly string $state,
        public readonly string $zipCode,
        public readonly string $country
    ) {}

    public function format(): string
    {
        return "{$this->street}, {$this->city}, {$this->state} {$this->zipCode}, {$this->country}";
    }
}

class Order
{
    public function setShippingAddress(Address $address): void
    {
        // ...
    }
}
```

Benefits:

- Validation can live in the value object
- Related behavior stays with the data
- Impossible to mix up parameter order

---

## Summary

| Principle | Key Takeaway  |
|-----------|---------------|
| Single Responsibility | One class, one reason to change |
| Keep Classes Small | If it's hard to name, it's too big |
| Composition Over Inheritance | Prefer "has-a" over "is-a" |
| Depend on Abstractions | Code to interfaces, not implementations |
| Constructor Injection | Make dependencies explicit |
| Encapsulate What Varies | Hide change behind interfaces |
| Design for Testability | If it's hard to test, redesign it |
| Avoid God Classes | Break down large, all-knowing classes |
| Use Value Objects | Group related primitives together |

Clean classes lead to clean systems. Apply these principles consistently, and your codebase will be easier to understand, test, and evolve.
