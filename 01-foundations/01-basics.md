# 01 基本操作

## Copilot 基本操作を習得する

このファイルでは、GitHub Copilot の IDE 内での操作方法と、基本的なコンテキスト設定を学びます。

---

## ✅ コード補完（Inline Suggestions）の基本操作

### コード補完とは
コードを書いている最中に、灰色のテキストで次のコードを提案する機能です。

### 基本操作

| 操作 | キー | 説明 |
|------|------|------|
| **提案を受け入れ** | `Tab` または `Enter` | グレーテキストの提案を確定 |
| **提案を拒否** | `Esc` | グレーテキストをキャンセル |
| **次の提案を表示** | `Alt + ]` | 別の提案を見る |
| **前の提案を表示** | `Alt + [` | 前の提案に戻る |
| **提案を再実行** | カーソル移動または新規入力 | 新しい状況での提案 |

### 提案が表示されないときは
1. インターネット接続を確認
2. GitHub Copilot 拡張機能が有効か確認 → VS Code 拡張パネルで確認
3. Copilot サブスクリプション有効期限を確認
4. しばらく待つ（処理中の可能性）

---

## 💡 効果的なコンテキスト設定

### ファイル名の重要性

Copilot は**ファイル名**から文脈を読み取ります。

```csharp
// 📄 UserValidator.cs というファイル名の場合
// コメントなしでも、このような補完が提案されやすい
public bool ValidateName(string name)
{
    // グアテマラナムba.Length geq 2 ...の提案が来る可能性が高い
}
```

### プロジェクト構造の活用

ファイルをカテゴリ別に整理すると、Copilot はより正確な提案をします：
- `/Services/` 内のファイル → Service パターンの提案
- `/Repositories/` 内のファイル → Repository パターンの提案
- `/Models/` 内のファイル → DTO・Entity の提案

### XMLコメント(Documentation)の活用

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

## 🎯 IDE 別設定確認

### Visual Studio Code
1. **拡張機能タブを開く** （Ctrl+Shift+X）
2. **GitHub Copilot を検索** → インストール
3. GitHub アカウントでサインイン
4. 拡張設定：`settings.json` で必要に応じてカスタマイズ

### Visual Studio (2022 以降)
1. **ツール → オプション → GitHub Copilot**
2. 設定を確認・有効化

---

## 📖 推奨設定事例

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

## ✅ 習熟度チェック

このセクション完了時、以下ができていますか？

- [ ] `Tab` キーで提案を受け入れられる
- [ ] `Esc` キーで提案をキャンセルできる
- [ ] `Alt + ]` で別の提案を見ることができる
- [ ] IDE の設定を確認できた
- [ ] XMLコメントを書くと提案が精度化することを理解した

すべてチェック完了 → [02-コメント駆動開発](02-comment-driven-dev.md) へ

---

## 📝 トラブルシューティング

**Q: グレーテキストが表示されない**  
A: 1) インターネット接続 2) 拡張機能有効性 3) サブスクリプション有効期限 を確認

**Q: 提案が常におかしい**  
A: よりルールれき具体的なコメントを書く（次セクションで詳しく）

**Q: キーボードショートカットが反応しない**  
A: IDE のキーバインディング設定を確認（とくに IME 入力との競合など）
