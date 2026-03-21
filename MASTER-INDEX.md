# GitHub Copilot ベストプラクティス（C#版）- マスターインデックス

このドキュメントは、プロジェクト全体の構造と各ファイルの関係を示します。

## 📚 プロジェクト構造

```
github-copilot-best-practices-csharp/
├── 00-COVER.md                          # ebook表紙・ランディングページ
├── README.md                            # ナビゲーションガイド
├── MASTER-INDEX.md                      # このファイル
│
├── 01-foundations/                      # 基礎知識セクション
│   ├── README.md
│   ├── 01-basics.md                     # IDE設定・キーボードショートカット
│   ├── 02-comment-driven-dev.md         # コメント駆動開発・XMLDoc
│   └── 03-beginner-scenarios.md         # 3つの初心者シナリオ
│
├── 02-core-features/                    # コア機能セクション
│   ├── README.md
│   ├── 01-inline-completions.md         # インラインコンプリーション
│   ├── 02-copilot-chat.md               # Copilot Chat機能
│   ├── 03-inline-edits.md               # インラインエディット
│   └── 04-agent-mode.md                 # エージェントモード
│
├── 03-hands-on-workflows/               # ハンズオン・ワークフロー
│   ├── README.md
│   ├── 01-design-patterns.md            # デザインパターン実装
│   ├── 02-performance-optimization.md   # パフォーマンス最適化
│   └── 03-testability.md                # テスト駆動設計
│
├── 04-role-mastery/                     # ロール別マスタリー
│   ├── README.md
│   ├── 01-beginner-developer.md         # 初級開発者ロードマップ
│   ├── 02-mid-level-developer.md        # 中級開発者スキル
│   ├── 03-senior-lead.md                # シニア・リードガイド
│   └── 04-qa-engineer.md                # QA エンジニアガイド
│
├── 05-team-operations/                  # チーム運用セクション
│   ├── README.md
│   ├── 01-team-standards.md             # チームスタンダードテンプレート
│   ├── 02-adoption-strategy.md          # 導入戦略・ロールアウト計画
│   ├── 03-metrics-measurement.md        # メトリクス・ROI測定
│   └── 04-copilot-instructions-template.md # .instructions.md テンプレート
│
└── 06-reference/                        # リファレンスセクション
    ├── README.md
    ├── 01-troubleshooting.md            # トラブルシューティング集
    ├── 02-quick-reference.md            # クイックリファレンス
    ├── 03-feature-by-role-matrix.md     # 機能×ロール対応表
    └── 04-project-templates.md          # プロジェクトテンプレート集
```

## 🎯 セクション別ガイド

### 01-foundations（基礎知識）
**対象者**: すべての開発者（特に初心者）  
**学習時間**: 3-5時間  
**目的**: Copyilotの基本操作と思考方法を習得

| ファイル | 内容 | 主要トピック |
|---------|------|-----------|
| **01-basics.md** | IDE統合・設定 | 環境設定、キーボードショートカット、コンテキスト管理 |
| **02-comment-driven-dev.md** | コメント駆動開発 | XMLDoc、3ステッププロセス、パターンマッチング |
| **03-beginner-scenarios.md** | 実践シナリオ | ファイル操作、LINQ、async/await 処理 |

### 02-core-features（コア機能）
**対象者**: すべての開発者  
**学習時間**: 5-8時間  
**目的**: Copilotの4つの主要機能を習得

| ファイル | 機能 | ユースケース |
|---------|------|-----------|
| **01-inline-completions.md** | インライン補完 | コード補完、パターン自動化 |
| **02-copilot-chat.md** | Chat機能 | 説明、リファクタリング、エラー解決 |
| **03-inline-edits.md** | インライン編集 | 最適化、エラーハンドリング |
| **04-agent-mode.md** | エージェント | 複数ファイル自動化、テスト生成 |

### 03-hands-on-workflows（ハンズオン）
**対象者**: 中級以上の開発者  
**学習時間**: 6-10時間  
**目的**: 実務的なワークフローでCopilotを活用

| ファイル | ワークフロー | 実装例 |
|---------|-----------|------|
| **01-design-patterns.md** | パターン構築 | Strategy/Repository パターン |
| **02-performance-optimization.md** | 最適化 | N+1問題、並列化、LINQ |
| **03-testability.md** | テスト設計 | DI、Moq、TDD |

### 04-role-mastery（ロール別）
**対象者**: 自分のロール別に学習  
**学習時間**: 各ロール 4-6週間  
**目的**: ロール固有のスキルとCopilot活用法を習得

| ロール | ファイル | キーパス |
|--------|---------|---------|
| **初級開発者** | 01-beginner-developer.md | 12週間ロードマップ、9つの学習ポイント |
| **中級開発者** | 02-mid-level-developer.md | 3つのキースキル、実践タスク |
| **シニア・リード** | 03-senior-lead.md | アーキテクチャ、チーム育成、リーダーシップ |
| **QAエンジニア** | 04-qa-engineer.md | テスト生成、ミューテーション、品質メトリクス |

### 05-team-operations（チーム運用）
**対象者**: エンジニアリングリード、チームマネージャー  
**学習時間**: 2-3日（チーム全体）  
**目的**: チーム導入と運用体制を構築

