# Dotnet Upgrade

## Abstract

This document refers to upgrading a `dotnet3.1` application to `dotnet 6.0`.

It will be split in the following chapters:

- Dotnet 5 [(msdn)](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-5)
- C# 9 [(msdn)](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9)
- Dotnet 6 [(msdn)](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-6)
- C# 10 [(msdn)](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10)

## Dotnet 5

### Single-file deployment + executable [(msdn)](https://docs.microsoft.com/en-us/dotnet/core/deploying/single-file/overview?tabs=cli)

Using the following 2 flags on the `.csproj`, we can achieve a **true** single fie deployment, not that 3.1 weak stuff:

```xml
    <PublishSingleFile>true</PublishSingleFile>
    <SelfContained>true</SelfContained>
```    

The main difference from `dotnet3.1` is that when a user runs your single file app, the host first extracts all files to a directory before running the application, whereas now `dotnet5.0` runs the code **directly** without the need to extract the files from the app.

**Usecases**: easier deployment, less space used.

---

### Runtime Optimizations  [(devblog)](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-5/)

Nothing much to note here, a lot of stuff are faster, use less memory, are more secure and generally implemented in a smarter fashion. Especially GC and RyuJIT. 
Read more in the devblog if you are curious.

**Usecases**: speed, memory, security.

--- 

### LINQ Optimizations [(devblog)](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-5/#linq)

Nothing much to see here, LINQ is faster since they moved sorting algorithms from the CoreCLR layer to Managed Code.
`OrderBy`, sometimes `SkipLast` and some `IEnumerator` performance changes.

**Usecases**: speed, memory.

---

## C# 9

### Records

TODO

---

### Init-only setters

As shown from the snippet bellow, you can use `init-only setted` properties in order to prevent a property changing it's value after construction + initialization has ended:

```csharp
    public record Car(int MaxSpeed); // same applies to records, structs and classes.

    public class Playground 
    {
        public static void DoWork()
        {
            var myCar = new Car(MaxSpeed: 42);

            myCar.MaxSpeed = 69; // Error! CS8852.

            var otherCar = myCar with { MaxSpeed = 43 }; // this is allowed

            otherCar.MaxSpeed = 69; // Error! CS8852.
        }
    }
```

**Usecases**: Initialization becomes (a lot) easier to reason, if you do not have to worry about something changed **ever**.

---

### Pattern matching enhancements

The following patterns were added:

```csharp
    public class Playground
    {
        public static bool CheckSomething(char c) =>
            c is
            >= 'a'      // multi-clause starts here.
            and < 'd'   // and
            or '5'      // or
            and not '4'; // not

        public static bool CheckNullability(object? obj) => obj is not null; // instead of obj != null;

        // also works on switch expressions.
        public static int DoWork(int inputValue) =>
            inputValue switch
            {
                > 42 or 0 => 42,
                42 or < 0 => -42,
                _ => throw new Exception()
            };
    }
```

**Usecases**: Every step we take towards Haskell syntax, we should be happy. Basically, more readability and conciseness.

---

### Less boilerplate on `new()`

Now you can omit the type on the right of a `new`, if it can easily deduced by Roslyn. Observe:

```csharp
    public class Car { } 

    public class Playground
    {
        public int Foo(Car car) => 42;

        public void Bar()
        {
            Car car = new(); // same as new Car(), deduced by the type left of the = operator.

            Foo(new()); // same as Foo(new Car()), deduced by Foo's signature;
        }
    }
```

**Usecases**: Conciseness, less boilerplate.

---

## Dotnet 6

### Hot Reload

Instantly apply changes to a running application, without the need to rebuild everything. Also applies to the `dotnet watch` cli command (not applicable to UI weak boys)

This also applies to `ASP.NET Core 6`, as well as some other web functionalities that do not really apply to our services, since we do not exactly use them as web apps but more like ETL pipelines. 

