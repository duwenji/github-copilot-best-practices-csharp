### Copilot で テスト自動化と品質保証を高度化する

QA エンジニアにとって重要な「テスト自動化」「不具合検出」「品質メトリクス」。Copilot をテストコード生成と品質戦略に活用します。

---

### 📊 QA エンジニア診断

- テストシナリオを自分で設計できる → **はい**
- 自動化テストコードを書ける（xUnit, Selenium など） → **はい/部分的**
- バグのバグナンバーを根本原因で分類できる → **はい**
- 品質メトリクス（テストカバレッジ、バグ検出率）を追跡できる → **はい**

↑ 3 つ以上「はい」なら、あなたは QA エンジニアです。

---

### 🎯 QA エンジニアが Copilot を活用する 3 つのシーン

#### シーン 1: 単体テストコード生成の自動化

**従来の苦労**：
```
「このビジネスロジック、テストケース 20 個必要」
→ テストコード手書き（2 日）→ 保守コスト（大）
```

**Copilot で解決**：

```
Step 1: 対象コードを Chat に貼り付け

public decimal CalculateDiscount(int quantity, bool isVip, 
    DateTime purchaseDate)
{
    var baseDiscount = quantity switch
    {
        >= 100 => 0.20m,
        >= 50 => 0.15m,
        >= 10 => 0.10m,
        _ => 0.05m
    };
    
    if (isVip)
        baseDiscount += 0.05m;
    
    // 2月のキャンペーン
    if (purchaseDate.Month == 2)
        baseDiscount += 0.10m;
    
    return Math.Min(baseDiscount, 0.50m); // 上限 50%
}

Step 2: Chat でリクエスト

「このメソッドの網羅的なユニットテストを生成してください。
正常系・異常系・境界値をカバーしてください。」

Step 3: Copilot がテストコード生成

[TestClass]
public class DiscountCalculationTests
{
    [TestMethod]
    [DataRow(100, false, 2024, 2, 1, 0.30)] // 100個 + 2月
    [DataRow(50, true, 2024, 3, 1, 0.20)]   // 50個 + VIP
    [DataRow(10, false, 2024, 1, 1, 0.10)]  // 境界値 10個
    [DataRow(5, false, 2024, 1, 1, 0.05)]   // 5個
    public void CalculateDiscount_ReturnsCorrectValue(
        int quantity, bool isVip, int year, 
        int month, int day, decimal expected)
    {
        // Arrange
        var sut = new DiscountCalculator();
        var purchaseDate = new DateTime(year, month, day);
        
        // Act
        var result = sut.CalculateDiscount(quantity, isVip, purchaseDate);
        
        // Assert
        Assert.AreEqual(expected, result);
    }
    
    [TestMethod]
    public void CalculateDiscount_CapsAt50Percent()
    {
        var sut = new DiscountCalculator();
        var result = sut.CalculateDiscount(
            1000, true, new DateTime(2024, 2, 1));
        Assert.IsTrue(result <= 0.50m);
    }
}

Step 4: レビュー & 調整

実際にテスト実行してみる
→ カバレッジ確認
→ 必要に応じて追加シナリオ追加
```

**効果**：
- テストコード作成時間: 2 日 → 1 時間（自動生成 + レビュー）
- テストケース網羅性: 80% → 95%

---

#### シーン 2: 統合テスト・E2E テストの効率化

**従来**：手作業テスト → 時間がかかり、漏れやすい

**Copilot で高度化**：

```csharp
// シナリオ：「新規注文 → 支払い → 確認メール送信」の統合テスト

public class OrderE2ETests : IAsyncLifetime
{
    [Fact]
    public async Task CompleteOrderFlow_ShouldSucceed()
    {
        // Arrange
        var client = _factory.CreateClient();
        var createOrderRequest = new CreateOrderRequest
        {
            CustomerEmail = "test@example.com",
            Items = new[]
            {
                new OrderItemRequest { ProductId = 1, Quantity = 2 }
            },
            PaymentMethod = "CreditCard"
        };
        
        // Act 1: 注文作成
        var response1 = await client.PostAsJsonAsync(
            "/api/orders", createOrderRequest);
        Assert.Equal(HttpStatusCode.Created, response1.StatusCode);
        
        var orderJson = await response1.Content.ReadAsStringAsync();
        var orderId = JsonSerializer.Deserialize<Order>(orderJson).Id;
        
        // Act 2: 支払い処理
        var paymentRequest = new ProcessPaymentRequest
        {
            OrderId = orderId,
            CardNumber = "4111111111111111"
        };
        
        var response2 = await client.PostAsJsonAsync(
            $"/api/orders/{orderId}/payment", paymentRequest);
        Assert.Equal(HttpStatusCode.OK, response2.StatusCode);
        
        // Act 3: 確認ステータス
        var response3 = await client.GetAsync(
            $"/api/orders/{orderId}");
        var finalOrder = await response3.Content
            .ReadAsAsync<Order>();
        
        // Assert
        Assert.Equal(OrderStatus.Confirmed, finalOrder.Status);
        // メール送信確認（モック）
        _mockEmailService.Verify(
            s => s.SendAsync(It.Is<Email>(
                e => e.To == "test@example.com")),
            Times.Once);
    }
}
```

**Copilot の活用**：
- Chat で「このビジネスフロー、E2E テストコード生成」
- 複数ステップのテストシナリオを自動生成
- HTTP リクエスト/レスポンス検証コードを生成

---

