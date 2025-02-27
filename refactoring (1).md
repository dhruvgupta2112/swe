# Refactoring Examples

## Extract Method

Refactor long methods into smaller, more manageable ones.

### Before Refactoring:

```c#
public void ProcessOrder(Order order)
{
    Console.WriteLine("Starting order processing...");
    // Validate order
    if (order == null || order.Items.Count == 0)
    {
        Console.WriteLine("Invalid order.");
        return;
    }

    // Calculate total
    decimal total = 0;
    foreach (var item in order.Items)
    {
        total += item.Price * item.Quantity;
    }

    Console.WriteLine($"Order total: {total}");
    Console.WriteLine("Order processed successfully.");
}
```

### After Refactoring:
```c#
public void ProcessOrder(Order order)
{
    Console.WriteLine("Starting order processing...");

    if (!ValidateOrder(order)) return;

    decimal total = CalculateOrderTotal(order);
    Console.WriteLine($"Order total: {total}");
    Console.WriteLine("Order processed successfully.");
}

private bool ValidateOrder(Order order)
{
    if (order == null || order.Items.Count == 0)
    {
        Console.WriteLine("Invalid order.");
        return false;
    }
    return true;
}

private decimal CalculateOrderTotal(Order order)
{
    return order.Items.Sum(item => item.Price * item.Quantity);
}
```

## Replace Magic Numbers with Constants

Improves code readability and maintainability.

### Before Refactoring:
```c#
public decimal CalculateDiscount(decimal amount)
{
    if (amount > 1000)
    {
        return amount * 0.1m; // 10% discount
    }
    return 0;
}
```

### After Refactoring:

```c#
public class DiscountCalculator
{
    private const decimal DiscountThreshold = 1000m;
    private const decimal DiscountRate = 0.1m;

    public decimal CalculateDiscount(decimal amount)
    {
        if (amount > DiscountThreshold)
        {
            return amount * DiscountRate;
        }
        return 0;
    }
}
```

## Introduce Parameter Object

Replace multiple parameters with a single object when they logically belong together.

### Before Refactoring:
```c#
public void CreateUser(string firstName, string lastName, string email)
{
    Console.WriteLine($"Creating user: {firstName} {lastName}, Email: {email}");
}
```

### After Refactoring:
```c#
public class User
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}

public void CreateUser(User user)
{
    Console.WriteLine($"Creating user: {user.FirstName} {user.LastName}, Email: {user.Email}");
}
```

## Replace Nested Conditionals with Guard Clauses

Simplifies code by reducing nesting levels.

### Before Refactoring:
```c#
public void ProcessPayment(decimal amount)
{
    if (amount > 0)
    {
        if (amount < 10000)
        {
            Console.WriteLine("Payment processed.");
        }
        else
        {
            Console.WriteLine("Payment exceeds the limit.");
        }
    }
    else
    {
        Console.WriteLine("Invalid payment amount.");
    }
}
```

### After Refactoring:
```c#
public void ProcessPayment(decimal amount)
{
    if (amount <= 0)
    {
        Console.WriteLine("Invalid payment amount.");
        return;
    }

    if (amount >= 10000)
    {
        Console.WriteLine("Payment exceeds the limit.");
        return;
    }

    Console.WriteLine("Payment processed.");
}
```

## Encapsulate Collection

Avoid exposing raw collections to provide better control over access and manipulation.

### Before Refactoring:
```c#
public class Team
{
    public List<string> Members = new List<string>();
}
```
### After Refactoring:
```c#
public class Team
{
    private readonly List<string> _members = new List<string>();

    public IReadOnlyList<string> Members => _members;

    public void AddMember(string member)
    {
        _members.Add(member);
    }

    public void RemoveMember(string member)
    {
        _members.Remove(member);
    }
}
```

## Use Polymorphism Instead of Conditionals

Scenario: Calculating Shipping Costs

### Before Refactoring: Using Conditionals
```c#
public class ShippingCostCalculator
{
    public decimal CalculateShippingCost(string shippingMethod, decimal weight)
    {
        if (shippingMethod == "Standard")
        {
            return weight * 1.0m; // Standard: $1.00 per kg
        }
        else if (shippingMethod == "Express")
        {
            return weight * 2.0m; // Express: $2.00 per kg
        }
        else if (shippingMethod == "Overnight")
        {
            return weight * 3.0m; // Overnight: $3.00 per kg
        }
        else
        {
            throw new ArgumentException("Unsupported shipping method.");
        }
    }
}

// Usage
var calculator = new ShippingCostCalculator();
decimal cost = calculator.CalculateShippingCost("Express", 5);
Console.WriteLine($"Shipping Cost: {cost}");
```