Check a look [here for ASP.NET Core 6](https://docs.microsoft.com/en-us/aspnet/core/release-notes/aspnetcore-6.0?view=aspnetcore-6.0).

**Usecases**: Faster development time, less waiting time for Visual Studio to do its work.

---

### FileStream Performance [(devblog)](https://devblogs.microsoft.com/dotnet/file-io-improvements-in-dotnet-6/)

Filestream, as the dotnet team says *verbatim* is:

`File I/O is better, stronger, faster!`

The main change is that it never blocks on asyncronous calls **on Windows**, so asynchronicity is more performant.

**Usecases**: Performance.

---

### DateOnly, TimeOnly [(devblog)](https://devblogs.microsoft.com/dotnet/date-time-and-time-zone-enhancements-in-net-6/)

Shiny, new types for explicitly only date and explicitly only time instances.

DateOnly snippet:

```csharp
// Construction and properties
DateOnly d1 = new DateOnly(2021, 5, 31);
Console.WriteLine(d1.Year);      // 2021
Console.WriteLine(d1.Month);     // 5
Console.WriteLine(d1.Day);       // 31
Console.WriteLine(d1.DayOfWeek); // Monday

// Manipulation
DateOnly d2 = d1.AddMonths(1);  // You can add days, months, or years. Use negative values to subtract.
Console.WriteLine(d2);     // "6/30/2021"  notice no time
```

TimeOnly snippet:

```csharp
// Construction and properties
TimeOnly t1 = new TimeOnly(16, 30);
Console.WriteLine(t1.Hour);      // 16
Console.WriteLine(t1.Minute);    // 30
Console.WriteLine(t1.Second);    // 0

// You can add hours, minutes, or a TimeSpan (using negative values to subtract).
TimeOnly t2 = t1.AddHours(10);
Console.WriteLine(t2);     // "2:30 AM"  notice no date, and we crossed midnight
```

**Usecases**: More purposeful semantics when declaring temporal variables.

---

### PriorityQueue

The new `PriorityQueue<TElement,TPriority>` class represents a collection of items that have both a value and a priority. 

Items are dequeued in increasing priority order—that is, the item with the lowest priority value is dequeued first. 

This class implements a [min heap](https://en.wikipedia.org/wiki/Binary_heap) data structure.

**Usecases**: Queue with priority, having O(logn) worst-case, amortized inserts and peeks/pops.

---

### LINQ Extensions

Dotnet 6 added a LOT of new LINQ extension methods. Observe:

```csharp
    public class Playground
    {
        public void Bar()
        {
            IEnumerable<int> numbers = new List<int> { 1, 2, 3, 42, -100 };
            IEnumerable<Car> cars = new List<Car> { new Car(1), new Car(42), new Car(2) };

            // gets the count if and only if no enumeration is needed.
            var succeeded = numbers.TryGetNonEnumeratedCount(out int howMany);

            // Splits the list into arrays, resulting in: [1,2],[3,42],[-100]
            var chunked = numbers.Chunk(2);

            // returns a CAR (not the speed!) with the maximum speed, here "Car { Speed = 42 }"
            var maxBySpeed = cars.MaxBy(car => car.Speed);
            // returns a CAR (not the speed!) with the minimum speed, here "Car { Speed = 1 }"
            var minBySpeed = cars.MinBy(car => car.Speed);

            // allowed indexes counted from the end, returns 42
            var found = numbers.ElementAt(^2);
            // allowed indexes counted from the end, returns 42
            var foundOrDefault = numbers.ElementAtOrDefault(^2);

            // allows default value specification, returns 42. Same applies to LastOrDefault/SingleOrDefault.
            var defaulted = new List<int>().FirstOrDefault(number => number % 2 == 0, defaultValue: 42);

            // allows specifying a comparer, returns the MAX car by the default comparer.
            var maxValue = cars.Max(Comparer<Car>.Default);
            // allows specifying a comparer, returns the MIN car by the default comparer.
            var minValue = cars.Min(Comparer<Car>.Default);

            // allows range-syntax, same as numbers.Take(3).Skip(1), returns [2,3]
            var take3Skip1 = numbers.Take(1..3);

            // allows THREE enumerables to be zipped to one, returns {[1,1,1],[2,2,2],[3,3,3]}
            var smallList = new List<int> { 1, 2, 3 };
            var tripleZipped = smallList.Zip(smallList, smallList);
        }
    }
```

**Usecases**: You really have to ask? You would be homeless without LINQ.

---

## C# 10

### Global `using`

TODO

---

### Record structs 

TODO

---

### Extended `with` functionality

TODO

---

### File-scoped namespace declaration 

TODO

---

### Property pattern-matching syntactic sugar 

TODO

---

### Roslyn lambda expression inference

TODO

---

### Constant string interpolation

TODO

---

### Deconstruction readability

TODO

---

### Roslyn null-check inference

TODO

---

### `CallerArgumentExpression` Attribute

TODO

---


## Further Notes for Absolute Nerds

- Dotnet 5 [Trimming/Member Trimming](https://devblogs.microsoft.com/dotnet/app-trimming-in-net-5/)
- Even more [Trimming/Member Trimming](https://docs.microsoft.com/en-us/dotnet/core/deploying/trimming/trim-self-contained)
- System.Text.Json [Better performance and stuff](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-5/#json)
- Generic Math [Experimental, haskell stuff, equally cool and useless](https://devblogs.microsoft.com/dotnet/preview-features-in-net-6-generic-math/)
- Top level statements [Maybe useful for Program.cs](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#top-level-statements)
- Immutability [Really good article about deciding on the optimal collection](https://docs.microsoft.com/en-us/archive/msdn-magazine/2017/march/net-framework-immutable-collections)
