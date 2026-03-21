# 03 テスト容易性

## テストしやすい設計にリファクタリング

「このコード、テストが書きやすい/難しい」の差は設計に由来します。依存性注入とモックを活用した設計を学びます。

---

## 🎯 シーン：依存性が多いコードをテスト可能に

### ❌ テスト困難なコード

```csharp
public class OrderProcessor
{
    // 直接 new していて、モック化できない！
    public async Task<bool> ProcessOrder(int orderId)
    {
        // 直接データベースに接続
        using var conn = new SqlConnection("connection string");
        var order = await conn.QueryFirstOrDefaultAsync<Order>(
            "SELECT * FROM Orders WHERE Id = @Id", new { Id = orderId });
        
        // 直接メール送信
        using var smtp = new SmtpClient("smtp.example.com");
        await smtp.SendMailAsync(new MailMessage(
            "from@example.com", 
            order.CustomerEmail, 
            "Order Processed", 
            "Your order has been processed"
        ));
        
        return true;
    }
}

// テスト困難な理由：
// 1. SqlConnection をモック化できない
// 2. SmtpClient をモック化できない
// 3. 実際のメール送信が実行される
// 4. 実際のデータベース接続が必要
```

### ✅ テスト可能な設計

**ステップ 1: インターフェース定義**

```csharp
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(int orderId);
    Task UpdateAsync(Order order);
}

public interface INotificationService
{
    Task SendOrderConfirmationAsync(int orderId, string customerEmail);
}

public interface IPaymentService
{
    Task<PaymentResult> ProcessPaymentAsync(int orderId, decimal amount);
}
```

**ステップ 2: 依存性注入でクラス実装**

```csharp
public class OrderProcessor
{
    private readonly IOrderRepository _orderRepository;
    private readonly INotificationService _notificationService;
    private readonly IPaymentService _paymentService;
    private readonly ILogger<OrderProcessor> _logger;
    
    // コンストラクタで依存性を注入
    public OrderProcessor(
        IOrderRepository orderRepository,
        INotificationService notificationService,
        IPaymentService paymentService,
        ILogger<OrderProcessor> logger)
    {
        _orderRepository = orderRepository;
        _notificationService = notificationService;
        _paymentService = paymentService;
        _logger = logger;
    }
    
    public async Task<OrderResult> ProcessOrderAsync(int orderId)
    {
        try
        {
            _logger.LogInformation("注文処理開始: {OrderId}", orderId);
            
            // 注文の取得
            var order = await _orderRepository.GetByIdAsync(orderId);
            if (order == null)
            {
                return OrderResult.Failed("注文が見つかりません");
            }
            
            // 支払い処理
            var paymentResult = await _paymentService.ProcessPaymentAsync(
                orderId, order.TotalAmount);
            
            if (!paymentResult.Success)
            {
                return OrderResult.Failed($"支払い失敗: {paymentResult.Error}");
            }
            
            // 注文ステータス更新
            order.Status = OrderStatus.Processed;
            order.PaymentTransactionId = paymentResult.TransactionId;
            await _orderRepository.UpdateAsync(order);
            
            // 通知送信
            await _notificationService.SendOrderConfirmationAsync(
                orderId, order.CustomerEmail);
            
            _logger.LogInformation("注文処理完了: {OrderId}", orderId);
            
            return OrderResult.Success(orderId, paymentResult.TransactionId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "注文処理エラー: {OrderId}", orderId);
            return OrderResult.Failed("注文処理中にエラーが発生しました");
        }
    }
}
```

**ステップ 3: テストコード**

