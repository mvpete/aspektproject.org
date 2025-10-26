---
title: 'Advanced Topics'
type: "page"
layout: "documentation"
menu: "documentation"
weight: 10
---

# Advanced Topics

## Return Value Interception

Implement `IAspectExitHandler<T>` to intercept and modify return values:

```csharp
using Aspekt;

public class ValidationAspect : Aspect, IAspectExitHandler<string>
{
    public string OnExit(MethodArguments args, string result)
    {
        // Validate or transform the return value
        if (string.IsNullOrEmpty(result))
        {
            return "Default Value";
        }
        return result.Trim();
    }
}

public class DataService
{
    [ValidationAspect]
    public string GetUserName(int userId)
    {
        // Your implementation
        return "  John Doe  ";
    }
    // Returns: "John Doe" (trimmed)
}
```

## Multiple Aspect Handlers

A single aspect can handle multiple return types:

```csharp
public class CachingAspect : Aspect, 
    IAspectExitHandler<string>, 
    IAspectExitHandler<int>
{
    private static Dictionary<string, object> cache = new();

    public string OnExit(MethodArguments args, string result)
    {
        cache[args.FullName] = result;
        return result;
    }

    public int OnExit(MethodArguments args, int result)
    {
        cache[args.FullName] = result;
        return result;
    }
}
```

## Async/Await Patterns

Aspekt 2.0 provides comprehensive async support:

### Async Interception

```csharp
public class AsyncTimingAspect : Aspect
{
    private readonly Dictionary<string, Stopwatch> timers = new();

    public override async ValueTask OnEntryAsync(
        MethodArguments args, 
        CancellationToken cancellationToken = default)
    {
        var timer = Stopwatch.StartNew();
        timers[args.FullName] = timer;
        await Console.Out.WriteLineAsync(
            $"[{DateTime.Now:HH:mm:ss}] Starting async operation: {args.Name}");
    }

    public override async ValueTask OnExitAsync(
        MethodArguments args, 
        CancellationToken cancellationToken = default)
    {
        if (timers.TryGetValue(args.FullName, out var timer))
        {
            timer.Stop();
            await Console.Out.WriteLineAsync(
                $"[{DateTime.Now:HH:mm:ss}] Completed {args.Name} in {timer.ElapsedMilliseconds}ms");
            timers.Remove(args.FullName);
        }
    }
}
```

### Task Continuations

```csharp
public class TaskContinuationAspect : Aspect
{
    public override void OnExit(MethodArguments args)
    {
        // This works with both sync and async methods
        Console.WriteLine($"Method {args.Name} completed");
    }
}

public class AsyncService
{
    [TaskContinuationAspect]
    public async Task<string> FetchDataAsync()
    {
        await Task.Delay(100);
        return "Data";
    }
    // OnExit is called after the task completes
}
```

## Contract Programming

### Precondition Validation

Validate method inputs using the `RequiresArgumentAttribute`:

```csharp
using Aspekt.Contracts;

public class PaymentService
{
    // Validate amount is positive
    [RequiresArgument("amount", typeof(decimal), 
        Contract.Comparison.GreaterThan, 0)]
    public void ProcessPayment(decimal amount, string currency)
    {
        // amount is guaranteed to be > 0
    }

    // Validate string is not null or empty
    [RequiresArgument("email", typeof(string), 
        Contract.Constraint.NotNullOrEmpty)]
    public void SendReceipt(string email)
    {
        // email is guaranteed to be valid
    }

    // Multiple constraints
    [RequiresArgument("age", typeof(int), 
        Contract.Comparison.GreaterThanOrEqual, 18)]
    [RequiresArgument("age", typeof(int), 
        Contract.Comparison.LessThan, 120)]
    public void RegisterAdult(string name, int age)
    {
        // age is guaranteed to be between 18 and 119
    }
}
```

### Postcondition Validation

Use `EnsureAttribute` to validate method results:

```csharp
using Aspekt.Contracts;

public class InventoryService
{
    [Ensure(typeof(int), Contract.Comparison.GreaterThanOrEqual, 0)]
    public int GetStockLevel(string productId)
    {
        // Return value must be >= 0
        return Database.GetStock(productId);
    }
}
```

### Class Invariants

Maintain class invariants across method calls:

```csharp
using Aspekt.Contracts;

public class BankAccount
{
    [FieldInvariant(Contract.Comparison.GreaterThanOrEqual, 0)]
    private decimal balance = 0;

    public void Withdraw(decimal amount)
    {
        balance -= amount;
        // Invariant automatically checked after method
        // Will throw if balance < 0
    }

    public void Deposit(decimal amount)
    {
        balance += amount;
        // Invariant checked here too
    }
}
```

## Performance Optimization

### Conditional Weaving

Skip aspect execution based on conditions:

