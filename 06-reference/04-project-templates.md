### よくあるプロジェクト種別ごとの「Copilot Best Practice」ガイド

各プロジェクトタイプの初期セットアップと推奨パターンを紹介します。

---

### 🎯 プロジェクトタイプ別ガイド

#### 1️⃣ Web API (ASP.NET Core)

**典型的なレイヤ構造**:
```
Presentation (Controllers, DTOs)
    ↓ (dependency)
Application (Services, Use Cases)
    ↓
Domain (Entities, Value Objects)
    ↓
Infrastructure (Repository, External Services)
```

**Copilot Best Practices**:

```csharp
// 1️⃣ Domain Entity
[Chat] 「UserAggregateRoot エンティティ定義」
public class User
{
    public int Id { get; set; }
    public string Email { get; set; }
    
    public void ChangeEmail(string newEmail)
    {
        // Domain logic
    }
}

// 2️⃣ Repository Interface
[Chat] 「IRepository<T> パターンに従うユーザーリポジトリを」
public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task<User> GetByEmailAsync(string email);
    Task AddAsync(User user);
    Task UpdateAsync(User user);
    Task SaveChangesAsync();
}

// 3️⃣ Application Service
[Chat] 「CreateUserUseCase を実装」
public class CreateUserService
{
    private readonly IUserRepository _repository;
    
    public async Task<UserDto> CreateUserAsync(CreateUserRequest request)
    {
        var user = new User { Email = request.Email };
        await _repository.AddAsync(user);
        await _repository.SaveChangesAsync();
        return MapToDto(user);
    }
}

// 4️⃣ API Controller
[Copilot Inline] コメントで endpoint 説明
[HttpPost("users")]
public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request)
{
    var result = await _userService.CreateUserAsync(request);
    return CreatedAtAction(nameof(GetUser), new { id = result.Id }, result);
}

// 5️⃣ Tests
[Agent Mode] 「このサービスの包括的なユニットテスト」
public class CreateUserServiceTests { ... }
```

**Copilot アクティベーション頻度**:
- Inline: 50% (boilerplate DTOs, simple methods)
- Chat: 30% (architecture questions, patterns)
- Edits: 15% (refactorings across files)
- Agent: 5% (test generation)

---

#### 2️⃣ Console Application / Worker Service

**典型的なシーン**:
- バッチ処理
- スケジューラタスク
- バックグラウンドジョブ

**Copilot Best Practices**:

```csharp
// 1️⃣ Dependency Injection Setup
var builder = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
        services.AddScoped<IDataProcessingService, DataProcessingService>();
        services.AddScoped<IRepository<Record>, RecordRepository>();
        services.AddHostedService<ProcessingWorker>();
    });

var host = builder.Build();
await host.RunAsync();

// [Chat] 「Dependency Injection セットアップ生成」でテンプレート

// 2️⃣ Main Processing Logic
public class ProcessingWorker : BackgroundService
{
    // [Inline] コメント + Copilot で _service.ProcessAsync() 補完
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                await _processingService.ProcessAsync(stoppingToken);
                await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Processing error");
            }
        }
    }
}

// 3️⃣ Error Handling & Retry
// [Chat] 「Polly retry policy の設定」で resilience

public void ConfigureRetryPolicy(IServiceCollection services)
{
    var retryPolicy = Policy
        .Handle<HttpRequestException>()
        .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
        .WaitAndRetryAsync(
            retryCount: 3,
            sleepDurationProvider: retryAttempt =>
                TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))
        );
    
    services.AddSingleton(retryPolicy);
}

// 4️⃣ Tests
// [Agent Mode] 「BackgroundService のユニットテスト」
public class ProcessingWorkerTests
{
    [Fact]
    public async Task ExecuteAsync_ProcessesRecords_Successfully()
    {
        // Arrange
        var mockService = new Mock<IDataProcessingService>();
        var worker = new ProcessingWorker(mockService.Object, logger);
        
        // Act
        var cts = new CancellationTokenSource(TimeSpan.FromSeconds(1));
        await worker.StartAsync(cts.Token);
        
        // Assert
        mockService.Verify(s => s.ProcessAsync(It.IsAny<CancellationToken>()), Times.Once);
    }
}
```

