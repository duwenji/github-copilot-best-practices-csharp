### プロジェクト固有の Copilot ガイドラインをテンプレートして、全チームで統一的に使う

`.github/copilot-instructions.md` は Copilot AI が参照する「このプロジェクトのルール」。チームのコーディング標準、設計パターン、セキュリティ要件を明示することで、生成されるコードの品質を飛躍的に向上させます。

---

### 📋 テンプレート全体

完全なテンプレートを以下に示します。プロジェクト特性に合わせてカスタマイズしてください。

---

````markdown
# GitHub Copilot Instructions

このドキュメントは、このプロジェクトで Copilot を使用する際の指針です。
AI の提案がこれらのルールに従うように指定します。

## プロジェクト概要

### Project Identity
- **Name**: E-Commerce Backend Platform
- **Tech Stack**: 
  - Runtime: .NET 8
  - Framework: ASP.NET Core
  - ORM: Entity Framework Core 8
  - Database: SQL Server 2022
  - Message Queue: RabbitMQ
  - Cache: Redis

### Team
- **Size**: 10 engineers
- **Experience Level**: Mix (2 juniors, 5 mid-level, 3 seniors)
- **Code Review**: Required before merge

### Target Metrics
- Test Coverage: ≥ 80%
- Code Duplication: < 5%
- Avg Cyclomatic Complexity: < 8
- P95 API Response Time: < 500ms

---

## コーディング基準

### 言語と命名規則

#### Classes & Interfaces
```csharp
// ✅ Correct
public interface IOrderService { }
public class OrderService : IOrderService { }
public class CreateOrderCommand { }
public record OrderDto(int Id, string CustomerName);

// ❌ Incorrect
public interface OrderService { }
public class Order_Service { }
public class order_service { }
```

#### Methods & Parameters
```csharp
// ✅ Correct
public async Task<OrderResult> ProcessOrderAsync(int orderId)
public bool IsValidPaymentMethod(PaymentMethodType method)
public void LogOrderCreated(Order order)

// ❌ Incorrect
public async Task<OrderResult> process_order(int orderId)
public bool checkPayment(PaymentMethodType method)
public void LogOrder(Order o)
```

#### Variables & Fields
```csharp
// ✅ Correct
private readonly IOrderRepository _orderRepository;
private decimal totalAmount;
const int MaxRetries = 3;

// ❌ Incorrect
private IOrderRepository OrderRepository;
private decimal total_amount;
const int max_retries = 3;
```

---

## アーキテクチャとデザインパターン

### 層構造（Clean Architecture）

```
┌─────────────────────────┐
│  Presentation Layer     │  Controllers, DTOs, API Contracts
├─────────────────────────┤
│  Application Layer      │  Services, Use Cases, Validation
├─────────────────────────┤
│  Domain Layer           │  Entities, Value Objects, Interfaces
├─────────────────────────┤
│  Infrastructure Layer   │  DB Access, External Services, Config
└─────────────────────────┘
```

**重要**: 層の依存関係は上からのみ。下層は上層に依存しない (Dependency Inversion)

### 必須デザインパターン

#### 1. Repository Pattern（すべてのデータアクセス）

```csharp
// ✅ Always: Interface definition first
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
    Task SaveChangesAsync();
}

// ✅ Implementation
public class OrderRepository : IRepository<Order>
{
    private readonly AppDbContext _context;
    
    public async Task<Order> GetByIdAsync(int id)
    {
        return await _context.Orders
            .AsNoTracking()
            .FirstOrDefaultAsync(o => o.Id == id);
    }
    
    // ... 他メソッド
}

// ❌ Never: Direct DbContext access in services
// 悪い例：DbContext を Service に inject
```

#### 2. Strategy Pattern（複数実装が必要な場合）

このプロジェクトでは以下が Strategy 候補：
- **Payment Processing**: CreditCard, PayPal, BankTransfer
- **Shipping Methods**: Standard, Express, International
- **Notification**: Email, SMS, Push

