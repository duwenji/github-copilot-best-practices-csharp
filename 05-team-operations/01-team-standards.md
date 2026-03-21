### コード品質と開発効率のための共通ルールを作る

チーム全体で Copilot を効果的に活用するには「標準」が必須。一貫したコード品質、保守性、Copilot との付き合い方を定義します。

---

### 🏢 チーム標準のねらい

#### 問題シーン
```
チーム 5 人が個々に Copilot を使う…

開発者 A: 支払い処理を Strategy パターンで実装
開発者 B: if-else で複数条件分岐
開発者 C: 前のコードをコピーして修正

↓ 結果

コードレビュー地獄：
- 「このコードはなぜ Strategy ?」
- 「なぜ if-else ?」
- 「重複 が多い…」
- パターン論争に時間浪費

→ 品質 , スピード低下
```

#### 解決策：チーム標準を決める
```
チーム会：「複数実装パターンはどう実装する？」
→ 結論：「Strategy パターン を標準とする」
→ `.github/copilot-instructions.md` に記載
→ 全員が Copilot にこの指示を与える
→ 新規実装は自動的に Strategy で生成

↓ 結果

- パターン論争 0件
- コードレビュー時間 50% 削減
- 新人の Learning Curve 短期化
```

---

### 📋 チーム標準ドキュメントのテンプレート

#### 1. プロジェクト基本情報

```markdown
## Project Information
- **Team Name**: E-Commerce Backend Team
- **Tech Stack**: .NET 8, EF Core 8, SQL Server
- **Repository**: github.com/ourcompany/ecommerce-api
- **Team Size**: 5 engineers
- **Target Coverage**: 80% unit test
```

#### 2. 命名規則

```markdown
## Code Naming Conventions

### Classes & Interfaces
- PascalCase: `UserService`, `IRepository<T>`
- Action-oriented: `UserAppService` (application service)
- Domain-driven: `User` (entity), `UserId` (value object)

### Methods
- PascalCase + action verb: `GetUserAsync`, `ProcessPaymentAsync`
- Async methods: `*Async` suffix required
- Query methods: `Get*`, `Find*`, `List*`
- Mutating methods: `Create*`, `Update*`, `Delete*`

### Variables & Fields
- Private fields: `_userRepository` (leading underscore)
- Local variables: `camelCase` for all
- Constants: `UPPER_SNAKE_CASE`

### Database
- Table names: `PascalCase` or `snake_case` (チーム決定)
- Column names: `PascalCase`
- Stored procedures: `sp_Action_EntityName`
```

#### 3. デザインパターン標準

```markdown
## Design Patterns

### When to Use
| Pattern | Use Case | Example |
|---------|----------|---------|
| Strategy | Multiple implementations | Payment methods (CreditCard, PayPal, BankTransfer) |
| Repository | Data access abstraction | IUserRepository |
| Factory | Complex object creation | PaymentStrategyFactory |
| Dependency Injection | Loose coupling | Constructor injection |
| Decorator | Feature enhancement | Caching decorator |

### Standard Pattern: Strategy
For all cases where multiple implementations exist.
```csharp
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request);
}

public class StrategyFactory
{
    public IPaymentStrategy GetStrategy(string paymentMethod)
    {
        return paymentMethod switch
        {
            "CreditCard" => new CreditCardPaymentStrategy(...),
            "PayPal" => new PayPalPaymentStrategy(...),
            _ => throw new NotSupportedException()
        };
    }
}
```

### Standard Pattern: Repository
For all data access.
```csharp
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}
```

### Standard Pattern: Unit of Work
For transaction management across multiple repositories.
```csharp
public interface IUnitOfWork : IDisposable
{
    IRepository<User> Users { get; }
    IRepository<Order> Orders { get; }
    Task SaveChangesAsync();
}
```
```

#### 4. エラーハンドリング標準