```csharp
public class ConditionalLoggingAspect : Aspect
{
    private readonly bool enabled;

    public ConditionalLoggingAspect(bool enabled = true)
    {
        this.enabled = enabled;
    }

    public override void OnEntry(MethodArguments args)
    {
        if (!enabled) return;
        
        Console.WriteLine($"Entering: {args.Name}");
    }
}

// Enable/disable per method
[ConditionalLogging(enabled: false)]
public void HighPerformanceMethod() { }
```

### Aspect Composition

Combine aspects efficiently:

```csharp
// Base aspect for common functionality
public abstract class BaseAspect : Aspect
{
    protected void LogMessage(string message)
    {
        Console.WriteLine($"[{DateTime.Now:HH:mm:ss.fff}] {message}");
    }
}

// Specialized aspects inherit common functionality
public class DetailedLoggingAspect : BaseAspect
{
    public override void OnEntry(MethodArguments args)
    {
        LogMessage($"Entering {args.Name} with {args.Arguments.Count} arguments");
    }
}

public class SimpleLoggingAspect : BaseAspect
{
    public override void OnEntry(MethodArguments args)
    {
        LogMessage($"Entering {args.Name}");
    }
}
```

## Testing with Aspects

### Unit Testing Aspects

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

[TestClass]
public class LoggingAspectTests
{
    [TestMethod]
    public void OnEntry_LogsMethodName()
    {
        // Arrange
        var aspect = new LoggingAspect();
        var args = new MethodArguments("TestMethod", "Full.TestMethod", 
            new Arguments(Array.Empty<object>()), null);
        
        using var writer = new StringWriter();
        Console.SetOut(writer);
        
        // Act
        aspect.OnEntry(args);
        
        // Assert
        Assert.IsTrue(writer.ToString().Contains("TestMethod"));
    }
}
```

### Integration Testing

```csharp
[TestClass]
public class AspectIntegrationTests
{
    [TestMethod]
    public void WovenMethod_CallsAspect()
    {
        // Arrange
        var service = new MyService();
        
        // Act - method has aspect applied
        var result = service.ProcessData(42);
        
        // Assert - verify aspect was executed
        Assert.IsNotNull(result);
        Assert.AreEqual(42, MyLoggingAspect.LastCapturedArgument);
    }
}
```

## Custom Aspect Attributes

Create reusable aspect configurations:

```csharp
// Custom attribute that applies multiple aspects
[AttributeUsage(AttributeTargets.Method)]
public class AuditableAttribute : Attribute
{
    // This could be expanded to apply multiple aspects
}

// Specialized logging attribute
public class DebugLogAttribute : LoggingAspect
{
    public DebugLogAttribute() : base(
        logLevel: Levels.Debug,
        logParameters: true,
        logExecutionTime: true)
    {
    }
}

// Usage
[DebugLog]
public void ComplexOperation(int value)
{
    // Automatically logged with debug level
}
```

## Best Practices

### 1. Keep Aspects Focused
Each aspect should handle one concern:
```csharp
// Good: Focused on one responsibility
public class PerformanceLoggingAspect : Aspect { }
public class SecurityAuditAspect : Aspect { }

// Avoid: Multiple responsibilities
public class EverythingAspect : Aspect { } // Don't do this
```

### 2. Handle Exceptions Carefully
```csharp
public class SafeAspect : Aspect
{
    public override void OnException(MethodArguments args, Exception e)
    {
        try
        {
            // Log or handle the exception
            LogException(e);
        }
        catch
        {
            // Never throw from OnException
            // The original exception will be re-thrown
        }
    }
}
```

### 3. Consider Performance Impact
```csharp
public class EfficientAspect : Aspect
{
    // Cache expensive operations
    private static readonly ConcurrentDictionary<string, string> cache = new();

    public override void OnEntry(MethodArguments args)
    {
        // Minimal overhead
        if (IsLoggingEnabled)
        {
            QuickLog(args.Name);
        }
    }
}
```

### 4. Use Dependency Injection
```csharp
public class InjectedLoggingAspect : Aspect
{
    private readonly ILogger logger;

    public InjectedLoggingAspect(ILogger logger)
    {
        this.logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public override void OnEntry(MethodArguments args)
    {
        logger.LogInformation($"Entering {args.Name}");
    }
}
```

## Troubleshooting

### Common Issues

**Aspect Not Executing**
- Verify the project builds successfully
- Check that the method is not inlined by the compiler
- Ensure the aspect class is public and derives from `Aspect`

**Performance Issues**
- Profile aspect overhead with a simple stopwatch
- Consider conditional weaving for high-frequency methods
- Cache expensive operations in aspect constructors

**Async Issues**
- Use `OnEntryAsync`/`OnExitAsync` for async-specific logic
- Remember that `OnExit` is called after async methods complete
- Handle cancellation tokens properly

## Further Reading

- [Examples](/examples) - Comprehensive code examples
- [Getting Started](/documentation/getting-started) - Basic setup and usage
- [GitHub Repository](https://github.com/mvpete/aspekt) - Source code and issues
- [NuGet Package](https://www.nuget.org/packages/Aspekt/) - Latest releases
