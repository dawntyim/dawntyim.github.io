---
title: "Garbage Collection"
date: 2023-01-10T22:22:26-08:00
draft: true
author: "Dawn Yim"
summary: ""
tags: ["C#", "Programming"]
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
---
# Garbage Collection and Disposable vs Finalization

## IDisposable vs Finalizer

`IDisposable` interface is provided in C# to provide a mechanism for releasing unmanaged resources directly by developers. Althought the garbage collector deallocate unused objects, but it is not possible to predict when the garbage collection will occur. 

On the contrary, a finalizer is a method that is called by the garbage collector when an object is not in use anymore. Finalizaing `~Object ();` is freeing resources and perform cleanup operations before it’s reclaimed by garbage collection.

### How Finalization Works

An object does not have default implementation for finalizer. If an object implements a finalizer, GC adds that object to a ‘finalization queue’ and the objects in these queues are proceeded before other objects. 

### Best Practice

Provide a finalizer that calls the Dispose method in case the Dispose method is not called. In this way, a unused resource will be disposed eventually. 

To prevent from the finalizer releasing the resource again, you can use `GC.SupressFinalize` method. 

```sql
public class Example
{
	public class DisposeExample : IDisposable
	{
		private IntPtr pointer;
		public DisposeExample(IntPtr pointer)
		{
			this.pointer = pointer;
		}

		public void Dispose()
		{
			Dispose(disposing: true)
			GC.Suppressfinalize(this)
		}

		protected virtual void Dispose(bool disposing)
		{
			if(!this.disposed)
			{
				// dispose unmanaged resources
				component.Dispose();
			}
			CloseHandle(handle);
			handle = IntPtr.Zero;
			disposed = true;
		}
	
		~DisposeExample()
		{
			Dispose(disposing: false);
		}
	}
}
```

## Garbage Collection

There are two types of garbage collecting mechanism. One is ‘mark-and-sweep’ like C# uses, and the other is ‘reference counter’. 

### C#: Mark and Sweep

GC (Garbage Collector) marks objects that are still in use and then sweeps (frees) the memory occupied by objects that are not in use anymore. However, during the ‘mark’ phase, if a GC encounters an object with a finalizer, it queues the object to ‘finalization queue’ then move on to the next object. It waits until the next GC cycle to run the finalizer. 

Depending on  the object characteristics, the objects with unused resources can be problematic in terms of memory usage and leak. In general, it is advised to use IDisposable class to free memory space as early as possible for the objects that can potentially harm memory usage. 

### C#: Generations

C# garbage collector has two generations (generation 0 and generation 1) to differentiate short-lived and long-lived objects. GC starts from checking generation 0 that has short-lived objects in. If it’s full, GC triggers collection and move on to the next generation and does the same thing. 

### Reference counting

Each object maintains a count of the number of references that point to it and when the count reaches zero, the object is considered in no use. This can cause circular references where two objects are pointing each other and not being collected by GC. This will result in memory leak. 

It's important to notice that, in this example, the cache does not handle the expiration of the cache items, you can use the MemoryCache constructors, Add methods or the CacheItemPolicy class to set expiration policies for the cache items.

So, to combine all these, I want to use refcount concurrent dictionary to ensure disposable object cache is not disposed when accessed by other components. We want to ensure this policy is kept even for sliding expirations. Also, we don't want the concurrent dictionary size to be too big. Give me the example code for this

```csharp

```