**Copilot アクティベーション頻度**:
- Inline: 40% (boilerplate service setup)
- Chat: 40% (retry policies, error handling patterns)
- Edits: 15% (refactoring retry logic across files)
- Agent: 5% (test generation)

---

#### 3️⃣ Class Library / SDK

**典型的なシーン**:
- NuGet パッケージ
- 他プロジェクトに imported

**Copilot Best Practices**:

```csharp
// 1️⃣ Public Interface 設計（最重要）
// [Chat] 「この SDK を使う側の視点で公開 API を」
public interface IPaymentGateway
{
    Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request);
    Task<PaymentRefundResult> RefundPaymentAsync(string transactionId, decimal amount);
}

// 2️⃣ Implementation with Sensible Defaults
[Inline] コメント → Copilot 補完
public class StripePaymentGateway : IPaymentGateway
{
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
    {
        // Implementation
    }
}

// 3️⃣ XMLDoc (Critical for Library Code)
// [Chat] 「このクラスのもう少し詳しい XMLDoc」
/// <summary>
/// Stripe payment processing gateway implementation.
/// 
/// 使用例:
/// <code>
/// var gateway = new StripePaymentGateway(apiKey);
/// var result = await gateway.ProcessPaymentAsync(new PaymentRequest { ... });
/// </code>
/// </summary>
public class StripePaymentGateway { }

// 4️⃣ Strong Validation
// [Copilot] 入力検証は必須
public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
{
    if (request == null)
        throw new ArgumentNullException(nameof(request));
    
    if (string.IsNullOrWhiteSpace(request.CardNumber))
        throw new ArgumentException("Card number required", nameof(request));
    
    // ... processing
}

// 5️⃣ Extensive Tests + Examples
// [Agent Mode] 「各メソッドの正常系・異常系・境界値テスト」
public class StripePaymentGatewayTests { }

// Plus: Example project (ExampleConsole)
public static async Task Main()
{
    var gateway = new StripePaymentGateway("sk_test_...");
    var result = await gateway.ProcessPaymentAsync(new PaymentRequest { ... });
    Console.WriteLine($"Payment {(result.Success ? "succeeded" : "failed")}");
}
```

**Copilot アクティベーション頻度**:
- Inline: 30% (defensive coding, small logic)
- Chat: 50% (API design, public interface review)
- Edits: 10% (validation across methods)
- Agent: 10% (comprehensive test generation + examples)

---

#### 4️⃣ Microservice

**典型的なアーキテクチャ**:
```
Service A (OrderService)
    ↓ (HTTP/gRPC)
Service B (PaymentService)
    ↓
Service C (NotificationService)
```

**Copilot Best Practices**:

```csharp
// 1️⃣ Resilience Patterns (Circuit Breaker, Retry)
// [Chat] 「マイクロサービス間の呼び出し、Polly circuit breaker」
services.AddHttpClient<IPaymentServiceClient>()
    .AddTransientHttpErrorPolicy(p => p
        .Handle<HttpRequestException>()
        .OrResult(r => !r.IsSuccessStatusCode)
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 3,
            durationOfBreak: TimeSpan.FromSeconds(30)
        )
    );

// 2️⃣ Async Messaging (Service Communication)
// [Chat] 「RabbitMQ で OrderCreated イベント publish」
public class OrderService
{
    public async Task<CreateOrderResult> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order { ... };
        await _repository.AddAsync(order);
        await _repository.SaveChangesAsync();
        
        // Publish event
        var orderCreatedEvent = new OrderCreatedEvent { OrderId = order.Id };
        await _messageBus.PublishAsync(orderCreatedEvent);
        
        return CreateOrderResult.Success(order.Id);
    }
}

// 3️⃣ Distributed Tracing / Logging
// [Chat] 「OpenTelemetry / Application Insights 設定」
services.AddApplicationInsightsTelemetry();
services.AddOpenTelemetry()
    .WithTracing(builder => builder
        .AddHttpClientInstrumentation()
        .AddAspNetCoreInstrumentation()
        .AddConsoleExporter()
    );

// 4️⃣ Health Checks
// [Inline] コメント → Copilot 実装
// [HttpGet("health")]
public async Task<IActionResult> HealthCheck()
{
    var dbHealthy = await _database.HealthCheckAsync();
    return dbHealthy ? Ok("Healthy") : StatusCode(503, "Unhealthy");
}

// 5️⃣ Contract Tests
// [Agent Mode] 「PaymentService との contract test」
public class PaymentServiceContractTests
{
    [Fact]
    public async Task ProcessPaymentAsync_WithValidRequest_ReturnsSuccess()
    {
        // Mock PaymentService behavior
        // Assert OrderService integrates correctly
    }
}
```

