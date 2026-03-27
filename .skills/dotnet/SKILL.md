---
name: dotnet
description: |
  C#, .NET, WPF/MVVM, EF Core, async/await, DI, xUnit test ve modern
  .NET gelistirme kurallari icin kullan.
---

# C# .NET KURALLARI

Bu kurallar C#, .NET, WPF, WinForms ve modern .NET gelistirme icin gecerlidir.

---

## 1. KOD STILI

### Naming Convention

| TIP | FORMAT | ORNEK |
|-----|--------|-------|
| Class, Struct | PascalCase | UserService, OrderItem |
| Interface | I prefix | IUserRepository, ICommand |
| Method, Property | PascalCase | GetUserById, IsEnabled |
| Private field | _camelCase | _userService, _isReady |
| Local, Parameter | camelCase | userId, itemCount |
| Async method | Async suffix | GetDataAsync, SaveAsync |
| Constant | PascalCase | MaxRetryCount |
| Enum | PascalCase | ConnectionState |
| Generic type | T prefix | TEntity, TResult |

### Dosya Yapisi

```csharp
using System;
using Microsoft.Extensions.Logging;
using MyProject.Models;

namespace MyProject.Services;

public class UserService : IUserService
{
    private readonly ILogger<UserService> _logger;

    public UserService(ILogger<UserService> logger)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public async Task<User?> GetByIdAsync(int id, CancellationToken ct = default)
    {
        // Implementation
    }
}
```

---

## 2. ASYNC/AWAIT

### ConfigureAwait

```csharp
// Library: ConfigureAwait(false)
public async Task<Data> GetDataAsync()
{
    return await _client.GetAsync(url).ConfigureAwait(false);
}

// UI: ConfigureAwait yok (UI context gerekli)
private async void Button_Click(object sender, EventArgs e)
{
    var data = await GetDataAsync();
    TextBlock.Text = data.ToString();
}
```

### CancellationToken

```csharp
public async Task<Result> ProcessAsync(CancellationToken ct = default)
{
    ct.ThrowIfCancellationRequested();
    return await _service.DoWorkAsync(ct);
}
```

### Task vs ValueTask

```csharp
// Task: Genel kullanim, caching yok
public async Task<Data> GetDataAsync() { }

// ValueTask: Sonuc genellikle cache'den geliyorsa (allocation azaltir)
public async ValueTask<Data> GetCachedDataAsync()
{
    if (_cache.TryGet(key, out var data))
        return data;  // Allocation yok
    return await FetchFromDbAsync();
}
```

---

## 3. WPF / MVVM

### ViewModel Base

```csharp
public abstract class ViewModelBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler? PropertyChanged;

    protected bool SetProperty<T>(ref T field, T value,
        [CallerMemberName] string? name = null)
    {
        if (EqualityComparer<T>.Default.Equals(field, value))
            return false;
        field = value;
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
        return true;
    }
}
```

### RelayCommand (ICommand)

```csharp
public class RelayCommand : ICommand
{
    private readonly Action<object?> _execute;
    private readonly Func<object?, bool>? _canExecute;

    public RelayCommand(Action<object?> execute, Func<object?, bool>? canExecute = null)
    {
        _execute = execute ?? throw new ArgumentNullException(nameof(execute));
        _canExecute = canExecute;
    }

    public event EventHandler? CanExecuteChanged;
    public bool CanExecute(object? parameter) => _canExecute?.Invoke(parameter) ?? true;
    public void Execute(object? parameter) => _execute(parameter);
    public void RaiseCanExecuteChanged() => CanExecuteChanged?.Invoke(this, EventArgs.Empty);
}
```

### Custom Control & Attached Property

```csharp
// Attached Property
public static class GridHelper
{
    public static readonly DependencyProperty HighlightProperty =
        DependencyProperty.RegisterAttached("Highlight", typeof(bool),
            typeof(GridHelper), new PropertyMetadata(false, OnHighlightChanged));

    public static bool GetHighlight(DependencyObject obj)
        => (bool)obj.GetValue(HighlightProperty);

    public static void SetHighlight(DependencyObject obj, bool value)
        => obj.SetValue(HighlightProperty, value);

    private static void OnHighlightChanged(DependencyObject d,
        DependencyPropertyChangedEventArgs e) { }
}
```

---

## 4. EF CORE PATTERN'LERI

### DbContext ve Repository Karari

Not:
- EF Core'da `DbContext` zaten Unit of Work ve Repository davranisi saglar
- Generic repository varsayilan tercih olmamali
- Ek soyutlama ancak domain'e ozel sorgu, test siniri veya provider ayrimi gerekiyorsa eklenmeli