```csharp
// Strategy interface
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request);
    string PaymentMethodName { get; }
}

// Strategy implementations
public class CreditCardPaymentStrategy : IPaymentStrategy
{
    // ...
}

// Strategy factory
public class PaymentStrategyFactory
{
    public IPaymentStrategy GetStrategy(PaymentMethodType method)
    {
        return method switch
        {
            PaymentMethodType.CreditCard => new CreditCardPaymentStrategy(...),
            PaymentMethodType.PayPal => new PayPalPaymentStrategy(...),
            _ => throw new NotSupportedException()
        };
    }
}

// Usage: Never if-else chains
public class OrderService
{
    public async Task<OrderResult> ProcessOrderAsync(Order order)
    {
        var strategy = _paymentFactory.GetStrategy(order.PaymentMethod);
        var result = await strategy.ProcessPaymentAsync(...);
        // ...
    }
}
```

#### 3. Dependency Injection（always）

Constructor injection ONLY. No Service Locator pattern.

```csharp
// ✅ Correct: Constructor injection
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentService _paymentService;
    private readonly ILogger<OrderService> _logger;
    
    public OrderService(
        IOrderRepository orderRepository,
        IPaymentService paymentService,
        ILogger<OrderService> logger)
    {
        _orderRepository = orderRepository;
        _paymentService = paymentService;
        _logger = logger;
    }
}

// ❌ Incorrect: Service Locator
public class OrderService
{
    private IOrderRepository _repository = 
        ServiceLocator.GetService<IOrderRepository>();
}
```

---

## エラーハンドリングとロギング

### 必須のエラーハンドリング

すべての public async method は try-catch-finally が必須：

```csharp
public async Task<ApiResponse<OrderDto>> CreateOrderAsync(CreateOrderRequest request)
{
    try
    {
        // 1. Input validation
        if (request == null || string.IsNullOrWhiteSpace(request.CustomerName))
        {
            _logger.LogWarning("CreateOrder called with invalid input");
            return ApiResponse<OrderDto>.BadRequest("Customer name is required");
        }
        
        // 2. Business logic
        _logger.LogInformation("Creating order for customer: {CustomerName}", request.CustomerName);
        
        var order = new Order
        {
            CustomerName = request.CustomerName,
            CreatedDate = DateTime.UtcNow
        };
        
        await _orderRepository.AddAsync(order);
        await _orderRepository.SaveChangesAsync();
        
        // 3. Success response
        _logger.LogInformation("Order created successfully: {OrderId}", order.Id);
        return ApiResponse<OrderDto>.Success(
            new OrderDto(order.Id, order.CustomerName));
    }
    catch (InvalidOperationException ex)
    {
        _logger.LogError(ex, "Invalid operation during order creation");
        return ApiResponse<OrderDto>.BadRequest(ex.Message);
    }
    catch (DbUpdateException ex)
    {
        _logger.LogError(ex, "Database error during order creation");
        return ApiResponse<OrderDto>.InternalError("Failed to save order");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Unexpected error during order creation");
        return ApiResponse<OrderDto>.InternalError("An unexpected error occurred");
    }
}
```

### ロギング規則

```csharp
// ✅ Correct logging patterns
_logger.LogInformation("User login attempt: {UserId}", userId);
_logger.LogWarning("Retry attempt {RetryCount}/{MaxRetries} for operation", 
    currentRetry, maxRetries);
_logger.LogError(exception, "Failed to process payment for order {OrderId}", orderId);

// ❌ Avoid
_logger.LogInformation($"User login: {userId}"); // Use structured logging
_logger.LogInformation("Failed to process"); // No context
```

---

## テスト基準

### 単体テスト (xUnit)

