---
title: "Asynchronous Pattern"
date: 2023-01-16T22:22:26-08:00
draft: true
author: "Dawn Yim"
summary: ""
tags: ["C#", "Programming", "DesignPattern"]
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
---
# Asynchronous Pattern

### Asynchronous vs Synchronous

Asynchronous and synchronous programming handle long running operations differently. 

Synchronous: Blocks programming execution until a long running operation completes. 

Asynchronous: Doesn’t block programming execution waiting for long running operation results. 

Asynchronous program splits programs into small tasks that can be executed independently and concurrently. 

### When to use?

- When need to return results quickly e.g., UI. Not to let any long running operation cause unresponsiveness in an application.
- When concurrently invoking multiple asynchronous methods synchronously. This can improve scalability and performance of your application. Otherwise, any long running operations will delay initiation of other operations, which result in decreased benefits of concurrency.
- Examples of expensive long-running operations: Compute-bound tasks, I/O bound tasks, and HTTP Post/Put requests etc.

# C# **Task-based Asynchronous Pattern (TAP)**

TAP is the recommended design pattern when implementing asynchronous workflow in C#. Task or Task<TResult> types in the System.Threading.Tasks namespace can be used to implement the concept. 

Starting from .NET Framework 4.5, any method with async keyword and returning Task or Task<TResult> types will be considered and transformed into asynchronous methods.

```csharp
public static void Executor1(string message)
{
    Console.WriteLine("This is Executor 1");
    Console.WriteLine(message);
}

public static void Execute(Action<string> action, string message)
{
    // Do something..
    action(message);
    // Do something..
}

public static void RequestExecutionWithExistingAction()
{
    Execute(Executor1, message)
}

public static void RequestExecutionWithArrowFunction()
{
    Execute(
        message =>
        {
            Console.WriteLine("This is arrow executor");
            Console.WriteLine(message);
        },
        "This is the message.. "
    )
}
```
