---
title: "Examples"
menu : "main"
type: "page"
layout: "simple"
---

{{% jumbotron %}}

# Examples
Get going with some simple examples that will show you the basics of aspect oriented programming.

{{< button href= "https://github.com/mvpete/aspekt" class= "btn-outline-primary" >}}Download Now{{</ button >}}

{{% /jumbotron %}}

{{% content %}}

###  Hello Logging.

{{< highlight csharp >}}
using Aspekt;
class MyFirstAspect : Aspect
{
    public override OnEntry(Args...)
    {
        Console.WriteLine("Aspekt Wuz Here");
    }
} 
{{< /highlight >}}

{{% /content %}}