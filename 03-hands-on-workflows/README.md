## 実践的ワークフロー | Hands-On Workflows

### 📚 このセクションについて

**対象**: 中級者（基本操作習得済み）  
**学習時間**: 2-3 時間  
**前提知識**: `02-core-features` の 4 つの機能理解

---

### 🎯 このセクションのゴール

02-core-features で学んだ 4 つの機能を**組み合わせて**、実務的なワークフローで活用できるようになります：

- ✅ **Design Patterns** — デザインパターンの適用とリファクタリング
- ✅ **Performance Optimization** — パフォーマンス改善のプロセス
- ✅ **Testability** — テスト容易性を考慮した設計・実装

---

### 📖 コンテンツ構成

#### [01-design-patterns.md](01-design-patterns.md)
**課題**: 既存コードにパターンを適用したい  
**学ぶこと**:
- Strategy パターンの応用
- Repository パターンの実装
- 依存性注入の設定
- Copilot を使ったリファクタリングプロセス

**実装例**: 支払い処理、データアクセス層の再設計

**時間**: 45-60 分

---

#### [02-performance-optimization.md](02-performance-optimization.md)
**課題**: 遅いコードを効率化したい  
**学ぶこと**:
- LINQ クエリの最適化（N+1 問題など）
- Entity Framework のベストプラクティス
- 並列処理・非同期処理の活用
- Copilot を使った最適化のプロセス

**実装例**: 大量データ検索、複数 API 並列呼び出し

**時間**: 45-60 分

---

#### [03-testability.md](03-testability.md)
**課題**: このコードってテストしにくい...  
**学ぶこと**:
- 依存性注入でテスト可能に
- モック・スタブの設定（Moq）
- テスト容易な設計の原則
- Copilot を使ったテストコード自動生成

**実装例**: Web API のユニット/統合テスト

**時間**: 45-60 分

---

### 🧪 学習の進め方

#### パターン A: テーマ順に学ぶ（推奨）

```
① Design Patterns
   （基本的なパターンと Copilot の活用）
   ↓
② Performance Optimization
   （既存コードの改善）
   ↓
③ Testability
   （品質保証との連携）
```

#### パターン B: 自分の課題から学ぶ

| 悩み | 該当ファイル |
|------|-----------|
| 「このコード複雑だ」 | → 01-design-patterns.md |
| 「このコード遅い」 | → 02-performance-optimization.md |
| 「テストが書きにくい」 | → 03-testability.md |

---

### 🎯 各ワークフローの特徴

| ワークフロー | 主な Copilot 機能 | 期待される改善 |
|-----------|-----------------|--------------|
| **Design Patterns** | Chat + Agent | コード可読性 ↑↑ |
| **Performance** | Chat + Inline Edits | 実行速度 ↑↑ |
| **Testability** | Agent + Chat | テストカバレッジ ↑↑ |

---

### ✅ 習熟度チェック

#### Design Patterns
- [ ] Strategy パターンの使い分けを理解
- [ ] Copilot を使ってリポジトリパターンを実装
- [ ] 依存性注入の設定ができた

#### Performance Optimization
- [ ] N+1 クエリ問題を特定できる
- [ ] LINQ クエリを最適化できる
- [ ] 並列処理の実装ができた

#### Testability
- [ ] インターフェースを使った依存性注入ができる
- [ ] Moq を使ったモック設定ができる
- [ ] テストコードの自動生成ができた

**すべてチェック完了** → [04-role-mastery](../04-role-mastery/) へ進み、ロール別の詳細ガイドを学びます

---

### 🚀 使用技術スタック（例）

このセクションで扱う主な技術：

- **Design Patterns**: C#, SOLID 原則
- **Performance**: Entity Framework Core, LINQ, async/await
- **Testability**: xUnit, Moq, FluentAssertions

---

### 📙 ファイル一覧

| ファイル | 内容 | 焦点 |
|---------|------|------|
| [01-design-patterns.md](01-design-patterns.md) | パターン適用＋リファクタリング | 設計 |
| [02-performance-optimization.md](02-performance-optimization.md) | 最適化プロセス | 実行効率 |
| [03-testability.md](03-testability.md) | テスト容易設計 | 品質 |
| **README.md** | **このファイル** | **ナビゲーション** |

---

### 💡 学習のコツ

1. **1つのワークフローを完全に追う** — 提示されたコード例を全部「自分で書く」
2. **Copilot の提案を理解する** — なぜこの提案か、を常に考える
3. **自分のプロジェクトに適用** — 学んだパターンを実務コードに試す

---

### 🔗 関連参考資料

- [Design Patterns in C#](https://www.refactoring.guru/design-patterns/csharp)
- [Entity Framework Core Performance](https://docs.microsoft.com/en-us/ef/core/performance/)
- [xUnit.net Documentation](https://xunit.net/docs/getting-started/netcore)