### After Refactoring: Using Polymorphism

We refactor the if-else logic into polymorphic behavior by creating a base class for shipping methods and specific implementations for each type.

```c#
// Base Class:

public abstract class ShippingMethod
{
    public abstract decimal CalculateCost(decimal weight);
}

// Concrete Implementations:

public class StandardShipping : ShippingMethod
{
    public override decimal CalculateCost(decimal weight) => weight * 1.0m;
}

public class ExpressShipping : ShippingMethod
{
    public override decimal CalculateCost(decimal weight) => weight * 2.0m;
}

public class OvernightShipping : ShippingMethod
{
    public override decimal CalculateCost(decimal weight) => weight * 3.0m;
}
```

**Factory for Shipping Methods:**
To dynamically select the shipping method, we use a factory pattern.

```c#
public static class ShippingMethodFactory
{
    public static ShippingMethod Create(string shippingMethod)
    {
        return shippingMethod switch
        {
            "Standard" => new StandardShipping(),
            "Express" => new ExpressShipping(),
            "Overnight" => new OvernightShipping(),
            _ => throw new ArgumentException("Unsupported shipping method.")
        };
    }
}

// Refactored Calculator:
public class ShippingCostCalculator
{
    public decimal CalculateShippingCost(string shippingMethod, decimal weight)
    {
        var method = ShippingMethodFactory.Create(shippingMethod);
        return method.CalculateCost(weight);
    }
}

// Usage:
var calculator = new ShippingCostCalculator();
decimal cost = calculator.CalculateShippingCost("Express", 5);
Console.WriteLine($"Shipping Cost: {cost}");
```

#### Benefits of the Refactored Design

1.	Extensibility:
Adding a new shipping method (e.g., “DroneShipping”) only requires creating a new class that inherits from ShippingMethod. No changes are needed in the calculator or factory.
2.	Single Responsibility:
Each shipping method is encapsulated in its own class, adhering to the Single Responsibility Principle (SRP).
3.	Readability:
The code is easier to read and reason about, as each shipping method’s logic is clearly separated.
4.	Open-Closed Principle:
The system is open for extension but closed for modification, a core principle of robust software design.


# Non-standard refactoring examples for tackling real-world issues in unique ways.
 

## Problem: “Replace Hardcoded Rules with a Configuration-Driven System”

A system uses hardcoded logic for business rules, making it difficult to update rules without redeploying the application.

### Before Refactoring:

```c#
public decimal CalculateTax(string region, decimal amount)
{
    if (region == "US")
    {
        return amount * 0.07m; // 7% tax
    }
    else if (region == "EU")
    {
        return amount * 0.2m; // 20% tax
    }
    else if (region == "AU")
    {
        return amount * 0.1m; // 10% tax
    }
    else
    {
        throw new ArgumentException("Unsupported region");
    }
}
```
### After Refactoring:

Load tax rules dynamically from a configuration file or database.
```c#
public class TaxCalculator
{
    private readonly Dictionary<string, decimal> _taxRates;

    public TaxCalculator(Dictionary<string, decimal> taxRates)
    {
        _taxRates = taxRates;
    }

    public decimal CalculateTax(string region, decimal amount)
    {
        if (!_taxRates.TryGetValue(region, out var taxRate))
        {
            throw new ArgumentException("Unsupported region");
        }
        return amount * taxRate;
    }
}

// Configuration
var taxRates = new Dictionary<string, decimal>
{
    { "US", 0.07m },
    { "EU", 0.2m },
    { "AU", 0.1m }
};

// Usage
var calculator = new TaxCalculator(taxRates);
decimal tax = calculator.CalculateTax("US", 1000);
```

## Problem: “Replace If-Else Chain with Metadata-Based Strategy”

A system uses a long if-else chain to determine actions based on user roles.

### Before Refactoring:
```c#
public string GetDashboard(User user)
{
    if (user.Role == "Admin")
    {
        return "Admin Dashboard";
    }
    else if (user.Role == "Manager")
    {
        return "Manager Dashboard";
    }
    else if (user.Role == "Employee")
    {
        return "Employee Dashboard";
    }
    else
    {
        return "Guest Dashboard";
    }
}
```
### After Refactoring:

