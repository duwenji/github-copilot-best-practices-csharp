### 選択したコードを直接修正する

Inline Edits は「選択したコード」に対して修正指示を出す機能。範囲を限定した改善が可能です。

---

### 💡 テクニック 1: コードの最適化

#### シーン：このコード遅いから最適化したい

```csharp
// ❌ 選択したコード（複雑度: O(n²)）
public List<int> FindDuplicates(int[] items)
{
    var duplicates = new List<int>();
    for (int i = 0; i < items.Length; i++)
    {
        for (int j = i + 1; j < items.Length; j++)
        {
            if (items[i] == items[j] && !duplicates.Contains(items[i]))
            {
                duplicates.Add(items[i]);
            }
        }
    }
    return duplicates;
}

// 指示: 「パフォーマンスを最適化して」
```

**Copilot による修正案:**

```csharp
/// <summary>
/// ハッシュセットを使用して重複を効率的に検出（O(n)）
/// </summary>
public List<int> FindDuplicatesOptimized(int[] items)
{
    var seen = new HashSet<int>();
    var duplicates = new HashSet<int>();
    
    foreach (var item in items)
    {
        if (seen.Contains(item))
        {
            duplicates.Add(item);
        }
        else
        {
            seen.Add(item);
        }
    }
    
    return duplicates.ToList();
}

// パフォーマンス比較
public void ComparePerformance()
{
    var testData = Enumerable.Range(0, 10000)
        .Concat(Enumerable.Range(0, 5000))
        .ToArray();
    
    var sw = new System.Diagnostics.Stopwatch();
    
    sw.Start();
    var result1 = FindDuplicates(testData);
    sw.Stop();
    Console.WriteLine($"最適化前: {sw.ElapsedMilliseconds}ms");
    
    sw.Restart();
    var result2 = FindDuplicatesOptimized(testData);
    sw.Stop();
    Console.WriteLine($"最適化後: {sw.ElapsedMilliseconds}ms");
    // 結果: 最適化前 1500ms 以上 → 最適化後 10-20ms
}
```

#### ポイント
- 「パフォーマンスを最適化して」 + 具体的な改善観点（O(n²) → O(n) など）
- 修正後のメソッド名を変更してくれることが多い（わかりやすく）

---

### 💡 テクニック 2: エラーハンドリングの追加

#### シーン：このコード例外処理がない

```csharp
// ❌ 選択したコード（エラーハンドリングなし）
public Config ReadConfig(string filename)
{
    var json = File.ReadAllText(filename);
    return JsonSerializer.Deserialize<Config>(json);
}

// 指示: 「エラーハンドリングを追加して」
```

**Copilot による拡張:**

```csharp
using System.Text.Json;
using System.IO;

public class ConfigService
{
    private readonly ILogger<ConfigService> _logger;
    
    public ConfigService(ILogger<ConfigService> logger)
    {
        _logger = logger;
    }
    
    /// <summary>
    /// 設定ファイルを安全に読み込む
    /// </summary>
    public async Task<Config> ReadConfigAsync(string filename)
    {
        // ファイル存在チェック
        if (!File.Exists(filename))
        {
            _logger.LogError("設定ファイルが見つかりません: {Filename}", filename);
            throw new FileNotFoundException($"設定ファイルが見つかりません: {filename}");
        }
        
        try
        {
            var json = await File.ReadAllTextAsync(filename);
            var config = JsonSerializer.Deserialize<Config>(json);
            
            if (config == null)
                throw new InvalidOperationException("設定ファイルが空です");
            
            ValidateConfig(config);
            _logger.LogInformation("設定ファイルを読み込みました: {Filename}", filename);
            return config;
        }
        catch (JsonException ex)
        {
            _logger.LogError(ex, "JSON形式が不正です: {Filename}", filename);
            throw;
        }
        catch (UnauthorizedAccessException ex)
        {
            _logger.LogError(ex, "ファイルへのアクセス権限がありません: {Filename}", filename);
            throw;
        }
    }
    
    private void ValidateConfig(Config config)
    {
        var required = new[] { "Database", "ApiKey", "Port" };
        var missing = required.Where(r =>
            typeof(Config).GetProperty(r)?.GetValue(config) == null
        ).ToList();
        
        if (missing.Any())
            throw new InvalidOperationException($"必須設定が不足しています: {string.Join(", ", missing)}");
    }
}

public class Config
{
    public string Database { get; set; }
    public string ApiKey { get; set; }
    public int Port { get; set; }
}
```

#### ポイント
- 「エラーハンドリングを追加して」
- null チェック、例外種別の分類、ログ出力まで対応

---

### 💡 テクニック 3: 複雑な変換

#### シーン：XMLをJSONに変換したい

```csharp
// 選択したコード
// 「XML パースして JSON に変換してください」

XDocument doc = XDocument.Load("data.xml");
// XML を JSON フォーマットに変換
```

**Copilot による実装:**

```csharp
public class XmlToJsonConverter
{
    public static string ConvertXmlToJson(string xmlFilePath)
    {
        try
        {
            XDocument doc = XDocument.Load(xmlFilePath);
            var jsonObject = ConvertElement(doc.Root);
            return JsonSerializer.Serialize(jsonObject, new JsonSerializerOptions { WriteIndented = true });
        }
        catch (XmlException ex)
        {
            throw new InvalidOperationException("XML形式が不正です", ex);
        }
    }
    
    private static Dictionary<string, object> ConvertElement(XElement element)
    {
        var result = new Dictionary<string, object>();
        
        // 属性を処理
        foreach (var attr in element.Attributes())
        {
            result[$"@{attr.Name}"] = attr.Value;
        }
        
        // 子要素を処理
        var children = element.Elements().GroupBy(e => e.Name.LocalName);
        foreach (var group in children)
        {
            var childList = group.Select(ConvertElement).ToList();
            
            if (childList.Count == 1)
                result[group.Key] = childList[0];
            else
                result[group.Key] = childList;
        }
        
        // テキスト値を処理
        if (element.HasElements)
            return result;
        else
            return new Dictionary<string, object> { { element.Name.LocalName, element.Value } };
    }
}
```

#### ポイント
- 複雑な変換は「指示」として明確に
- 再帰処理など下層的なロジックも生成

---

### 🎯 有効な指示パターン

| 指示内容 | 例 |
|---------|---|
| **最適化** | 「パフォーマンスを最適化して」「LINQ で書き直して」 |
| **エラー処理** | 「エラーハンドリングを追加して」「nullチェックを追加して」 |
| **フォーマット** | 「非同期に変換して」「Async/Await パターンに」 |
| **セキュリティ** | 「入力検証を追加して」「SQL インジェクション対策を」 |
| **テスト** | 「テスト可能な設計に変更して」 |

---

### ⚠️ 使用上の注意

#### ❌ 避けるべき使用法
- **大量行のコード選択** ← Chat を使う
- **複数メソッド同時修正** ← 1つずつ修正する
- **あいまいな指示** ← 具体的に

#### ✅ 推奨
- **メソッド単位で修正**
- **1つの観点で修正** （複数観点は複数回実行）
- **修正前後を比較** してから確定

---

### ✅ 習熟度チェック

- [ ] コードを選択して修正指示ができた
- [ ] 複数行のコード修正ができた
- [ ] 複雑な変換を依頼できた
- [ ] 修正後のコードを確認してから確定

すべてチェック完了 → [04-agent-mode.md](04-agent-mode.md) へ

---

### 🔗 参考

- [02-copilot-chat.md](02-copilot-chat.md) — Chat との使い分け
- [Entity Framework Core Best Practices](https://learn.microsoft.com/en-us/ef/core/)