```csharp
public class OrderProcessorTests
{
    [Fact]
    public async Task ProcessOrderAsync_ShouldSucceed_WhenValidOrder()
    {
        // Arrange
        var orderId = 1;
        var order = new Order 
        { 
            Id = orderId, 
            TotalAmount = 100, 
            CustomerEmail = "test@example.com" 
        };
        
        var mockRepo = new Mock<IOrderRepository>();
        mockRepo.Setup(r => r.GetByIdAsync(orderId))
            .ReturnsAsync(order);
        
        var mockPayment = new Mock<IPaymentService>();
        mockPayment.Setup(p => p.ProcessPaymentAsync(orderId, 100))
            .ReturnsAsync(PaymentResult.Success("tx123"));
        
        var mockNotification = new Mock<INotificationService>();
        var mockLogger = new Mock<ILogger<OrderProcessor>>();
        
        var processor = new OrderProcessor(
            mockRepo.Object,
            mockNotification.Object,
            mockPayment.Object,
            mockLogger.Object
        );
        
        // Act
        var result = await processor.ProcessOrderAsync(orderId);
        
        // Assert
        Assert.True(result.Success);
        mockRepo.Verify(r => r.UpdateAsync(It.Is<Order>(o => 
            o.Status == OrderStatus.Processed)), Times.Once);
        mockNotification.Verify(n => n.SendOrderConfirmationAsync(
            orderId, order.CustomerEmail), Times.Once);
    }
    
    [Fact]
    public async Task ProcessOrderAsync_ShouldFail_WhenOrderNotFound()
    {
        // Arrange
        var mockRepo = new Mock<IOrderRepository>();
        mockRepo.Setup(r => r.GetByIdAsync(It.IsAny<int>()))
            .ReturnsAsync((Order)null);
        
        var mockPayment = new Mock<IPaymentService>();
        var mockNotification = new Mock<INotificationService>();
        var mockLogger = new Mock<ILogger<OrderProcessor>>();
        
        var processor = new OrderProcessor(
            mockRepo.Object,
            mockNotification.Object,
            mockPayment.Object,
            mockLogger.Object
        );
        
        // Act
        var result = await processor.ProcessOrderAsync(999);
        
        // Assert
        Assert.False(result.Success);
        Assert.Equal("注文が見つかりません", result.Error);
    }
}
```

### テスト容易性の改善
- ✅ **Moq でモック化** — インターフェース経由のため簡単
- ✅ **実装と隔離** — テストが実装の詳細に依存しない
- ✅ **複数ケースをカバー** — 正常系・異常系を独立テスト

---

## 🎯 テスト可能な設計の原則

### 原則 1: Interface を活用

```csharp
// ✅ テスト可能
public class Service
{
    private readonly IRepository _repository;
    
    public Service(IRepository repository)
    {
        _repository = repository;
    }
}

// ❌ テスト困難
public class Service
{
    private readonly ConcreteRepository _repository = new();
}
```

### 原則 2: 依存性を外部から注入

```csharp
// ✅ テスト可能（DI コンテナが自動配線）
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
}

// ❌ テスト困難（ベタ書き）
public class UserService
{
    private IUserRepository _userRepository = ServiceLocator.GetRepository();
}
```

### 原則 3: ビジネスロジックと I/O を分離

```csharp
// ❌ テスト困難（ロジック + DB 操作が混在）
public async Task<bool> ValidateOrder(Order order)
{
    var isValid = order.Amount > 0 && order.Items.Count > 0;
    await _context.SaveChangesAsync(); // I/O と混在
    return isValid;
}

// ✅ テスト可能（ロジックだけ）
public bool ValidateOrder(Order order)
{
    // ビジネスロジックのみ
    return order.Amount > 0 && order.Items.Count > 0;
}

// I/O は別メソッドで
public async Task SaveOrderAsync(Order order)
{
    if (ValidateOrder(order)) // ロジック呼び出し
    {
        await _context.SaveChangesAsync();
    }
}
```

---

## ✅ 習熟度チェック

- [ ] インターフェース設計ができた
- [ ] 依存性注入の実装ができた
- [ ] Moq を使ったモック設定ができた
- [ ] テストケースを複数作成できた
- [ ] 正常系・異常系・境界値をテストできた

すべてチェック完了 → [04-role-mastery/README](../04-role-mastery/README.md) へ

---

## 🔗 参考

- [Unit Testing Best Practices](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
- [Moq Documentation](https://github.com/Moq/moq4/wiki/Quickstart)
- [xUnit.net Documentation](https://xunit.net/docs/getting-started/netcore)
- [Dependency Injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)