```csharp
public interface IOrderReadRepository
{
    Task<Order?> GetByIdAsync(int id, CancellationToken ct = default);
    Task<IReadOnlyList<OrderSummaryDto>> ListOpenOrdersAsync(CancellationToken ct = default);
}

public sealed class OrderReadRepository : IOrderReadRepository
{
    private readonly AppDbContext _db;

    public OrderReadRepository(AppDbContext db)
    {
        _db = db;
    }

    public Task<Order?> GetByIdAsync(int id, CancellationToken ct = default)
        => _db.Orders.FirstOrDefaultAsync(x => x.Id == id, ct);

    public Task<IReadOnlyList<OrderSummaryDto>> ListOpenOrdersAsync(CancellationToken ct = default)
        => _db.Orders
            .Where(x => x.Status == OrderStatus.Open)
            .Select(x => new OrderSummaryDto(x.Id, x.CustomerName, x.Total))
            .ToListAsync(ct);
}
```

### Unit of Work

```csharp
public interface IUnitOfWork : IDisposable
{
    Task<int> SaveChangesAsync(CancellationToken ct = default);
}
```

---

## 5. MINIMAL API (.NET 6+)

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IOrderService, OrderService>();

var app = builder.Build();

app.MapGet("/api/orders", async (IOrderService service) =>
    Results.Ok(await service.GetAllAsync()));

app.MapGet("/api/orders/{id}", async (int id, IOrderService service) =>
    await service.GetByIdAsync(id) is { } order
        ? Results.Ok(order)
        : Results.NotFound());

app.MapPost("/api/orders", async (CreateOrderDto dto, IOrderService service) =>
{
    var order = await service.CreateAsync(dto);
    return Results.Created($"/api/orders/{order.Id}", order);
});

app.Run();
```

---

## 6. ANTI-PATTERNS

### async void

```csharp
// YANLIS
public async void ProcessAsync() { }

// DOGRU
public async Task ProcessAsync() { }

// TEK ISTISNA: Event handler
private async void Button_Click(object sender, EventArgs e)
{
    try { await ProcessAsync(); }
    catch (Exception ex) { _logger.LogError(ex, "Failed"); }
}
```

### .Result ve .Wait()

```csharp
// YANLIS: Deadlock riski
var result = GetDataAsync().Result;
GetDataAsync().Wait();

// DOGRU
var result = await GetDataAsync();
```

### UI Thread Blocking

```csharp
// YANLIS
private void Button_Click(object sender, EventArgs e)
{
    Thread.Sleep(5000);  // UI donuyor!
}

// DOGRU
private async void Button_Click(object sender, EventArgs e)
{
    await Task.Delay(5000);
}
```

### IDisposable

```csharp
// YANLIS
var client = new HttpClient();
var result = client.GetAsync(url);
// Dispose yok!

// DOGRU
using var client = new HttpClient();
var result = await client.GetAsync(url);
```

### Bos Exception

```csharp
// YANLIS
try { DoSomething(); }
catch { }

// DOGRU
try { DoSomething(); }
catch (SpecificException ex)
{
    _logger.LogError(ex, "Context: {Message}", ex.Message);
    throw;
}
```

---

## 7. NULL HANDLING

```csharp
// Null check
if (value is not null) { }

// Null-conditional
var length = value?.Length ?? 0;

// Guard clause (.NET 6+)
ArgumentNullException.ThrowIfNull(user);
ArgumentException.ThrowIfNullOrWhiteSpace(name);
```

---

## 8. DEPENDENCY INJECTION

```csharp
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repo;
    private readonly ILogger<OrderService> _logger;

    public OrderService(IOrderRepository repo, ILogger<OrderService> logger)
    {
        _repo = repo ?? throw new ArgumentNullException(nameof(repo));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}

// Registration
services.AddSingleton<IConfigService, ConfigService>();
services.AddScoped<IOrderService, OrderService>();
services.AddTransient<IEmailService, EmailService>();
```

---

## 9. MODERN C#

### Pattern Matching

```csharp
if (obj is string { Length: > 0 } text) { }

var result = status switch
{
    Status.Active => "Running",
    Status.Stopped => "Finished",
    _ => "Unknown"
};
```

### Records

```csharp
public record User(int Id, string Name, string Email);
var updated = user with { Email = "new@email.com" };
```

### File-Scoped Namespace

```csharp
namespace MyProject.Services;

public class MyService { }
```

### Global Using (ImplicitUsings)

```csharp
// GlobalUsings.cs
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
```

---

## 10. PERFORMANCE

### Span<T> ve Memory

```csharp
// Heap allocation olmadan dizi isleme
ReadOnlySpan<char> span = "Hello, World!".AsSpan();
var word = span[0..5];  // "Hello" - allocation yok
```

### StringBuilder

```csharp
// YANLIS: String concatenation (O(n^2))
string result = "";
for (int i = 0; i < 1000; i++)
    result += i.ToString();

// DOGRU
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i);
string result = sb.ToString();
```

### Lazy Initialization

```csharp
private readonly Lazy<ExpensiveObject> _expensive = new(() => new ExpensiveObject());
public ExpensiveObject Expensive => _expensive.Value;
```