```csharp
[TestClass]
public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _mockRepository;
    private readonly Mock<IPaymentService> _mockPayment;
    private readonly ILogger<OrderService> _logger;
    private readonly OrderService _sut; // System Under Test
    
    public OrderServiceTests()
    {
        _mockRepository = new Mock<IOrderRepository>();
        _mockPayment = new Mock<IPaymentService>();
        _logger = new Mock<ILogger<OrderService>>().Object;
        _sut = new OrderService(_mockRepository.Object, _mockPayment.Object, _logger);
    }
    
    // Test naming: [MethodName]_[Scenario]_[ExpectedResult]
    [Theory]
    [InlineData(100, true)]    // Normal case
    [InlineData(0, false)]     // Edge case
    [InlineData(-1, false)]    // Invalid case
    public async Task ProcessOrder_WithAmount_ReturnsExpectedResult(
        decimal amount, bool shouldSucceed)
    {
        // Arrange
        var order = new Order { Id = 1, Amount = amount };
        _mockRepository.Setup(r => r.GetByIdAsync(1))
            .ReturnsAsync(order);
        
        // Act
        var result = await _sut.ProcessOrderAsync(1);
        
        // Assert
        if (shouldSucceed)
        {
            Assert.True(result.Success);
        }
        else
        {
            Assert.False(result.Success);
        }
    }
}
```

### テストカバレッジ要件

- **Target**: ≥ 80% overall coverage
- **Focus**: Business logic layer (90%+)
- **Data access**: Integration tests OK
- **UI/API endpoint**: E2E tests OK

---

## パフォーマンス要件

### データベースクエリ最適化

```csharp
// ❌ Bad: N+1 query problem
var orders = await _context.Orders.ToListAsync();
foreach (var order in orders)
{
    order.Customer = await _context.Customers
        .FirstOrDefaultAsync(c => c.Id == order.CustomerId);
}

// ✅ Good: Single query with Include
var orders = await _context.Orders
    .Include(o => o.Customer)
    .AsNoTracking()  // For read-only queries
    .ToListAsync();

// ✅ Good: Projection (最小限のカラムのみ取得)
var orderSummaries = await _context.Orders
    .Select(o => new OrderSummaryDto
    {
        Id = o.Id,
        CustomerName = o.Customer.Name,
        Total = o.Total
    })
    .ToListAsync();
```

### API レスポンスタイム Targets

| Endpoint Type | P50 | P95 | P99 |
|--------------|-----|-----|-----|
| GET (simple) | 50ms | 100ms | 200ms |
| GET (with joins) | 100ms | 300ms | 500ms |
| POST (create) | 200ms | 500ms | 1000ms |
| Batch/Report | 1s | 5s | 10s |

### Caching Strategy

```csharp
// Redis cache for reference data (5-min TTL)
public async Task<List<ShippingMethod>> GetShippingMethodsAsync()
{
    const string cacheKey = "shipping_methods";
    const int cacheDurationMinutes = 5;
    
    if (_cache.TryGetValue(cacheKey, out List<ShippingMethod> cached))
    {
        return cached;
    }
    
    var methods = await _repository.GetShippingMethodsAsync();
    _cache.Set(cacheKey, methods, TimeSpan.FromMinutes(cacheDurationMinutes));
    return methods;
}
```

---

## セキュリティ要件

### Mandatory Security Checks

```csharp
// ❌ NEVER: Hardcoded credentials
const string connectionString = "Server=...;Password=admin123";

// ✅ ALWAYS: Environment variables / Secrets
var connectionString = _configuration.GetConnectionString("DefaultConnection");

// ❌ NEVER: SQL concatenation (SQL Injection risk)
var sql = $"SELECT * FROM Orders WHERE Id = {orderId}";

// ✅ ALWAYS: Parameterized queries
var order = await _context.Orders
    .FromSqlInterpolated($"SELECT * FROM Orders WHERE Id = {orderId}")
    .FirstOrDefaultAsync();

// ❌ NEVER: Trust user input
public IActionResult Search(string query)
{
    return Ok(_context.Products
        .Where(p => p.Name.Contains(query))  // Can contain XSS
        .ToList());
}

// ✅ ALWAYS: Validate & sanitize
public IActionResult Search(string query)
{
    if (string.IsNullOrWhiteSpace(query) || query.Length > 100)
        return BadRequest("Invalid search query");
    
    return Ok(_context.Products
        .Where(p => EF.Functions.Like(p.Name, $"%{query}%"))
        .ToList());
}
```

### Authentication & Authorization

