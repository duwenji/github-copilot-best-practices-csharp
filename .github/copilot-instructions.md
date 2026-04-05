# Copilot Instructions

このリポジトリでは、電子書籍生成を shared-copilot-skills 経由で実行する。

## Ebook Build
- 入口: `.github/skills-config/ebook-build/invoke-build.ps1`
- 設定: `.github/skills-config/ebook-build/github-copilot-best-practices-csharp.build.json`
- メタデータ: `.github/skills-config/ebook-build/github-copilot-best-practices-csharp.metadata.yaml`
- 章構成前提: `00-COVER.md` と `01-*` 以降の章フォルダ/章ファイル

## Shared skill discovery order
1. `.github/skills/shared-skills/*`
2. `.github/skills/*`
3. `../shared-copilot-skills/*`
