# 01 デザインパターン適用

## 既存コードにパターンを適用してリファクタリング

中級者にとって重要な「デザインパターンを実務に適用する」スキルを learning-by-doing で習得します。

---

## 🎯 シナリオ：支払い処理にストラテジーパターンを適用

### 状況
「複数の支払い方法（クレジットカード、PayPal、銀行振込）に対応したい」

### ❌ リファクタリング前：if-else で分岐

```csharp
public class OrderProcessor
{
    public async Task<bool> ProcessPaymentAsync(Order order, string paymentMethod)
    {
        if (paymentMethod == "CreditCard")
        {
            // クレジットカード処理
            var result = await _creditCardService.ChargeAsync(...);
            // ...
        }
        else if (paymentMethod == "PayPal")
        {
            // PayPal 処理
            var result = await _payPalService.CreatePaymentAsync(...);
            // ...
        }
        else if (paymentMethod == "BankTransfer")
        {
            // 銀行振込処理
            var result = await _bankService.RegisterTransferAsync(...);
            // ...
        }
        
        return true;
    }
}

// 問題：
// 1. 新しい支払い方法追加時に ProcessPaymentAsync を修正必要
// 2. 各方法のロジックが混在して読みづらい
// 3. テストが複雑
```

### ✅ リファクタリング後：Strategy パターン

**ステップ 1: インターフェース定義**

```csharp
/// <summary>
/// 支払い処理の戦略インターフェース
/// </summary>
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request);
    string PaymentMethod { get; }
}
```

**ステップ 2: 各戦略の実装**

```csharp
public class CreditCardPaymentStrategy : IPaymentStrategy
{
    private readonly ICreditCardService _creditCardService;
    private readonly ILogger<CreditCardPaymentStrategy> _logger;
    
    public string PaymentMethod => "CreditCard";
    
    public CreditCardPaymentStrategy(ICreditCardService creditCardService, ILogger<CreditCardPaymentStrategy> logger)
    {
        _creditCardService = creditCardService;
        _logger = logger;
    }
    
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
    {
        try
        {
            _logger.LogInformation("クレジットカード支払い処理開始: {Amount}", request.Amount);
            
            var result = await _creditCardService.ChargeAsync(
                request.CardNumber,
                request.ExpiryDate,
                request.Cvv,
                request.Amount
            );
            
            return new PaymentResult
            {
                Success = result.Success,
                TransactionId = result.TransactionId,
                Message = result.Message
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "クレジットカード支払いエラー");
            return PaymentResult.Failed("決済処理中にエラーが発生しました");
        }
    }
}

public class PayPalPaymentStrategy : IPaymentStrategy
{
    private readonly IPayPalService _payPalService;
    private readonly ILogger<PayPalPaymentStrategy> _logger;
    
    public string PaymentMethod => "PayPal";
    
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
    {
        var result = await _payPalService.CreatePaymentAsync(
            request.PayPalEmail,
            request.Amount,
            request.Currency
        );
        
        return new PaymentResult
        {
            Success = result.Status == "completed",
            TransactionId = result.Id,
            ApprovalUrl = result.ApprovalUrl,
            Message = result.Message
        };
    }
}
```

**ステップ 3: コンテキスト（処理オーケストレーター）**

```csharp
/// <summary>
/// 支払い処理のコンテキスト（戦略経由で実行）
/// </summary>
public class PaymentContext
{
    private readonly Dictionary<string, IPaymentStrategy> _strategies;
    private readonly ILogger<PaymentContext> _logger;
    
    public PaymentContext(IEnumerable<IPaymentStrategy> strategies, ILogger<PaymentContext> logger)
    {
        _strategies = strategies.ToDictionary(s => s.PaymentMethod);
        _logger = logger;
    }
    
    public async Task<PaymentResult> ExecutePaymentAsync(PaymentRequest request)
    {
        if (!_strategies.TryGetValue(request.PaymentMethod, out var strategy))
        {
            _logger.LogWarning("未対応の支払い方法: {PaymentMethod}", request.PaymentMethod);
            return PaymentResult.Failed($"未対応の支払い方法です: {request.PaymentMethod}");
        }
        
        _logger.LogInformation("支払い実行: {PaymentMethod}, {Amount}", 
            request.PaymentMethod, request.Amount);
        
        return await strategy.ProcessPaymentAsync(request);
    }
}
```

**ステップ 4: 依存性注入の設定**

```csharp
// Program.cs
services.AddScoped<IPaymentStrategy, CreditCardPaymentStrategy>();
services.AddScoped<IPaymentStrategy, PayPalPaymentStrategy>();
services.AddScoped<IPaymentStrategy, BankTransferPaymentStrategy>();
services.AddScoped<PaymentContext>();
```

### 効果
- ✅ **新しい支払い方法追加が簡単** — 新クラス作成 + DI 登録のみ
- ✅ **テストが容易** — 各戦略のモック化が簡単
- ✅ **責務分離** — 各支払い方法のロジックが独立
- ✅ **保守性向上** — 既存コード修正不要（Open/Closed 原則）

---

## 🎯 リポジトリパターンの応用

### シーン：複数のデータソース（SQL, NoSQL, API）に対応

```csharp
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
}

// SQL Database 実装
public class SqlUserRepository : IRepository<User>
{
    private readonly DbContext _context;
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
    
    // ... 実装
}

// MongoDB 実装
public class MongoUserRepository : IRepository<User>
{
    private readonly IMongoCollection<User> _collection;
    
    public async Task<User> GetByIdAsync(int id)
    {
        // MongoDB クエリ
    }
    
    // ... 実装
}

// 使用時：どのリポジトリか意識しない
public class UserService
{
    private readonly IRepository<User> _userRepository;
    
    public UserService(IRepository<User> userRepository)
    {
        _userRepository = userRepository;
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        return await _userRepository.GetByIdAsync(id);
    }
}
```

---

## ✅ 習熟度チェック

- [ ] Strategy パターンの概念が理解できた
- [ ] 既存の if-else 分岐を Strategy に置き換えられた
- [ ] インターフェース設計ができた
- [ ] 依存性注入の設定ができた
- [ ] 新しいパターン追加の簡単さを実感

すべてチェック完了 → [02-performance-optimization.md](02-performance-optimization.md) へ

---

## 🔗 参考

- [Design Patterns in C#](https://refactoring.guru/design-patterns/csharp)
- [Strategy Pattern](https://refactoring.guru/design-patterns/strategy)
- [SOLID Principles](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#solid)
