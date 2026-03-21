# 01 トラブルシューティング

## よくあるエラーと解決策を索引形式で整理

Copilot や C# 開発での困った場合、ここを検索してください。

---

## 🔍 問題インデックス

| 症状 | 原因 | 解決策 |
|------|------|--------|
| [Copilot が提案してくれない](#copilot-が提案してくれない) | コメント曖昧 / Context 不足 | コメント詳細化 / 関連コード表示 |
| [Copilot 提案が外れてる](#copilot-提案が外れてる) | パターン不一致 / 仕様誤解 | Chat で仕様確認 / コメント再作成 |
| [テストが赤になる](#テストが赤になる) | 実装ミス / モック設定誤り | Step-by-step デバッグ / 仕様確認 |
| [NullReferenceException](#nullreferenceexception) | Null チェック漏れ | Stack trace 確認 / Null 値検証追加 |
| [N+1 クエリ問題](#n1-クエリ問題) | Include 漏れ | Include 追加 / SQL Plan 確認 |
| [パフォーマンス低下](#パフォーマンス低下) | クエリ非効率 / キャッシュなし | SQL Plan 測定 / キャッシュ導入 |
| [セキュリティ警告](#セキュリティ警告) | SQL Injection / 入力未検証 | Parameterized Query / Input validation |
| [Unit Test カバレッジ不足](#unit-test-カバレッジ不足) | テストケース欠落 | Copilot でテスト生成 / 動機筋条件確認 |
| [Code Review 指摘多発](#code-review-指摘多発) | チーム標準不理解 | `.github/copilot-instructions.md` 確認 |
| [Async 関連エラー](#async-関連エラー) | ConfigureAwait 漏れ / Deadlock | ConfigureAwait(false) 追加 / 同期型排除 |

---

## Copilot が提案してくれない

### 症状
```
コメントを書いたのに Copilot が補完提案を出さない
Ctrl+Enter して Chat 開いても無反応
```

### 原因と対策

**原因 1: コメントが曖昧**
```csharp
// ❌ 曖昧
// ユーザーを処理する

// ✅ 詳細
// メールアドレスが @example.com のドメインのみの
// ユーザーをフィルタリングして、フィルタ結果を返す
```

**解決**: コメント詳細化。特に：
- 「入力」と「出力」を明記
- ビジネスロジック（なぜ？）を書く
- 複数行コメントを使う

**原因 2: Context がない**
```csharp
// ❌ Copilot が context を失う
var users = ...   // この行の1000行前でクラス定義

// ✅ Copilot が context を保つ
// 関連する interface / class を同一ファイルに
// また是非 chat で「このメソッドで できることは？」と聞く
```

**解決**：
- Chat を使って詳細説明
- スクリーンショット + 説明を貼り付け
- 関連ファイルを parallel で開く

---

## Copilot 提案が外れてる

### 症状
```csharp
// Copilot がこう生成した
var result = await service.DoSomething();

// 但し実際には…
// - DoSomething は存在しない（DoSomethingElse が正）
// - 返り値の型が違う
// - Exception handle が必要
```

### 原因と対策

**原因 1: テンプレート不一致**

Copilot は「似たコード」で提案します。期待と違う場合：

```csharp
// ❌ Copilot が古いパターンで提案
public bool ProcessPayment(PaymentMethod method)
{
    if (method == "CreditCard") { ... }
    else if (method == "PayPal") { ... }
}

// ✅ Strategy パターン を期待していた
public async Task<PaymentResult> ProcessPaymentAsync(Order order)
{
    var strategy = _paymentFactory.GetStrategy(order.PaymentMethod);
    return await strategy.ProcessPaymentAsync(order);
}
```

**解決**：
1. Chat で「Strategy パターンで実装してください」と明示
2. `.github/copilot-instructions.md` を Studio に教える
3. 既存コード（new Strategy 実装）をを Copilot に見せる

**原因 2: 仕様誤解**

```csharp
// コメント：「ユーザーを取得」
// Copilot が ALL users を返す

// ✅ より詳細に
// メールが検証済みのユーザーのみを SELECT して、
// ページング対応で最初の 10 件を返す
```

**解決**：コメントで「ビジネス要件」を書く

---

## テストが赤になる

### 症状
```
[xUnit] Test failed: AssertionError
実装コードは「できてる」ようなのにテストが失敗
```

### 原因と対策

**原因 1: モック設定ミス**

```csharp
// ❌ モック返却値を設定忘れ
var mockRepo = new Mock<IOrderRepository>();
// .Setup(...) を書かないと NullReferenceException

// ✅ モック設定を明示的に
mockRepo.Setup(r => r.GetByIdAsync(1))
    .ReturnsAsync(new Order { Id = 1, Total = 100 });
```

**解決**：
- `Mock<T>.Setup()` を忘れずに
- 全デリケンデンシーにモック設定を
- Moq ドキュメント参照：https://github.com/moq/moq4/wiki/Quickstart

**原因 2: Expected vs Actual の混同**

```csharp
// ❌ テストロジックが逆
[Fact]
public void Multiply_2_3_Should_Return_6()
{
    var result = calc.Multiply(2, 3);
    Assert.Equal(2, result);  // ← 期待値と実結果が逆！
}

// ✅ 正しい順序
[Fact]
public void Multiply_2_3_Should_Return_6()
{
    var result = calc.Multiply(2, 3);
    Assert.Equal(6, result);
}
```

**解決**：Assert.Equal(expected, actual) の順序確認

**原因 3: Async 処理の待ち忘れ**

```csharp
// ❌ await 漏れ
var result = _sut.ProcessOrderAsync(1);  // Task を await せず
Assert.True(result.IsCompleted);  // 即座に失敗

// ✅ await を付ける
var result = await _sut.ProcessOrderAsync(1);
Assert.True(result.Success);
```

**解決**：async メソッドには必ず `await` をつける

---

## NullReferenceException

### 症状
```
System.NullReferenceException: Object reference not set 
to an instance of an object.
```

### 原因と対策

**原因 1: 存在しないプロパティアクセス**

```csharp
// ❌ Null の可能性
var customer = await _repository.GetByIdAsync(999);
var email = customer.Email;  // ← NullReferenceException

// ✅ Null チェック
var customer = await _repository.GetByIdAsync(999);
if (customer == null)
{
    throw new InvalidOperationException("Customer not found");
}
var email = customer.Email;
```

**解決**：
- ?? (null coalescing) operator 使用
- null check を明示的に
- C# 8+ nullable annotations を活用

```csharp
// Modern C#
public string? GetCustomerEmail(int customerId)
{
    var customer = await _repository.GetByIdAsync(customerId);
    return customer?.Email;  // null-safe operator で安全
}
```

**原因 2: コレクション要素へのアクセス**

```csharp
// ❌ リスト空チェックなし
var orders = await _repository.GetOrders();
var firstOrder = orders.First();  // InvalidOperationException

// ✅ チェック
var orders = await _repository.GetOrders();
var firstOrder = orders.FirstOrDefault();  // null 返す
if (firstOrder != null)
{
    // 処理
}
```

**解決**：`FirstOrDefault()` + null check

---

## N+1 クエリ問題

### 症状
```
データベースクエリが大量発行
- 初回: 1 クエリ（顧客取得）
- ループで N クエリ（各顧客ごとの注文取得）
= 合計 N+1 クエリ
```

### 原因と対策

**原因: Include 漏れ**

```csharp
// ❌ N+1 問題
var customers = await _context.Customers.ToListAsync();  // 1 query
foreach (var customer in customers)
{
    var orders = await _context.Orders
        .Where(o => o.CustomerId == customer.Id)
        .ToListAsync();  // N queries
}

// ✅ Include で 1 query だけ
var customersWithOrders = await _context.Customers
    .Include(c => c.Orders)  // Eager Load
    .ToListAsync();
```

**解決チェックリスト**：
- [ ] Entity 関連データを取得する？ → Include を付ける
- [ ] Select で投影している？ → Projection で必要カラムのみ
- [ ] Loop 内で query ？ → 外に出す or Include

### SQL Plan で確認

```sql
-- SQL Server Management Studio で実行計画確認
-- 1 query のはず（複数の JOIN が見えるのは OK）
SELECT * FROM Customers c
LEFT JOIN Orders o ON c.Id = o.CustomerId
```

---

## パフォーマンス低下

### 症状
```
API が遅い（1秒以上）
Response time が P95: 2000ms 超えてる
```

### 原因と対策

**原因 1: クエリが遅い**

```sql
-- Slow query log から確認
SELECT * FROM Orders
WHERE CreatedDate > @Date  -- インデックスなし

-- 対策: インデックス作成
CREATE INDEX IX_Orders_CreatedDate 
ON Orders(CreatedDate);
```

**原因 2: プログラム的に遅い**

```csharp
// ❌ メモリ load 多い
var allOrders = await _context.Orders.ToListAsync();  // 100万件
var filtered = allOrders.Where(o => o.Amount > 1000).ToList();

// ✅ DB で filter
var filtered = await _context.Orders
    .Where(o => o.Amount > 1000)
    .ToListAsync();
```

**原因 3: キャッシュなし**

```csharp
// ❌ 毎回 DB query
public async Task<List<ShippingMethod>> GetMethodsAsync()
{
    return await _context.ShippingMethods.ToListAsync();
}

// ✅ キャッシュ 5 分
public async Task<List<ShippingMethod>> GetMethodsAsync()
{
    const string key = "shipping_methods";
    if (_cache.TryGetValue(key, out var cached))
        return cached;
    
    var methods = await _context.ShippingMethods.ToListAsync();
    _cache.Set(key, methods, TimeSpan.FromMinutes(5));
    return methods;
}
```

**デバッグ方法**：
1. Application Insights で遅い endpoint を特定
2. SQL Profiler で遅いクエリを特定
3. SQL Plan で重い操作を確認
4. Index / キャッシュで改善

---

## セキュリティ警告

### 症状
```
SonarQube / Copilot: "SQL Injection 脆弱性"
"入力未検証"
```

### 原因と対策

**原因 1: SQL 文字列連結**

```csharp
// ❌ SQL Injection 脆弱性
var sql = $"SELECT * FROM Orders WHERE CustomerId = {customerId}";
var result = _context.Orders.FromSqlRaw(sql).ToList();

// ✅ Parameterized Query
var result = _context.Orders
    .FromSqlInterpolated($"SELECT * FROM Orders WHERE CustomerId = {customerId}")
    .ToList();

// または LINQ
var result = _context.Orders
    .Where(o => o.CustomerId == customerId)
    .ToList();
```

**原因 2: 入力未検証**

```csharp
// ❌ XSS 脆弱性
public IActionResult Search(string query)
{
    return Ok(_context.Products
        .Where(p => p.Name.Contains(query))  // ← User input そのまま
        .ToList());
}

// ✅ 入力検証 + サニタイズ
public IActionResult Search(string query)
{
    if (string.IsNullOrWhiteSpace(query) || query.Length > 100)
        return BadRequest("Invalid query");
    
    var results = _context.Products
        .Where(p => EF.Functions.Like(p.Name, $"%{query}%"))
        .ToList();
    return Ok(results);
}
```

**解決チェックリスト**：
- [ ] すべての DB query は parameterized か？
- [ ] User input は検証・sanitation しているか？
- [ ] API endpoint は [Authorize] か？
- [ ] Sensitive data はログに出ていないか？

---

## Unit Test カバレッジ不足

### 症状
```
Code Coverage: 45% (目標 80%)
どのメソッドをテストればいい？
```

### 原因と対策

**原因 1: テストケース不十分**

```csharp
// ❌ Happy path のみ
[Fact]
public void ProcessOrder_ShouldSucceed()
{
    var order = new Order { Amount = 100 };
    var result = _sut.ProcessOrder(order);
    Assert.True(result.Success);
}

// ✅ 正常系 + 異常系 + 境界値
[Theory]
[InlineData(100, true)]     // 正常
[InlineData(0, false)]      // 境界値
[InlineData(-1, false)]     // 異常
public void ProcessOrder_WithAmount_ReturnsExpected(
    decimal amount, bool shouldSucceed)
{
    var order = new Order { Amount = amount };
    var result = _sut.ProcessOrder(order);
    Assert.Equal(shouldSucceed, result.Success);
}
```

**対策**：
- 正常系 1 テスト + 異常系 2-3 テスト の最低 3 ケース
- Copilot に「このメソッドの comprehensive unit tests を生成」と依頼

**原因 2: Internal/Private メソッドのテスト**

```csharp
// ❌ Private メソッドはテストできない
private bool ValidateEmail(string email) { ... }

// ✅ Public interface で検証、Private は使用側でテスト
public OrderResult CreateOrder(CreateOrderRequest request)
{
    if (!ValidateEmail(request.Email))
        return OrderResult.Failed(...);
    // ...
}

// テスト
[Fact]
public void CreateOrder_InvalidEmail_ReturnsFailed()
{
    var result = _sut.CreateOrder(new CreateOrderRequest { Email = "invalid" });
    Assert.False(result.Success);  // ValidateEmail が間接的にテストされる
}
```

**解決**：
- カバレッジ計測（Sonarqube / Codecov）
- Copilot でテスト自動生成
- Business logic 層に 90% coverage チェック

---

## Code Review 指摘多発

### 症状
```
PR を出す
→ 10 個の指摘を受ける
→ 毎回同じ指摘ばかり
```

### 原因と対策

**原因: チーム標準の理解不足**

```csharp
// ❌ 指摘 1: 命名規則
private OrderRepo repo;      // ← _orderRepository にすべき
private decimal total_price; // ← totalPrice にすべき

// ❌ 指摘 2: エラーハンドリングなし
public void ProcessOrder(Order order)
{
    _repository.Save(order);  // ← Exception handling なし
}

// ❌ 指摘 3: テストなし
public async Task ComplexBusiness Logic() { ... }
// テストコードなし
```

**解決**：
1. `.github/copilot-instructions.md` を読む（作業前に）
2. Copilot に「チーム標準に従ったコード生成」を頼む
3. Template からコピペする

---

## Async 関連エラー

### 症状
```
deadlock が発生
別スレッドから呼び出すと hang する
```

### 原因と対策

**原因 1: ConfigureAwait(false) 漏れ**

```csharp
// ❌ UI context に依存（Web API では不要）
var result = await someAsyncMethod();

// ✅ Context-free
var result = await someAsyncMethod().ConfigureAwait(false);
```

**原因 2: 同期型で Async 呼び出し**

```csharp
// ❌ Deadlock
public void ProcessOrder(int orderId)
{
    var order = _service.GetOrderAsync(orderId).Result;  // ← Wait !
}

// ✅ Async 伝播
public async Task ProcessOrderAsync(int orderId)
{
    var order = await _service.GetOrderAsync(orderId);
}
```

**解決チェックリスト**：
- [ ] Async chain 全体で await を使用
- [ ] `.Result` / `.Wait()` を使わない
- [ ] `ConfigureAwait(false)` を Library code に追加

---

## 📞 さらに困った場合

1. **Slack**: #copilot-questions に質問
2. **Wiki**: Team wiki で past solutions を検索
3. **Copilot Chat**: 「エラーメッセージ [ここに貼り付け]、原因と対策」
4. **Stack Overflow**: Tag `csharp` と `entity-framework` で検索

---

## ✅ トラブルシューティング習慣

- [ ] Google / Stack Overflow で「エラーメッセージ + 言語」で検索
- [ ] 根本原因を Copilot Chat で確認
- [ ] テスト駆動で修正し、回帰防止
- [ ] Team wiki に「今日の学び」を記録
