# 03 初心者向けシナリオ

## 初心者が最初に取り組むべき 3 つの実装シナリオ

実際のコード例を通じて、Copilot の活用法を学びます。各シナリオは簡潔な実装から段階的に複雑化します。

---

## シナリオ 1: ファイル操作の実装

### 状況
「テキストファイルを読み込んで、各行をリストとして返す機能を作りたい」

### ステップ 1: 意図をコメントで伝える

```csharp
// テキストファイルを読み込んで、各行をリストとして返す
public List<string> ReadFileLines(string filename)
{
    // ここから Copilot が提案を出す
}
```

### ステップ 2: Copilot の提案を確認

Copilot による提案例：

```csharp
public List<string> ReadFileLines(string filename)
{
    var lines = new List<string>();
    
    try
    {
        using (var reader = new StreamReader(filename))
        {
            string line;
            while ((line = reader.ReadLine()) != null)
            {
                lines.Add(line);
            }
        }
    }
    catch (FileNotFoundException)
    {
        Console.WriteLine($"ファイルが見つかりません: {filename}");
    }
    catch (IOException ex)
    {
        Console.WriteLine($"ファイル読み込みエラー: {ex.Message}");
    }
    
    return lines;
}
```

### ステップ 3: より モダンな書き方に改善

チャットで「より モダンな書き方を提案して」と聞くと：

```csharp
public async IAsyncEnumerable<string> ReadFileLinesAsync(string filename)
{
    using var reader = new StreamReader(filename);
    
    while (await reader.ReadLineAsync() is string line)
    {
        yield return line;
    }
}
```

---

## シナリオ 2: LINQ の使い方を学ぶ

### 状況
「コレクション操作を効率的に行いたい。フォアーチループではなく LINQ を使いたい」

### ステップ 1: 基本的な foreach 版

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
var evenNumbers = new List<int>();

foreach (var num in numbers)
{
    if (num % 2 == 0)
    {
        evenNumbers.Add(num);
    }
}
```

### ステップ 2: Copilot に LINQ 版を提案してもらう

「このコードを LINQ で書き直して」と指示 →

```csharp
var evenNumbersLinq = numbers.Where(n => n % 2 == 0).ToList();
```

### ステップ 3: より複雑な例に応用

```csharp
var students = new List<Student>
{
    new Student { Name = "田中", Score = 85, Grade = "A" },
    new Student { Name = "鈴木", Score = 92, Grade = "A" },
    new Student { Name = "佐藤", Score = 78, Grade = "B" }
};

// グループ化と集計
var gradeStats = students
    .GroupBy(s => s.Grade)
    .Select(g => new
    {
        Grade = g.Key,
        AverageScore = g.Average(s => s.Score),
        Count = g.Count(),
        TopStudent = g.OrderByDescending(s => s.Score).First().Name
    })
    .ToList();
```

### 学ぶべきポイント
- `Where()` — フィルタリング
- `Select()` — データ変換
- `GroupBy()` — グループ化
- `Average()`, `Count()` — 集計

---

## シナリオ 3: 非同期プログラミングの導入

### 状況
「Web API からデータを取得したい。非同期処理を正しく書きたい」

### ❌ 間違った非同期処理

```csharp
// 危険：using と例外処理がない
public async Task<string> GetDataAsync()
{
    using var client = new HttpClient();
    var result = await client.GetStringAsync("https://api.example.com/data");
    return result;
}
```

### ✅ ベストプラクティス版

```csharp
public class DataService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<DataService> _logger;
    
    public DataService(HttpClient httpClient, ILogger<DataService> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }
    
    /// <summary>
    /// 非同期でデータを取得する
    /// </summary>
    public async Task<Result<T>> GetDataAsync<T>(
        string endpoint, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            // キャンセレーショントークンを渡す
            var response = await _httpClient.GetAsync(endpoint, cancellationToken);
            
            // 成功レスポンスを確認
            response.EnsureSuccessStatusCode();
            
            // コンテンツを非同期で読み取り
            var json = await response.Content.ReadAsStringAsync(cancellationToken);
            
            // デシリアライズ
            var data = JsonSerializer.Deserialize<T>(json, new JsonSerializerOptions
            {
                PropertyNameCaseInsensitive = true
            });
            
            return Result<T>.Success(data);
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "HTTPリクエストエラー: {Endpoint}", endpoint);
            return Result<T>.Failure("ネットワークエラーが発生しました");
        }
        catch (JsonException ex)
        {
            _logger.LogError(ex, "JSON解析エラー: {Endpoint}", endpoint);
            return Result<T>.Failure("データ形式が不正です");
        }
        catch (OperationCanceledException)
        {
            _logger.LogWarning("リクエストがキャンセルされました: {Endpoint}", endpoint);
            return Result<T>.Failure("リクエストがキャンセルされました");
        }
    }
}

// 結果をラップするResult型
public class Result<T>
{
    public bool IsSuccess { get; set; }
    public T Data { get; set; }
    public string Error { get; set; }
    
    public static Result<T> Success(T data) 
        => new() { IsSuccess = true, Data = data };
    
    public static Result<T> Failure(string error) 
        => new() { IsSuccess = false, Error = error };
}
```

### 学ぶべきポイント
- ✅ **CancellationToken を渡す** — キャンセル対応
- ✅ **EnsureSuccessStatusCode()** — HTTP エラーチェック
- ✅ **ライングリ例外処理** — HttpRequestException, JsonException など分ける
- ✅ **ログ出力** — エラー診断用に情報を記録
- ✅ **Result パターン** — 例外を投げず結果を返す

---

## 🎯 実装の流れ

各シナリオで推奨される進め方：

1. **「状況」を理解** — どんな問題を解く？
2. **シンプル版を見る** — 最小限のコード例
3. **Copilot で改善** — より良い書き方を試す
4. **学ぶべきポイント** — 実装パターンを理解

---

## 💡 各シナリオから学べることの活用先

### シナリオ 1（ファイル操作）から学べること
- `using` ステートメント — リソース管理
- 例外処理 — エラーハンドリング
- 非同期ファイル操作 — `async/await`

**活用先**: ログ読み込み、設定ファイル解析、BOM ファイル処理など

### シナリオ 2（LINQ）から学べること
- メソッドチェーン — 流暢なコード
- ラムダ式 — 匿名関数
- 遅延評価 — 効率的なメモリ使用

**活用先**: リストフィルタリング、データ変換、ソート、集計処理など

### シナリオ 3（非同期処理）から学べること
- async/await パターン — 非ブロッキング処理
- CancellationToken — キャンセル対応
- 例外処理の詳細 — エラー分類

**活用先**: Web API 呼び出し、データベース操作、ファイル I/O など

---

## ✅ 習熟度チェック

このセクション完了時、以下ができていますか？

- [ ] 3 つのシナリオの実装例が理解できた
- [ ] シナリオ 1：ファイル操作を自分で実装できた
- [ ] シナリオ 2：LINQ で操作を実装できた  
- [ ] シナリオ 3：非同期処理の重要ポイントが理解できた
- [ ] 「Copilot での改善提案」のいずれかを試した

すべてチェック完了 → [04-role-mastery/01-beginner-developer.md](../04-role-mastery/01-beginner-developer.md) へ

---

## 🔗 参考資料

- [LINQ Basic Concepts](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/linq/)
- [Asynchronous programming patterns](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [Exception handling best practices](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)