```markdown
## Error Handling

### All public methods MUST include structured error handling:

```csharp
public async Task<ApiResponse<T>> GetDataAsync<T>(int id)
{
    try
    {
        _logger.LogInformation("Fetching {EntityType} with ID {Id}", typeof(T).Name, id);
        
        var data = await _repository.GetByIdAsync(id);
        
        if (data == null)
        {
            _logger.LogWarning("Entity not found: {EntityType} ID {Id}", typeof(T).Name, id);
            return ApiResponse<T>.NotFound($"{typeof(T).Name} not found");
        }
        
        return ApiResponse<T>.Success(data);
    }
    catch (InvalidOperationException ex)
    {
        _logger.LogError(ex, "Invalid operation for {EntityType}", typeof(T).Name);
        return ApiResponse<T>.BadRequest(ex.Message);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Unexpected error fetching {EntityType}", typeof(T).Name);
        return ApiResponse<T>.InternalError("An unexpected error occurred");
    }
}
```
```

#### 5. テスト基準

```markdown
## Testing Standards

### Unit Tests (xUnit)
- **Coverage Target**: ≥ 80% for business logic
- **Naming**: `[MethodName]_[Scenario]_[ExpectedResult]`
  Example: `ProcessPayment_WithValidCard_ReturnsSuccess`
- **Tools**: xUnit, Moq, FluentAssertions
- **Location**: `*.Tests` directory, mirror structure

### Test File Template
```csharp
public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _mockRepository;
    private readonly Mock<IPaymentService> _mockPayment;
    private readonly OrderService _sut; // System Under Test
    
    public OrderServiceTests()
    {
        _mockRepository = new Mock<IOrderRepository>();
        _mockPayment = new Mock<IPaymentService>();
        _sut = new OrderService(_mockRepository.Object, _mockPayment.Object);
    }
    
    [Theory]
    [InlineData(100)]
    [InlineData(1000)]
    public async Task ProcessOrder_WithValidAmount_ReturnsSuccess(decimal amount)
    {
        // Arrange
        var order = new Order { Id = 1, Amount = amount };
        _mockRepository.Setup(r => r.GetByIdAsync(1)).ReturnsAsync(order);
        _mockPayment.Setup(p => p.ProcessAsync(amount))
            .ReturnsAsync(PaymentResult.Success("tx123"));
        
        // Act
        var result = await _sut.ProcessOrderAsync(1);
        
        // Assert
        Assert.True(result.Success);
        _mockPayment.Verify(p => p.ProcessAsync(amount), Times.Once);
    }
}
```

### Integration Tests
- **Scope**: Multiple components
- **Tools**: WebApplicationFactory, xUnit
- **Frequency**: Run before merge

### E2E Tests
- **Scope**: Full API workflow
- **Tools**: WebApplicationFactory, custom test client
- **Frequency**: Run in CI/CD pipeline (slower, so selective)
```

#### 6. パフォーマンス基準

```markdown
## Performance Standards

### API Response Time
- Normal queries: ≤ 500ms (P95 percentile)
- Search queries: ≤ 2000ms (P95)
- Bulk operations: ≤ 5000ms (P95)

### Database Query
- Single query: ≤ 100ms
- Batch query: ≤ 500ms total
- Report queries: ≤ 5000ms

### Monitoring
- Use Application Insights / New Relic
- Set alerts for P95 > threshold
- Weekly performance review in team standup

## Optimization Patterns
1. **Always use `.AsNoTracking()`** for read-only queries
2. **Include related data**: `.Include(x => x.Orders)`
3. **Filter at DB level**: Never `.ToList()` before `.Where()`
4. **Parallel API calls**: `Task.WhenAll()`
5. **Cache immutable data**: 5-min cache for reference data
```

#### 7. Copilot 使用ルール

