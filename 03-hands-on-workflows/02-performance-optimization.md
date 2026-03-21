### 遅いコードを見つけて、効率的に改善する

パフォーマンス問題の診断と改善は、中級者必須スキル。Copilot を使った効率的なプロセスを学びます。

---

### 🎯 シーン 1: N+1 クエリ問題の解決

#### ❌ 非効率なコード

```csharp
// ❌ N+1 クエリ問題：顧客取得（1回） + 各顧客の注文取得（N回）
public class OrderService
{
    private readonly AppDbContext _context;
    
    public async Task<List<CustomerOrderSummary>> GetCustomerOrdersSummaryAsync()
    {
        var customers = await _context.Customers.ToListAsync();
        
        var result = new List<CustomerOrderSummary>();
        foreach (var customer in customers)
        {
            var orders = await _context.Orders
                .Where(o => o.CustomerId == customer.Id)
                .ToListAsync();
            
            result.Add(new CustomerOrderSummary
            {
                CustomerId = customer.Id,
                CustomerName = customer.Name,
                OrderCount = orders.Count,
                TotalAmount = orders.Sum(o => o.Amount)
            });
        }
        
        return result;
    }
}

// 問題：顧客が 1000 人いたら、クエリ 1001 回！
```

#### ✅ 最適化版：Include + Select

```csharp
public class OrderService
{
    private readonly AppDbContext _context;
    
    /// <summary>
    /// 顧客別の注文サマリーを取得（クエリ 1 回）
    /// </summary>
    public async Task<List<CustomerOrderSummary>> GetCustomerOrdersSummaryAsync()
    {
        return await _context.Customers
            .Include(c => c.Orders) // 関連データを一度に取得
            .AsNoTracking()          // 読み取り専用（パフォーマンス向上）
            .Select(c => new CustomerOrderSummary
            {
                CustomerId = c.Id,
                CustomerName = c.Name,
                OrderCount = c.Orders.Count,
                TotalAmount = c.Orders.Sum(o => o.Amount)
            })
            .ToListAsync();
    }
}
```

#### 効果
- **クエリ数**: 1001回 → 1回 (1000倍高速化!)
- **メモリ使用**: Include で自動管理
- **実装の簡潔性**: LINQ で宣言的に表現

---

### 🎯 シーン 2: 複数 API の並列呼び出し

#### ❌ 非効率版：順次処理

```csharp
public class DataService
{
    private readonly HttpClient _httpClient;
    
    // ❌ 各 API を順番に呼び出し（遅い）
    public async Task<AggregatedData> GetAggregatedDataAsync(int id)
    {
        var product = await _httpClient.GetAsync($"/api/products/{id}");
        var reviews = await _httpClient.GetAsync($"/api/reviews/{id}");
        var stock = await _httpClient.GetAsync($"/api/stock/{id}");
        var pricing = await _httpClient.GetAsync($"/api/pricing/{id}");
        
        // 全部完了するのに各 API の合計時間がかかる
        // 例：100ms + 100ms + 100ms + 100ms = 400ms
    }
}
```

#### ✅ 最適化版：Task.WhenAll

```csharp
public class DataService
{
    private readonly HttpClient _httpClient;
    
    /// <summary>
    /// 複数 API を並列で呼び出し
    /// </summary>
    public async Task<AggregatedData> GetAggregatedDataAsync(int id)
    {
        // すべての API を同時実行
        var productTask = FetchProductAsync(id);
        var reviewsTask = FetchReviewsAsync(id);
        var stockTask = FetchStockAsync(id);
        var pricingTask = FetchPricingAsync(id);
        
        // すべての結果を待つ
        await Task.WhenAll(productTask, reviewsTask, stockTask, pricingTask);
        
        return new AggregatedData
        {
            Product = await productTask,
            Reviews = await reviewsTask,
            Stock = await stockTask,
            Pricing = await pricingTask
        };
        
        // 実行時間：max(100, 100, 100, 100) = 100ms (4倍高速化!)
    }
    
    private async Task<ProductData> FetchProductAsync(int id)
    {
        var response = await _httpClient.GetAsync($"/api/products/{id}");
        return await response.Content.ReadAsAsync<ProductData>();
    }
    
    // ... 他の Fetch メソッド
}
```

---

### 🎯 シーン 3: LINQ クエリの最適化

#### ❌ 非効率版：メモリ上でフィルタリング

```csharp
// ❌ すべてのデータをメモリに読み込んでから処理
var expensiveOrders = _context.Orders
    .ToList()  // すべてメモリに！
    .Where(o => o.Amount > 1000)
    .Where(o => o.OrderDate > DateTime.Now.AddMonths(-1))
    .OrderByDescending(o => o.Amount)
    .Take(10)
    .ToList();

// 問題：100万件のデータをメモリに読み込むと メモリ不足！
```

#### ✅ 最適化版：データベースで処理

```csharp
// ✅ データベースで処理 → 結果だけメモリに
var expensiveOrders = _context.Orders
    .Where(o => o.Amount > 1000)
    .Where(o => o.OrderDate > DateTime.Now.AddMonths(-1))
    .OrderByDescending(o => o.Amount)
    .Take(10)
    .ToListAsync();

// メリット：
// 1. SQL が WHERE 句で事前フィルタリング
// 2. 結果（最大10件）だけメモリに
// 3. 実行速度も高速
```

---

### 🎯 シーン 4: キャッシングの活用

#### 状況
「同じクエリが何度も実行される」

```csharp
public class ProductService
{
    private readonly AppDbContext _context;
    private readonly IMemoryCache _cache;
    
    /// <summary>
    /// 商品を ID で取得（キャッシュ活用）
    /// </summary>
    public async Task<Product> GetProductAsync(int productId)
    {
        const string cacheKey = $"product_{productId}";
        
        // キャッシュから取得
        if (_cache.TryGetValue(cacheKey, out Product cachedProduct))
        {
            return cachedProduct;
        }
        
        // キャッシュにない場合はデータベースから
        var product = await _context.Products.FindAsync(productId);
        
        if (product != null)
        {
            // 5分間キャッシュ保持
            _cache.Set(cacheKey, product, TimeSpan.FromMinutes(5));
        }
        
        return product;
    }
}
```

**効果**:
- 1回目のリクエスト: DB アクセス（遅い）
- 2-5分間のリクエスト: キャッシュから即座に返答（高速！）

---

### 🔧 Copilot を使った最適化フロー

#### ステップ 1: 遅いコードを選択

```csharp
var expensiveOrders = _context.Orders.ToList()...
```

#### ステップ 2: Chat で質問

```
「このコードが遅い理由は？最適化方法を教えてください」
```

#### ステップ 3: Copilot が説明

```
「.ToList() ですべてをメモリに読み込んでいます。
WHERE を LINQ クエリで実行して、結果のみ取得してください。」
```

#### ステップ 4: Inline Edits で修正

単純な最適化（ToList 削除など）は「該当部分選択 → パフォーマンス最適化」で自動修正。

---

### ✅ 習熟度チェック

- [ ] N+1 クエリ問題が理解できた
- [ ] Include を使ったデータベースアクセス最適化ができた
- [ ] 複数 API の並列化ができた
- [ ] LINQ での事前フィルタリングができた
- [ ] キャッシング実装ができた

すべてチェック完了 → [03-testability.md](03-testability.md) へ

---

### 🔗 参考

- [Entity Framework Core Performance](https://learn.microsoft.com/en-us/ef/core/performance/)
- [LINQ to Objects Performance](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic)
- [Async/Await Best Practices](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
