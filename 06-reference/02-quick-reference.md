# 02 クイックリファレンス

## よく使うコマンド、パターン、キーボードショートカットを素早く参照

Ctrl+F で検索して使用。

---

## ⌨️ キーボードショートカット

### Copilot（全 IDE）

| 操作 | VS Code | Visual Studio |
|------|---------|---------------|
| Inline Completions 表示 | `Ctrl+I` | `Ctrl+\` |
| Chat ウィンドウ | `Ctrl+Shift+I` | `Ctrl+\` (then Chat tab) |
| 次の提案 | `Ctrl+]` | `Alt+]` |
| 前の提案 | `Ctrl+[` | `Alt+[` |
| 拒否（Dismiss） | `Escape` | `Escape` |

### Visual Studio 2022

| 操作 | ショートカット |
|------|----------------|
| Solution Explorer | `Ctrl+Alt+L` |
| Debug Start/Stop | `F5` / `Shift+F5` |
| Build | `Ctrl+Shift+B` |
| Run Tests | `Ctrl+R, T` |
| Go to Definition | `F12` |
| Find All References | `Shift+F12` |
| Rename | `Ctrl+R, R` or `F2` |
| Format Document | `Ctrl+K, Ctrl+D` |
| Format Selection | `Ctrl+K, Ctrl+F` |

### VS Code

| 操作 | ショートカット |
|------|----------------|
| Command Palette | `Ctrl+Shift+P` |
| Find | `Ctrl+F` |
| Replace | `Ctrl+H` |
| Go to Definition | `F12` |
| Peek Definition | `Alt+F12` |
| Terminal | `Ctrl+` |
| Run Task | `Ctrl+Shift+P` → Run Task |

---

## 💬 Copilot Chat よく使うプロンプト

### コード理解

```
このメソッドが何をしているか説明して

[メソッドコードを貼り付け]
```

### リファクタリング

```
このコード、SOLID 原則に従うようにリファクタリングして

[コード]
```

### テスト生成

```
このクラスの包括的な xUnit テストを生成してください
正常系、異常系、境界値をカバーしてください

[クラスコード]
```

### バグ調査

```
このエラー、原因と対策を教えてください

[エラースタックトレース + コード]
```

### パフォーマンス診断

```
このクエリが遅い理由は？最適化案を教えてください

[LINQ or SQL]
```

### セキュリティ見直し

```
このコード、セキュリティ脆弱性を指摘してください

[コード]
```

### パターン相談

```
複数の支払い方法（CreditCard, PayPal, BankTransfer）に対応する場合、
どのデザインパターンが適切ですか？実装例も示してください
```

---

## 🎯 よく使う C# パターン

### 非同期処理

```csharp
// 基本形
public async Task<T> GetDataAsync<T>(int id)
{
    var data = await _repository.GetByIdAsync(id);
    return data;
}

// 複数 Task の並列実行
await Task.WhenAll(
    task1,
    task2,
    task3
);

// Task チェーン
var result = await _service.GetAsync()
    .ContinueWith(t => ProcessAsync(t.Result))
    .Unwrap();
```

### エラーハンドリング

```csharp
try
{
    // リスク操作
}
catch (InvalidOperationException ex)
{
    _logger.LogError(ex, "Invalid operation");
    throw;  // 再スロー
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error");
    return BadRequest("エラーが発生しました");
}
finally
{
    // Cleanup
}
```

### Null チェック

```csharp
// Null-coalescing
var name = user?.Name ?? "Unknown";

// Null-conditional
var email = customer?.Email;  // null if customer is null

// Null-coalescing assignment
user ??= new User();

// Pattern matching
if (order is { Status: OrderStatus.Pending, Amount: > 1000 })
{
    // Order is pending and amount > 1000
}
```

### LINQ パターン

```csharp
// Filter, Map, Reduce
var result = _context.Orders
    .Where(o => o.Amount > 100)           // Filter
    .Select(o => o.CustomerName)          // Map
    .Distinct()
    .ToList();

// GroupBy
var grouped = _context.Orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new { 
        CustomerId = g.Key, 
        Count = g.Count(), 
        Total = g.Sum(o => o.Amount) 
    })
    .ToList();

// Join
var joined = _context.Orders
    .Join(_context.Customers,
        o => o.CustomerId,
        c => c.Id,
        (o, c) => new { o.Id, c.Name, o.Amount })
    .ToList();

// Pagination
var page2 = _context.Orders
    .OrderByDescending(o => o.CreatedDate)
    .Skip(10)
    .Take(10)
    .ToListAsync();
```

### 依存性注入

```csharp
// Startup Register (Program.cs / Startup.cs)
services.AddScoped<IOrderService, OrderService>();
services.AddScoped<IOrderRepository, OrderRepository>();
services.AddSingleton<ICache, MemoryCache>();

// Constructor Injection
public class OrderService
{
    public OrderService(IOrderRepository repo, ILogger<OrderService> logger)
    {
        // ...
    }
}
```

### Strategy パターン

```csharp
// Interface
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessAsync(PaymentRequest req);
}

// Implementations
public class CreditCardPaymentStrategy : IPaymentStrategy { }