```markdown
## Copilot Usage Guidelines

### ✅ Recommended Uses
1. **Comment-Driven Development**
   ```csharp
   // Get all users whose email is invalid, log their emails, and return count
   ```
   → Copilot generates: `.Where(u => !IsValidEmail(u.Email)).ForEach(...)`

2. **Test Code Generation**
   - Paste method signature → Copilot generates unit tests
   - Adjust as needed (Copilot gets 60-70% right)

3. **Boilerplate Code**
   - Repository implementations
   - DTO mappings
   - API endpoint basic structure

4. **Documentation**
   - Generate XMLDoc comments from code
   - Generate README sections

### ❌ Avoid
1. **Blind Trust** — Review all Copilot code suggestions
2. **Security Assumptions** — Always sanitize inputs, parameterize queries
3. **AI-First Design** — Design first, then code with Copilot
4. **No Testing** — All Copilot code must pass tests before merge

### Copilot Verification Checklist
- [ ] Code follows team conventions (naming, patterns)
- [ ] Includes error handling
- [ ] Includes logging
- [ ] Includes unit tests
- [ ] No SQL injection vulnerabilities
- [ ] No null reference exceptions
```

#### 8. コードレビューチェックリスト

```markdown
## Code Review Checklist

Reviewer は以下をチェック：

### Functional Correctness
- [ ] Logic matches issue/PR description
- [ ] Edge cases handled
- [ ] Error scenarios covered

### Code Quality
- [ ] Follows team naming conventions
- [ ] Uses standard design patterns
- [ ] No code duplication
- [ ] Single responsibility per method
- [ ] Cyclomatic complexity < 10

### Testing
- [ ] Unit tests present
- [ ] Test cases cover normal + edge + error scenarios
- [ ] Copilot-generated tests reviewed & understood

### Security
- [ ] No hardcoded credentials
- [ ] SQL queries parameterized
- [ ] Input validation present
- [ ] No sensitive data in logs

### Performance
- [ ] Database queries optimized (`.AsNoTracking()`, `.Include()`)
- [ ] No N+1 query problems
- [ ] Expensive operations cached if applicable

### Documentation
- [ ] XMLDoc comments for public methods
- [ ] Complex logic explained
- [ ] API contract documented

### Copilot Generated Code
- [ ] All Copilot-generated code explicitly noted
- [ ] Code doesn't blindly trust AI suggestions
- [ ] Security review if external inputs involved
```

---

### 🚀 チーム標準の導入ステップ

#### Week 1: 基本ルールを決定
```
Team meeting (2時間):
1. 命名規則確認（既存 + 更新する箇所）
2. デザインパターン標準（Strategy, Repository, etc.）
3. エラーハンドリング標準
4. テスト基準（覆陰率, ツール, 場所）
```

#### Week 2: ドキュメント作成 & CI/CD パイプライン更新
```
1. `.github/copilot-instructions.md` 作成
2. `.editorconfig` 作成（自動フォーマット）
3. linter / formatter 設定（StyleCop, EditorConfig Analyzer）
4. Git pre-commit hook で強制
```

#### Week 3: チーム全体トレーニング
```
1. 標準ドキュメントの説明会（30分）
2. Copilot + 標準の演習（1時間）
3. 質問受け付けタイム
```

#### Week 4: 実装 + フィードバック
```
新機能開発で標準を使用
→ コードレビューで「標準守れてるか」チェック
→ 月末レトロスペクティブで「標準、改善点」協議
```

---

### ✅ チェックリスト

- [ ] チーム全体で「デザインパターン標準」を決定
- [ ] `.github/copilot-instructions.md` を作成
- [ ] `.editorconfig` を設定
- [ ] コードレビュー チェックリスト を定義
- [ ] チーム全体で標準をレビュー＆承認
- [ ] 学習資料を共有（Wiki / Confluence）

---

### 🔗 参考

- [EditorConfig](https://editorconfig.org/)
- [StyleCop Analyzers](https://github.com/DotNetAnalyzers/StyleCopAnalyzers)
- [C# Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