Use a dictionary or metadata to map roles to their actions.
```c#
public class DashboardProvider
{
    private readonly Dictionary<string, string> _dashboards = new()
    {
        { "Admin", "Admin Dashboard" },
        { "Manager", "Manager Dashboard" },
        { "Employee", "Employee Dashboard" },
        { "Guest", "Guest Dashboard" }
    };

    public string GetDashboard(User user)
    {
        return _dashboards.TryGetValue(user.Role, out var dashboard)
            ? dashboard
            : "Default Dashboard";
    }
}

// Usage
var provider = new DashboardProvider();
string dashboard = provider.GetDashboard(user);
```

## Problem: “Extract Workflow to a Pipeline Pattern”

You have a complex workflow with multiple sequential steps. Adding or reordering steps requires modifying core logic.

### Before Refactoring:
```c#
public void ProcessData(Data data)
{
    // Step 1
    CleanData(data);
    // Step 2
    TransformData(data);
    // Step 3
    ValidateData(data);
    // Step 4
    SaveData(data);
}
```
### After Refactoring:

Refactor the workflow into a pipeline of steps that can be extended or reordered dynamically.
```c#
public interface IDataProcessorStep
{
    void Execute(Data data);
}

public class DataProcessingPipeline
{
    private readonly List<IDataProcessorStep> _steps = new();

    public void AddStep(IDataProcessorStep step)
    {
        _steps.Add(step);
    }

    public void Execute(Data data)
    {
        foreach (var step in _steps)
        {
            step.Execute(data);
        }
    }
}

// Example Steps
public class CleanDataStep : IDataProcessorStep
{
    public void Execute(Data data) => Console.WriteLine("Cleaning data...");
}

public class TransformDataStep : IDataProcessorStep
{
    public void Execute(Data data) => Console.WriteLine("Transforming data...");
}

public class ValidateDataStep : IDataProcessorStep
{
    public void Execute(Data data) => Console.WriteLine("Validating data...");
}

// Usage
var pipeline = new DataProcessingPipeline();
pipeline.AddStep(new CleanDataStep());
pipeline.AddStep(new TransformDataStep());
pipeline.AddStep(new ValidateDataStep());
pipeline.Execute(new Data());
```

## Problem: “Simplify Event-Driven Code with Reactive Extensions (Rx)”

A system has complex event-handling code, often leading to race conditions and hard-to-follow logic.

### Before Refactoring:
```c#
public void MonitorSensor(Sensor sensor)
{
    sensor.OnTemperatureChanged += (sender, temp) =>
    {
        if (temp > 100)
        {
            Console.WriteLine("Temperature is too high!");
        }
    };

    sensor.OnPressureChanged += (sender, pressure) =>
    {
        if (pressure > 200)
        {
            Console.WriteLine("Pressure is too high!");
        }
    };
}
```

### After Refactoring:

Use Reactive Extensions (Rx) for declarative event handling.
```c#
public void MonitorSensor(Sensor sensor)
{
    var temperatureStream = Observable.FromEventPattern<TemperatureChangedEventArgs>(
        h => sensor.OnTemperatureChanged += h,
        h => sensor.OnTemperatureChanged -= h
    );

    var pressureStream = Observable.FromEventPattern<PressureChangedEventArgs>(
        h => sensor.OnPressureChanged += h,
        h => sensor.OnPressureChanged -= h
    );

    temperatureStream
        .Where(e => e.EventArgs.Temperature > 100)
        .Subscribe(e => Console.WriteLine("Temperature is too high!"));

    pressureStream
        .Where(e => e.EventArgs.Pressure > 200)
        .Subscribe(e => Console.WriteLine("Pressure is too high!"));
}
```

## Problem : “Replace Procedural Reporting Logic with Report Builder”

You generate reports with repetitive code for formatting and data aggregation.

## Before Refactoring:
```c#
public void GenerateReport()
{
    Console.WriteLine("Report Header");
    Console.WriteLine("-------------");
    Console.WriteLine("Data: 123");
    Console.WriteLine("Total: $456");
}
```

### After Refactoring:

Introduce a report builder to encapsulate the logic.
```c#
public class ReportBuilder
{
    private readonly StringBuilder _report = new();

    public ReportBuilder AddHeader(string header)
    {
        _report.AppendLine(header);
        _report.AppendLine(new string('-', header.Length));
        return this;
    }

    public ReportBuilder AddSection(string content)
    {
        _report.AppendLine(content);
        return this;
    }

    public string Build() => _report.ToString();
}

// Usage
var report = new ReportBuilder()
    .AddHeader("Report Header")
    .AddSection("Data: 123")
    .AddSection("Total: $456")
    .Build();

Console.WriteLine(report);
```

