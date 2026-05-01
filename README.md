# スペック駆動開発テンプレート

Claude Code と連携した**スペック駆動開発**のスターターテンプレートです。ドキュメントを整備しながら機能を実装していく開発フローが組み込まれています。

---

## このテンプレートについて

### スペック駆動開発とは

「何を作るか」を定義した**永続ドキュメント**（`docs/`）と、「今回何をするか」を定義した**ステアリングファイル**（`.steering/`）の2層構造で開発を管理するアプローチです。

```
docs/                        ← プロジェクト全体の「北極星」（頻繁に更新されない）
  product-requirements.md    ← プロダクト要求定義書
  functional-design.md       ← 機能設計書
  architecture.md            ← 技術仕様書
  repository-structure.md    ← リポジトリ構造定義書
  development-guidelines.md  ← 開発ガイドライン
  glossary.md                ← ユビキタス言語定義
  ideas/                     ← 壁打ち・ブレインストーミングの成果物

.steering/                   ← 作業単位のドキュメント（作業ごとに新規作成）
  YYYYMMDD-[タスク名]/
    requirements.md          ← 今回の作業の要求内容
    design.md                ← 実装アプローチ
    tasklist.md              ← 具体的なタスクリスト（進捗管理）
```

### 基本的な開発フロー

```
1. ドキュメント作成  →  2. 作業計画  →  3. 実装  →  4. 検証  →  5. ドキュメント更新
     (docs/)              (.steering/)    (src/)
```

---

## 技術スタック

| 項目 | 内容 |
|------|------|
| 開発環境 | Dev Container |
| Node.js | v24.11.0 |
| 言語 | TypeScript 5.x |
| パッケージマネージャー | npm |
| テスト | Vitest |
| リント | ESLint 9 + typescript-eslint |
| フォーマッター | Prettier |
| Git フック | Husky + lint-staged |

---

## セットアップ

### 1. このテンプレートを新しいリポジトリとして使う

**1-1. テンプレートをクローン**

```bash
git clone <このリポジトリの URL> my-project
cd my-project
```

**1-2. GitHub に新しいリポジトリを作成して push**

```bash
# 既存の origin を削除
git remote remove origin
```

ブラウザで GitHub を開き、新しいリポジトリを作成します（README の自動生成はオフにしてください）。

```bash
# GitHub CLI で認証（未認証の場合）
gh auth login

# 作成したリポジトリを origin として登録して push
git remote add origin https://github.com/<your-username>/my-project.git
git branch -M main
git push -u origin main
```

### 2. Dev Container で開く

Visual Studio Code で「Reopen in Container」を選択すると、以下が自動構築されます。

- Node.js 環境の構築
- `npm install` の実行
- Claude Code 最新版のインストール

> Dev Container を利用する際は、事前に Docker のインストールが必要です。

### 3. Claude Code を起動

```bash
claude
```

---

## 開発手順

### 初回セットアップ: `/setup-project`

プロジェクト開始時に一度だけ実行します。Claude との対話形式で `docs/` 配下の6つの永続ドキュメントを作成します。

```
> /setup-project
```

事前に `docs/ideas/` にアイデアや要件メモを置いておくと、それを元に各ドキュメントを生成します。

**作成されるドキュメント:**

1. `docs/product-requirements.md` — プロダクト要求定義書
2. `docs/functional-design.md` — 機能設計書
3. `docs/architecture.md` — 技術仕様書
4. `docs/repository-structure.md` — リポジトリ構造定義書
5. `docs/development-guidelines.md` — 開発ガイドライン
6. `docs/glossary.md` — ユビキタス言語定義

### 機能追加: `/add-feature [機能名]`

新機能を実装するときに使います。ステアリングファイルの作成から実装・検証・振り返りまでを自動で実行します。

```
> /add-feature ユーザー認証
> /add-feature 商品一覧ページ
```

**自動実行される流れ:**

1. `.steering/YYYYMMDD-[機能名]/` を作成
2. 永続ドキュメントと既存コードを調査
3. `requirements.md`, `design.md`, `tasklist.md` を生成
4. `tasklist.md` の全タスクを順番に実装
5. `implementation-validator` サブエージェントで品質検証
6. `npm test`, `npm run lint`, `npm run typecheck` を実行
7. 振り返りを `tasklist.md` に記録

### 日常的な使い方

基本は普通の会話で依頼します。

```
> PRDに新機能を追加してください
> architecture.mdのパフォーマンス要件を見直して
> glossary.mdに新しいドメイン用語を追加
```

---

## npm scripts

```bash
npm run build       # TypeScript のビルド
npm run dev         # ウォッチモードでビルド
npm test            # テスト実行
npm run test:watch  # ウォッチモードでテスト
npm run lint        # ESLint によるリント
npm run format      # Prettier によるフォーマット
npm run typecheck   # 型チェック（ビルドなし）
```

---

## Claude Code カスタマイズ構成

`.claude/` 配下にプロジェクト固有の設定が含まれています。

### スラッシュコマンド（`.claude/commands/`）

Claude との会話で `/コマンド名` として呼び出せる定義済みワークフローです。

| コマンド | 説明 |
|----------|------|
| `/setup-project` | 初回セットアップ。`docs/ideas/` の内容を元に6つの永続ドキュメントを対話的に作成する |
| `/add-feature [機能名]` | 新機能の追加。計画・実装・検証・振り返りまでを完全自動で実行する |
| `/review-docs [パス]` | 指定したドキュメントを `doc-reviewer` サブエージェントで詳細レビューする |

### スキル（`.claude/skills/`）

コマンドや Claude が内部で呼び出す、特定用途のガイドとテンプレートのセットです。

| スキル | 説明 |
|--------|------|
| `steering` | `.steering/` ファイルの作成・進捗管理・振り返り記録を担う中核スキル。計画モード / 実装モード / 振り返りモードの3モードを持つ |
| `prd-writing` | プロダクト要求定義書（PRD）の作成ガイドとテンプレート |
| `functional-design` | 機能設計書の作成ガイドとテンプレート |
| `architecture-design` | アーキテクチャ設計書の作成ガイドとテンプレート |
| `repository-structure` | リポジトリ構造定義書の作成ガイドとテンプレート |
| `development-guidelines` | 開発ガイドラインの作成ガイドとテンプレート |
| `glossary-creation` | ユビキタス言語定義（用語集）の作成ガイドとテンプレート |

### サブエージェント（`.claude/agents/`）

特定のタスクに特化した独立動作のエージェント定義です。メインの会話コンテキストを消費せずに並列実行できます。

| エージェント | 説明 |
|--------------|------|
| `doc-reviewer` | ドキュメントの品質を「完全性・明確性・一貫性・実装可能性・測定可能性」の5観点で評価し、改善提案をレポート出力する |
| `implementation-validator` | 実装コードをスペックとの整合性・コード品質・テストカバレッジ・セキュリティ・パフォーマンスの観点で検証し、結果をレポート出力する |

### その他の設定ファイル

| ファイル | 説明 |
|----------|------|
| `CLAUDE.md` | プロジェクトメモリ。Claude が会話のたびに読み込む指示書 |
| `.claude/settings.json` | Claude Code のパーミッション設定 |
