## チーム運営 | Team Operations

### 📚 このセクションについて

**対象**: リード・マネージャー、チーム導入責任者  
**学習時間**: 2-3 時間  
**前提知識**: Copilot の基本理解 + ロール別ガイドの確認

---

### 🎯 このセクションのゴール

GitHub Copilot をチーム全体に展開し、**組織的な生産性向上**を実現します：

- ✅ **チーム標準化** — コーディング規約をファイルに落とし込む
- ✅ **採用戦略** — パイロット～定着化のロードマップ
- ✅ **メトリクス計測** — 効果の可視化と改善ループ
- ✅ **カスタム指示** — プロジェクト固有の指示ファイル作成

---

### 📖 コンテンツ構成

#### [01-team-standards.md](01-team-standards.md)
**課題**: Copilot がチーム規約に沿ったコードを生成したい  
**学ぶこと**:
- `.github/copilot-instructions.md` の作成方法
- C# コーディング規約の定義
- 命名規則・エラーハンドリング標準
- `.editorconfig` との連携

**成果物**: チーム共通指示ファイルテンプレート

**時間**: 45-60 分

---

#### [02-adoption-strategy.md](02-adoption-strategy.md)
**課題**: Copilot をチーム全体に導入したい  
**学ぶこと**:
- パイロット導入フェーズ（1-2 週間）
- チーム展開フェーズ（1 ヶ月）
- 定着化フェーズ（3 ヶ月）
- ライセンス管理・サポート体制

**成果物**: チーム導入ロードマップ

**時間**: 60-90 分

---

#### [03-metrics-measurement.md](03-metrics-measurement.md)
**課題**: Copilot の効果ってどう測ればいい？  
**学ぶこと**:
- 定量的指標（受け入れ率、生産性、バグ率）
- 定性的指標（満足度、コード品質評価）
- スプレッドシートでのダッシュボード作成
- 経営層への報告方法

**成果物**: 計測テンプレート、レポート例

**時間**: 45-60 分

---

#### [04-copilot-instructions-template.md](04-copilot-instructions-template.md)
**課題**: カスタム指示ファイルって何を書く？  
**学ぶこと**:
- 指示ファイルの構造
- セクション別の書き方（技術スタック、規約、テスト方針）
- 各フレームワーク（ASP.NET, EF Core など）の設定例
- テンプレートの定期更新方法

**成果物**: すぐに使えるテンプレート＆カスタマイズ例

**時間**: 60-90 分

---

### 🎯 3つの活動パターン

#### パターン A: 新しいプロジェクト立ち上げ
```
① Team Standards で規約を定義
   ↓
② Adoption Strategy で初回導入
   ↓
③ Copilot Instructions Template でカスタマイズ
   ↓
④ Metrics Measurement で効果追跡
```

#### パターン B: 既存プロジェクトへの導入
```
① Copilot Instructions Template から開始
   （既存の規約を反映）
   ↓
② Adoption Strategy パイロットフェーズ
   ↓
③ Metrics Measurement で改善検討
```

#### パターン C: チーム全体への展開（全社）
```
① Team Standards（全社統一規約）
   ↓
② Adoption Strategy（多段階展開）
   ↓
③ Metrics Measurement（全体ダッシュボード）
   ↓
④ Copilot Instructions Template（プロジェクト個別化）
```

---

### 📊 成果物と産出物

| ファイル | 成果物 | 用途 |
|---------|-------|------|
| 01-team-standards.md | `.github/copilot-instructions.md` | チーム共有 |
| 02-adoption-strategy.md | 導入ロードマップ＋チェックリスト | 経営層報告 |
| 03-metrics-measurement.md | 計測テンプレート＆レポート | 効果可視化 |
| 04-copilot-instructions-template.md | カスタマイズ済みテンプレート | プロジェクト導入 |

---

### 🚀 推奨実装順

#### ⏱️ 1 週目: 準備
1. 01-team-standards で規約定義
2. `.github/copilot-instructions.md` 作成・試運用

#### ⏱️ 2 週目: パイロット
1. 02-adoption-strategy でパイロットチーム選定
2. 基本的な指示ファイルで試運用開始

#### ⏱️ 3-4 週目: チーム展開
1. 02-adoption-strategy の展開フェーズ実行
2. 学習会（[04-role-mastery](../04-role-mastery/) 活用）

#### 📅 1 ヶ月後: 定着化
1. 03-metrics-measurement で初回計測
2. 改善点を指示ファイルに反映

---

### ✅ チーム導入チェックリスト

#### パイロット導入完了時
- [ ] パイロットメンバーが基本操作を習得した
- [ ] `.github/copilot-instructions.md` が作成された
- [ ] 初回フィードバック会を実施した
- [ ] 成功事例とチャレンジを共有した

#### チーム展開完了時
- [ ] 全メンバーにライセンス配布完了
- [ ] チーム学習会を実施した
- [ ] 指示ファイルが全メンバーに共有された
- [ ] Q&A チャネルが設置された

#### 定着化フェーズ
- [ ] 活用状況レビュー（月次）開始
- [ ] ベストプラクティス共有会を開催
- [ ] カスタム指示ファイルの改善ループ開始
- [ ] メトリクス計測＆報告開始

---

### 👥 各ロールの責任

| ロール | 責務 | 時間/月 |
|--------|------|--------|
| **チーム導入リード** | 全体調整、ロードマップ実行 | 10-15h |
| **テックリード** | 指示ファイル作成・保守 | 5-8h |
| **メンバー** | 活用＋フィードバック | 2-3h (初月) |
| **マネージャー** | 進捗管理、報告書作成 | 5h |

---

### 📈 期待できる改善

#### 短期（1-3 ヶ月）
- コード生成受け入れ率: 20-30%
- コードレビュー工数: 15-20% 削減
- オンボーディング期間: 10-15% 短縮

#### 中期（3-6 ヶ月）
- 生産性向上: 20-30%
- バグ発生率: 10-15% 削減
- チーム満足度: 70% 以上

#### 長期（6-12 ヶ月）
- 新機能開発リードタイム: 20-30% 短縮
- 技術的負債削減: 定量化開始
- キャリア成長: 個別評価

---

### 🎯 よくある質問

**Q: ライセンス費用対効果は？**  
→ 03-metrics-measurement.md の ROI 計算例を参照

**Q: セキュリティは大丈夫？**  
→ 01-team-standards.md のセキュリティセクション参照

**Q: 全社導入にはどのくらい?**  
→ 02-adoption-strategy.md の多段階展開モデル参照

**Q: 古いプロジェクトにも適用できる?**  
→ [06-reference](../06-reference/) のトラブルシューティング参照

---

### 📙 ファイル一覧

| ファイル | 内容 | 対象 |
|---------|------|------|
| [01-team-standards.md](01-team-standards.md) | 規約・指示ファイル | リード |
| [02-adoption-strategy.md](02-adoption-strategy.md) | 導入ロードマップ | マネージャー |
| [03-metrics-measurement.md]( 03-metrics-measurement.md) | 効果測定 | マネージャー/リード |
| [04-copilot-instructions-template.md](04-copilot-instructions-template.md) | カスタマイズテンプレート | リード |
| **README.md** | **このファイル** | **全員** |

---

### 🔗 関連ドキュメント

- [GitHub Copilot ライセンス管理](https://docs.github.com/ja/enterprise-cloud@latest/admin/policies)
- [Copilot for Business Pricing](https://github.com/features/copilot/plans)
- [Enterprise Security & Compliance](https://github.com/enterprise/security)