**Copilot アクティベーション頻度**:
- Inline: 30% (setup, simple methods)
- Chat: 50% (resilience patterns, messaging, tracing)
- Edits: 10% (refactoring across files)
- Agent: 10% (contract/integration tests)

---

#### 5️⃣ Console App (Simple Automation / Scripts)

**典型的なシーン**:
- データマイグレーション
- レポート生成
- 定期メンテナンス

**Copilot Best Practices**:

```csharp
// [Chat] 「CSV ファイルを読み込んで DB に insert するコンソールアプリ」
using var reader = new StreamReader("data.csv");
using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

await foreach (var record in csv.GetRecordsAsync<UserRecord>())
{
    var user = new User { Email = record.Email, Name = record.Name };
    await _repository.AddAsync(user);
}

await _repository.SaveChangesAsync();
Console.WriteLine("✅ Data imported successfully");
```

**Copilot アクティベーション頻度**:
- Inline: 70% (file I/O, parsing, simple transforms)
- Chat: 30% (error handling strategy, optimization hints)

---

### 📋 すべてのプロジェクトタイプに共通

#### 1. `.github/copilot-instructions.md` 必須

```markdown
## プロジェクト固有の Copilot ガイド

### Tech Stack
- .NET 8
- EF Core 8
- SQL Server
- xUnit + Moq

### Design Patterns
- Repository Pattern for all data access
- Strategy Pattern for multiple implementations
- Dependency Injection (constructor only)

### Naming Conventions
- PascalCase for classes/methods
- _camelCase for private fields

### Security
- Parameterized queries ONLY
- No hardcoded credentials
- Input validation on all public methods
```

#### 2. テストテンプレート

すべてのプロジェクトで同じテスト構造：
```
ProjectName/
├─ ProjectName.csproj
├─ src/
│  ├─ Services/
│  ├─ Repositories/
│  └─ Models/
└─ tests/
   └─ ProjectName.Tests/
      ├─ Services/
      ├─ Repositories/
      └─ Models/
```

#### 3. CI/CD Pipeline

```yaml
# .github/workflows/test.yml
- Build
- Unit Tests (xUnit)
- Code Coverage Analysis (Sonarqube)
- Publish (if coverage ≥ 80%)
```

---

### ✅ プロジェクト初期化チェック

```
☐ .github/copilot-instructions.md を作成
☐ README.md に Project Structure を記載
☐ Initial test project を作成
☐ Directory structure を上記に従って organize
☐ Build script を confirm
☐ CI/CD pipeline を setup
☐ Team に説明会を実施
```

---

### 🚀 推奨ワークフロー

#### Day 1: セットアップ
```
1. git clone
2. dotnet build
3. dotnet test
4. copilot-instructions.md を read
5. IDE opening
```

#### Day 2-3: First Feature
```
1. Issue/Task を読む
2. コメントで intent を書く → Copilot 補完
3. Chat でパターン確認
4. テスト駆動で develop
5. Code Review
```

---

### 📚 参考

- [Clean Architecture in .NET](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Microservices Patterns](https://microservices.io/patterns/)
- [Resilience and chaos engineering](https://learn.microsoft.com/en-us/azure/architecture/framework/resiliency/)
