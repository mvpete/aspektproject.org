---
title: 'Getting Started'
type: "page"
layout: "documentation"
menu: "documentation"
---

# Getting Started

## Introduction
Get started with Aspekt - the lightweight, free and open-source AOP library, now modernized for .NET 8.0, .NET 6.0, and .NET Standard 2.1. Add cross-cutting concerns to your applications with minimal effort and maximum clarity.

## Quick Start

### Prerequisites
- .NET 8.0 SDK, .NET 6.0 SDK, or any .NET Standard 2.1 compatible runtime
- Your favorite IDE (Visual Studio 2022, VS Code, or Rider)

### Installation via NuGet

Add Aspekt to your project using the .NET CLI:

```bash
dotnet add package Aspekt
```

Or via Package Manager Console:

```powershell
Install-Package Aspekt
```

Or add it directly to your `.csproj` file:

```xml
<PackageReference Include="Aspekt" Version="2.1.0" />
```

### Your First Aspect

1. **Create an Aspect Class**

Create a new class that inherits from `Aspekt.Aspect`:

```csharp
using Aspekt;

public class SimpleLoggingAspect : Aspect
{
    public override void OnEntry(MethodArguments args)
    {
        Console.WriteLine($"Method {args.Name} called");
    }
    
    public override void OnExit(MethodArguments args)
    {
        Console.WriteLine($"Method {args.Name} completed");
    }
}
```

2. **Apply the Aspect**

Use your aspect as an attribute on any method:

```csharp
public class Calculator
{
    [SimpleLoggingAspect]
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

3. **Build Your Project**

When you build your project, Aspekt automatically weaves your aspects into the IL code. The weaving happens post-compilation via MSBuild integration.

```bash
dotnet build
```

4. **Run and See the Magic**

```csharp
var calc = new Calculator();
var result = calc.Add(5, 3);
// Output:
// Method Add called
// Method Add completed
```

## Understanding Aspect Methods

Aspekt provides several interception points:

### OnEntry
Called before the method body executes. Perfect for:
- Logging method entry
- Validating preconditions
- Starting performance timers
- Setting up resources

```csharp
public override void OnEntry(MethodArguments args)
{
    // Access method name
    Console.WriteLine($"Entering: {args.Name}");
    
    // Access method arguments
    foreach (var arg in args.Arguments.Values)
    {
        Console.WriteLine($"  Argument: {arg}");
    }
}
```

### OnExit
Called before any return statement. Perfect for:
- Logging method exit
- Validating postconditions
- Stopping performance timers
- Cleaning up resources

```csharp
public override void OnExit(MethodArguments args)
{
    Console.WriteLine($"Exiting: {args.Name}");
}
```

### OnException
Called when the method throws an exception. Perfect for:
- Logging exceptions
- Error recovery
- Retry logic
- Exception transformation

```csharp
public override void OnException(MethodArguments args, Exception e)
{
    Console.WriteLine($"Exception in {args.Name}: {e.Message}");
    // Exception is re-thrown automatically after this method
}
```

## Modern Async Support

Aspekt includes first-class async support:

```csharp
public class AsyncAspect : Aspect
{
    public override async ValueTask OnEntryAsync(
        MethodArguments args, 
        CancellationToken cancellationToken = default)
    {
        await Console.Out.WriteLineAsync($"Async entering: {args.Name}");
    }
    
    public override async ValueTask OnExitAsync(
        MethodArguments args, 
        CancellationToken cancellationToken = default)
    {
        await Console.Out.WriteLineAsync($"Async exiting: {args.Name}");
    }
}
```

## Accessing Method Information

The `MethodArguments` object provides rich information about the intercepted method:

```csharp
public override void OnEntry(MethodArguments args)
{
    // Method name
    string name = args.Name;
    
    // Full method signature
    string fullName = args.FullName;
    
    // Formatted name for display
    string formatted = args.FormattedName;
    
    // Access arguments
    Arguments arguments = args.Arguments;
    int count = arguments.Count;
    object[] values = arguments.Values.ToArray();
    
    // Get specific argument by index
    object firstArg = arguments.GetArgumentByIndex(0);
    
    // Get argument by name
    object namedArg = arguments.GetArgumentValueByName("paramName");
    
    // The instance being called (null for static methods)
    object? instance = args.This;
}
```

## Next Steps

Now that you have the basics:

1. **Explore Examples**: Check out the [Examples](/examples) page for comprehensive use cases
2. **Learn Contract Programming**: Use `Aspekt.Contracts` for design-by-contract programming
3. **Advanced Features**: Implement `IAspectExitHandler<T>` to intercept and modify return values
4. **Performance**: Use the built-in `LoggingAspect` for production-ready logging with performance tracking

## Framework Compatibility

Aspekt supports:
- ✅ .NET 8.0 (Long Term Support)
- ✅ .NET 6.0 (Long Term Support)  
- ✅ .NET Standard 2.1 (broad compatibility)

This means Aspekt works on:
- Windows, Linux, and macOS
- ASP.NET Core applications
- Console applications
- Library projects
- Blazor applications
- Azure Functions (.NET 6+)

## Troubleshooting

### Aspects Not Being Applied

If your aspects aren't being woven:

1. Ensure you're building the project (weaving happens at build time)
2. Check that your aspect class derives from `Aspekt.Aspect`
3. Verify the aspect attribute is applied correctly
4. Clean and rebuild: `dotnet clean && dotnet build`

### Build Warnings

If you see weaving warnings, you can suppress specific warnings:

```csharp
[IgnoreAspectWarning(1001, 1002)]
public void MyMethod() { }
```

## Getting Help

- 📖 **Documentation**: Read through examples and this guide
- 🐛 **Issues**: Report bugs on [GitHub Issues](https://github.com/mvpete/aspekt/issues)
- 💬 **Discussions**: Ask questions on [GitHub Discussions](https://github.com/mvpete/aspekt/discussions)
- 📦 **NuGet**: Check the [Aspekt NuGet page](https://www.nuget.org/packages/Aspekt/)

Happy aspect weaving! 🎯