// Factory
public class PaymentFactory
{
    public IPaymentStrategy GetStrategy(string method) =>
        method switch
        {
            "CreditCard" => new CreditCardPaymentStrategy(),
            "PayPal" => new PayPalPaymentStrategy(),
            _ => throw new NotSupportedException()
        };
}

// Usage
var strategy = _factory.GetStrategy(order.PaymentMethod);
var result = await strategy.ProcessAsync(request);
```

### Unit Test テンプレート

```csharp
public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _mockRepo;
    private readonly OrderService _sut;
    
    public OrderServiceTests()
    {
        _mockRepo = new Mock<IOrderRepository>();
        _sut = new OrderService(_mockRepo.Object);
    }
    
    [Theory]
    [InlineData(100, true)]
    [InlineData(0, false)]
    public async Task ProcessOrder_WithAmount_ReturnsExpected(
        decimal amount, bool expected)
    {
        // Arrange
        var order = new Order { Amount = amount };
        
        // Act
        var result = await _sut.ProcessOrderAsync(order);
        
        // Assert
        Assert.Equal(expected, result.Success);
    }
}
```

---

## 📊 Entity Framework Core パターン

### Context Setup

```csharp
// DbContext
public class AppDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    public DbSet<Customer> Customers { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer(_connectionString);
    }
}

// Register (Program.cs)
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(configuration.GetConnectionString("Default")));
```

### Querying

```csharp
// Simple Query
var order = await _context.Orders.FindAsync(orderId);

// Filter
var orders = await _context.Orders
    .Where(o => o.Status == OrderStatus.Pending)
    .ToListAsync();

// Include (Eager Load)
var orders = await _context.Orders
    .Include(o => o.Customer)
    .Include(o => o.Items)
    .ToListAsync();

// Select (Projection)
var orderSummary = await _context.Orders
    .Select(o => new OrderDto
    {
        Id = o.Id,
        Total = o.Total,
        Customer = o.Customer.Name
    })
    .ToListAsync();

// AsNoTracking (Read-only optimization)
var orders = await _context.Orders
    .AsNoTracking()
    .ToListAsync();
```

### Mutations

```csharp
// Create
var order = new Order { ... };
_context.Orders.Add(order);
await _context.SaveChangesAsync();

// Update
var order = await _context.Orders.FindAsync(id);
order.Status = OrderStatus.Paid;
await _context.SaveChangesAsync();

// Delete
_context.Orders.Remove(order);
await _context.SaveChangesAsync();

// Bulk Operations (via Query)
await _context.Orders
    .Where(o => o.CreatedDate < DateTime.Now.AddYears(-1))
    .ExecuteDeleteAsync();
```

---

## 🔐 セキュリティ チェックリスト

Every PR Review:

- [ ] SQL Injection? (Parameterized queries?)
- [ ] XSS? (Input sanitization?)
- [ ] CORS? (Whitelist origins only)
- [ ] Authentication? ([Authorize] attrs?)
- [ ] Secrets in code? (Use appsettings/environment)
- [ ] Sensitive logging? (Avoid PII in logs)
- [ ] Error messages? (Don't expose internals)

---

## 📈 パフォーマンス チェックリスト

Every Performance Critical Code:

- [ ] N+1 queries? (Include/Projection?)
- [ ] .ToList() at wrong place?
- [ ] Caching implemented?
- [ ] Connection pooling?
- [ ] Indexes on filtered columns?
- [ ] Async all the way?
- [ ] ConfigureAwait(false) in libraries?

---

## 🧪 テストキー チェックリスト

Every Unit Test:

- [ ] Happy path 1 + Edge1 + Error1 minimum?
- [ ] Mocks configured?
- [ ] Verification of calls?
- [ ] No shared state between tests?
- [ ] Fast (< 100ms per test)?
- [ ] Isolated from external dependencies?

---

## 📝 ドキュメント テンプレート

### XMLDoc (Public methods)

```csharp
/// <summary>
/// 指定 ID の注文を取得
/// </summary>
/// <param name="orderId">注文 ID</param>
/// <returns>注文オブジェクト、見つからない場合は null</returns>
/// <exception cref="InvalidOperationException">ID が無効な場合</exception>
public async Task<Order> GetOrderAsync(int orderId)
{
    // ...
}
```

### Code Comment (Complex logic)

```csharp
// Strategy パターンで支払い方法を選択
// 各戦略は IPaymentStrategy を実装し、
// Factory が適切なインスタンスを返す
var strategy = _paymentFactory.GetStrategy(order.PaymentMethod);
```

---

## 🚀 チームオンボーディングチェック

New Team Member Quick Links:

1. Read: [01-foundations/01-basics.md](../01-foundations/01-basics.md)
2. Setup: IDE + Copilot extension
3. Learn: [02-core-features/](../02-core-features/)
4. Code: Pair programming on simple bug fix
5. Test: Run `dotnet test`
6. Review: Submit PR, get feedback
7. Culture: Attend team standup

---

## 🔗 外部リソース

- [C# Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/)
- [Async Best Practices](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
- [Refactoring.Guru Patterns](https://refactoring.guru/design-patterns/csharp)
- [xUnit Documentation](https://xunit.net/docs/getting-started/netcore)
