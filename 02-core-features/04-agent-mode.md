### 複数ファイルにまたがる大規模タスクを自動実装

Agent Mode は Copilot のもっとも高度な機能。複数ファイルを自動生成・修正して、大規模な機能追加やリファクタリングを一度に実行できます。

---

### 💡 テクニック 1: 新機能の追加

#### シーン：ユーザー認証機能を一から追加したい

```
ユーザー: ユーザー認証機能を追加して。要件：
- パスワードはハッシュ化して保存（BCrypt）
- JWTトークンを発行
- ログイン/ログアウトAPI（コントローラー）
- レート制限（1分間に5回まで）
```

**Copilot（エージェント）による自動生成ファイル:**

```
📁 C#Project/
├── Controllers/
│   └── AuthController.cs（新規作成）
├── Services/
│   ├── AuthService.cs（新規作成）
│   └── JwtService.cs（新規作成）
├── Models/
│   ├── User.cs（更新）
│   └── LoginRequest.cs（新規作成）
├── Middleware/
│   └── RateLimitMiddleware.cs（新規作成）
├── appsettings.json（更新）
└── Program.cs（更新）
```

**生成されるコード例:**

```csharp
// AuthController.cs
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;
    
    [HttpPost("login")]
    public async Task<ActionResult<AuthResponse>> Login([FromBody] LoginRequest request)
    {
        var result = await _authService.LoginAsync(request.Email, request.Password);
        
        if (!result.Success)
            return Unauthorized(result.Message);
        
        return Ok(result);
    }
    
    [HttpPost("logout")]
    [Authorize]
    public async Task<IActionResult> Logout()
    {
        await _authService.LogoutAsync(User.FindFirst(ClaimTypes.NameIdentifier)?.Value);
        return Ok("ログアウトしました");
    }
}

// AuthService.cs
public class AuthService : IAuthService
{
    private readonly IUserRepository _userRepository;
    private readonly IJwtService _jwtService;
    private readonly ILogger<AuthService> _logger;
    
    public async Task<AuthResponse> LoginAsync(string email, string password)
    {
        var user = await _userRepository.GetByEmailAsync(email);
        
        if (user == null || !BCrypt.Net.BCrypt.Verify(password, user.PasswordHash))
        {
            _logger.LogWarning("ログイン失敗: {Email}", email);
            return AuthResponse.Failure("メールやパスワードが正しくありません");
        }
        
        var token = _jwtService.GenerateToken(user);
        return AuthResponse.Success(token);
    }
}
```

#### ポイント
- **複数ファイル作成・更新を自動化**
- **依存関係の自動設定**（DI コンテナなど）
- **エラーハンドリング** まで対応

---

### 💡 テクニック 2: アーキテクチャリファクタリング

#### シーン：データアクセス層を Repository パターンにリファクタリング

```
ユーザー: 
@workspace このプロジェクトのデータアクセス層を
リポジトリパターンにリファクタリングして
```

**Copilot（エージェント）の実行内容:**

1. `IRepository<T>` インターフェースを自動作成
2. 各エンティティのリポジトリクラスを実装
3. DbContext から直接呼び出している部分を修正
4. 依存性注入を設定（Program.cs）

**生成されるファイル:**

```csharp
// Repositories/IRepository.cs
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
}

// Repositories/UserRepository.cs
public class UserRepository : IRepository<User>
{
    private readonly AppDbContext _context;
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
    
    // ... 他のメソッド実装
}

// Program.cs（更新）
services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
services.AddScoped<IUserRepository, UserRepository>();
```

---

### 💡 テクニック 3: テスト自動生成

#### シーン：このクラスのユニットテストを生成したい

```
ユーザー: このクラスのユニットテストを生成して

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
    
    public double Divide(int a, int b)
    {
        if (b == 0)
            throw new DivideByZeroException("0で割ることはできません");
        return (double)a / b;
    }
}
```

**Copilot による自動生成:**

```csharp
// CalculatorTests.cs
using Xunit;
using FluentAssertions;

public class CalculatorTests
{
    private readonly Calculator _calculator;
    
    public CalculatorTests()
    {
        _calculator = new Calculator();
    }
    
    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(-1, -2, -3)]
    [InlineData(0, 0, 0)]
    public void Add_ShouldReturnSum_WhenCalled(int a, int b, int expected)
    {
        var result = _calculator.Add(a, b);
        result.Should().Be(expected);
    }
    
    [Theory]
    [InlineData(10, 2, 5.0)]
    [InlineData(5, 2, 2.5)]
    [InlineData(-10, 2, -5.0)]
    public void Divide_ShouldReturnQuotient_WhenValidInputs(int a, int b, double expected)
    {
        var result = _calculator.Divide(a, b);
        result.Should().Be(expected);
    }
    
    [Fact]
    public void Divide_ShouldThrowDivideByZeroException_WhenDivisorIsZero()
    {
        Action act = () => _calculator.Divide(10, 0);
        
        act.Should().Throw<DivideByZeroException>()
            .WithMessage("0で割ることはできません");
    }
}
```

### ポイント
- **テストケース自動生成** （正常系・異常系・境界値）
- **Xunit + FluentAssertions** の自動選択
- **命名規則** に従った自動生成

---

## 🎯 エージェントモードの適用シーン

| 要件 | 所要時間 | 複雑度 |
|-----|--------|------|
| **新機能追加**（複数ファイル） | 5-15 分 | ⭐⭐⭐ |
| **アーキテクチャ リファクタリング** | 10-30 分 | ⭐⭐⭐⭐ |
| **テスト自動生成** | 5-10 分 | ⭐⭐ |
| **データベースマイグレーション** | 5-10 分 | ⭐⭐ |
| **API エンドポイント一括追加** | 10-20 分 | ⭐⭐⭐ |

---

## ⚠️ エージェントモードの注意点

### ❌ 避けるべき指示
- 「いい感じにして」（曖昧）
- 「全部リファクタリングして」（スコープ不明確）
- 「セキュリティを上げて」（観点不明確）

### ✅ 推奨される指示
- 「Repository パターンを適用して」（パターン指定）
- 「ユーザー認証機能を追加：[具体的要件]」（スコープ明確）
- 「BCrypt を使ったパスワードハッシュ化」（実装技術指定）

### 代替案の活用
- エージェントが生成コードに不満
  → `@workspace` 指示で修正指示を追加
  → 「生成ファイルを削除して」で巻き戻し可能（git で）

---

## ✅ 習熟度チェック

- [ ] 複数ファイル作成指示ができた
- [ ] 要件を明確に述べて指示できた
- [ ] 生成されたコードを確認できた
- [ ] 依存性注入が自動設定されたことを理解

すべてチェック完了 → [03-hands-on-workflows/README](../03-hands-on-workflows/README.md) へ

---

## 🔗 参考

- [GitHub Copilot for Enterprise](https://github.com/features/copilot/plans)
- [Design Patterns in C#](https://refactoring.guru/design-patterns/csharp)
- [Dependency Injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)
