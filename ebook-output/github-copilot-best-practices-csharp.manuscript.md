# GitHub Copilot ベストプラクティス（C# 版） {#cover-00-cover}

**C# 開発で生産性と品質を高める実践ガイド**

GitHub Copilot を C# 開発に取り入れ、基礎操作から実装パターン、テスト、チーム導入までを
一冊で学べる実践ガイドです。

> 💡 ブラウザで https://duwenji.github.io/spa-quiz-app/ を開くと、関連トピックをクイズ形式で復習できます。

- 著者: 杜 文吉
- 対象: C# 開発者 / Tech Lead / QA
- テーマ: `Inline Chat` / `Edits` / `Agent Mode` / TDD

**この教材で学べること**
- Copilot の主要機能と使い分け
- C# 実装パターンと設計改善
- テスト駆動開発と品質保証
- チーム導入と標準化の進め方

# Foundations {#chapter-01-foundations}

## Basics {#section-01-foundations-01-basics}

#### Copilot 基本操作を習得する

このファイルでは、GitHub Copilot の IDE 内での操作方法と、基本的なコンテキスト設定を学びます。

---

#### ✅ コード補完（Inline Suggestions）の基本操作

##### コード補完とは
コードを書いている最中に、灰色のテキストで次のコードを提案する機能です。

##### 基本操作

| 操作 | キー | 説明 |
|------|------|------|
| **提案を受け入れ** | `Tab` または `Enter` | グレーテキストの提案を確定 |
| **提案を拒否** | `Esc` | グレーテキストをキャンセル |
| **次の提案を表示** | `Alt + ]` | 別の提案を見る |
| **前の提案を表示** | `Alt + [` | 前の提案に戻る |
| **提案を再実行** | カーソル移動または新規入力 | 新しい状況での提案 |

##### 提案が表示されないときは
1. インターネット接続を確認
2. GitHub Copilot 拡張機能が有効か確認 → VS Code 拡張パネルで確認
3. Copilot サブスクリプション有効期限を確認
4. しばらく待つ（処理中の可能性）

---

#### 💡 効果的なコンテキスト設定

##### ファイル名の重要性

Copilot は**ファイル名**から文脈を読み取ります。

```csharp
// 📄 UserValidator.cs というファイル名の場合
// コメントなしでも、このような補完が提案されやすい
public bool ValidateName(string name)
{
    // グアテマラナムba.Length geq 2 ...の提案が来る可能性が高い
}
```

##### プロジェクト構造の活用

ファイルをカテゴリ別に整理すると、Copilot はより正確な提案をします：
- `/Services/` 内のファイル → Service パターンの提案
- `/Repositories/` 内のファイル → Repository パターンの提案
- `/Models/` 内のファイル → DTO・Entity の提案

##### XMLコメント(Documentation)の活用

```csharp
/// <summary>
/// ユーザーリストから、最終ログインが30日以上前のユーザーを抽出
/// </summary>
/// <param name="users">ユーザーリスト</param>
/// <returns>非アクティブユーザーリスト</returns>
public List<User> MarkInactiveUsers(List<User> users)
{
    // XMLコメントがあることで、メソッド本体の提案の精度が上がる
}
```

---

#### 🎯 IDE 別設定確認

##### Visual Studio Code
1. **拡張機能タブを開く** （Ctrl+Shift+X）
2. **GitHub Copilot を検索** → インストール
3. GitHub アカウントでサインイン
4. 拡張設定：`settings.json` で必要に応じてカスタマイズ

##### Visual Studio (2022 以降)
1. **ツール → オプション → GitHub Copilot**
2. 設定を確認・有効化

---

#### 📖 推奨設定事例

```json
{
  "github.copilot.enable": {
    "*": true,
    "plaintext": false,
    "markdown": true
  },
  "github.copilot.advanced": {
    "debug.overrideEngine": "gpt-4",
    "debug.testOverrideProxyUrl": "",
    "debug.testOverridePort": ""
  }
}
```

---

#### ✅ 習熟度チェック

このセクション完了時、以下ができていますか？

- [ ] `Tab` キーで提案を受け入れられる
- [ ] `Esc` キーで提案をキャンセルできる
- [ ] `Alt + ]` で別の提案を見ることができる
- [ ] IDE の設定を確認できた
- [ ] XMLコメントを書くと提案が精度化することを理解した

すべてチェック完了 → [02-コメント駆動開発](#section-01-foundations-02-comment-driven-dev) へ

---

#### 📝 トラブルシューティング

**Q: グレーテキストが表示されない**  
A: 1) インターネット接続 2) 拡張機能有効性 3) サブスクリプション有効期限 を確認

**Q: 提案が常におかしい**  
A: よりルールれき具体的なコメントを書く（次セクションで詳しく）

**Q: キーボードショートカットが反応しない**  
A: IDE のキーバインディング設定を確認（とくに IME 入力との競合など）

## Comment Driven Dev {#section-01-foundations-02-comment-driven-dev}

#### 効果的なコメントで Copilot の精度を上げる

GitHub Copilot は**具体的なコメント**から、正確なコードを生成します。このセクションでは、コメント駆動開発のテクニックを学びます。

---

#### 🎯 基本原則：曖昧さを避ける

##### ❌ 悪い例 — 曖昧なコメント

```csharp
// Before: 曖昧なコメント
// ユーザーを処理する

public List<User> ProcessUsers(List<User> users)
{
    // Copilot の提案: 何をするのか不明確 → 汎用的な提案に
}
```

##### ✅ 良い例 — 具体的なコメント

```csharp
// After: 具体的なコメント（Copilot が正確に提案）
// ユーザーリストから、最終ログインが30日以上前のユーザーを抽出し、
// 非アクティブユーザーとしてマークする
public List<User> MarkInactiveUsers(List<User> users)
{
    var thirtyDaysAgo = DateTime.Now.AddDays(-30);
    
    foreach (var user in users)
    {
        if (user.LastLogin < thirtyDaysAgo)
        {
            user.Status = UserStatus.Inactive;
        }
    }
    
    return users;
}
```

---

#### 📝 効果的なコメント構造

##### パターン 1：目的 + 条件 + 結果

```csharp
// [テーマ]から、[条件]な[対象]を[処理内容]

// 例 1
// "注文リスト"から、"金額が100,000円以上の"注文を抽出し、
// VIP顧客として分類する
public List<Order> ClassifyVipOrders(List<Order> orders)
{
    // Copilot: filter → classify の 2 ステップが明確
}

// 例 2
// "スコアリスト"から、"指定スコア以上の"学生を抽出して、
// 名前を返す
public List<string> FindHighScorers(List<Student> students, double threshold = 80.0)
{
    // Copilot: filter → map は標準的
}
```

##### パターン 2：エッジケースを明記

```csharp
// 平均スコアを計算する。
// - リストが空の場合は null を返す
// - null 参照は例外をスロー
public double? CalculateAverageScore(List<double> scores)
{
    if (scores == null || scores.Count == 0)
        return null;
    
    return scores.Average();
}
```

##### パターン 3：外部ライブラリの動作を明記

```csharp
// メールアドレスを検証する。
// - 有効な形式なら true を返す
// - System.Net.Mail.MailAddress を使用
public bool ValidateEmail(string email)
{
    try
    {
        var addr = new System.Net.Mail.MailAddress(email);
        return addr.Address == email;
    }
    catch
    {
        return false;
    }
}
```

---

#### 🔧 XMLコメント (Documentation Comments) の活用

XMLコメントを書くことで、Copilot はメソッドの意図を完全に理解します。

##### 基本構造

```csharp
/// <summary>
/// スコアの平均を計算する
/// </summary>
/// <param name="scores">スコアのリスト</param>
/// <returns>平均値、リストが空の場合はnull</returns>
public double? CalculateAverageScore(List<double> scores)
{
    if (scores == null || scores.Count == 0)
        return null;
    
    return scores.Average();
}

/// <summary>
/// しきい値以上のスコアを持つ学生の名前を返す
/// </summary>
/// <param name="students">学生リスト（Name, Score キー）</param>
/// <param name="threshold">しきい値（デフォルト: 80.0）</param>
/// <returns>条件を満たす学生の名前リスト</returns>
public List<string> FindHighScorers(
    List<Dictionary<string, object>> students, 
    double threshold = 80.0)
{
    return students
        .Where(s => s.ContainsKey("Score") && Convert.ToDouble(s["Score"]) >= threshold)
        .Select(s => s["Name"].ToString())
        .ToList();
}
```

##### XMLコメント の要素

| タグ | 用途 | 例 |
|-----|------|---|
| `<summary>` | メソッドの 1 行説明 | 「ユーザーを非アクティブと標記」 |
| `<param>` | パラメータ説明 | `<param name="users">ユーザーリスト</param>` |
| `<returns>` | 戻り値説明 | `<returns>非アクティブユーザーリスト</returns>` |
| `<exception>` | 例外のドキュメント | `<exception cref="ArgumentNullException">」users が null の場合</exception>` |
| `<remarks>` | 詳細説明（オプション） | 実装上の注意など |

##### 効果的な XMLコメント の例

```csharp
/// <summary>
/// 配送料金を計算する
/// </summary>
/// <param name="weight">荷物の重さ（kg）</param>
/// <param name="distance">配送距離（km）</param>
/// <param name="express">速達配送フラグ（デフォルト: false）</param>
/// <param name="international">国際配送フラグ（デフォルト: false）</param>
/// <returns>計算された配送料金（円）</returns>
/// <remarks>
/// 計算ロジック：
/// - 基本料金: 500円
/// - 重さ: kg × 100円
/// - 距離: km × 10円
/// - 国際: 基本×3倍、距離×5倍
/// - 速達: + 1000円
/// </remarks>
public decimal CalculateShippingCost(
    decimal weight,
    decimal distance,
    bool express = false,
    bool international = false)
{
    // Copilot は複雑な計算ロジックでも、remarks で指定した通りに実装
}
```

---

#### 💡 コメント駆動開発のプロセス

##### ステップ 1: 目的を明記

```csharp
// 「何を達成するのか」を最初に書く
// ユーザーリストから、過去30日間ログインていないユーザーを非アクティブにマーク
public void ProcessInactiveUsers(List<User> users)
{
    // Copilot はこのコメントから実装を提案
}
```

##### ステップ 2: エッジケースを記載

```csharp
// 注記：
// - null リストは例外をスロー
// - 空リストは空を返す
// - ログイン日時が null の場合はアクティブと判定
```

##### ステップ 3: 例を示す（複雑な場合）

```csharp
// 例：
// Input: [User{LastLogin: 2024-01-01}, User{LastLogin: 2024-03-01}]
// 今日が 2024-03-31 の場合、最初のユーザーが非アクティブ判定
```

---

#### 🎯 シナリオ別コメントテンプレート

##### データ変換

```csharp
// [入力データ]を[処理内容]して、[出力データ]に変換
// 例：
// "ユーザーリスト"を"アクティブユーザーのみフィルタ"して、
// "ユーザーIDリスト"に変換
public List<int> GetActiveUserIds(List<User> users)
{
    // ...
}
```

##### バリデーション

```csharp
// [対象]が[条件]を満たすか検証
// 成功時: true、失敗時: false
// エッジケース: [対象]が null の場合は例外をスロー
public bool ValidateUser(User user)
{
    // ...
}
```

##### 計算・統計

```csharp
// [データセット]の[指標]を計算
// 空の場合は[デフォルト値]を返す
// 計算式：[数式]
public double CalculateMetric(List<decimal> data)
{
    // ...
}
```

---

#### ✅ 習熟度チェック

このセクション完了時、以下ができていますか？

- [ ] 曖昧なコメントと具体的なコメントの違いが理解できた
- [ ] 「目的 + 条件 + 結果」パターンでコメント書ける
- [ ] XMLコメント の基本構造（summary, param, returns）をかける
- [ ] 複雑なロジックをコメントで説明できる
- [ ] Copilot の提案精度が上がることを実感した