| ファイル | 目的 | 成果物 |
|---------|-----|------|
| **01-team-standards.md** | 標準化 | チームスタンダードテンプレート |
| **02-adoption-strategy.md** | 導入計画 | 3段階ロールアウト計画 |
| **03-metrics-measurement.md** | 測定 | ROI計算、KPIダッシュボード |
| **04-copilot-instructions-template.md** | カスタマイズ | 組織用.instructions.md テンプレート |

### 06-reference（リファレンス）
**対象者**: すべての開発者（必要に応じて参照）  
**学習時間**: 随時参照  
**目的**: 問題解決・パターン検索

| ファイル | 用途 | 内容 |
|---------|-----|------|
| **01-troubleshooting.md** | トラブル解決 | 12+シナリオ、根本原因、解決策 |
| **02-quick-reference.md** | クイックルックアップ | ショートカット、C#パターン、EFCore |
| **03-feature-by-role-matrix.md** | 対応表 | 機能 × ロール グリッド |
| **04-project-templates.md** | テンプレート | Web API、Console、Library など5種類 |

## 🗺️ 学習パス

### パス1: 初心者向け（推奨期間: 3-4週間）
1. 00-COVER.md を読む（5分）
2. README.md でナビゲーション確認（5分）
3. `01-foundations/` 全て学習（3-5時間）
   - 実践: `03-beginner-scenarios.md` の3シナリオを実装
4. `02-core-features/01-02-03.md` 学習（4-6時間）
   - 実践: 簡単なプロジェクトで各機能を試す
5. `04-role-mastery/01-beginner-developer.md` で12週間ロードマップを確認

**完了後**: インラインコンプリーションとChatの基本操作習得 ✓

### パス2: 中級開発者向け（推奨期間: 4-6週間）
1. `02-core-features/` 全て学習（5-8時間）
2. `03-hands-on-workflows/` 全て学習（6-10時間）
3. `04-role-mastery/02-mid-level-developer.md` の3スキルを実践開発で適用
4. `05-team-operations/01-team-standards.md` でチーム標準を理解

**完了後**: デザインパターン、パフォーマンス最適化、テスト設計で生産性向上 ✓

### パス3: シニア・リード向け（推奨期間: 1-2週間）
1. `04-role-mastery/03-senior-lead.md` で要件確認（2時間）
2. `05-team-operations/` 全て学習（6-8時間）
3. チームスタンダードの策定（2-3日）
4. 採用戦略の実行準備

**完了後**: チーム採用とCopilot統合戦略の策定 ✓

### パス4: QA・テスト向け（推奨期間: 2-3週間）
1. `01-foundations/` 基本習得（2-3時間）
2. `02-core-features/04-agent-mode.md` テスト生成機能を学習（2時間）
3. `04-role-mastery/04-qa-engineer.md` 全て学習（6-8時間）
4. `06-reference/01-troubleshooting.md` でテスト関連シナリオを実践

**完了後**: テスト自動化とテスト品質メトリクスの改善 ✓

## 📊 ファイル統計

### ファイル数
- セクション: 6個
- コンテンツファイル: 22個
- READMEファイル: 7個（セクション6 + ルート1）
- **合計**: 35個

### コンテンツボリューム
- 推定総行数: 15,100行
- 平均ファイルサイズ: 430行
- コード例: 100+個
- チェックリスト: 20+個

### カバー範囲
- 機能領域: インライン補完、Chat、エディット、エージェント
- スキルレベル: 初心者～シニア
- ロール: 開発者（4タイプ）、チームリード、QA
- トピック: 23領域

## 🔗 クロスリファレンス

### 「初心者向けシナリオ」の参照箇所
- `03-beginner-scenarios.md` ← 主要コンテンツ
- `01-basics.md` → 環境設定の詳細
- `04-role-mastery/01-beginner-developer.md` → 発展的な学習パス

### 「パフォーマンス最適化」の参照箇所
- `03-hands-on-workflows/02-performance-optimization.md` ← 主要コンテンツ
- `02-core-features/01-inline-completions.md` → LINQ最適化の基礎
- `04-role-mastery/02-mid-level-developer.md` → 実務的な適用例

### 「チーム標準化」の参照箇所
- `05-team-operations/01-team-standards.md` ← テンプレート
- `02-core-features/` → 各機能の標準的な使い方
- `06-reference/02-quick-reference.md` → コピペ可能なテンプレート

## 🚀 次のステップ

1. **フェーズ4**: Ebook-build 設定確認
   - build.json と metadata.yaml の検証
   - 00-COVER.md が着地ページとして機能するか確認

2. **フェーズ5**: EPUB/AZW3/MOBI 生成
   - ebook-build 実行
   - TOC生成の検証
   - 出力ファイルの差し込みテスト

3. **デプロイ**: GitHub リリース作成
   - 生成されたebookファイルを ebook-output/ に配置
   - リリースノート作成
   - チーム通知

## 📝 注釈

- **ebook-build の互換性**: すべてのファイルは `\d{2}-.*\.md` パターンに準拠
- **相対リンク**: セクション内のリンクは相対パスで記述（ebook互換性確保）
- **コード例**: すべてC#3.11以降、.NET 8.0以降で動作確認
- **チェックリスト**: コピペ可能なマークダウン形式（word/スプレッドシート対応）

---

**最終更新**: 2026年3月21日  
**アーキテクチャ**: 6セクション構造  
**互換性**: Ebook-build v2.0+, Markdown 🎉
