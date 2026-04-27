# Anthropic Agent Skills 仕様調査結果

> 2026-04-27 research-tech 調査

## 1. SKILL.md の必須仕様

### フロントマター（agentskills.io/specification が一次ソース）

| フィールド | 必須 | 制約 |
|---|---|---|
| `name` | Yes | 最大64文字。小文字英数字とハイフンのみ。先頭・末尾ハイフン禁止。連続ハイフン（`--`）禁止。**ディレクトリ名と一致必須** |
| `description` | Yes（強く推奨） | 最大1024文字。「何をするか」と「いつ使うか」を両方書く |
| `license` | No | ライセンス名またはライセンスファイルへの参照 |
| `compatibility` | No | 最大500文字。環境要件（必要コマンド、ネットワーク等） |
| `metadata` | No | 任意のキー・バリューマップ |
| `allowed-tools` | No | スペース区切りのツール名（experimental） |

### Claude Code 固有の追加フィールド

| フィールド | 用途 |
|---|---|
| `when_to_use` | トリガー補足説明。`description`と合算で1,536文字上限 |
| `disable-model-invocation` | `true`にするとClaude自動起動を禁止し `/skill-name` 手動のみに |
| `user-invocable` | `false`にするとメニューから非表示 |
| `argument-hint` | オートコンプリートのヒント（例: `[url] [depth]`） |
| `context` | `fork` にするとサブエージェントとして独立実行 |
| `agent` | `context: fork` 時のエージェント種別 |
| `allowed-tools` | スキルアクティブ中に承認なしで使えるツール |
| `model` | スキル起動時のモデル上書き |
| `effort` | 推論effort（`low`〜`max`） |
| `paths` | glob パターンでスキルが自動起動するファイルパスを限定 |

### description の書き方（最重要）

良い例:
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.

ポイント:
- 「何をするか（機能一覧）」+ 「いつ使うか（トリガーとなるユーザー発言・状況）」の2層構造
- description はスキルの自動起動判断に直結

## 2. ディレクトリ構成

公式定義:
```
skill-name/
├── SKILL.md          # 必須
├── scripts/          # 任意: 実行コード
├── references/       # 任意: 詳細ドキュメント（オンデマンドロード）
├── assets/           # 任意: テンプレート・データ
└── examples/         # 任意（Claude Code 拡張）
```

ファイル名規約:
- `SKILL.md` は大文字固定
- ディレクトリ名 = `name` フィールドと完全一致必須

公式サンプル:
- webapp-testing: github.com/anthropics/skills/tree/main/skills/webapp-testing
- skill-creator: github.com/anthropics/skills/tree/main/skills/skill-creator

## 3. 配置・実行モデル

| スコープ | パス |
|---|---|
| 個人 | `~/.claude/skills/<skill-name>/SKILL.md` |
| プロジェクト | `.claude/skills/<skill-name>/SKILL.md` |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` |

ライブ変更検出: 既存ディレクトリ配下の追加・編集・削除は**セッション内で即時反映**

## 4. 配布・PR 要件

- 公式リポ: github.com/anthropics/skills （Apache 2.0）
- Open PR: 562件、Open Issue: 213件（2026-04-27時点）
- CONTRIBUTING.md なし、採択基準は非明文
- **WordPress / CMS 系スキルは現在ゼロ**（先行者枠あり）

## 5. ベストプラクティス

- Description は "pushy" に書く（undertriggering > overtriggering）
- Progressive Disclosure: SKILL.md は概要、詳細は references/ に分離
- 命令形（imperative）で指示
- 出力フォーマットをテンプレートで示す
- `compatibility` フィールドで依存を明示

## アンチパターン

- description が短すぎる
- SKILL.md に全情報を詰め込む
- ディレクトリ名と name フィールドが不一致
- `--` を name に使う
- 組織固有の内部ツールを前提とした Skill

## 情報源

- agentskills.io/specification
- code.claude.com/docs/en/skills
- github.com/anthropics/skills
- github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md
- github.com/anthropics/skills/tree/main/skills/webapp-testing
