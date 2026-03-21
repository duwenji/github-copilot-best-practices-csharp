# 02 中級開発者向けガイド

## 実装スキルを深掘りして、チーム貢献度を高める

中級者にとって重要な「設計パターン」「パフォーマンス」「保守性」。これらを Copilot で効率的に習得・実装します。

---

## 📊 中級者へのかんたん診断

- 独立したメソッドを書ける → **はい**
- デザインパターンを聞いたことがある → **はい**
- 「なぜテストが必要か」を説明できる → **はい**
- パフォーマンス問題の原因を調査できる → **はい/部分的**

↑ ほとんど「はい」なら、あなたは中級者です。

---

## 🎯 中級者が習得すべき 3つのスキル

### スキル 1: 設計パターンの実装

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
1. [02-core-features/02-copilot-chat.md](../02-core-features/02-copilot-chat.md) で「このコード、リファクタリング候補のパターンは？」と聞く
2. Copilot が Strategy/Factory/Repository などを提案
3. 実装して、PR で指摘を受ける
4. フィードバック反映

### スキル 2: パフォーマンスの診断と改善

「このコードなぜ遅い？」と自分で問い, 改善できる力。

**診断フロー**：
1. ログ/メトリクスで遅い部分を特定
2. Copilot Chat: 「このクエリが遅い理由は？」
3. Copilot が「N+1 問題」などを指摘
4. Include や parallel processing で改善
5. ベンチマーク取得 → 改善効果を確認

例：[03-hands-on-workflows/02-performance-optimization.md](../03-hands-on-workflows/02-performance-optimization.md)

### スキル 3: テスト可能な設計

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

例：[03-hands-on-workflows/03-testability.md](../03-hands-on-workflows/03-testability.md)

---

## 🚀 中級者向け実践タスク（月 1 本程度）

### タスク 例：既存コードの設計パターン適用

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

## 📋 中級者チェックリスト

### 設計スキル
- [ ] 3 つ以上のデザインパターン（Strategy, Repository, Factory）を実装した
- [ ] インターフェース設計を「ユーザー視点」で考慮した
- [ ] 既存コードをリファクタして、SOLID 原則を適用した
- [ ] パターンの適用前後で テスト数を比較できた
- [ ] 「このコードはどのパターンを使うべき？」と聞かれて説明できた

### パフォーマンススキル
- [ ] N+1 クエリ問題を診断・解決した
- [ ] Include/ThenInclude を使った最適化ができた
- [ ] 複数 API の並列化（Task.WhenAll）ができた
- [ ] LINQ での事前フィルタリング（.ToList() 除去）ができた
- [ ] SQL レベルでのパフォーマンス測定ができた

### テストスキル
- [ ] ユニットテスト（xUnit）を 10 個以上書いた
- [ ] モック（Moq）を使った テスト可能な設計ができた
- [ ] テストコード量 ≥ 実装コード量（推奨）
- [ ] CI/CD パイプラインでテスト自動実行される
- [ ] バグが出たら「テストで事前検出できたはず」と思考できた

### コード品質スキル
- [ ] CODE REVIEW で指摘される問題を自分で発見できるようになった
- [ ] PR のコメントで「なぜそう設計したか」を説明できた
- [ ] リファクタリング後も「すべてのテストがパス」を確認する習慣
- [ ] チーム内で「パターン適用例」を共有できた

### Copilot 活用スキル
- [ ] Chat で「デザインパターンの相談」ができた
- [ ] Inline Edits で「複数箇所の一括リファクタリング」ができた
- [ ] テストコード生成を効率的に活用できた
- [ ] Copilot の提案を「盲信」せず、「検証してから採用」できた

---

## 💼 中級者から上級者への進路

以下のいずれかに注力：

### 進路 A: スペシャリスト化
- **領域**: 例）EF Core のエキスパート、LINQ 最適化スペシャリスト
- **キャリア**: コンサルタント、アーキテクト方向
- **次の学習**: [03-hands-on-workflows/](../03-hands-on-workflows/) を完全習得 + 実務分野での深掘り

### 進路 B: リーダーシップ化
- **領域**: チームの設計基準策定、コードレビューリーダー
- **キャリア**: シニアエンジニア、テックリード方向
- **次の学習**: [04-role-mastery/03-senior-lead.md](03-senior-lead.md) へ

---

## 🔗 参考リソース

- [SOLID 原則](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles)
- [デザインパターン in C#](https://refactoring.guru/design-patterns/csharp)
- [Entity Framework Core Performance](https://learn.microsoft.com/en-us/ef/core/performance/)
- [Unit Testing Best Practices](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)

---

## 📞 よくある質問

**Q: Copilot がすべてコード生成してくれるなら、自分で設計する必要ある？**

A: はい。Copilot は「実装」を生成してくれますが、「設計決定」はあなたがする必要があります。
- Copilot: 「実装しますね」
- あなた: 「どのパターンを使うべき？」「なぜそう設計した？」を判断

設計スキルなく Copilot を使うと、「動くがメンテナンス困難」なコードになります。

**Q: テストを書く時間がない。中級者はテスト必須？**

A: テストがないと「リファクタリングが怖くなり」「バグが増える」。
逆にテストがあると「リファクタリング → 品質向上」のサイクルが回ります。
Copilot はテストコード生成で時間短縮できるので、この機会に TDD 導入をお勧めします。