```csharp
// ✅ Check authorization attribute on all protected endpoints
[Authorize(Roles = "Admin")]
[HttpDelete("orders/{id}")]
public async Task<IActionResult> DeleteOrder(int id)
{
    // ...
}

// ✅ Validate user ownership
[HttpPut("orders/{id}")]
public async Task<IActionResult> UpdateOrder(int id, UpdateOrderRequest request)
{
    var order = await _orderRepository.GetByIdAsync(id);
    if (order.UserId != User.GetUserId())
    {
        return Forbid();  // 403 Forbidden
    }
    // ...
}
```

---

## 外部サービス・API 連携

### Resilience Patterns

```csharp
// Use Polly for retry + circuit breaker
public class PaymentService
{
    public PaymentService(HttpClient httpClient, IAsyncPolicy<HttpResponseMessage> policy)
    {
        _httpClient = httpClient;
        _policy = policy;
    }
    
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
    {
        try
        {
            var response = await _policy.ExecuteAsync(
                () => _httpClient.PostAsJsonAsync("/api/process-payment", request));
            
            if (!response.IsSuccessStatusCode)
            {
                _logger.LogError("Payment API error: {StatusCode}", response.StatusCode);
                return PaymentResult.Failed("Payment processing failed");
            }
            
            return PaymentResult.Success(...);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Payment service unavailable");
            return PaymentResult.Failed("Payment service unavailable");
        }
    }
}

// Startup configuration
services.AddHttpClient<IPaymentService, PaymentService>()
    .AddTransientHttpErrorPolicy(p => p.WaitAndRetryAsync(
        retryCount: 3,
        sleepDurationProvider: retryAttempt =>
            TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))));
```

---

## Comment-Driven Development

与えられた機能要全成を書く際は、まず詳細なコメントを書いてから実装をする。

```csharp
// ✅ Always: Detailed comments describe intent
/// <summary>
/// 注文の支払いを処理する
/// 1. 注文を DB から読み込む
/// 2. 有効性チェック（金額 > 0, 顧客存在）
/// 3. 支払いサービスで処理
/// 4. 成功時: 注文ステータスを Paid に更新、確認メール送信
/// 5. 失敗時: エラーログ + エラーレスポンス返却
/// </summary>
public async Task<OrderResult> ProcessPaymentAsync(int orderId)
{
    // Implementation...
}

// Then Copilot will generate better code based on intent
```

---

## Copilot Usage Guidelines

### ✅ Do
- Use Copilot for boilerplate code (Repository implementations, DTOs)
- Use Chat for architecture questions
- Use Inline Edits for refactoring
- Verify all security-sensitive code
- Run tests before merge

### ❌ Don't
- Trust Copilot blind (review all suggestions)
- Use for complex business logic without understanding
- Copy-paste without testing
- Ignore security warnings
- Skip code review because Copilot generated it

---

## 参考リソース

- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Entity Framework Core Best Practices](https://learn.microsoft.com/en-us/ef/core/performance/)
- [.NET Security Best Practices](https://learn.microsoft.com/en-us/dotnet/standard/security/)

---

## Questions?

Ask in #copilot-questions on Slack
````

---

### 📝 カスタマイズのポイント

実際に使用する際には、以下を調整してください：

1. **Tech Stack**: Rails, Python, Node.js など別言語なら全部置き換え
2. **Team Size / Experience**: 小さく focused なチームなら簡潔に
3. **デザインパターン**: 自チームで標準化しているパターンのみ記載
4. **パフォーマンス基準**: e-commerce（このテンプレート）と遠く管理画面でも異なる
5. **セキュリティ**: PCI-DSS など compliance 要件に合わせる

---

### ✅ Copilot Instructions 導入チェックリスト

- [ ] `.github/copilot-instructions.md` ファイルを作成
- [ ] チーム全体で内容をレビュー + approve
- [ ] デフォルトブランチに merge
- [ ] すべてのエンジニアが `.github/copilot-instructions.md` を Copilot に教えた
- [ ] 最初 month は遵守状況を code review で確認
- [ ] 3 ヶ月ごとに更新・改善を協議

---

### 🔗 参考テンプレート

- [GitHub Copilot Documentation](https://docs.github.com/en/copilot/using-github-copilot/prompt-engineering)
- [Awesome Copilot Instructions](https://github.com/search?q=copilot-instructions.md)
