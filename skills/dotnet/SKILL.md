---
name: dotnet
description: Apply modern .NET, C#, async, DI, MVVM, and testing practices with minimal-risk edits.
---

# Dotnet

## When to use me

Use this skill for:

- C# and .NET code
- ASP.NET, services, worker apps, and libraries
- WPF or MVVM-style UI code
- Entity Framework Core
- xUnit or integration-test work

## What I enforce

- dependency injection over hidden global state
- async/await over blocking waits
- explicit cancellation where long-running work exists
- constructor validation and clear null handling
- small, focused classes over god objects

## Critical checks

- avoid `.Result` and `.Wait()` in async flows
- avoid `async void` except event handlers
- respect existing project style and nullable annotations
- confirm config, startup, and project-file edits before widening scope