#### シーン 3: 品質メトリクスと不具合分析

**よくある課題**：
```
「今月のバグ率は？」
→ 「えっと管理表を見て...」（情報が散在）
```

**Copilot で構造化**：

```sql
-- Copilot に「バグ市場別の傾向チェッククエリ」を生成してもらう

SELECT 
    DATETRUNC('month', CreatedDate) AS Month,
    Severity,
    COUNT(*) AS BugCount,
    COUNT(CASE WHEN FixedDate IS NOT NULL THEN 1 END) AS FixedCount,
    AVG(DATEDIFF(day, CreatedDate, ISNULL(FixedDate, GETDATE()))) 
        AS AvgDaysToFix
FROM Bugs
WHERE ProjectId = @ProjectId
GROUP BY DATETRUNC('month', CreatedDate), Severity
ORDER BY Month DESC;
```

**分析結果**：
```
Month      | Severity | BugCount | FixedCount | AvgDaysToFix
2024-02    | High     | 5        | 4          | 2.5 days
2024-02    | Medium   | 15       | 12         | 5.2 days
2024-02    | Low      | 8        | 6          | 12.1 days

→ 「今月 High バグが増加傾向」「Medium バグの修正が遅い」を発見
→ チーム会で対策検討
```

---

### 🔧 テスト自動化フレームワークの設計

#### Unit Test (xUnit)

テストする対象: メソッドの logic

```csharp
[Theory]
[InlineData(1, 10, 10)]      // 正常系
[InlineData(0, 10, 0)]       // 境界値
[InlineData(-1, 10, -10)]    // 異常値
public void Multiply_ReturnsCorrectProduct(
    int a, int b, int expected)
{
    var calc = new Calculator();
    var result = calc.Multiply(a, b);
    Assert.Equal(expected, result);
}
```

#### Integration Test (xUnit + Moq)

テストする対象: 複数コンポーネント間の連携

```csharp
[Fact]
public async Task CreateOrder_ShouldCallPaymentService()
{
    // 複数コンポーネント間のやり取りをテスト
    var orderService = new OrderService(
        _mockRepository.Object, 
        _mockPaymentService.Object);
    
    await orderService.CreateOrderAsync(...);
    
    // PaymentService が正しく呼ばれたか確認
    _mockPaymentService.Verify(
        p => p.ProcessPaymentAsync(...), Times.Once);
}
```

#### E2E Test (WebApplicationFactory)

テストする対象: API 全体（認証 → 処理 → 応答）

```csharp
[Fact]
public async Task HealthCheck_ReturnsOk()
{
    var response = await _client.GetAsync("/health");
    response.EnsureSuccessStatusCode();
}
```

---

### 📋 QA エンジニアチェックリスト

#### テスト設計スキル
- [ ] テストシナリオを要件から自動導出できた
- [ ] 正常系・異常系・境界値を網羅的に列挙できた
- [ ] 複数テストケースを効率的にカバーできた（Data-Driven テスト）

#### テスト自動化スキル
- [ ] Unit Test (xUnit) で 80% 以上カバレッジ達成
- [ ] Integration Test で複数コンポーネント連携を検証
- [ ] E2E Test で full workflow を自動検証
- [ ] Copilot でテストコード生成できた

#### 不具合分析スキル
- [ ] バグを「機能別」「重大度別」「期間別」で分類できた
- [ ] 「High バグが多い機能」を特定できた
- [ ] 修正までの平均日数を計測できた
- [ ] 「この機能の issue が減った」を定量測定できた

#### 品質向上スキル
- [ ] テストカバレッジを改善した（50% → 80%）
- [ ] バグ検出率を向上させた（本番バグ減）
- [ ] テスト実行時間を短縮した（30分 → 5分）

#### Copilot 活用スキル
- [ ] テストコード自動生成で生産性向上
- [ ] 複雑なテストロジックを Chat で相談
- [ ] SQL クエリ（品質メトリクス）を Copilot で生成

---

### 🚀 高度な活用：Mutation Testing

**Mutation Testing とは**：
「テストの質を測る」テスト。テストが実装の変更を検出できるか検証。

```csharp
// 元のコード
public int Add(int a, int b)
{
    return a + b;
}

// Mutation 1（バグを意図的に挿入）
public int Add(int a, int b)
{
    return a - b;  // + を - に変更
}

→ テストが引っかかったか？ Yes → Good bug detection
→ テストが引っかからなかったか？ No → 不十分なテスト

Copilot に「Mutation Testing クエリ」を生成してもらう
→ テストの厳密さを測定
```

---

### 📞 QA よくある質問

**Q: Copilot が生成したテスト、信頼できる？**

A: 「そのまま使う」ではなく「レビューして使う」。
- Copilot: テストコード全体の 60-70% を生成
- あなた（QA）: ビジネス要件に合わせて調整

**Q: Unit Test と E2E Test どっち優先？**

A: 「ピラミッド型」推奨：
```
     /\           E2E (5%)        遅い, 脆い
    /  \
   /____\         Integration (15%)
  /      \
 /________\       Unit (80%)  速い, 安定
```

Unit で基盤を固め、E2E で full flow 検証。

---

### 🔗 参考リソース

- [xUnit.net Documentation](https://xunit.net/)
- [Moq Framework](https://github.com/moq/moq4)
- [FluentAssertions](https://fluentassertions.com/)
- [WebApplicationFactory for Testing](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests)
- [Mutation Testing](https://github.com/stryker-mutator/stryker-net)