すべてチェック完了 → [03-初心者向けシナリオ](#section-01-foundations-03-beginner-scenarios) へ

---

#### 🔗 関連資料

- [C# XML Documentation Comments](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/)
- [Microsoft Learn: Code documentation](https://learn.microsoft.com/en-us/dotnet/fundamentals/coding-style/coding-conventions)

## Beginner Scenarios {#section-01-foundations-03-beginner-scenarios}

#### 初心者が最初に取り組むべき 3 つの実装シナリオ

実際のコード例を通じて、Copilot の活用法を学びます。各シナリオは簡潔な実装から段階的に複雑化します。

---

#### シナリオ 1: ファイル操作の実装

##### 状況
「テキストファイルを読み込んで、各行をリストとして返す機能を作りたい」

##### ステップ 1: 意図をコメントで伝える

```csharp
// テキストファイルを読み込んで、各行をリストとして返す
public List<string> ReadFileLines(string filename)
{
    // ここから Copilot が提案を出す
}
```

##### ステップ 2: Copilot の提案を確認

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

##### ステップ 3: より モダンな書き方に改善

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

#### シナリオ 2: LINQ の使い方を学ぶ

##### 状況
「コレクション操作を効率的に行いたい。フォアーチループではなく LINQ を使いたい」

##### ステップ 1: 基本的な foreach 版

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

##### ステップ 2: Copilot に LINQ 版を提案してもらう

「このコードを LINQ で書き直して」と指示 →

```csharp
var evenNumbersLinq = numbers.Where(n => n % 2 == 0).ToList();
```

##### ステップ 3: より複雑な例に応用

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

##### 学ぶべきポイント
- `Where()` — フィルタリング
- `Select()` — データ変換
- `GroupBy()` — グループ化
- `Average()`, `Count()` — 集計

---

#### シナリオ 3: 非同期プログラミングの導入

##### 状況
「Web API からデータを取得したい。非同期処理を正しく書きたい」

##### ❌ 間違った非同期処理

```csharp
// 危険：using と例外処理がない
public async Task<string> GetDataAsync()
{
    using var client = new HttpClient();
    var result = await client.GetStringAsync("https://api.example.com/data");
    return result;
}
```

##### ✅ ベストプラクティス版

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

##### 学ぶべきポイント
- ✅ **CancellationToken を渡す** — キャンセル対応
- ✅ **EnsureSuccessStatusCode()** — HTTP エラーチェック
- ✅ **ライングリ例外処理** — HttpRequestException, JsonException など分ける
- ✅ **ログ出力** — エラー診断用に情報を記録
- ✅ **Result パターン** — 例外を投げず結果を返す

---

#### 🎯 実装の流れ

各シナリオで推奨される進め方：

1. **「状況」を理解** — どんな問題を解く？
2. **シンプル版を見る** — 最小限のコード例
3. **Copilot で改善** — より良い書き方を試す
4. **学ぶべきポイント** — 実装パターンを理解

---

#### 💡 各シナリオから学べることの活用先

##### シナリオ 1（ファイル操作）から学べること
- `using` ステートメント — リソース管理
- 例外処理 — エラーハンドリング
- 非同期ファイル操作 — `async/await`

**活用先**: ログ読み込み、設定ファイル解析、BOM ファイル処理など

##### シナリオ 2（LINQ）から学べること
- メソッドチェーン — 流暢なコード
- ラムダ式 — 匿名関数
- 遅延評価 — 効率的なメモリ使用

**活用先**: リストフィルタリング、データ変換、ソート、集計処理など

##### シナリオ 3（非同期処理）から学べること
- async/await パターン — 非ブロッキング処理
- CancellationToken — キャンセル対応
- 例外処理の詳細 — エラー分類

**活用先**: Web API 呼び出し、データベース操作、ファイル I/O など

---

#### ✅ 習熟度チェック

このセクション完了時、以下ができていますか？

- [ ] 3 つのシナリオの実装例が理解できた
- [ ] シナリオ 1：ファイル操作を自分で実装できた
- [ ] シナリオ 2：LINQ で操作を実装できた  
- [ ] シナリオ 3：非同期処理の重要ポイントが理解できた
- [ ] 「Copilot での改善提案」のいずれかを試した

すべてチェック完了 → [04-role-mastery/01-beginner-developer.md](#section-04-role-mastery-01-beginner-developer) へ

---

#### 🔗 参考資料

- [LINQ Basic Concepts](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/linq/)
- [Asynchronous programming patterns](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [Exception handling best practices](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)

# Core Features {#chapter-02-core-features}

## Inline Completions {#section-02-core-features-01-inline-completions}

#### コード補完のテクニックをマスターする

Inline Completions はもっとも頻繁に使う機能です。コメント駆動開発と組み合わせて、精度を最大化します。

---

#### 💡 テクニック 1: コメント駆動開発

##### パターン：具体的なコメント = 精度の高い提案

```csharp
// Before: 曖昧なコメント
// ユーザーを処理する

// After: 具体的なコメント
// ユーザーリストから、最終ログインが30日以上前のユーザーを抽出し、
// 非アクティブユーザーとしてマークする
public List<User> MarkInactiveUsers(List<User> users)
{
    var thirtyDaysAgo = DateTime.Now.AddDays(-30);
    
    foreach (var user in users)
    {
        if (user.LastLogin < thirtyDaysAgo)
        {
            user.Status = UserStatus.Inactive;
        }
    }
    
    return users;
}

// 使用例
var users = new List<User>
{
    new User { Name = "田中", LastLogin = DateTime.Parse("2024-02-01"), Status = UserStatus.Active },
    new User { Name = "鈴木", LastLogin = DateTime.Parse("2024-01-15"), Status = UserStatus.Active }
};
```

##### 効果：
- **前**: コメントがあいまい → 汎用的な提案
- **後**: コメント具体的 → 目的に沿った実装が提案される

---

#### 💡 テクニック 2: XMLコメントで精度向上

XMLDocumentation コメントがあると、Copilot はメソッドの意図を完全に理解します。

```csharp
/// <summary>
/// スコアの平均を計算する
/// </summary>
/// <param name="scores">スコアのリスト</param>
/// <returns>平均値、リストが空の場合はnull</returns>
public double? CalculateAverageScore(List<double> scores)
{
    // XMLコメントがあることで、適切な実装を提案
    if (scores == null || scores.Count == 0)
        return null;
    
    return scores.Average();
}

/// <summary>
/// しきい値以上のスコアを持つ学生の名前を返す
/// </summary>
/// <param name="students">学生リスト</param>
/// <param name="threshold">しきい値（デフォルト: 80.0）</param>
/// <returns>条件を満たす学生の名前リスト</returns>
public List<string> FindHighScorers(
    List<Dictionary<string, object>> students, 
    double threshold = 80.0)
{
    return students
        .Where(s => s.ContainsKey("Score") && Convert.ToDouble(s["Score"]) >= threshold)
        .Select(s => s["Name"].ToString())
        .ToList();
}
```

---

#### 💡 テクニック 3: 繰り返しパターンの自動化

同じパターンのメソッドを複数書くと、Copilot は自動的に次のメソッドを提案します。

```csharp
public class UserValidator
{
    public bool ValidateName(string name)
    {
        return name.Length >= 2 && name.All(char.IsLetter);
    }
    
    // ここから Copilot は同じパターンで次を提案
    public bool ValidateEmail(string email)
    {
        try
        {
            var addr = new System.Net.Mail.MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }
    
    public bool ValidateAge(int age)
    {
        return age >= 0 && age <= 150;
    }
    
    public bool ValidatePhone(string phone)
    {
        var digits = new string(phone.Where(char.IsDigit).ToArray());
        return digits.Length >= 10;
    }
}
```

##### 効果：
- 最初のメソッドを丁寧に作成
- 次のメソッド名を書くと、前のパターンを自動提案
- パターン一致するやつは 1-2 秒で生成

---

#### 🎯 効果的な使用シーン

| シーン | コメント例 | 期待される提案 |
|--------|----------|-------------|
| CRUD 操作 | `// ユーザーを ID で取得` | `GetUserById(int id)` の実装 |
| バリデーション | `// メールアドレスが有効か检证` | `ValidateEmail(string email)` |
| データ変換 | `// リストをIDのマップに変換` | `ToDictionary()` 使用 |
| 計算・集計 | `// スコアの合計を計算` | `Sum()` 使用 |

---

#### ⚠️ 注意点と改善

##### よくある提案ミス

```csharp
// ❌ 不完全な提案が来たら Esc で拒否
public class OrderService
{
    public void ProcessOrder(Order order)
    {
        // 提案が変だったら Esc キーで拒否して書き直し
    }
}

// ✅ 具体的にコメント追加して再度提案依頼
public class OrderService
{
    // 注文を処理する。支払い確認→在庫確保→出荷準備の順序で実行
    public void ProcessOrder(Order order)
    {
        // より具体的なコメントで精度が上がる
    }
}
```

---

#### ✅ 習熟度チェック

- [ ] コメント駆動開発でコード補完の精度が上がることを確認
- [ ] XMLコメントを使った提案を経験
- [ ] 繰り返しパターンで複数メソッド自動生成
- [ ] Tab キーで提案受け入れまでの流れが自然

すべてチェック完了 → [02-copilot-chat.md](#section-02-core-features-02-copilot-chat) へ

---

#### 🔗 参考

- [01-foundations/02-comment-driven-dev.md](#section-01-foundations-02-comment-driven-dev) — コメント駆動の詳細
- [XML Documentation Comments](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/)

## Copilot Chat {#section-02-core-features-02-copilot-chat}

#### チャットインターフェースで対話的に支援を受ける

Copilot Chat は単なるコード生成ではなく、「相談」のツール。設計から実装、改善まで幅広く活用できます。

---

#### 💡 テクニック 1: コード説明を依頼

##### シーン：複雑なコードの意味を理解したい

ユーザー: このコードは何をしていますか？

```csharp
var result = items
    .Where(x => !string.IsNullOrEmpty(x))
    .Select(x => x.Trim().ToLower())
    .Distinct()
    .ToList();
```

Copilot: このコードは以下の処理を行っています：
1. `Where(x => !string.IsNullOrEmpty(x))` - 空またはnullの要素を除外
2. `Select(x => x.Trim().ToLower())` - 各要素の前後の空白を除去し、小文字に変換
3. `Distinct()` - 重複する要素を除去
4. `ToList()` - 結果をリストに変換

つまり、大文字小文字を区別せず、空白を無視したユニークな文字列のリストを返します。

##### ポイント
- 選択したコード + 「このコードは何をしていますか？」で説明依頼
- 複雑なロジックは分解して説明してもらう

---

#### 💡 テクニック 2: リファクタリングの提案

##### シーン：コードが複雑で読みづらい

ユーザー: このコードをリファクタリングして、もっと読みやすくしてください

```csharp
public double Calc(double x)
{
    if (x > 0)
    {
        if (x < 100)
        {
            return x * 1.1;
        }
        else
        {
            return x * 0.9;
        }
    }
    else
    {
        return 0;
    }
}
```

Copilot: 以下のようにリファクタリングできます：

```csharp
/// <summary>
/// 価格を条件に応じて調整する
/// </summary>
/// <param name="price">調整前の価格</param>
/// <returns>調整後の価格（0以上の値）</returns>
public double CalculateAdjustedPrice(double price)
{
    if (price <= 0)
        return 0;
    
    // 100未満は10%アップ、100以上は10%ダウン
    return price < 100 
        ? price * 1.1  // 10%アップ
        : price * 0.9; // 10%ダウン
}
```

主な改善点：
- 意味のあるメソッド名とパラメータ名
- XMLコメントの追加
- 条件式の明確化（三項演算子）
- マジックナンバーにコメント

##### ポイント
- 改善したい メソッドを選択
- 「リファクタリングして」 + 目的を指示
- 複数の案を提示してもらうことも可能

---

#### 💡 テクニック 3: エラー解決

##### シーン：エラーが出ていて、原因がわからない

ユーザー: 以下のエラーが出ています。原因と解決方法を教えてください

```csharp
var json = File.ReadAllText("data.json");
var data = JsonSerializer.Deserialize<List<Item>>(json);
// Error: System.Text.Json.JsonException: The JSON value could not be converted to System.Collections.Generic.List`1[Item]
```

Copilot: このエラーはJSONの形式が期待するList<Item>と一致しないことが原因です。

解決策：

```csharp
using System.Text.Json;

// 1. JSONの構造を確認
var json = File.ReadAllText("data.json");
Console.WriteLine(json); // JSONの内容を表示

// 2. 適切なオプションを設定
var options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true, // プロパティ名の大文字小文字を区別しない
    AllowTrailingCommas = true,         // 末尾のカンマを許可
    ReadCommentHandling = JsonCommentHandling.Skip // コメントをスキップ
};

try
{
    // 3. エラーハンドリング付きでデシリアライズ
    var data = JsonSerializer.Deserialize<List<Item>>(json, options);
    
    if (data == null)
    {
        Console.WriteLine("データがnullです");
    }
}
catch (JsonException ex)
{
    Console.WriteLine($"JSON解析エラー: {ex.Message}");
    Console.WriteLine($"エラー位置: 行 {ex.LineNumber}, 位置 {ex.BytePositionInLine}");
}
```

原因の詳細：
- JSONが配列ではなくオブジェクト
- プロパティ名の大小文字が異なる
- JSON に不正な形式がある

##### ポイント
- エラーメッセージをコピペ
- 「原因と解決方法を教えて」と聞く
- コード例まで提示してくれることが多い

---

#### 💡 テクニック 4: ドキュメント生成

##### シーン：XMLコメントを自動生成したい

ユーザー: このメソッドにXMLコメントを追加してください

```csharp
public async Task<List<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default)
{
    return await _userRepository
        .GetActiveUsersAsync(cancellationToken)
        .ConfigureAwait(false);
}
```

Copilot: 

```csharp
/// <summary>
/// アクティブユーザーを非同期で取得する
/// </summary>
/// <param name="cancellationToken">キャンセレーショントークン</param>
/// <returns>アクティブユーザーのリスト</returns>
/// <exception cref="OperationCanceledException">キャンセルされた場合</exception>
public async Task<List<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default)
{
    return await _userRepository
        .GetActiveUsersAsync(cancellationToken)
        .ConfigureAwait(false);
}
```

##### ポイント
- メソッドを選択
- 「XMLコメントを追加してください」
- `<summary>`, `<param>`, `<returns>`, `<exception>` が自動生成

---

#### 🎯 効果的な質問パターン

| 質問 | 効果 |
|------|------|
| 「このコードは何をしていますか？」 | コード解説 |
| 「リファクタリングしてください」 | 改善提案 |
| 「ベストプラクティスはありますか？」 | パターンの提示 |
| 「テストコードを書いてください」 | テスト自動生成 |
| 「パフォーマンスを最適化してください」 | 速度改善 |

---

#### ⚠️ Chat で注意すべき点

##### ❌ 避けるべき質問
- 「このコード直して」（具体的でない）
- 「速くして」（どの観点？）
- 「いい感じにして」（曖昧）

##### ✅ 効果的な質問
- 「このコードの N+1 クエリ問題を解決してください」
- 「LINQ で書き直してください」
- 「メモリ効率を改善してください」

---

#### ✅ 習熟度チェック

- [ ] Chat パネルを開いて質問できる
- [ ] コード説明を依頼できた
- [ ] リファクタリング提案を受けた
- [ ] エラー解決の相談ができた
- [ ] XMLコメント自動生成を試した

すべてチェック完了 → [03-inline-edits.md](#section-02-core-features-03-inline-edits) へ

---

#### 🔗 参考

- [GitHub Copilot Chat FAQ](https://github.com/features/copilot)
- [C# Best Practices](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)

## Inline Edits {#section-02-core-features-03-inline-edits}

#### 選択したコードを直接修正する

Inline Edits は「選択したコード」に対して修正指示を出す機能。範囲を限定した改善が可能です。

---

#### 💡 テクニック 1: コードの最適化

##### シーン：このコード遅いから最適化したい

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

##### ポイント
- 「パフォーマンスを最適化して」 + 具体的な改善観点（O(n²) → O(n) など）
- 修正後のメソッド名を変更してくれることが多い（わかりやすく）

---

#### 💡 テクニック 2: エラーハンドリングの追加

##### シーン：このコード例外処理がない

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

##### ポイント
- 「エラーハンドリングを追加して」
- null チェック、例外種別の分類、ログ出力まで対応

---

#### 💡 テクニック 3: 複雑な変換

##### シーン：XMLをJSONに変換したい

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

##### ポイント
- 複雑な変換は「指示」として明確に
- 再帰処理など下層的なロジックも生成

---

#### 🎯 有効な指示パターン

| 指示内容 | 例 |
|---------|---|
| **最適化** | 「パフォーマンスを最適化して」「LINQ で書き直して」 |
| **エラー処理** | 「エラーハンドリングを追加して」「nullチェックを追加して」 |
| **フォーマット** | 「非同期に変換して」「Async/Await パターンに」 |
| **セキュリティ** | 「入力検証を追加して」「SQL インジェクション対策を」 |
| **テスト** | 「テスト可能な設計に変更して」 |

---

#### ⚠️ 使用上の注意

##### ❌ 避けるべき使用法
- **大量行のコード選択** ← Chat を使う
- **複数メソッド同時修正** ← 1つずつ修正する
- **あいまいな指示** ← 具体的に

##### ✅ 推奨
- **メソッド単位で修正**
- **1つの観点で修正** （複数観点は複数回実行）
- **修正前後を比較** してから確定

---

#### ✅ 習熟度チェック

- [ ] コードを選択して修正指示ができた
- [ ] 複数行のコード修正ができた
- [ ] 複雑な変換を依頼できた
- [ ] 修正後のコードを確認してから確定

すべてチェック完了 → [04-agent-mode.md](#section-02-core-features-04-agent-mode) へ

---

#### 🔗 参考

- [02-copilot-chat.md](#section-02-core-features-02-copilot-chat) — Chat との使い分け
- [Entity Framework Core Best Practices](https://learn.microsoft.com/en-us/ef/core/)

## Agent Mode {#section-02-core-features-04-agent-mode}

#### 複数ファイルにまたがる大規模タスクを自動実装

Agent Mode は Copilot のもっとも高度な機能。複数ファイルを自動生成・修正して、大規模な機能追加やリファクタリングを一度に実行できます。

---

#### 💡 テクニック 1: 新機能の追加

##### シーン：ユーザー認証機能を一から追加したい

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

##### ポイント
- **複数ファイル作成・更新を自動化**
- **依存関係の自動設定**（DI コンテナなど）
- **エラーハンドリング** まで対応

---

#### 💡 テクニック 2: アーキテクチャリファクタリング

##### シーン：データアクセス層を Repository パターンにリファクタリング

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

#### 💡 テクニック 3: テスト自動生成

##### シーン：このクラスのユニットテストを生成したい

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

#### ポイント
- **テストケース自動生成** （正常系・異常系・境界値）
- **Xunit + FluentAssertions** の自動選択
- **命名規則** に従った自動生成

---

### 🎯 エージェントモードの適用シーン

| 要件 | 所要時間 | 複雑度 |
|-----|--------|------|
| **新機能追加**（複数ファイル） | 5-15 分 | ⭐⭐⭐ |
| **アーキテクチャ リファクタリング** | 10-30 分 | ⭐⭐⭐⭐ |
| **テスト自動生成** | 5-10 分 | ⭐⭐ |
| **データベースマイグレーション** | 5-10 分 | ⭐⭐ |
| **API エンドポイント一括追加** | 10-20 分 | ⭐⭐⭐ |

---

### ⚠️ エージェントモードの注意点

#### ❌ 避けるべき指示
- 「いい感じにして」（曖昧）
- 「全部リファクタリングして」（スコープ不明確）
- 「セキュリティを上げて」（観点不明確）

#### ✅ 推奨される指示
- 「Repository パターンを適用して」（パターン指定）
- 「ユーザー認証機能を追加：[具体的要件]」（スコープ明確）
- 「BCrypt を使ったパスワードハッシュ化」（実装技術指定）

#### 代替案の活用
- エージェントが生成コードに不満
  → `@workspace` 指示で修正指示を追加
  → 「生成ファイルを削除して」で巻き戻し可能（git で）

---

### ✅ 習熟度チェック

- [ ] 複数ファイル作成指示ができた
- [ ] 要件を明確に述べて指示できた
- [ ] 生成されたコードを確認できた
- [ ] 依存性注入が自動設定されたことを理解

すべてチェック完了 → [03-hands-on-workflows/README](#chapter-03-hands-on-workflows) へ

---

### 🔗 参考

- [GitHub Copilot for Enterprise](https://github.com/features/copilot/plans)
- [Design Patterns in C#](https://refactoring.guru/design-patterns/csharp)
- [Dependency Injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)

# Hands on Workflows {#chapter-03-hands-on-workflows}

## Design Patterns {#section-03-hands-on-workflows-01-design-patterns}

#### 既存コードにパターンを適用してリファクタリング

中級者にとって重要な「デザインパターンを実務に適用する」スキルを learning-by-doing で習得します。

---

#### 🎯 シナリオ：支払い処理にストラテジーパターンを適用

##### 状況
「複数の支払い方法（クレジットカード、PayPal、銀行振込）に対応したい」

##### ❌ リファクタリング前：if-else で分岐

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

##### ✅ リファクタリング後：Strategy パターン

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

##### 効果
- ✅ **新しい支払い方法追加が簡単** — 新クラス作成 + DI 登録のみ
- ✅ **テストが容易** — 各戦略のモック化が簡単
- ✅ **責務分離** — 各支払い方法のロジックが独立
- ✅ **保守性向上** — 既存コード修正不要（Open/Closed 原則）

---

#### 🎯 リポジトリパターンの応用

##### シーン：複数のデータソース（SQL, NoSQL, API）に対応

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

#### ✅ 習熟度チェック

- [ ] Strategy パターンの概念が理解できた
- [ ] 既存の if-else 分岐を Strategy に置き換えられた
- [ ] インターフェース設計ができた
- [ ] 依存性注入の設定ができた
- [ ] 新しいパターン追加の簡単さを実感

すべてチェック完了 → [02-performance-optimization.md](#section-03-hands-on-workflows-02-performance-optimization) へ

---

#### 🔗 参考

- [Design Patterns in C#](https://refactoring.guru/design-patterns/csharp)
- [Strategy Pattern](https://refactoring.guru/design-patterns/strategy)
- [SOLID Principles](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#solid)

## Performance Optimization {#section-03-hands-on-workflows-02-performance-optimization}

#### 遅いコードを見つけて、効率的に改善する

パフォーマンス問題の診断と改善は、中級者必須スキル。Copilot を使った効率的なプロセスを学びます。

---

#### 🎯 シーン 1: N+1 クエリ問題の解決

##### ❌ 非効率なコード

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

##### ✅ 最適化版：Include + Select

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

##### 効果
- **クエリ数**: 1001回 → 1回 (1000倍高速化!)
- **メモリ使用**: Include で自動管理
- **実装の簡潔性**: LINQ で宣言的に表現

---

#### 🎯 シーン 2: 複数 API の並列呼び出し

##### ❌ 非効率版：順次処理

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

##### ✅ 最適化版：Task.WhenAll

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

#### 🎯 シーン 3: LINQ クエリの最適化

##### ❌ 非効率版：メモリ上でフィルタリング

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

##### ✅ 最適化版：データベースで処理

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

#### 🎯 シーン 4: キャッシングの活用

##### 状況
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

#### 🔧 Copilot を使った最適化フロー

##### ステップ 1: 遅いコードを選択

```csharp
var expensiveOrders = _context.Orders.ToList()...
```

##### ステップ 2: Chat で質問

```
「このコードが遅い理由は？最適化方法を教えてください」
```

##### ステップ 3: Copilot が説明

```
「.ToList() ですべてをメモリに読み込んでいます。
WHERE を LINQ クエリで実行して、結果のみ取得してください。」
```

##### ステップ 4: Inline Edits で修正

単純な最適化（ToList 削除など）は「該当部分選択 → パフォーマンス最適化」で自動修正。

---

#### ✅ 習熟度チェック

- [ ] N+1 クエリ問題が理解できた
- [ ] Include を使ったデータベースアクセス最適化ができた
- [ ] 複数 API の並列化ができた
- [ ] LINQ での事前フィルタリングができた
- [ ] キャッシング実装ができた

すべてチェック完了 → [03-testability.md](#section-03-hands-on-workflows-03-testability) へ

---

#### 🔗 参考

- [Entity Framework Core Performance](https://learn.microsoft.com/en-us/ef/core/performance/)
- [LINQ to Objects Performance](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic)
- [Async/Await Best Practices](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)

## Testability {#section-03-hands-on-workflows-03-testability}

#### テストしやすい設計にリファクタリング

「このコード、テストが書きやすい/難しい」の差は設計に由来します。依存性注入とモックを活用した設計を学びます。

---

#### 🎯 シーン：依存性が多いコードをテスト可能に

##### ❌ テスト困難なコード

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

##### ✅ テスト可能な設計

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

##### テスト容易性の改善
- ✅ **Moq でモック化** — インターフェース経由のため簡単
- ✅ **実装と隔離** — テストが実装の詳細に依存しない
- ✅ **複数ケースをカバー** — 正常系・異常系を独立テスト

---

#### 🎯 テスト可能な設計の原則

##### 原則 1: Interface を活用

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

##### 原則 2: 依存性を外部から注入

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

##### 原則 3: ビジネスロジックと I/O を分離

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

#### ✅ 習熟度チェック

- [ ] インターフェース設計ができた
- [ ] 依存性注入の実装ができた
- [ ] Moq を使ったモック設定ができた
- [ ] テストケースを複数作成できた
- [ ] 正常系・異常系・境界値をテストできた

すべてチェック完了 → [04-role-mastery/README](#chapter-04-role-mastery) へ

---

#### 🔗 参考

- [Unit Testing Best Practices](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
- [Moq Documentation](https://github.com/Moq/moq4/wiki/Quickstart)
- [xUnit.net Documentation](https://xunit.net/docs/getting-started/netcore)
- [Dependency Injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)

# Role Mastery {#chapter-04-role-mastery}

## Beginner Developer {#section-04-role-mastery-01-beginner-developer}

#### 基礎をしっかり習得して、コーディング効率を高める

初級者にとって重要な「基本の習得」と「習慣形成」にフォーカス。Copilot で効率的に学習を進めるコツを紹介します。

---

#### 📚 学習ロードマップ

##### フェーズ 1：IDE 環境構築（1-2日）
- ✅ [01-foundations/01-basics.md](#section-01-foundations-01-basics) を完了
- Visual Studio Code または Visual Studio 2022 のセットアップ
- Copilot 拡張機能のインストール

##### フェーズ 2：C# の基本構文（1-2週間）
基本概念を Copilot のコメント駆動開発で習得

```csharp
// コメントで「やりたいこと」を説明
// 配列から偶数のみを抽出して、2倍にしたリストを返す

// Copilot に提案させる → 学習する
var numbers = new[] { 1, 2, 3, 4, 5 };
var evenDoubled = numbers
    .Where(n => n % 2 == 0)
    .Select(n => n * 2)
    .ToList();
```

学ぶべきキーワード:
- `foreach`, `for`, `while` — ループ制御
- `if`, `else` — 条件分岐
- `List<T>`, `Dictionary<K,V>` — コレクション
- `LINQ` — データ処理（Where, Select, FirstOrDefault）
- `try-catch` — 例外処理

##### フェーズ 3：Copilot の活用パターン習得（1週間）
- [02-core-features/01-inline-completions.md](#section-02-core-features-01-inline-completions)
- [02-core-features/02-copilot-chat.md](#section-02-core-features-02-copilot-chat)

---

#### 🎯 初級者が陥りやすい落とし穴と解決策

##### ❌ 落とし穴 1: コメントが曖昧 → Copilot が外れた提案

```csharp
// ❌ 曖昧なコメント
// ユーザーを処理する

// ✅ 明確なコメント
// メールアドレスが無効なユーザーをフィルタリングして、
// 無効なユーザーのメールアドレスをログに出力
var invalidUsers = users
    .Where(u => !IsValidEmail(u.Email))
    .ToList();

invalidUsers.ForEach(u => 
    _logger.LogWarning("Invalid email: {Email}", u.Email));
```

**対策**：
1. コメントに「何をするのか」を書く
2. 「入力」と「出力」を明確にする
3. 複数行のコメントを書く（細部まで説明）

##### ❌ 落とし穴 2: 重複コードをコピペ

```csharp
// ❌ 同じロジックが3か所にある
class UserService
{
    public bool ValidateEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

class OrderService
{
    public bool ValidateEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

class PaymentService
{
    public bool ValidateEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

// ✅ 共通メソッドに抽出
public static class ValidationHelper
{
    public static bool IsValidEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}
```

**対策**：
1. 同じコードが 3 か所以上ある → 共通メソッドに抽出
2. Copilot は「メソッド抽出」を提案できる → 選択 + Chat で「メソッド化してください」

##### ❌ 落とし穴 3: エラー処理の欠落

```csharp
// ❌ エラーハンドリングなし
public async Task<User> GetUserAsync(int userId)
{
    var response = await _httpClient.GetAsync($"/api/users/{userId}");
    var json = await response.Content.ReadAsStringAsync();
    return JsonSerializer.Deserialize<User>(json);
}

// ✅ エラー処理あり
public async Task<User> GetUserAsync(int userId)
{
    try
    {
        var response = await _httpClient.GetAsync($"/api/users/{userId}");
        
        if (!response.IsSuccessStatusCode)
        {
            _logger.LogError("User API error: {StatusCode}", response.StatusCode);
            throw new HttpRequestException($"User API returned {response.StatusCode}");
        }
        
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<User>(json) 
            ?? throw new InvalidOperationException("User data is null");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to get user {UserId}", userId);
        throw;
    }
}
```

**対策**：
1. すべての async メソッドに try-catch を追加
2. ネットワークエラー → ログ出力 + 例外 throw
3. Copilot に「エラーハンドリング追加」と依頼

---

#### 💡 初級者向けの効率的な Copilot 使い方

##### パターン 1: 仕様確認（Chat）

```
仕様: ユーザーリストを年齢でソートして、最初の10人を返すメソッドを書いてください。
複数のソート基準（昇順/降順）に対応してください。
```

Copilot が複数パターンの実装を提案 → 学習しながら選択

##### パターン 2: リファクタリング質問（Chat）

```
このコードが読みづらい理由は何ですか？
改善案を提案してください。

[該当コード貼り付け]
```

Copilot が「ネストが深い」「変数名が不明確」などを指摘 + 改善案提示

##### パターン 3: デバッグ質問（Chat）

```
このメソッドで NullReferenceException が発生しています。
原因と改善案を教えてください。

[エラーが出るコード貼り付け]
```

Copilot が null チェック漏れなどを指摘

---

#### ✅ 初級者チェックリスト

毎週、以下をチェック。すべて「できている」になるまで前に進まない。

##### Week 1-2: 基本構文習得
- [ ] `foreach` ループで配列を走査できた
- [ ] LINQ `Where`, `Select` で中級レベルのフィルタリングができた
- [ ] Dictionary の初期化と取得ができた
- [ ] try-catch で例外処理を書いた
- [ ] 変数名が「意図」を表現している（❌ `x, data` → ✅ `userName, totalAmount`）

##### Week 3-4: Copilot 基本活用
- [ ] コメント駆動開発を 5回以上実践した
- [ ] Chat で「このコードを改善してください」と質問できた
- [ ] 提案コードの「なぜ」を理解できた
- [ ] 自分で簡単な機能を Copilot なしで実装できた（重要！）
- [ ] ペアプログラミングで他者に説明できた

##### Month 2: スキル定着
- [ ] 月に 1回、既存コードのリファクタリングを完了した
- [ ] テストコード（xUnit）を書いた
- [ ] Pull Request を提出して、コードレビューを受けた
- [ ] 「Copilot はこうすれば上手く使える」をチームに説明できた

---

#### 🔗 次のステップ

準備ができたら：
1. 簡単な実装タスクを自分で完了する
2. その後、 [02-mid-level-developer.md](#section-04-role-mastery-02-mid-level-developer) へ進む
3. 3-6か月で中級者レベルへ到達

**注意**：初級者が「Copilot に全部書かせる」は禁止！必ず「Copilot が何をしてくれたか理解してから進める」を習慣に。

---

#### 📚 参考資料

- [C# の基本](https://learn.microsoft.com/ja-jp/dotnet/csharp/fundamentals/types/)
- [LINQ を使用してデータをクエリします](https://learn.microsoft.com/ja-jp/dotnet/csharp/fundamentals/functional/async)
- [例外処理](https://learn.microsoft.com/ja-jp/dotnet/csharp/fundamentals/exceptions/)

## Mid Level Developer {#section-04-role-mastery-02-mid-level-developer}

#### 実装スキルを深掘りして、チーム貢献度を高める

中級者にとって重要な「設計パターン」「パフォーマンス」「保守性」。これらを Copilot で効率的に習得・実装します。

---

#### 📊 中級者へのかんたん診断

- 独立したメソッドを書ける → **はい**
- デザインパターンを聞いたことがある → **はい**
- 「なぜテストが必要か」を説明できる → **はい**
- パフォーマンス問題の原因を調査できる → **はい/部分的**

↑ ほとんど「はい」なら、あなたは中級者です。

---

#### 🎯 中級者が習得すべき 3つのスキル

##### スキル 1: 設計パターンの実装

中級者は「正しいパターンを選んで、正しく実装できる」がマスト。

```csharp
// シーン：複数の支払い方法に対応
// 初級者の実装❌
public bool ProcessPayment(Order order, string method)
{
    if (method == "CreditCard")
    {
        // クレジットカード処理
    }
    else if (method == "PayPal")
    {
        // PayPal 処理
    }
    // 新しい方法追加時に改造必要

    // 中級者の実装✅
    // Strategy パターンで実装（[03-hands-on-workflows/01-design-patterns.md](../03-hands-on-workflows/01-design-patterns.md) 参照）
    public async Task<PaymentResult> ProcessPaymentAsync(Order order)
    {
        var strategy = _paymentStrategyFactory.GetStrategy(order.PaymentMethod);
        return await strategy.ProcessPaymentAsync(order);
    }
    // 新しい方法追加時：実装追加だけ（既存コード変更不要）
}
```

**習得方法**：
1. [02-core-features/02-copilot-chat.md](#section-02-core-features-02-copilot-chat) で「このコード、リファクタリング候補のパターンは？」と聞く
2. Copilot が Strategy/Factory/Repository などを提案
3. 実装して、PR で指摘を受ける
4. フィードバック反映

##### スキル 2: パフォーマンスの診断と改善

「このコードなぜ遅い？」と自分で問い, 改善できる力。

**診断フロー**：
1. ログ/メトリクスで遅い部分を特定
2. Copilot Chat: 「このクエリが遅い理由は？」
3. Copilot が「N+1 問題」などを指摘
4. Include や parallel processing で改善
5. ベンチマーク取得 → 改善効果を確認

例：[03-hands-on-workflows/02-performance-optimization.md](#section-03-hands-on-workflows-02-performance-optimization)

##### スキル 3: テスト可能な設計

「テストを書きながら設計できる」は強力なスキル。

**TDD (Test-Driven Development) 流**：
1. インターフェース定義
2. テストコード作成（テスト駆動）
3. 実装
4. テスト実行
5. リファクタリング

Copilot の活用：
- Chat で「このクラスのユニットテストを書いてください」
- テストコード雛形を生成 → 修正 → 実装

例：[03-hands-on-workflows/03-testability.md](#section-03-hands-on-workflows-03-testability)

---

#### 🚀 中級者向け実践タスク（月 1 本程度）

##### タスク 例：既存コードの設計パターン適用

```
対象：OrderService の ProcessPayment メソッド
現状：if-else で複数の支払い方法に対応（約 50 行）
目標：Strategy パタートムで実装

進行手順：
1. 現在のコードを読む → 改善点を Copilot に質問
2. Strategy パターン の設計を提案してもらう
3. IPaymentStrategy インターフェース + 実装クラス群を作成
4. テストコード作成
5. PR 提出 → レビュー → マージ

期待効果：
- 新しい支払い方法追加がメソッド修正不要に
- テスト カバレッジ向上
- 既存機能へのリグレッション 0（テストで保証）
```

---

#### 📋 中級者チェックリスト

##### 設計スキル
- [ ] 3 つ以上のデザインパターン（Strategy, Repository, Factory）を実装した
- [ ] インターフェース設計を「ユーザー視点」で考慮した
- [ ] 既存コードをリファクタして、SOLID 原則を適用した
- [ ] パターンの適用前後で テスト数を比較できた
- [ ] 「このコードはどのパターンを使うべき？」と聞かれて説明できた

##### パフォーマンススキル
- [ ] N+1 クエリ問題を診断・解決した
- [ ] Include/ThenInclude を使った最適化ができた
- [ ] 複数 API の並列化（Task.WhenAll）ができた
- [ ] LINQ での事前フィルタリング（.ToList() 除去）ができた
- [ ] SQL レベルでのパフォーマンス測定ができた

##### テストスキル
- [ ] ユニットテスト（xUnit）を 10 個以上書いた
- [ ] モック（Moq）を使った テスト可能な設計ができた
- [ ] テストコード量 ≥ 実装コード量（推奨）
- [ ] CI/CD パイプラインでテスト自動実行される
- [ ] バグが出たら「テストで事前検出できたはず」と思考できた

##### コード品質スキル
- [ ] CODE REVIEW で指摘される問題を自分で発見できるようになった
- [ ] PR のコメントで「なぜそう設計したか」を説明できた
- [ ] リファクタリング後も「すべてのテストがパス」を確認する習慣
- [ ] チーム内で「パターン適用例」を共有できた

##### Copilot 活用スキル
- [ ] Chat で「デザインパターンの相談」ができた
- [ ] Inline Edits で「複数箇所の一括リファクタリング」ができた
- [ ] テストコード生成を効率的に活用できた
- [ ] Copilot の提案を「盲信」せず、「検証してから採用」できた

---

#### 💼 中級者から上級者への進路

以下のいずれかに注力：

##### 進路 A: スペシャリスト化
- **領域**: 例）EF Core のエキスパート、LINQ 最適化スペシャリスト
- **キャリア**: コンサルタント、アーキテクト方向
- **次の学習**: [03-hands-on-workflows/](#chapter-03-hands-on-workflows) を完全習得 + 実務分野での深掘り

##### 進路 B: リーダーシップ化
- **領域**: チームの設計基準策定、コードレビューリーダー
- **キャリア**: シニアエンジニア、テックリード方向
- **次の学習**: [04-role-mastery/03-senior-lead.md](#section-04-role-mastery-03-senior-lead) へ

---

#### 🔗 参考リソース

- [SOLID 原則](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles)
- [デザインパターン in C#](https://refactoring.guru/design-patterns/csharp)
- [Entity Framework Core Performance](https://learn.microsoft.com/en-us/ef/core/performance/)
- [Unit Testing Best Practices](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)

---

#### 📞 よくある質問

**Q: Copilot がすべてコード生成してくれるなら、自分で設計する必要ある？**

A: はい。Copilot は「実装」を生成してくれますが、「設計決定」はあなたがする必要があります。
- Copilot: 「実装しますね」
- あなた: 「どのパターンを使うべき？」「なぜそう設計した？」を判断

設計スキルなく Copilot を使うと、「動くがメンテナンス困難」なコードになります。

**Q: テストを書く時間がない。中級者はテスト必須？**

A: テストがないと「リファクタリングが怖くなり」「バグが増える」。
逆にテストがあると「リファクタリング → 品質向上」のサイクルが回ります。
Copilot はテストコード生成で時間短縮できるので、この機会に TDD 導入をお勧めします。

## Senior Lead {#section-04-role-mastery-03-senior-lead}

#### アーキテクチャ設計と組織的影響力で、事業価値を最大化する

シニアリードにとって重要な「システム設計」「技術意思決定」「チーム育成」。Copilot をこれらの スケーラビリティ向上に活用します。

---

#### 📊 シニアリード診断

- コンポーネント間の依存関係を設計図で説明できる → **はい**
- パフォーマンス問題の根本原因を追跡できる → **はい**
- 「新入社員が 3 か月で生産性を上げる」設計ができた → **はい**
- チーム全体のコード品質を向上させた実績 → **はい**

↑ すべて「はい」なら、あなたはシニアリード候補です。

---

#### 🏗️ シニアリードの責務

##### 責務 1: アーキテクチャ設計と意思決定

「どうやって作るか」を決めるのがシニアリード。

**具体例**：マイクロサービスの導入検討

```
Business Requirement:
- グローバル展開で複数地域からのアクセス増加
- 決済システムの障害が他サービスに影響しないこと
- 開発チームの並列化（複数チームの独立開発）

Design Options:
A) モノリシック（現状） → スケーリング困難
B) マイクロサービス → 複雑度増加、運用コスト
C) Modular Monolith → 中庸

決定プロセス：
1. Copilot に「マイクロサービス vs モノリシック メリット・デメリット」を質問
2. テーマ別アーキテクチャドキュメント作成
3. 技術検証（PoC）で仮説検証
4. チーム投票 + 経営層に報告
5. 決定 + ロードマップ策定
```

**Copilot の活用**：
- Chat で「うちのシナリオに合うアーキテクチャは？」と相談
- アーキテクチャ検討ドキュメント の骨子を生成
- 意思決定をスピードアップ

##### 責務 2: チーム育成と標準化

「誰が書いても同じ品質」を実現するのがシニアリード。

**具体例**：`.github/copilot-instructions.md` の策定

````markdown
# Copilot Instructions

## Project Context
- **Tech Stack**: .NET 8, EF Core 8, Clean Architecture
- **Target**: e-commerce platform with 1M+ users

## Code Standards

### Design Patterns
1. Dependency Injection （Microsoft.Extensions.DependencyInjection）
2. Repository Pattern （IRepository<T> interface）
3. Strategy Pattern （支払い方法等の複数実装）

### Naming Conventions
- Classes: `PascalCase`
- Methods: `PascalCase`
- Private fields: `_camelCase`
- Async methods: `*Async` suffix

### Error Handling
All public methods MUST include try-catch and logging:
```csharp
try
{
    // Implementation
}
catch (Exception ex)
{
    _logger.LogError(ex, "Method failed");
    throw;
}
```

### Testing
- Target: ≥ 80% code coverage
- Tools: xUnit, Moq, FluentAssertions
- Naming: `[MethodName]_[Scenario]_[ExpectedResult]`

## Performance Requirements
- API response time: ≤ 500ms (P95)
- Database query: ≤ 100ms (P95)
- Memory: ≤ 500MB per application

## Security
- All external input validation required
- SQL injection prevention: parameterized queries
- CORS: Whitelist only approved origins
````

**チーム波及効果**：
- 新人が「このプロジェクトの標準」を Copilot に教えられる
- Pull Request レビューの軸が統一される
- 「パターン論争」が減る

##### 責務 3: パフォーマンス＆スケーラビリティの管理

「システムが成長しても、スピードを失わない」がミッション。

**具体例**：ページ読み込み時間が 2 秒 → 500ms へ

```
診断フロー：
1. Application Insights で遅い操作を特定 →「注文詳細ページ」
2. DB 実行計画を確認 → N+1 クエリ
3. 改善案を Copilot に相談
4. Copilot が Include + Select で最適化コード生成
5. ベンチマーク取得 → 2秒 → 500ms 確認
6. チーム会で共有 → パターン化 → 他ページにも適用
```

**Copilot の活用**：
- 遅いクエリを Chat に貼り付け → 「最適化方法を教えてください」
- 生成案をテスト環境で検証 → 採用
- 「N+1 解決パターン集」としてドキュメント化

---

#### 🎓 シニアリードの本領発揮：技術的負債の削減

##### シーン：「新機能追加が遅い」問題

**根本原因分析** (5W1H)：
```
現象: 簡単な機能が 2 週間かかる
→ 原因 1: コード複雑度が高い（Cyclomatic Complexity）
→ 原因 2: テストがない（変更時に 10 個のバグ出現）
→ 原因 3: ドメイン知識が散在（仕様書が複数箇所に）

改善計画:
1. チーム全体で「最も複雑な 5 つのモジュール」を特定
2. 各モジュールを分析
3. リファクタリング計画書を作成（優先度 + 期限）
4. 各開発者にアサイン → Copilot でテストコード生成サポート
5. Re-test + パフォーマンス計測
6. 6 か月後：「機能追加が 1 週間に短縮」を達成

投資効果:
- 開発スピード 2倍化
- バグ率 70% 低下
- 開発者 morale 向上
```

**Copilot の活用**：
- リファクタリング前後での テストコード自動生成
- テスト駆動で変更を確実にする
- ドキュメント生成（XMLDoc → API ドキュメント）で知識を可視化

---

#### 📋 シニアリードチェックリスト

##### アーキテクチャスキル
- [ ] システム全体の責務分離を設計図で説明できた
- [ ] 複数のアーキテクチャ（Clean Arch, CQRS, Microservices）の トレードオフを説明できた
- [ ] スケーラビリティ要件を技術選定に反映できた
- [ ] データベース設計（正規化、インデックス）を指導できた

##### 運用・組織スキル
- [ ] `.github/copilot-instructions.md` を策定して チーム全体に展開した
- [ ] Code Review 基準を文書化して、チーム内で統一できた
- [ ] テクニカル負債を定量化（「リファクタリング X 時間で 2 倍スピード」）できた
- [ ] 新人育成プログラム（初級 → 中級 → 上級パス）を設計した

##### リーダーシップスキル
- [ ] チーム内で「技術的意思決定」をファシリテーションできた
- [ ] 「なぜこの設計？」をエンジニア以外にも説明できた
- [ ] リスクを明確に伝えた上で判断を下せた（決断力）
- [ ] チーム内の 「このパターンがベストプラクティス」という文化をつくった

##### Copilot 活用スキル
- [ ] アーキテクチャドキュメント生成で時間短縮できた
- [ ] チーム全体の Copilot Instructions を策定した
- [ ] ジュニアの リファクタリング + テスト生成を Copilot でサポートできた
- [ ] Copilot の提案を「完全採用/完全却下」ではなく「改良」できた

---

#### 🚀 高度な活用例：制度的改善

##### 例 1: 「技術夜間」オンボーディング

```
新入エンジニア M さんが入社
→ 初級パス（[01-beginner-developer.md](01-beginner-developer.md)）を 2 週間で習得
→ 「Copilot コメント駆動開発」で C# 基本を習得
→ 最初の機能担当（既存機能の小さなバグ修正）を自分で完了

速度:
従来: 1 か月（メンター付き、ペアプロ多め）
新方式: 2 週間（Copilot + ドキュメント）

効果:
- 開発生産性: 200% 向上
- メンター負担: 80% 削減
- 新人満足度: 向上（自分のペースで学習可能）
```

##### 例 2: デイリースタンドアップの効率化

```
従来（15分）:
- エンジニア 1: 「昨日 X 完了、今日 Y します」
- エンジニア 2: 「昨日 P & Q、ブロッカー Z」
- ...

新方式（5分）:
- Copilot が「進捗サマリー」を自動生成（GitHub Issues から）
- 「ブロッカーあり」のチーム員だけ詳細共有
- Slack に「進捗サマリー」投稿 → async 確認も可能
```

---

#### 💡 シニアリード必読：組織へのレバレッジ

**最高のコード** vs **組織に機能するコード**

```
❌ よくある誤り：
- 「完璧なアーキテクチャを設計して 3 か月」
- 「チーム全員が理解できるまで設計説明会 10 回」

✅ シニアリードの正解：
- 「 70 点のアーキテクチャを 1 週間で決定」
- 「決めたら、ドキュメント（Copilot 生成）と実装で示す」
- 「3 か月後『アーキテクチャをこう改善したい』とチーム提案」
```

スピード × チーム理解 > 完璧さ

---

#### 🔗 次のステップ

##### キャリアパス
1. **テックリード化**（あなたが現在地）
   - チーム全体の技術戦略を決定
   - 組織拡大時、複数チームの調整役

2. **アーキテクト化**
   - 全社のテック戦略
   - 複数プロダクト間の標準化

3. **VPE/CTO 方向**
   - 事業と技術の接起点
   - 採用・育成・組織文化

---

#### 📚 参考文献

- 「エンジニアリング組織論」（マネジメントの科学的基礎）
- 「ビルドメンターシック」（リーダーシップ）
- [Microsoft Architecture Guides](https://learn.microsoft.com/en-us/azure/architecture/)
- [Enterprise Architecture Patterns](https://martinfowler.com/articles/patterns-of-distributed-systems/)

## QA Engineer {#section-04-role-mastery-04-qa-engineer}

#### Copilot で テスト自動化と品質保証を高度化する

QA エンジニアにとって重要な「テスト自動化」「不具合検出」「品質メトリクス」。Copilot をテストコード生成と品質戦略に活用します。

---

#### 📊 QA エンジニア診断

- テストシナリオを自分で設計できる → **はい**
- 自動化テストコードを書ける（xUnit, Selenium など） → **はい/部分的**
- バグのバグナンバーを根本原因で分類できる → **はい**
- 品質メトリクス（テストカバレッジ、バグ検出率）を追跡できる → **はい**

↑ 3 つ以上「はい」なら、あなたは QA エンジニアです。

---

#### 🎯 QA エンジニアが Copilot を活用する 3 つのシーン

##### シーン 1: 単体テストコード生成の自動化

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

##### シーン 2: 統合テスト・E2E テストの効率化

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

##### シーン 3: 品質メトリクスと不具合分析

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

#### 🔧 テスト自動化フレームワークの設計

##### Unit Test (xUnit)

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

##### Integration Test (xUnit + Moq)

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

##### E2E Test (WebApplicationFactory)

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

#### 📋 QA エンジニアチェックリスト

##### テスト設計スキル
- [ ] テストシナリオを要件から自動導出できた
- [ ] 正常系・異常系・境界値を網羅的に列挙できた
- [ ] 複数テストケースを効率的にカバーできた（Data-Driven テスト）

##### テスト自動化スキル
- [ ] Unit Test (xUnit) で 80% 以上カバレッジ達成
- [ ] Integration Test で複数コンポーネント連携を検証
- [ ] E2E Test で full workflow を自動検証
- [ ] Copilot でテストコード生成できた

##### 不具合分析スキル
- [ ] バグを「機能別」「重大度別」「期間別」で分類できた
- [ ] 「High バグが多い機能」を特定できた
- [ ] 修正までの平均日数を計測できた
- [ ] 「この機能の issue が減った」を定量測定できた

##### 品質向上スキル
- [ ] テストカバレッジを改善した（50% → 80%）
- [ ] バグ検出率を向上させた（本番バグ減）
- [ ] テスト実行時間を短縮した（30分 → 5分）

##### Copilot 活用スキル
- [ ] テストコード自動生成で生産性向上
- [ ] 複雑なテストロジックを Chat で相談
- [ ] SQL クエリ（品質メトリクス）を Copilot で生成

---

#### 🚀 高度な活用：Mutation Testing

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

#### 📞 QA よくある質問

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

#### 🔗 参考リソース

- [xUnit.net Documentation](https://xunit.net/)
- [Moq Framework](https://github.com/moq/moq4)
- [FluentAssertions](https://fluentassertions.com/)
- [WebApplicationFactory for Testing](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests)
- [Mutation Testing](https://github.com/stryker-mutator/stryker-net)

# Team Operations {#chapter-05-team-operations}

## Team Standards {#section-05-team-operations-01-team-standards}

#### コード品質と開発効率のための共通ルールを作る

チーム全体で Copilot を効果的に活用するには「標準」が必須。一貫したコード品質、保守性、Copilot との付き合い方を定義します。

---

#### 🏢 チーム標準のねらい

##### 問題シーン
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

##### 解決策：チーム標準を決める
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

#### 📋 チーム標準ドキュメントのテンプレート

##### 1. プロジェクト基本情報

```markdown
## Project Information
- **Team Name**: E-Commerce Backend Team
- **Tech Stack**: .NET 8, EF Core 8, SQL Server
- **Repository**: github.com/ourcompany/ecommerce-api
- **Team Size**: 5 engineers
- **Target Coverage**: 80% unit test
```

##### 2. 命名規則

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

##### 3. デザインパターン標準

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

##### 4. エラーハンドリング標準

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

##### 5. テスト基準

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

##### 6. パフォーマンス基準

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

##### 7. Copilot 使用ルール

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

##### 8. コードレビューチェックリスト

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

#### 🚀 チーム標準の導入ステップ

##### Week 1: 基本ルールを決定
```
Team meeting (2時間):
1. 命名規則確認（既存 + 更新する箇所）
2. デザインパターン標準（Strategy, Repository, etc.）
3. エラーハンドリング標準
4. テスト基準（覆陰率, ツール, 場所）
```

##### Week 2: ドキュメント作成 & CI/CD パイプライン更新
```
1. `.github/copilot-instructions.md` 作成
2. `.editorconfig` 作成（自動フォーマット）
3. linter / formatter 設定（StyleCop, EditorConfig Analyzer）
4. Git pre-commit hook で強制
```

##### Week 3: チーム全体トレーニング
```
1. 標準ドキュメントの説明会（30分）
2. Copilot + 標準の演習（1時間）
3. 質問受け付けタイム
```

##### Week 4: 実装 + フィードバック
```
新機能開発で標準を使用
→ コードレビューで「標準守れてるか」チェック
→ 月末レトロスペクティブで「標準、改善点」協議
```

---

#### ✅ チェックリスト

- [ ] チーム全体で「デザインパターン標準」を決定
- [ ] `.github/copilot-instructions.md` を作成
- [ ] `.editorconfig` を設定
- [ ] コードレビュー チェックリスト を定義
- [ ] チーム全体で標準をレビュー＆承認
- [ ] 学習資料を共有（Wiki / Confluence）

---

#### 🔗 参考

- [EditorConfig](https://editorconfig.org/)
- [StyleCop Analyzers](https://github.com/DotNetAnalyzers/StyleCopAnalyzers)
- [C# Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)

## Adoption Strategy {#section-05-team-operations-02-adoption-strategy}

#### チーム全体で Copilot を段階的に導入し、生産性を最大化する

「Copilot を買った → 全員使わせる」では失敗。段階的な導入で、チーム文化に合わせた活用を実現します。

---

#### 📊 導入失敗パターン

```
❌ 失敗例：
- 突然「全員 10月からCopilot 必須」と発表
- トレーニング不足
- チーム内のスキル差を無視
- 「Copilot が全部やる」という期待のみ

結果：
- 初級者：「Copilot が生成したコド誤り対応に時間」→ 生産性低下
- 中級者：「便利だが品質不安」→ 使用躊躇
- チーム全体：「Copilot 検証コスト > 生産性向上」→ 冷笑的
```

##### ✅ 成功する導入
```
1. Pilot phase（小規模）
2. 段階的展開（1 チーム → 複数チーム）
3. フィードバック反映
4. 継続的改善

結果：
- 生産性 30% 向上
- 品質 15% 向上（テスト自動生成で検出率↑）
- チーム満足度 高い
```

---

#### 🎯 3 フェーズ導入プラン

##### フェーズ 1：パイロット（1 か月）

**目標**：「Copilot で何ができるか」をチーム内で検証

**参加者**：5-10 人の有志（主に中堅以上）

**タスク**：
1. **Week 1-2: 学習**
   - Copilot 基本操作（コメント駆動、Chat、Edits）
   - チーム標準の理解 (`.github/copilot-instructions.md`)
   - 簡単なタスク（バグ修正）で実践

2. **Week 3: 本格試用**
   - 通常の新機能開発で Copilot を使用
   - 生成コード品質を記録（テスト coverage, code review comments）

3. **Week 4: レトロスペクティブ**
   - メンバーアンケート：「生産性向上？」「品質は？」「課題は？」
   - メトリクス確認：「テストカバレッジ向上？」「レビュー時間短縮？」
   - 改善点协议

**成功指標**：
- [ ] テスト カバレッジ 10% 以上向上
- [ ] コード レビュー時間 20% 短縮
- [ ] チームメンバー 70% 以上が「有用」と評価

---

##### フェーズ 2：展開（2-4 か月）

**目標**：段階的に全チームへ拡大

**展開スケジュール**：
```
Month 1: Pilot team (5-10 人) → レトロスペクティブ
Month 2: Pilot + Early Adopter (10-15 人)
Month 3: + Mainstream (25-30 人)
Month 4: Company-wide (全員)
```

**各段階での取り組み**：

**Early Adopter フェーズ** (Month 2)：
```
新メンバーの追加：
1. Pilot チーム が「ベストプラクティス」を共有
2. 新メンバーに「学習パス」提供
   - [04-role-mastery/01-beginner-developer.md](../04-role-mastery/01-beginner-developer.md)
   - または中級向けパス
3. Pair programming で Copilot の使い方をメンタリング
4. 簡単なバグ修正で習熟
```

**Mainstream フェーズ** (Month 3)：
```
全員が Basic インストール済み状態
1. 全社勉強会（40分）：
   - 「Copilot で何ができる」デモ
   - ベストプラクティス共有（Pilot チーム事例）
   - チーム標準の説明

2. チーム別ワークショップ（各 30分）
   - 「うちのチームで Copilot を使うなら？」
   - チーム特有の課題を解決

3. 1 on 1 Q&A セッション
   - 個別の疑問や懸念に対応
```

**Company-wide フェーズ** (Month 4)：
```
展開完了
1. 全員が Copilot を自分のペースで活用開始
2. Slack チャネル「#copilot-tips」で Tips 共有
3. 月 1 回「Copilot 活用事例」LT
```

---

##### フェーズ 3: 最適化と継続改善（6 か月以上）

**目標**：組織的な競争優位性へ昇華

**継続的改善タスク**：

1. **月次レトロスペクティブ** (30分)
   ```
   アジェンダ：
   - メトリクス確認（コード品質、テストカバレッジ、生産性）
   - Best Practice 共有
   - 課題協議 + 改善案
   - `.github/copilot-instructions.md` の更新判断
   ```

2. **四半期ごとのスキルアップ**
   `
   例）Q2 目標：「テスト自動化で 95% coverage 達成」
   - Copilot でテストコード自動生成
   - チーム全体で TDD を導入
   - メトリクスで効果測定
   ```

3. **組織拡大時の標準化**
   ```
   新しいチーム が増えたら：
   - 既存チームの `.github/copilot-instructions.md` を提供
   - 学習パス を提供
   - Pair programming でメンタリング
   ```

---

### 📈 導入中のKPI 追跡

#### Week 1-4: Pilot フェーズ KPI

| メトリクス | 測定方法 | 目標値 | 実績 |
|-----------|--------|-------|------|
| **生産性向上** | コミット数/人/週 | 10% ↑ | ? |
| **テスト カバレッジ** | Code Coverage % | +10% | ? |
| **コードレビュー時間** | 平均 review time | -20% | ? |
| **チーム満足度** | アンケート（5段階） | ≥3.5 | ? |
| **Quality Issues** | リグレッション バグ数 | -15% | ? |

#### Month 1-4: 展開フェーズ KPI

| メトリクス | 月 1 | 月 2 | 月 3 | 月 4 |
|-----------|------|------|------|------|
| **利用ユーザー数** | 10 | 15 | 30 | 40 |
| **機能別利用率** | 設計:50% Chat:60% | 60% 65% | 70% 70% | 75% 75% |
| **生産性向上** | 3% | 8% | 15% | 20% |
| **テスト coverage** | +2% | +5% | +10% | +12% |

---

### 🎓 教育・サポート体制

#### 1. オンボーディングパス (New メンバー)

**Day 1: インストール & 初回体験**
- Copilot 拡張インストール
- `.github/copilot-instructions.md` を読む
- Chat で簡単な質問（「Hello world プログラム書いてください」）

**Day 2-3: ハンズオンワークショップ**
- コメント駆動開発の実践
- 実際のバグ修正を Copilot で対応
- メンター が横について Q&A

**Week 2: 実務投入**
- 簡単な機能 assign
- Copilot を使いながら実装
- Code Review で「良かった点」「改善点」フィードバック

**Week 4: Retrospective**
- 「Copilot で困ったことは？」を聞く
- Tips や Patterns を共有

#### 2. Support チャネル

**Slack チャネル**：`#copilot-help`
- 誰でも質問可能
- Senior エンジニアが 24時間以内回答

**Wiki / Confluence**
- FAQ 集
- チーム固有の Tips
- アンチパターン集（「こうしてはいけない」）

**月 1 回 LT タイム** (30分)
```
「先週こんなふうに Copilot 使ったら便利だった」を共有
→ Slack にアップロード
→ Best Practice Database へ蓄積
```

---

### ⚠️ の リスク と対策

| リスク | 根本原因 | 対策 |
|--------|---------|------|
| Copilot が生成したバグ入りコード実装 | 品質チェック不足 | テスト Coverage 80% 以上を必須 + Code Review 強化 |
| セキュリティ脆弱性（SQL injection等） | AI 盲信 | チーム標準で「入力検証 + Parameterized Queries」を明記 |
| スキル低下（初級者が Copilot 依存） | 学習不足 | 初級者パス で「理解してから使う」を強調 |
| チーム内スキル格差拡大 | 早期導入者 vs 非導入者 | 全員対象の段階的導入」制 |

---

### ✅ 導入チェックリスト

#### フェーズ 1: パイロット準備
- [ ] Pilot チームメンバー決定（5-10 人）
- [ ] MDD 化 3 フェーズ導入計画を作成 + approval
- [ ] チーム標準 (`.github/copilot-instructions.md`) を Draft
- [ ] 学習教材 整備（このドキュメント、チュートリアル）
- [ ] Slack チャネル `#copilot-questions` 作成

#### フェーズ 1: 実施期間（1 か月）
- [ ] Week 1-2: Pilot team learning + basic exercises
- [ ] Week 3: Real feature development with Copilot
- [ ] Week 4: Retrospective + metrics collection
- [ ] Decision: "Continue to Phase 2?" approval

#### フェーズ 2: 段階的展開
- [ ] Early Adopter team onboarding + pair programming
- [ ] Mainstream team 全社勉強会 開催
- [ ] チーム別ワークショップ実施
- [ ] 月次レトロスペクティブ開始
- [ ] KPI ダッシュボード 構築

#### フェーズ 3: 継続改善
- [ ] 月次リトロで「Best Practice」を共有
- [ ] 四半期ごとのスキルアップにテーマ設定
- [ ] Wiki/FAQ を充実
- [ ] 新メンバーへのオンボーディング standardize

---

### 💬 よくある質問

**Q: Copilot 導入で本当に生産性が上がる？**

A: 「うまく導入できれば」上がります。
- 成功事例: テスト自動化で 1時間/人/週 短縮
- 失敗事例: 検証コストが大きく、生産性低下

成功の鍵：
1. チーム標準を決める
2. 段階的に導入（無理強いしない）
3. フィードバック反映
4. 継続改善

**Q: 初級者が Copilot に依存しすぎていないか心配**

A: 正当な心配。対策：
- 初級パス で「理解 → 実装」の順序を強調
- Code Review で「このコード、なぜ Copilot が提案した？」を聞く
- ペアプログラミング で メンター が横にいる
- テスト駆動で「自分で設計」習慣を つける

**Q: すべてのチームが同じペースで進める必要ある？**

A: いいえ。チーム特性に応じて：
- QA チーム: テスト自動化を重点
- バックエンド: 設計 & パフォーマンス
- インフラ: IaC コード生成

但し「チーム標準」と「学習パス」は会社共通で。

## Metrics Measurement {#section-05-team-operations-03-metrics-measurement}

#### Copilot の効果を定量的に測定して、投資効果を証明する

「Copilot を導入したら生産性が上がった」は主観的。データで測定し、改善サイクルを回します。

---

#### 📊 測定すべき主な KPI

##### 1. 生産性メトリクス

###### コミット数 / 人 / 週

```sql
SELECT 
    DATETRUNC('week', AuthorDate) AS Week,
    Author,
    COUNT(*) AS CommitCount,
    COUNT(DISTINCT Repo) AS RepoCount,
    AVG(LinesAdded) AS AvgLinesPerCommit
FROM GitLog
WHERE AuthorDate >= DATEADD(month, -6, GETDATE())
GROUP BY DATETRUNC('week', AuthorDate), Author
ORDER BY Week DESC;
```

**解釈**：
- 導入前後でコミット数 の増加 → 生産性向上の兆候（但し質と量は別）
- 留意点：「多い = 高品質」ではない。テストカバレッジと合わせて見る

###### Pull Request マージ時間

```sql
SELECT 
    DATETRUNC('day', MergedDate) AS Day,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY DATEDIFF(hour, CreatedDate, MergedDate)) AS P50_Hours,
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY DATEDIFF(hour, CreatedDate, MergedDate)) AS P95_Hours,
    COUNT(*) AS MergedPRCount
FROM PullRequests
WHERE MergedDate >= DATEADD(month, -6, GETDATE())
GROUP BY DATETRUNC('day', MergedDate)
ORDER BY Day DESC;
```

**期待値**：
- 導入前: P95 = 24-48 時間
- 導入後: P95 = 12-24 時間（リビュー　＆　修正速度向上）

---

##### 2. コード品質メトリクス

###### テスト カバレッジ

```csharp
// Sonarqube / OpenCover から取得
public class CodeQualityMetrics
{
    public decimal OverallCoverage { get; set; }  // 現在 72% → 目標 85%
    public decimal UnitTestCoverage { get; set; }
    public decimal IntegrationTestCoverage { get; set; }
    public int NewTestsAddedPerWeek { get; set; }
}
```

**測定タイミング**：週 1 回（CI/CD パイプイン)
**期待値**：
- 導入前: 60-70%
- 3 ヶ月後: 75-80%
- 6 ヶ月後: 85% 以上

**理由**：Copilot は テストコード生成が得意。テストが増える = カバレッジ↑

###### コード品質スコア

```csharp
public class CodeQualityScore {
    public int CyclomaticComplexity { get; set; }      // 低い方が良い（< 10）
    public int DuplicateLinePercentage { get; set; }   // 低い方が良い（< 5%）
    public int MaintainabilityIndex { get; set; }      // 高い方が良い（> 70）
    public int IssuesCount { get; set; }               // Sonarqube detections
}
```

**ツール**：Sonarqube, Codacy, CodeFactor
**期待値**：
- Cyclomatic Complexity: 6 平均 → 5 以下
- Code Duplication: 10% → 5%
- Issues Count: 100 → 50（重大度別に分析）

###### バグ検出率（リグレッション）

```sql
SELECT 
    DATETRUNC('month', CreatedDate) AS Month,
    Severity,
    COUNT(*) AS BugCount,
    COUNT(CASE WHEN FixedDate IS NOT NULL THEN 1 END) AS FixedCount,
    AVG(DATEDIFF(day, CreatedDate, ISNULL(FixedDate, GETDATE()))) AS AvgDaysToFix
FROM BugTracking
WHERE ProjectId = @ProjectId
  AND CreatedDate >= DATEADD(month, -12, GETDATE())
GROUP BY DATETRUNC('month', CreatedDate), Severity
ORDER BY Month DESC;
```

**期待値**（Copilot 6 ヶ月導入後）：
```
Month    | High | Medium | Low | AvgDaysToFix
2024-02  | 3    | 8      | 5   | 2.1 days      (導入後)
2023-08  | 8    | 25     | 15  | 7.3 days      (導入前)
         |↓50% |↓68%    |↓67% |↓71% 改善
```

---

##### 3. チーム効率メトリクス

###### Code Review コメント数

```sql
SELECT 
    DATETRUNC('week', ReviewDate) AS Week,
    COUNT(*) AS ReviewCommentCount,
    AVG(CommentLength) AS AvgCommentLength,
    COUNT(DISTINCT ReviewerID) AS UniqueReviewers
FROM CodeReviewComments
WHERE ReviewDate >= DATEADD(month, -6, GETDATE())
GROUP BY DATETRUNC('week', ReviewDate)
ORDER BY Week DESC;
```

**解釈**：
- コメント数 ↓ → 「コード品質向上で指摘減」 OR 「レビュー不十分」（他メトリクスで確認）
- コメント内容の質 → 「パターン指摘」から「複雑な設計相談」へシフト（高度化）

###### デプロイ頻度（本番リリース）

```sql
SELECT 
    DATETRUNC('week', DeployDate) AS Week,
    COUNT(DISTINCT DeploymentID) AS DeploymentCount,
    COUNT(DISTINCT RepositoryName) AS RepoCount
FROM Deployments
WHERE DeployDate >= DATEADD(month, -6, GETDATE())
  AND Environment = 'Production'
GROUP BY DATETRUNC('week', DeployDate)
ORDER BY Week DESC;
```

**期待値**：
- 導入前: 2回/週
- 導入後: 5-7回/週（信頼度高いため、デプロイ frequency ↑）

---

#### 🎯 ROI計算：Copilot 導入投資回収

##### 前提

```
Team Size: 10 engineers
Average Salary: $150K/year
Copilot License: $120/person/year
= 合計投資: $1,200/year

メリット期待値（Copilot 活用なら）：
- テスト自動化: 1時間/人/週 短縮
- コードレビュー時間: 30分/人/週 短縮
- バグ修正: 20% 削減（テスト増 + 品質向上）
```

##### ROI 計算

```csharp
public class CopilotROI
{
    // 年間削減時間
    public double TestAutomationSavingsHours => 10 * 50 * 1; // 10人 × 50週 × 1時間
    public double CodeReviewSavingsHours => 10 * 50 * 0.5;   // 10人 × 50週 × 30分
    public double BugFixSavingsHours => 40; // 月 2本バグ削減 × 12ヶ月 × 1.67時間
    
    public double TotalSavingsHours => 
        TestAutomationSavingsHours + CodeReviewSavingsHours + BugFixSavingsHours;
    // = 500 + 250 + 40 = 790 時間
    
    // 金銭価値
    public double HourlyRate => 150_000 / 2000; // $75/hour
    public double TotalSavingsValue => TotalSavingsHours * HourlyRate;
    // = 790 × $75 = $59,250
    
    // ROI
    public double LicenseCost => 1_200;
    public double NetBenefit => TotalSavingsValue - LicenseCost;
    // = $59,250 - $1,200 = $58,050
    
    public double ROI => (NetBenefit / LicenseCost) * 100;
    // = 4,837% → 約 49 倍 ! (初年度)
}
```

**結論**：Copilot 採用で **初年度 ROI 49倍以上**（適切な導入の場合）

---

#### 📈 ダッシュボード事例

##### Weekly Dashboard（各チームが見る）

```markdown
## Week of 2024-02-12

### Productivity
- Commits/person: 4.2 (↑ 12% vs last week)
- PR merge time (P95): 18 hours (↓ 20% vs baseline)
- Code review comments: 23 (stable)

### Quality
- Test coverage: 78% (↑ 3% vs month start)
- Cyclomatic complexity avg: 5.2 (↓ 0.8 vs baseline)
- Production bugs filed: 2 (↓ 75% YoY)

### Copilot Usage
- Active users: 8/10 (↑)
- Chat sessions/person: 3.4
- Code generation usage: 42% of PRs

### Blocked Issues
- None

### Wins This Week
- "Used Copilot Chat to generate 15 unit tests in 30min" - Jane
- "Refactored OrderProcessor with Strategy pattern, coverage 60%→85%" - Bob
```

##### Monthly Executive Summary

```markdown
## Copilot 導入 3ヶ月レポート

### Key Metrics
| Metric | Baseline | Current | Change |
|--------|----------|---------|--------|
| **Test Coverage** | 65% | 78% | +20% |
| **Code Duplication** | 8.2% | 4.5% | -45% |
| **PR Merge Time** | 32h | 18h | -44% |
| **Production Bugs/Month** | 8 | 2 | -75% |
| **Developer Satisfaction** | 3.2/5 | 4.1/5 | +28% |

### ROI Analysis
- **Investment**: $1,200 (licenses)
- **Estimated Savings**: $58,050 (time + quality)
- **ROI**: 4,837%

### Risk Areas
- Code review comments slightly ↓ (monitor quality)
- 2 security issues detected in Copilot code (cf. process improved)

### Q2 Objectives
- [ ] Reach 85% test coverage
- [ ] Reduce cyclomatic complexity to <5 avg
- [ ] Zero security issues in Copilot-generated code

### Next Steps
1. Advanced Copilot training for mid-level team
2. Implement mutation testing to verify test quality
3. Establish code quality KPI for each component
```

---

#### 🔧 測定インフラの構築

##### ツールスタック推奨

```yaml
Version Control:
  - GitHub / GitLab API （コミット、PR メトリクス）

Code Quality:
  - Sonarqube Enterprise （complexity, coverage, duplication）
  - CodeFactor （継続監視）

Metrics Collection:
  - DataDog / New Relic （application performance）
  - Application Insights （Azure）

Dashboard & Visualization:
  - Grafana （dashboards）
  - Tableau / Power BI （executive reports）

Bug Tracking:
  - Jira / Azure DevOps （bug metrics）

Process:
  - Slack Integration （weekly notifications）
  - Automated reports （月 1 回メール）
```

##### データパイプライン例

```
GitHub API
    ↓
Data Lake (SQL Server / Snowflake)
    ↓
Transformation (dbt / stored procedures)
    ↓
Analytics DB (aggregated metrics)
    ↓
Dashboard (Grafana / BI Tool)
    ↓
Alerts (Slack / Email)
```

---

#### ✅ Copilot 効果測定チェックリスト

##### 導入前（ベースライン測定）
- [ ] 現在の テスト coverage を測定・記録
- [ ] バグ検出率を 1 ヶ月分記録
- [ ] PR のマージ平均時間を計測
- [ ] Code quality score を取得（Sonarqube）
- [ ] Developer satisfaction アンケート（5段階）
- [ ] 月間開発時間の内訳把握

##### 導入中（毎週追跡）
- [ ] GitHub Actions で weekly metrics 自動収集
- [ ] Slack チャネルに weekly summary post
- [ ] Sonarqube と coverage ツール連携
- [ ] Manual survey 月 1 回（どう感じてるか）

##### 導入後 3-6 ヶ月（評価）
- [ ] 主要 KPI の改善度を計測
- [ ] ROI を計算
- [ ] リスク領域（セキュリティなど）を確認
- [ ] Executive 向けレポート作成
- [ ] 継続投資判断

---

#### 📚 参考

- [DORA Metrics](https://dora.dev/) （DevOps Research & Assessment）
- [Sonarqube Documentation](https://docs.sonarqube.org/)
- [Engineering Metrics](https://www.microsoft.com/en-us/research/publication/engineering-metrics/)

## Copilot Instructions Template {#section-05-team-operations-04-copilot-instructions-template}

#### プロジェクト固有の Copilot ガイドラインをテンプレートして、全チームで統一的に使う

`.github/copilot-instructions.md` は Copilot AI が参照する「このプロジェクトのルール」。チームのコーディング標準、設計パターン、セキュリティ要件を明示することで、生成されるコードの品質を飛躍的に向上させます。

---

#### 📋 テンプレート全体

完全なテンプレートを以下に示します。プロジェクト特性に合わせてカスタマイズしてください。

---

````markdown
# GitHub Copilot Instructions

このドキュメントは、このプロジェクトで Copilot を使用する際の指針です。
AI の提案がこれらのルールに従うように指定します。

## プロジェクト概要

### Project Identity
- **Name**: E-Commerce Backend Platform
- **Tech Stack**: 
  - Runtime: .NET 8
  - Framework: ASP.NET Core
  - ORM: Entity Framework Core 8
  - Database: SQL Server 2022
  - Message Queue: RabbitMQ
  - Cache: Redis

### Team
- **Size**: 10 engineers
- **Experience Level**: Mix (2 juniors, 5 mid-level, 3 seniors)
- **Code Review**: Required before merge

### Target Metrics
- Test Coverage: ≥ 80%
- Code Duplication: < 5%
- Avg Cyclomatic Complexity: < 8
- P95 API Response Time: < 500ms

---

## コーディング基準

### 言語と命名規則

#### Classes & Interfaces
```csharp
// ✅ Correct
public interface IOrderService { }
public class OrderService : IOrderService { }
public class CreateOrderCommand { }
public record OrderDto(int Id, string CustomerName);

// ❌ Incorrect
public interface OrderService { }
public class Order_Service { }
public class order_service { }
```

#### Methods & Parameters
```csharp
// ✅ Correct
public async Task<OrderResult> ProcessOrderAsync(int orderId)
public bool IsValidPaymentMethod(PaymentMethodType method)
public void LogOrderCreated(Order order)

// ❌ Incorrect
public async Task<OrderResult> process_order(int orderId)
public bool checkPayment(PaymentMethodType method)
public void LogOrder(Order o)
```

#### Variables & Fields
```csharp
// ✅ Correct
private readonly IOrderRepository _orderRepository;
private decimal totalAmount;
const int MaxRetries = 3;

// ❌ Incorrect
private IOrderRepository OrderRepository;
private decimal total_amount;
const int max_retries = 3;
```

---

## アーキテクチャとデザインパターン

### 層構造（Clean Architecture）

```
┌─────────────────────────┐
│  Presentation Layer     │  Controllers, DTOs, API Contracts
├─────────────────────────┤
│  Application Layer      │  Services, Use Cases, Validation
├─────────────────────────┤
│  Domain Layer           │  Entities, Value Objects, Interfaces
├─────────────────────────┤
│  Infrastructure Layer   │  DB Access, External Services, Config
└─────────────────────────┘
```

**重要**: 層の依存関係は上からのみ。下層は上層に依存しない (Dependency Inversion)

### 必須デザインパターン

#### 1. Repository Pattern（すべてのデータアクセス）

```csharp
// ✅ Always: Interface definition first
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
    Task SaveChangesAsync();
}

// ✅ Implementation
public class OrderRepository : IRepository<Order>
{
    private readonly AppDbContext _context;
    
    public async Task<Order> GetByIdAsync(int id)
    {
        return await _context.Orders
            .AsNoTracking()
            .FirstOrDefaultAsync(o => o.Id == id);
    }
    
    // ... 他メソッド
}

// ❌ Never: Direct DbContext access in services
// 悪い例：DbContext を Service に inject
```

#### 2. Strategy Pattern（複数実装が必要な場合）

このプロジェクトでは以下が Strategy 候補：
- **Payment Processing**: CreditCard, PayPal, BankTransfer
- **Shipping Methods**: Standard, Express, International
- **Notification**: Email, SMS, Push

```csharp
// Strategy interface
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request);
    string PaymentMethodName { get; }
}

// Strategy implementations
public class CreditCardPaymentStrategy : IPaymentStrategy
{
    // ...
}

// Strategy factory
public class PaymentStrategyFactory
{
    public IPaymentStrategy GetStrategy(PaymentMethodType method)
    {
        return method switch
        {
            PaymentMethodType.CreditCard => new CreditCardPaymentStrategy(...),
            PaymentMethodType.PayPal => new PayPalPaymentStrategy(...),
            _ => throw new NotSupportedException()
        };
    }
}

// Usage: Never if-else chains
public class OrderService
{
    public async Task<OrderResult> ProcessOrderAsync(Order order)
    {
        var strategy = _paymentFactory.GetStrategy(order.PaymentMethod);
        var result = await strategy.ProcessPaymentAsync(...);
        // ...
    }
}
```

#### 3. Dependency Injection（always）

Constructor injection ONLY. No Service Locator pattern.

```csharp
// ✅ Correct: Constructor injection
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentService _paymentService;
    private readonly ILogger<OrderService> _logger;
    
    public OrderService(
        IOrderRepository orderRepository,
        IPaymentService paymentService,
        ILogger<OrderService> logger)
    {
        _orderRepository = orderRepository;
        _paymentService = paymentService;
        _logger = logger;
    }
}

// ❌ Incorrect: Service Locator
public class OrderService
{
    private IOrderRepository _repository = 
        ServiceLocator.GetService<IOrderRepository>();
}
```

---

## エラーハンドリングとロギング

### 必須のエラーハンドリング

すべての public async method は try-catch-finally が必須：

```csharp
public async Task<ApiResponse<OrderDto>> CreateOrderAsync(CreateOrderRequest request)
{
    try
    {
        // 1. Input validation
        if (request == null || string.IsNullOrWhiteSpace(request.CustomerName))
        {
            _logger.LogWarning("CreateOrder called with invalid input");
            return ApiResponse<OrderDto>.BadRequest("Customer name is required");
        }
        
        // 2. Business logic
        _logger.LogInformation("Creating order for customer: {CustomerName}", request.CustomerName);
        
        var order = new Order
        {
            CustomerName = request.CustomerName,
            CreatedDate = DateTime.UtcNow
        };
        
        await _orderRepository.AddAsync(order);
        await _orderRepository.SaveChangesAsync();
        
        // 3. Success response
        _logger.LogInformation("Order created successfully: {OrderId}", order.Id);
        return ApiResponse<OrderDto>.Success(
            new OrderDto(order.Id, order.CustomerName));
    }
    catch (InvalidOperationException ex)
    {
        _logger.LogError(ex, "Invalid operation during order creation");
        return ApiResponse<OrderDto>.BadRequest(ex.Message);
    }
    catch (DbUpdateException ex)
    {
        _logger.LogError(ex, "Database error during order creation");
        return ApiResponse<OrderDto>.InternalError("Failed to save order");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Unexpected error during order creation");
        return ApiResponse<OrderDto>.InternalError("An unexpected error occurred");
    }
}
```

### ロギング規則

```csharp
// ✅ Correct logging patterns
_logger.LogInformation("User login attempt: {UserId}", userId);
_logger.LogWarning("Retry attempt {RetryCount}/{MaxRetries} for operation", 
    currentRetry, maxRetries);
_logger.LogError(exception, "Failed to process payment for order {OrderId}", orderId);

// ❌ Avoid
_logger.LogInformation($"User login: {userId}"); // Use structured logging
_logger.LogInformation("Failed to process"); // No context
```

---

## テスト基準

### 単体テスト (xUnit)

```csharp
[TestClass]
public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _mockRepository;
    private readonly Mock<IPaymentService> _mockPayment;
    private readonly ILogger<OrderService> _logger;
    private readonly OrderService _sut; // System Under Test
    
    public OrderServiceTests()
    {
        _mockRepository = new Mock<IOrderRepository>();
        _mockPayment = new Mock<IPaymentService>();
        _logger = new Mock<ILogger<OrderService>>().Object;
        _sut = new OrderService(_mockRepository.Object, _mockPayment.Object, _logger);
    }
    
    // Test naming: [MethodName]_[Scenario]_[ExpectedResult]
    [Theory]
    [InlineData(100, true)]    // Normal case
    [InlineData(0, false)]     // Edge case
    [InlineData(-1, false)]    // Invalid case
    public async Task ProcessOrder_WithAmount_ReturnsExpectedResult(
        decimal amount, bool shouldSucceed)
    {
        // Arrange
        var order = new Order { Id = 1, Amount = amount };
        _mockRepository.Setup(r => r.GetByIdAsync(1))
            .ReturnsAsync(order);
        
        // Act
        var result = await _sut.ProcessOrderAsync(1);
        
        // Assert
        if (shouldSucceed)
        {
            Assert.True(result.Success);
        }
        else
        {
            Assert.False(result.Success);
        }
    }
}
```

### テストカバレッジ要件

- **Target**: ≥ 80% overall coverage
- **Focus**: Business logic layer (90%+)
- **Data access**: Integration tests OK
- **UI/API endpoint**: E2E tests OK

---

## パフォーマンス要件

### データベースクエリ最適化

```csharp
// ❌ Bad: N+1 query problem
var orders = await _context.Orders.ToListAsync();
foreach (var order in orders)
{
    order.Customer = await _context.Customers
        .FirstOrDefaultAsync(c => c.Id == order.CustomerId);
}

// ✅ Good: Single query with Include
var orders = await _context.Orders
    .Include(o => o.Customer)
    .AsNoTracking()  // For read-only queries
    .ToListAsync();

// ✅ Good: Projection (最小限のカラムのみ取得)
var orderSummaries = await _context.Orders
    .Select(o => new OrderSummaryDto
    {
        Id = o.Id,
        CustomerName = o.Customer.Name,
        Total = o.Total
    })
    .ToListAsync();
```

### API レスポンスタイム Targets

| Endpoint Type | P50 | P95 | P99 |
|--------------|-----|-----|-----|
| GET (simple) | 50ms | 100ms | 200ms |
| GET (with joins) | 100ms | 300ms | 500ms |
| POST (create) | 200ms | 500ms | 1000ms |
| Batch/Report | 1s | 5s | 10s |

### Caching Strategy

```csharp
// Redis cache for reference data (5-min TTL)
public async Task<List<ShippingMethod>> GetShippingMethodsAsync()
{
    const string cacheKey = "shipping_methods";
    const int cacheDurationMinutes = 5;
    
    if (_cache.TryGetValue(cacheKey, out List<ShippingMethod> cached))
    {
        return cached;
    }
    
    var methods = await _repository.GetShippingMethodsAsync();
    _cache.Set(cacheKey, methods, TimeSpan.FromMinutes(cacheDurationMinutes));
    return methods;
}
```

---

## セキュリティ要件

### Mandatory Security Checks

```csharp
// ❌ NEVER: Hardcoded credentials
const string connectionString = "Server=...;Password=admin123";

// ✅ ALWAYS: Environment variables / Secrets
var connectionString = _configuration.GetConnectionString("DefaultConnection");

// ❌ NEVER: SQL concatenation (SQL Injection risk)
var sql = $"SELECT * FROM Orders WHERE Id = {orderId}";

// ✅ ALWAYS: Parameterized queries
var order = await _context.Orders
    .FromSqlInterpolated($"SELECT * FROM Orders WHERE Id = {orderId}")
    .FirstOrDefaultAsync();

// ❌ NEVER: Trust user input
public IActionResult Search(string query)
{
    return Ok(_context.Products
        .Where(p => p.Name.Contains(query))  // Can contain XSS
        .ToList());
}

// ✅ ALWAYS: Validate & sanitize
public IActionResult Search(string query)
{
    if (string.IsNullOrWhiteSpace(query) || query.Length > 100)
        return BadRequest("Invalid search query");
    
    return Ok(_context.Products
        .Where(p => EF.Functions.Like(p.Name, $"%{query}%"))
        .ToList());
}
```

### Authentication & Authorization

```csharp
// ✅ Check authorization attribute on all protected endpoints
[Authorize(Roles = "Admin")]
[HttpDelete("orders/{id}")]
public async Task<IActionResult> DeleteOrder(int id)
{
    // ...
}

// ✅ Validate user ownership
[HttpPut("orders/{id}")]
public async Task<IActionResult> UpdateOrder(int id, UpdateOrderRequest request)
{
    var order = await _orderRepository.GetByIdAsync(id);
    if (order.UserId != User.GetUserId())
    {
        return Forbid();  // 403 Forbidden
    }
    // ...
}
```

---

## 外部サービス・API 連携

### Resilience Patterns

```csharp
// Use Polly for retry + circuit breaker
public class PaymentService
{
    public PaymentService(HttpClient httpClient, IAsyncPolicy<HttpResponseMessage> policy)
    {
        _httpClient = httpClient;
        _policy = policy;
    }
    
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
    {
        try
        {
            var response = await _policy.ExecuteAsync(
                () => _httpClient.PostAsJsonAsync("/api/process-payment", request));
            
            if (!response.IsSuccessStatusCode)
            {
                _logger.LogError("Payment API error: {StatusCode}", response.StatusCode);
                return PaymentResult.Failed("Payment processing failed");
            }
            
            return PaymentResult.Success(...);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Payment service unavailable");
            return PaymentResult.Failed("Payment service unavailable");
        }
    }
}

// Startup configuration
services.AddHttpClient<IPaymentService, PaymentService>()
    .AddTransientHttpErrorPolicy(p => p.WaitAndRetryAsync(
        retryCount: 3,
        sleepDurationProvider: retryAttempt =>
            TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))));
```

---

## Comment-Driven Development

与えられた機能要全成を書く際は、まず詳細なコメントを書いてから実装をする。

```csharp
// ✅ Always: Detailed comments describe intent
/// <summary>
/// 注文の支払いを処理する
/// 1. 注文を DB から読み込む
/// 2. 有効性チェック（金額 > 0, 顧客存在）
/// 3. 支払いサービスで処理
/// 4. 成功時: 注文ステータスを Paid に更新、確認メール送信
/// 5. 失敗時: エラーログ + エラーレスポンス返却
/// </summary>
public async Task<OrderResult> ProcessPaymentAsync(int orderId)
{
    // Implementation...
}

// Then Copilot will generate better code based on intent
```

---

## Copilot Usage Guidelines

### ✅ Do
- Use Copilot for boilerplate code (Repository implementations, DTOs)
- Use Chat for architecture questions
- Use Inline Edits for refactoring
- Verify all security-sensitive code
- Run tests before merge

### ❌ Don't
- Trust Copilot blind (review all suggestions)
- Use for complex business logic without understanding
- Copy-paste without testing
- Ignore security warnings
- Skip code review because Copilot generated it

---

## 参考リソース

- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Entity Framework Core Best Practices](https://learn.microsoft.com/en-us/ef/core/performance/)
- [.NET Security Best Practices](https://learn.microsoft.com/en-us/dotnet/standard/security/)

---

## Questions?

Ask in #copilot-questions on Slack
````

---

#### 📝 カスタマイズのポイント

実際に使用する際には、以下を調整してください：

1. **Tech Stack**: Rails, Python, Node.js など別言語なら全部置き換え
2. **Team Size / Experience**: 小さく focused なチームなら簡潔に
3. **デザインパターン**: 自チームで標準化しているパターンのみ記載
4. **パフォーマンス基準**: e-commerce（このテンプレート）と遠く管理画面でも異なる
5. **セキュリティ**: PCI-DSS など compliance 要件に合わせる

---

#### ✅ Copilot Instructions 導入チェックリスト

- [ ] `.github/copilot-instructions.md` ファイルを作成
- [ ] チーム全体で内容をレビュー + approve
- [ ] デフォルトブランチに merge
- [ ] すべてのエンジニアが `.github/copilot-instructions.md` を Copilot に教えた
- [ ] 最初 month は遵守状況を code review で確認
- [ ] 3 ヶ月ごとに更新・改善を協議

---

#### 🔗 参考テンプレート

- [GitHub Copilot Documentation](https://docs.github.com/en/copilot/using-github-copilot/prompt-engineering)
- [Awesome Copilot Instructions](https://github.com/search?q=copilot-instructions.md)

# Reference {#chapter-06-reference}

## Troubleshooting {#section-06-reference-01-troubleshooting}

#### よくあるエラーと解決策を索引形式で整理

Copilot や C# 開発での困った場合、ここを検索してください。

---

#### 🔍 問題インデックス

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

#### Copilot が提案してくれない

##### 症状
```
コメントを書いたのに Copilot が補完提案を出さない
Ctrl+Enter して Chat 開いても無反応
```

##### 原因と対策

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

#### Copilot 提案が外れてる

##### 症状
```csharp
// Copilot がこう生成した
var result = await service.DoSomething();

// 但し実際には…
// - DoSomething は存在しない（DoSomethingElse が正）
// - 返り値の型が違う
// - Exception handle が必要
```

##### 原因と対策

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

#### テストが赤になる

##### 症状
```
[xUnit] Test failed: AssertionError
実装コードは「できてる」ようなのにテストが失敗
```

##### 原因と対策

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

#### NullReferenceException

##### 症状
```
System.NullReferenceException: Object reference not set 
to an instance of an object.
```

##### 原因と対策

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

#### N+1 クエリ問題

##### 症状
```
データベースクエリが大量発行
- 初回: 1 クエリ（顧客取得）
- ループで N クエリ（各顧客ごとの注文取得）
= 合計 N+1 クエリ
```

##### 原因と対策

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

##### SQL Plan で確認

```sql
-- SQL Server Management Studio で実行計画確認
-- 1 query のはず（複数の JOIN が見えるのは OK）
SELECT * FROM Customers c
LEFT JOIN Orders o ON c.Id = o.CustomerId
```

---

#### パフォーマンス低下

##### 症状
```
API が遅い（1秒以上）
Response time が P95: 2000ms 超えてる
```

##### 原因と対策

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

#### セキュリティ警告

##### 症状
```
SonarQube / Copilot: "SQL Injection 脆弱性"
"入力未検証"
```

##### 原因と対策

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

#### Unit Test カバレッジ不足

##### 症状
```
Code Coverage: 45% (目標 80%)
どのメソッドをテストればいい？
```

##### 原因と対策

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

#### Code Review 指摘多発

##### 症状
```
PR を出す
→ 10 個の指摘を受ける
→ 毎回同じ指摘ばかり
```

##### 原因と対策

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

#### Async 関連エラー

##### 症状
```
deadlock が発生
別スレッドから呼び出すと hang する
```

##### 原因と対策

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

#### 📞 さらに困った場合

1. **Slack**: #copilot-questions に質問
2. **Wiki**: Team wiki で past solutions を検索
3. **Copilot Chat**: 「エラーメッセージ [ここに貼り付け]、原因と対策」
4. **Stack Overflow**: Tag `csharp` と `entity-framework` で検索

---

#### ✅ トラブルシューティング習慣

- [ ] Google / Stack Overflow で「エラーメッセージ + 言語」で検索
- [ ] 根本原因を Copilot Chat で確認
- [ ] テスト駆動で修正し、回帰防止
- [ ] Team wiki に「今日の学び」を記録

## Quick Reference {#section-06-reference-02-quick-reference}

#### よく使うコマンド、パターン、キーボードショートカットを素早く参照

Ctrl+F で検索して使用。

---

#### ⌨️ キーボードショートカット

##### Copilot（全 IDE）

| 操作 | VS Code | Visual Studio |
|------|---------|---------------|
| Inline Completions 表示 | `Ctrl+I` | `Ctrl+\` |
| Chat ウィンドウ | `Ctrl+Shift+I` | `Ctrl+\` (then Chat tab) |
| 次の提案 | `Ctrl+]` | `Alt+]` |
| 前の提案 | `Ctrl+[` | `Alt+[` |
| 拒否（Dismiss） | `Escape` | `Escape` |

##### Visual Studio 2022

| 操作 | ショートカット |
|------|----------------|
| Solution Explorer | `Ctrl+Alt+L` |
| Debug Start/Stop | `F5` / `Shift+F5` |
| Build | `Ctrl+Shift+B` |
| Run Tests | `Ctrl+R, T` |
| Go to Definition | `F12` |
| Find All References | `Shift+F12` |
| Rename | `Ctrl+R, R` or `F2` |
| Format Document | `Ctrl+K, Ctrl+D` |
| Format Selection | `Ctrl+K, Ctrl+F` |

##### VS Code

| 操作 | ショートカット |
|------|----------------|
| Command Palette | `Ctrl+Shift+P` |
| Find | `Ctrl+F` |
| Replace | `Ctrl+H` |
| Go to Definition | `F12` |
| Peek Definition | `Alt+F12` |
| Terminal | `Ctrl+` |
| Run Task | `Ctrl+Shift+P` → Run Task |

---

#### 💬 Copilot Chat よく使うプロンプト

##### コード理解

```
このメソッドが何をしているか説明して

[メソッドコードを貼り付け]
```

##### リファクタリング

```
このコード、SOLID 原則に従うようにリファクタリングして

[コード]
```

##### テスト生成

```
このクラスの包括的な xUnit テストを生成してください
正常系、異常系、境界値をカバーしてください

[クラスコード]
```

##### バグ調査

```
このエラー、原因と対策を教えてください

[エラースタックトレース + コード]
```

##### パフォーマンス診断

```
このクエリが遅い理由は？最適化案を教えてください

[LINQ or SQL]
```

##### セキュリティ見直し

```
このコード、セキュリティ脆弱性を指摘してください

[コード]
```

##### パターン相談

```
複数の支払い方法（CreditCard, PayPal, BankTransfer）に対応する場合、
どのデザインパターンが適切ですか？実装例も示してください
```

---

#### 🎯 よく使う C# パターン

##### 非同期処理

```csharp
// 基本形
public async Task<T> GetDataAsync<T>(int id)
{
    var data = await _repository.GetByIdAsync(id);
    return data;
}

// 複数 Task の並列実行
await Task.WhenAll(
    task1,
    task2,
    task3
);

// Task チェーン
var result = await _service.GetAsync()
    .ContinueWith(t => ProcessAsync(t.Result))
    .Unwrap();
```

##### エラーハンドリング

```csharp
try
{
    // リスク操作
}
catch (InvalidOperationException ex)
{
    _logger.LogError(ex, "Invalid operation");
    throw;  // 再スロー
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error");
    return BadRequest("エラーが発生しました");
}
finally
{
    // Cleanup
}
```

##### Null チェック

```csharp
// Null-coalescing
var name = user?.Name ?? "Unknown";

// Null-conditional
var email = customer?.Email;  // null if customer is null

// Null-coalescing assignment
user ??= new User();

// Pattern matching
if (order is { Status: OrderStatus.Pending, Amount: > 1000 })
{
    // Order is pending and amount > 1000
}
```

##### LINQ パターン

```csharp
// Filter, Map, Reduce
var result = _context.Orders
    .Where(o => o.Amount > 100)           // Filter
    .Select(o => o.CustomerName)          // Map
    .Distinct()
    .ToList();

// GroupBy
var grouped = _context.Orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new { 
        CustomerId = g.Key, 
        Count = g.Count(), 
        Total = g.Sum(o => o.Amount) 
    })
    .ToList();

// Join
var joined = _context.Orders
    .Join(_context.Customers,
        o => o.CustomerId,
        c => c.Id,
        (o, c) => new { o.Id, c.Name, o.Amount })
    .ToList();

// Pagination
var page2 = _context.Orders
    .OrderByDescending(o => o.CreatedDate)
    .Skip(10)
    .Take(10)
    .ToListAsync();
```

##### 依存性注入

```csharp
// Startup Register (Program.cs / Startup.cs)
services.AddScoped<IOrderService, OrderService>();
services.AddScoped<IOrderRepository, OrderRepository>();
services.AddSingleton<ICache, MemoryCache>();

// Constructor Injection
public class OrderService
{
    public OrderService(IOrderRepository repo, ILogger<OrderService> logger)
    {
        // ...
    }
}
```

##### Strategy パターン

```csharp
// Interface
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessAsync(PaymentRequest req);
}

// Implementations
public class CreditCardPaymentStrategy : IPaymentStrategy { }

// Factory
public class PaymentFactory
{
    public IPaymentStrategy GetStrategy(string method) =>
        method switch
        {
            "CreditCard" => new CreditCardPaymentStrategy(),
            "PayPal" => new PayPalPaymentStrategy(),
            _ => throw new NotSupportedException()
        };
}

// Usage
var strategy = _factory.GetStrategy(order.PaymentMethod);
var result = await strategy.ProcessAsync(request);
```

##### Unit Test テンプレート

```csharp
public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _mockRepo;
    private readonly OrderService _sut;
    
    public OrderServiceTests()
    {
        _mockRepo = new Mock<IOrderRepository>();
        _sut = new OrderService(_mockRepo.Object);
    }
    
    [Theory]
    [InlineData(100, true)]
    [InlineData(0, false)]
    public async Task ProcessOrder_WithAmount_ReturnsExpected(
        decimal amount, bool expected)
    {
        // Arrange
        var order = new Order { Amount = amount };
        
        // Act
        var result = await _sut.ProcessOrderAsync(order);
        
        // Assert
        Assert.Equal(expected, result.Success);
    }
}
```

---

#### 📊 Entity Framework Core パターン

##### Context Setup

```csharp
// DbContext
public class AppDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    public DbSet<Customer> Customers { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer(_connectionString);
    }
}

// Register (Program.cs)
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(configuration.GetConnectionString("Default")));
```

##### Querying

```csharp
// Simple Query
var order = await _context.Orders.FindAsync(orderId);

// Filter
var orders = await _context.Orders
    .Where(o => o.Status == OrderStatus.Pending)
    .ToListAsync();

// Include (Eager Load)
var orders = await _context.Orders
    .Include(o => o.Customer)
    .Include(o => o.Items)
    .ToListAsync();

// Select (Projection)
var orderSummary = await _context.Orders
    .Select(o => new OrderDto
    {
        Id = o.Id,
        Total = o.Total,
        Customer = o.Customer.Name
    })
    .ToListAsync();

// AsNoTracking (Read-only optimization)
var orders = await _context.Orders
    .AsNoTracking()
    .ToListAsync();
```

##### Mutations

```csharp
// Create
var order = new Order { ... };
_context.Orders.Add(order);
await _context.SaveChangesAsync();

// Update
var order = await _context.Orders.FindAsync(id);
order.Status = OrderStatus.Paid;
await _context.SaveChangesAsync();

// Delete
_context.Orders.Remove(order);
await _context.SaveChangesAsync();

// Bulk Operations (via Query)
await _context.Orders
    .Where(o => o.CreatedDate < DateTime.Now.AddYears(-1))
    .ExecuteDeleteAsync();
```

---

#### 🔐 セキュリティ チェックリスト

Every PR Review:

- [ ] SQL Injection? (Parameterized queries?)
- [ ] XSS? (Input sanitization?)
- [ ] CORS? (Whitelist origins only)
- [ ] Authentication? ([Authorize] attrs?)
- [ ] Secrets in code? (Use appsettings/environment)
- [ ] Sensitive logging? (Avoid PII in logs)
- [ ] Error messages? (Don't expose internals)

---

#### 📈 パフォーマンス チェックリスト

Every Performance Critical Code:

- [ ] N+1 queries? (Include/Projection?)
- [ ] .ToList() at wrong place?
- [ ] Caching implemented?
- [ ] Connection pooling?
- [ ] Indexes on filtered columns?
- [ ] Async all the way?
- [ ] ConfigureAwait(false) in libraries?

---

#### 🧪 テストキー チェックリスト

Every Unit Test:

- [ ] Happy path 1 + Edge1 + Error1 minimum?
- [ ] Mocks configured?
- [ ] Verification of calls?
- [ ] No shared state between tests?
- [ ] Fast (< 100ms per test)?
- [ ] Isolated from external dependencies?

---

#### 📝 ドキュメント テンプレート

##### XMLDoc (Public methods)

```csharp
/// <summary>
/// 指定 ID の注文を取得
/// </summary>
/// <param name="orderId">注文 ID</param>
/// <returns>注文オブジェクト、見つからない場合は null</returns>
/// <exception cref="InvalidOperationException">ID が無効な場合</exception>
public async Task<Order> GetOrderAsync(int orderId)
{
    // ...
}
```

##### Code Comment (Complex logic)

```csharp
// Strategy パターンで支払い方法を選択
// 各戦略は IPaymentStrategy を実装し、
// Factory が適切なインスタンスを返す
var strategy = _paymentFactory.GetStrategy(order.PaymentMethod);
```

---

#### 🚀 チームオンボーディングチェック

New Team Member Quick Links:

1. Read: [01-foundations/01-basics.md](#section-01-foundations-01-basics)
2. Setup: IDE + Copilot extension
3. Learn: [02-core-features/](#chapter-02-core-features)
4. Code: Pair programming on simple bug fix
5. Test: Run `dotnet test`
6. Review: Submit PR, get feedback
7. Culture: Attend team standup

---

#### 🔗 外部リソース

- [C# Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/)
- [Async Best Practices](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
- [Refactoring.Guru Patterns](https://refactoring.guru/design-patterns/csharp)
- [xUnit Documentation](https://xunit.net/docs/getting-started/netcore)

## Feature BY Role Matrix {#section-06-reference-03-feature-by-role-matrix}

#### 各ロールが「どの Copilot 機能」を「どんなシーンで」使うかのマトリックス

---

#### 📊 全体マトリックス

```
              Inline    Chat      Inline    Agent
ロール        完成推定   説明・質問 Edits    Mode
─────────────────────────────────────────────────
初級者         ⭐⭐    ⭐⭐      ⭐        -
中級者         ⭐⭐⭐  ⭐⭐⭐   ⭐⭐      ⭐
上級者/Lead   ⭐⭐    ⭐⭐⭐   ⭐⭐⭐    ⭐⭐
QA エンジ     ⭐      ⭐⭐     ⭐⭐      ⭐⭐⭐
```

---

#### 🎯 ロール別 詳細マッピング

##### 初級開発者（Beginner）

| 機能 | 使用頻度 | 用途例 | Tips |
|------|---------|--------|------|
| **Inline Completions** | 毎日 | 変数名補完、基本構文 | コメント詳細に |
| **Chat** | 毎日 | 「このコード何してる？」「いくらやり方は？」 | 「初級者なので優しく explain」 |
| **Inline Edits** | 週 1-2 | 「このコード簡潔に」（簡単なリファクタ） | 結果をレビューしてから採用 |
| **Agent Mode** | 使用禁止 | - | 理解不十分→危ない |

**典型的なワークフロー**：
```
朝「やることリスト」確認
→ 基本タスク assign
→ コメント書く
→ Copilot で補完
→ Manual テストで動作確認
→ Code Review for learning
```

---

##### 中級開発者（Mid-Level）

| 機能 | 使用頻度 | 用途例 | Tips |
|------|---------|--------|------|
| **Inline Completions** | 毎日 | Repository implementations, DTOs | チェック結果対応（S**テストで verify） |
| **Chat** | 毎日 | パターン選定、リファクタ戦略、難しいテスト | 「XYZ パターンで実装したい」と明示 |
| **Inline Edits** | 毎日 | 最適化、複数箇所同時リファクタリング | Manual verify 後に merge |
| **Agent Mode** | 週 1-2 | 小さなテスト作成、ドキュメント生成 | 指示を明確に。結果は review 必須 |

**典型的なワークフロー**：
```
スプリント開始
→ テスト駆動で interface 設計
→ Copilot で Strategy/Repository 実装
→ 複雑ロジックは Chat で相談
→ テスト coverage 80%+
→ Code Review（品質重点）
```

---

##### シニアリード（Senior/Lead）

| 機能 | 使用頻度 | 用途例 | Tips |
|------|---------|--------|------|
| **Inline Completions** | 週 1-2 | Example code 作成、複雑な変数操作 | 「正しいパターン」の examples として |
| **Chat** | 毎日 | アーキテクチャ検討、チーム標準化、レビュー相談 | 「我が社の context」を明示 |
| **Inline Edits** | 週 1 | 大規模リファクタリング（複数ファイル） | Draft as AI → Human review + refine |
| **Agent Mode** | 週数回 | テスト自動化サポート、ジュニア向けドキュメント生成 | チーム教育 leverage point |

**典型的なワークフロー**：
```
チーム会
→ 「新機能の設計パターンは？」を Copilot に相談
→ チーム標準を更新 (copilot-instructions.md)
→ ジュニア向け学習教材を Agent Mode で生成
→ Code Review リード（パターン・セキュリティ確認）
→ Metrics 追跡 + continuous improvement
```

---

##### QA エンジニア

| 機能 | 使用頻度 | 用途例 | Tips |
|------|---------|--------|------|
| **Inline Completions** | - | - | 使用せず（コード書かない） |
| **Chat** | 毎日 | テストシナリオ相談、バグ分析手法 | 「このバグ、どう分類する？」 |
| **Inline Edits** | 日 few | テストコード微調整 | テスト生成後の手直し |
| **Agent Mode** | 毎日 | テストコード自動生成、テスト matrix 作成 | **QA の生産性向上の key** |

**典型的なワークフロー**：
```
テスト計画
→ Agent Mode で「正常系・異常系・境界値」テスト自動生成
→ 手動テストケース整理
→ Mutation Testing で テスト品質確認
→ Metrics report（カバレッジ、バグ検出率）作成
```

---

#### 🎓 シーン別推奨フロー

##### シーン 1: "新しい API エンドポイント作成"

**初級**:
```
1. Comment で「ユーザーを ID で取得する GET endpoint」
2. Copilot が補完
3. Chat で「ビジネスロジック」確認
4. Test Manual （Postman）
5. Code Review
```

**中級**:
```
1. Interface / DTO 設計（自分で）
2. Repository 実装 → Copilot で補完
3. Service 層 → Strategy パターン Chat で相談
4. Unit Test → Agent Mode で生成
5. Code Review（設計重点）
```

**上級**:
```
1. チーム会「新 API 設計パターンは？」→ Copilot 相談
2. チーム標準更新（copilot-instructions.md）
3. ジュニア向け「こうやって実装しちゃう」Example を Agent Mode で生成
4. ジュニアに assign → Copilot 活用してね → Code Review
```

---

##### シーン 2: "バグ修正＋テスト追加"

**初級**:
```
1. バグ説明を読む
2. Chat「なぜこんなバグが起きるのか」
3. Copilot が explanation + fix 提案
4. Test Manual で確認
5. PR / Code Review
```

**中級**:
```
1. Stack trace 分析 → Copilot Chat で根本原因追跡
2. Unit Test ケース 思いつく → Chat でテスト案生成
3. Inline Edits で複数テストケース一括生成
4. Run tests → Coverage 確認
5. Code Review
```

**QA**:
```
1. バグ ticket から「テスト coverage gap」確認
2. Agent Mode で「このバグを catch する」テスト生成
3. Regression test suite に追加
4. Mutation testing で「テスト本当に検出できる？」確認
```

---

#### 📈 効率化の「鍵」

##### 初級 → 中級への jump

- [ ] Copilot Chat で設計パターン相談できるか
- [ ] Strategy / Repository 実装を Copilot で完了できるか
- [ ] テストコード generation を活用できるか

**Growth Metric**: 
- 同じタスク時間 -30%
- テストカバレッジ +20%
- Code review comment -40%（品質向上）

---

##### 中級 → 上級への jump

- [ ] API アーキテクチャを「設計 → チーム教育」に転換か
- [ ] Copilot を「ジュニア教育 tool」として活用できるか
- [ ] メトリクス driven で continuous improvement できるか

**Growth Metric**:
- チーム生産性 +50%
- 技術負債 reduction
- Onboarding time -60%（新人育成効率）

---

#### 💡 共通注意点

##### すべてのロール

- ✅ Copilot は「助手」（盲信するな）
- ✅ テストで verify してから採用
- ✅ セキュリティ手動確認必須（especially Copilot code）
- ✅ チーム標準 (`.github/copilot-instructions.md`) を教える
- ✅ 「Copilot が何をしたか」理解してから進める

##### Red Flags

- ❌ 「Copilot が生成したから OK」（No verification）
- ❌ テストなし で merge（AI code でも必須）
- ❌ チーム標準を無視した自由なパターン混在
- ❌ ジュニアが complete dependency（理解なし） → 長期的に危ない

---

#### 📊 チーム全体効果期待値

| Metric | Timeline | 初級→効果 | 中級→効果 | 上級→効果 |
|--------|----------|---------|---------|---------|
| **生産性** | 3m | +20% | +40% | +60% |
| **コード品質** | 3m | +15% | +30% | +25% |
| **テストカバレッジ** | 6m | +10% | +25% | +20% |
| **オンボーディング** | 6m | -30% | -50% | -70% |
| **チーム満足度** | 1m | +3.0 | +3.5 | +4.0 |

（5段階スケール）

---

#### 🚀 推奨導入順序

**Phase 1 (Week 1-2: 全員)**
- Copilot インストール
- [01-foundations/](#chapter-01-foundations) 学習
- Inline Completions で comment-driven dev 試す

**Phase 2 (Week 3-4: 初級+ 中級)**
- Chat を活用（パターン質問）
- Inline Edits で簡単リファクタリング
- テストコード generation trial

**Phase 2 Continues (Month 2+: 中級+ 上級)**
- Agent Mode て自動化
- `.github/copilot-instructions.md` チューニング
- チーム教育 leverage point として活用

---

#### 📞 Questions by Role

##### 初級: 「Copilot が提案したコード、信頼していい？」

A: **必ず verify してください**。
- テストを実行
- 「おかしい」と感じたら Chat で質問
- メンターに見てもらう

##### 中級: 「どこまで Copilot に任せる？」

A: **設計は自分で、実装は Copilot** の配分がベスト。
- Interface 設計 → 自分
- Repository 実装 → Copilot
- テストコード → Copilot + review

##### 上級: 「Copilot で何が変わる？」

A: **時間を「実装」から「アーキテクチャ + 教育」にシフト**。
- ジュニア教育の quality / speed up
- 技術負債 reduction に spotlight
- チーム文化 leveling up

##### QA: 「テスト完全自動化できる？」

A: **「網羅的」は可能。「creative/exploratory」は人間無し。**

- Unit / Integration test: AGI 80-90% 自動化可能
- 複雑なシナリオ: QA Engineers judgment 必須
- Mutation testing で「テストの質」確認

---

#### ✅ チェック

Are you using Copilot in the recommended way for your role?

- [ ] 毎日使用
- [ ] チーム標準を awareness している
- [ ] テストで verify している
- [ ] Chat で理由を確認している
- [ ] Pair で learning & mentoring している

## Project Templates {#section-06-reference-04-project-templates}

#### よくあるプロジェクト種別ごとの「Copilot Best Practice」ガイド

各プロジェクトタイプの初期セットアップと推奨パターンを紹介します。

---

#### 🎯 プロジェクトタイプ別ガイド

##### 1️⃣ Web API (ASP.NET Core)

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

##### 2️⃣ Console Application / Worker Service

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

##### 3️⃣ Class Library / SDK

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

##### 4️⃣ Microservice

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

##### 5️⃣ Console App (Simple Automation / Scripts)

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

#### 📋 すべてのプロジェクトタイプに共通

##### 1. `.github/copilot-instructions.md` 必須

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

##### 2. テストテンプレート

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

##### 3. CI/CD Pipeline

```yaml
# .github/workflows/test.yml
- Build
- Unit Tests (xUnit)
- Code Coverage Analysis (Sonarqube)
- Publish (if coverage ≥ 80%)
```

---

#### ✅ プロジェクト初期化チェック

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

#### 🚀 推奨ワークフロー

##### Day 1: セットアップ
```
1. git clone
2. dotnet build
3. dotnet test
4. copilot-instructions.md を read
5. IDE opening
```

##### Day 2-3: First Feature
```
1. Issue/Task を読む
2. コメントで intent を書く → Copilot 補完
3. Chat でパターン確認
4. テスト駆動で develop
5. Code Review
```

---

#### 📚 参考

- [Clean Architecture in .NET](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Microservices Patterns](https://microservices.io/patterns/)
- [Resilience and chaos engineering](https://learn.microsoft.com/en-us/azure/architecture/framework/resiliency/)
