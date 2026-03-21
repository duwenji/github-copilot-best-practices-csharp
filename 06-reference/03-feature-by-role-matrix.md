### 各ロールが「どの Copilot 機能」を「どんなシーンで」使うかのマトリックス

---

### 📊 全体マトリックス

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

### 🎯 ロール別 詳細マッピング

#### 初級開発者（Beginner）

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

#### 中級開発者（Mid-Level）

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

#### シニアリード（Senior/Lead）

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

#### QA エンジニア

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

### 🎓 シーン別推奨フロー

#### シーン 1: "新しい API エンドポイント作成"

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

#### シーン 2: "バグ修正＋テスト追加"

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

### 📈 効率化の「鍵」

#### 初級 → 中級への jump

- [ ] Copilot Chat で設計パターン相談できるか
- [ ] Strategy / Repository 実装を Copilot で完了できるか
- [ ] テストコード generation を活用できるか

**Growth Metric**: 
- 同じタスク時間 -30%
- テストカバレッジ +20%
- Code review comment -40%（品質向上）

---

#### 中級 → 上級への jump

- [ ] API アーキテクチャを「設計 → チーム教育」に転換か
- [ ] Copilot を「ジュニア教育 tool」として活用できるか
- [ ] メトリクス driven で continuous improvement できるか

**Growth Metric**:
- チーム生産性 +50%
- 技術負債 reduction
- Onboarding time -60%（新人育成効率）

---

### 💡 共通注意点

#### すべてのロール

- ✅ Copilot は「助手」（盲信するな）
- ✅ テストで verify してから採用
- ✅ セキュリティ手動確認必須（especially Copilot code）
- ✅ チーム標準 (`.github/copilot-instructions.md`) を教える
- ✅ 「Copilot が何をしたか」理解してから進める

#### Red Flags

- ❌ 「Copilot が生成したから OK」（No verification）
- ❌ テストなし で merge（AI code でも必須）
- ❌ チーム標準を無視した自由なパターン混在
- ❌ ジュニアが complete dependency（理解なし） → 長期的に危ない

---

### 📊 チーム全体効果期待値

| Metric | Timeline | 初級→効果 | 中級→効果 | 上級→効果 |
|--------|----------|---------|---------|---------|
| **生産性** | 3m | +20% | +40% | +60% |
| **コード品質** | 3m | +15% | +30% | +25% |
| **テストカバレッジ** | 6m | +10% | +25% | +20% |
| **オンボーディング** | 6m | -30% | -50% | -70% |
| **チーム満足度** | 1m | +3.0 | +3.5 | +4.0 |

（5段階スケール）

---

### 🚀 推奨導入順序

**Phase 1 (Week 1-2: 全員)**
- Copilot インストール
- [01-foundations/](../01-foundations/) 学習
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

### 📞 Questions by Role

#### 初級: 「Copilot が提案したコード、信頼していい？」

A: **必ず verify してください**。
- テストを実行
- 「おかしい」と感じたら Chat で質問
- メンターに見てもらう

#### 中級: 「どこまで Copilot に任せる？」

A: **設計は自分で、実装は Copilot** の配分がベスト。
- Interface 設計 → 自分
- Repository 実装 → Copilot
- テストコード → Copilot + review

#### 上級: 「Copilot で何が変わる？」

A: **時間を「実装」から「アーキテクチャ + 教育」にシフト**。
- ジュニア教育の quality / speed up
- 技術負債 reduction に spotlight
- チーム文化 leveling up

#### QA: 「テスト完全自動化できる？」

A: **「網羅的」は可能。「creative/exploratory」は人間無し。**

- Unit / Integration test: AGI 80-90% 自動化可能
- 複雑なシナリオ: QA Engineers judgment 必須
- Mutation testing で「テストの質」確認

---

### ✅ チェック

Are you using Copilot in the recommended way for your role?

- [ ] 毎日使用
- [ ] チーム標準を awareness している
- [ ] テストで verify している
- [ ] Chat で理由を確認している
- [ ] Pair で learning & mentoring している
