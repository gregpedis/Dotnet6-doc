# Dotnet Upgrade

## Abstract

This document refers to upgrading a `dotnet3.1` application to `dotnet 6.0`.

It will be split in the following chapters:

- Dotnet 5 [(msdn)](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-5)
- C# 9 [(msdn)](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9)
- Dotnet 6 [(msdn)](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-6)
- C# 10 [(msdn)](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10)

## Dotnet 5

---

### Single-file deployment + executable [(msdn)](https://docs.microsoft.com/en-us/dotnet/core/deploying/single-file/overview?tabs=cli)

Using the following 2 flags on the `.csproj`, we can achieve a **true** single fie deployment, not that 3.1 weak stuff:

```xml
    <PublishSingleFile>true</PublishSingleFile>
    <SelfContained>true</SelfContained>
```    

The main difference from `dotnet3.1` is that when a user runs your single file app, the host first extracts all files to a directory before running the application, whereas now `dotnet5.0` runs the code **directly** without the need to extract the files from the app.

**Usecases**: easier deployment, less space used.

---

### Optimizations  [(devblog)]([https://docs.microsoft.com/en-us/dotnet/core/deploying/single-file/overview?tabs=cli](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-5/))

Nothing much to note here, a lot of stuff are faster, use less memory, are more secure and generally implemented in a smarter fashion. Especially GC and RyuJIT. 
Read more in the devblog if you are curious.

**Usecases**: speed, memory, security.
