---
title: "Examples"
menu : "main"
type: "page"
layout: "simple"
---

{{% jumbotron %}}

# Examples
Get going with comprehensive examples that show you the power of aspect oriented programming with modern .NET 8.0/6.0.

{{< button href= "https://github.com/mvpete/aspekt" class= "btn-outline-primary" >}}Download Now{{</ button >}}

{{% /jumbotron %}}

{{% content %}}

## Getting Started with Aspekt

Aspekt is now modernized for .NET 8.0, .NET 6.0, and .NET Standard 2.1, bringing you the latest performance improvements and modern C# features.

### Installation

Install Aspekt via NuGet:

{{< highlight bash >}}
dotnet add package Aspekt
{{< /highlight >}}

---

## Basic Logging Aspect

Create your first aspect to add logging to any method:

{{< highlight csharp >}}
using Aspekt;

public class LoggingAspect : Aspect
{
    public override void OnEntry(MethodArguments args)
    {
        Console.WriteLine($"Entering: {args.FullName}");
        if (args.Arguments.Count > 0)
        {
            Console.WriteLine($"  Parameters: [{string.Join(", ", args.Arguments.Values)}]");
        }
    }

    public override void OnExit(MethodArguments args)
    {
        Console.WriteLine($"Exiting: {args.FullName}");
    }

    public override void OnException(MethodArguments args, Exception e)
    {
        Console.WriteLine($"Exception in {args.FullName}: {e.Message}");
    }
}

// Apply the aspect to your methods
public class Calculator
{
    [LoggingAspect]
    public int Add(int a, int b)
    {
        return a + b;
    }
}
{{< /highlight >}}

---

## Performance Monitoring

Track execution time of your methods:

{{< highlight csharp >}}
using Aspekt;
using System.Diagnostics;

public class PerformanceMonitoringAspect : Aspect
{
    private Stopwatch? _stopwatch;

    public override void OnEntry(MethodArguments args)
    {
        _stopwatch = Stopwatch.StartNew();
    }

    public override void OnExit(MethodArguments args)
    {
        _stopwatch?.Stop();
        Console.WriteLine($"{args.Name} executed in {_stopwatch?.ElapsedMilliseconds}ms");
    }
}

public class DataProcessor
{
    [PerformanceMonitoring]
    public void ProcessLargeDataSet(int[] data)
    {
        // Your processing logic here
        Array.Sort(data);
    }
}
{{< /highlight >}}

---

## Contract Programming - Input Validation

Use built-in contract aspects to enforce preconditions:

{{< highlight csharp >}}
using Aspekt.Contracts;

public class UserService
{
    // Validate that age parameter is greater than 0
    [RequiresArgument("age", typeof(int), Contract.Comparison.GreaterThan, 0)]
    public void RegisterUser(string name, int age)
    {
        // Registration logic - age is guaranteed to be > 0
        Console.WriteLine($"Registering user: {name}, age: {age}");
    }

    // Validate that email is not null or empty
    [RequiresArgument("email", typeof(string), Contract.Constraint.NotNullOrEmpty)]
    public void SendEmail(string email, string message)
    {
        // Email logic - email is guaranteed to be valid
        Console.WriteLine($"Sending email to: {email}");
    }
}
{{< /highlight >}}

---

## Return Value Interception

Modify or validate return values using `IAspectExitHandler<T>`:

{{< highlight csharp >}}
using Aspekt;

public class CacheAspect : Aspect, IAspectExitHandler<string>
{
    private static readonly Dictionary<string, string> Cache = new();

    public string OnExit(MethodArguments args, string result)
    {
        // Cache the result
        var key = $"{args.FullName}_{string.Join("_", args.Arguments.Values)}";
        Cache[key] = result;
        Console.WriteLine($"Cached result for {args.Name}");
        return result;
    }
}

public class DataService
{
    [CacheAspect]
    public string GetUserData(int userId)
    {
        // Expensive database call
        return $"User data for {userId}";
    }
}
{{< /highlight >}}

---

## Modern Async/Await Support

Aspekt supports modern async patterns:

{{< highlight csharp >}}
using Aspekt;

public class AsyncLoggingAspect : Aspect
{
    public override async ValueTask OnEntryAsync(MethodArguments args, 
        CancellationToken cancellationToken = default)
    {
        await Console.Out.WriteLineAsync($"Async entering: {args.FullName}");
    }

    public override async ValueTask OnExitAsync(MethodArguments args, 
        CancellationToken cancellationToken = default)
    {
        await Console.Out.WriteLineAsync($"Async exiting: {args.FullName}");
    }
}

public class AsyncService
{
    [AsyncLoggingAspect]
    public async Task<string> FetchDataAsync()
    {
        await Task.Delay(100);
        return "Data fetched";
    }
}
{{< /highlight >}}

---

## Exception Handling and Recovery

Implement sophisticated exception handling:

{{< highlight csharp >}}
using Aspekt;

public class RetryAspect : Aspect
{
    private readonly int _maxRetries;
    
    public RetryAspect(int maxRetries = 3)
    {
        _maxRetries = maxRetries;
    }

    public override void OnException(MethodArguments args, Exception e)
    {
        Console.WriteLine($"Exception caught in {args.Name}: {e.Message}");
        Console.WriteLine($"Retry logic could be implemented here");
        // In real implementation, you would coordinate with the weaver
        // to retry the method execution
    }
}

public class NetworkService
{
    [RetryAspect(maxRetries: 3)]
    public string FetchFromRemoteServer(string url)
    {
        // Network call that might fail
        throw new TimeoutException("Server timeout");
    }
}
{{< /highlight >}}

---

## Multiple Aspects

Combine multiple aspects on a single method:

{{< highlight csharp >}}
using Aspekt;
using Aspekt.Contracts;

public class BusinessService
{
    [PerformanceMonitoring]
    [LoggingAspect]
    [RequiresArgument("amount", typeof(decimal), Contract.Comparison.GreaterThan, 0)]
    public decimal ProcessPayment(decimal amount, string currency)
    {
        // Payment processing logic
        // Automatically logged, timed, and validated!
        return amount * 1.05m; // Add processing fee
    }
}
{{< /highlight >}}

---

## What's New in Aspekt

- ✨ **Modern .NET Support**: Targets .NET 8.0, .NET 6.0, and .NET Standard 2.1
- ⚡ **Performance**: Benefits from latest .NET performance improvements
- 🔒 **Nullable Reference Types**: Full support for modern C# null safety
- 🌐 **Cross-Platform**: Run on Windows, Linux, and macOS
- 🎯 **Enhanced Async**: Better async/await pattern support
- 📦 **Updated Dependencies**: Latest MSBuild and tooling support

{{% /content %}}