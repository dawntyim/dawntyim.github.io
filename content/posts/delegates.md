---
title: "Delegates"
date: 2023-01-19T22:22:26-08:00
draft: false
author: "Dawn Yim"
summary: ""
tags: ["C#", "Programming"]
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
---

# Delegates

Delegates predicate enhances flexiblility in code. Delegate delegates method call to other methods. Invoking a delegate means passing parameters to the target method via the delegate method and receiving return value from the target method via the delegate method

This simplest example will help you understand Delegate concept 

```csharp
public delegate void Delegator(string message);

public static void TargetMethod(string message)
{
	Console.Writeln("This is delegated method");
	Console.Writeln(message);
}

Delegator handler = TargetMethod;
handler("This is the message");
```

By stating a delegate method, you can plug in different methods for a delegate method. Delegates are used when passing a method as an argument of another method. Any method that matches the delegate type can be assigned to the delegates. This flexibility makes the delegate methods ideal for defining callback methods 

### Properties of Delegate

- Delegate types are sealed

They can’t be derived nor other classes can be derived from Delegate. 

- Instantiated Delegate is an object.

A delegate instance can be passed as a parameter or can be assigned as a property

- Can be used as a callback function

Continuing from the above example, the delegated handler method can be used as a callback to print given message.

```csharp
public static void PrintWithCallback(int param1, int param2, Delegator handler)
{
    handler($"The params are {param1} and {param2}");
}

PrintWithCallback(1, 2, handler);
```

- Multicasting

A delegate can call multiple methods when invoked. The methods will be called sequentially in the order of addition. Any changes by one method is visible to the next method. Subtraction operation works either. 

```csharp
public class WrapperClass
{
    public void Method1(string message) { }
    public void Method2(string message) { }
}

var obj = new WrapperClass();
Delegator combinedDelegator = obj.Method1 + obj.Method2;
combinedDelegator += handler;

combinedDelegator -= obj.Method1;
```

### Detailed Example

Let’s say there are multiple writer classes that are customized to write a simple story line for its genre. We can use a delegate method to callback any `WriteParagraph` method from the given genre. In other words, you can plug in any writer code programmatically using Delegate predicate. 

```csharp
public delegate void WriteParagraphFunc();

public interface IWriter
{
    void WriteParagraph();
}

// WriterScifi, WriterPoem, WriterNonfiction, WriterFantasy are inherited from this abstract classes
public abstract class Writer<T> : IWriter where T : Books
{
	abstract WriteParagraph();
}

// we have this binding dictionary to map each genre to customized writer class
public static readonly Dictionary<Genres, WriteParagraphFunc>
    GetBookWriterFunctions =
        new Dictionary<Genres, WriteParagraphFunc>
        {
            { Genres.Scifi, new WriterScifi().WriteParagraph },
            { Genres.Poem, new WriterPoem().WriteParagraph },
            { Genres.Nonfiction, new WriterNonfiction().WriteParagraph },
            { Genres.Fantasy, new WriterFantasy().WriteParagraph }
        };

// You can change the genre and the mapped  callback will be invoked. 
Genres myGenre = Genres.Scifi 
GetBookWriterFunctions[Genre]();
```

### `Action` and `Func<T>` Types

**`Func<T>`**and **`Action`** both delegate types in C#.

**`Func<T>`**can take zero or more input parameters and return a value of type **`T`**

```csharp
// Representation
public delegate TResult Func<in T, out TResult>(T arg);

// Func example that takes int input and return string object. 
Func<int, string> func = x => x.ToString();
string result = func(10); // result is "10"
```

**`Action`** can take zero or more input parameters, but does not return a value. 

```csharp
// Representation
public delegate void Action();

// Action example that takes two strings as input. 
Action<string, string> action = (s1, s2) => Console.WriteLine(s1 + s2);
action("Hello, ", "World!"); // prints "Hello, World!"
```

You can use arrow function to replace **`Func<T>`**or **`Action`** .  Here’s a simple example of how to use these types

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

public static void RequestExecutionWithExistingAction(string message)
{
    Execute(Executor1, message)
}

public static void RequestExecutionWithArrowFunction(string message)
{
    Execute(
        message =>
        {
            Console.WriteLine("This is arrow executor");
            Console.WriteLine(message);
        },
        message
    )
}
```
