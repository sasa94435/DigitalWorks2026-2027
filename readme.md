# DigitalWorks2026-2027

学生が **Cloudflare Pages** と **GitHub** を連携して公開した Web 作品を、GitHub API を介して全自動で一元収集・表示するギャラリーポータルサイトです。

学生側の設定（Topic 付与と URL 登録）と本ポータル側の自動ビルド（GitHub Actions + Cloudflare Deploy Hook）により、手作業でのリスト更新なしに最新の作品集を維持できます。

---

## 🎨 システム概要図

```text
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │  【学生の作業】                                                             │
 │                                                                             │
 │ ┌─────────────────────────┐   ① 自動デプロイ   ┌──────────────────────────┐ │
 │ │    学生のGitHub         ├───────────────────►│  学生個人のCloudflare    │ │
 │ │   リポジトリ            │                    │  Pages (作品サイト)      │ │
 │ │(Topic:digitalworks-2026)│                    │ (https://xxx.pages.dev)  │ │
 │ └────────────┬────────────┘                    └─────────────┬────────────┘ │
 └──────────────┼───────────────────────────────────────────────┼──────────────┘
                │                                               │
                │ ② APIでリポジトリ情報                         │ ④ サイト画面の
                │    (名前/説明/URL) を収集                     │    キャプチャ生成
                ▼                                               ▼
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │  【ポータルサイト（自動更新）】                                             │
 │                                                                             │
 │ ┌─────────────────────────┐                    ┌──────────────────────────┐ │
 │ │   ポータル用GitHub      │   ③ 自動ビルド     │  ポータルのCloudflare    │ │
 │ │   リポジトリ            ├───────────────────►│  Pages (一覧ギャラリー)   ├─► 🌐 閲覧者
 │ │  (Astro + GitHub API)   │                    │ (優秀作品 / 通常作品)    │ │
 │ └────────────▲────────────┘                    └──────────────────────────┘ │
 │              │                                                              │
 │              └─ (GitHub Actions が毎日0時に Deploy Hook を叩いて再ビルド)   
 └─────────────────────────────────────────────────────────────────────────────┘
```

---

## ✨ 主な機能・仕様

* **完全自動収集**: GitHub Search API を使用し、対象トピックが付与されたパブリックリポジトリをビルド時に自動取得します。
* **優秀作品のピックアップ表示**: `featured-2026` トピックが付いた作品を自動的に判別し、ページ最上部の「FEATURED WORKS」セクションに強調表示します。
* **公平なランダム表示**: ビルド実行ごとに各セクション内の表示順をランダムにシャッフルし、提出順による偏りを防ぎます。
* **サムネイル自動生成**: Microlink API を利用し、各作品の公開URLから最新のスクリーンショットを動的に取得・表示します。
* **レスポンシブ＆説明文重視のUI**: スマホ（1列）から大型ディスプレイ（4列）まで最適化されたグリッドを採用し、リポジトリ名よりも作品の説明文（Description）が引き立つカードデザインになっています。

---

## 🛠 採用技術 (Tech Stack)

* **Framework**: [Astro](https://astro.build/) (SSG)
* **Styling**: [Tailwind CSS](https://tailwindcss.com/)
* **Hosting**: [Cloudflare Pages](https://pages.cloudflare.com/)
* **API Integration**: GitHub REST API (`search/repositories`)
* **Thumbnail Generation**: [Microlink API](https://microlink.io/)
* **Automation**: GitHub Actions (Scheduled & Manual Deploy Hook)

---

## 📁 ディレクトリ構成

```text
├── .github/
│   └── workflows/
│       └── rebuild.yml      # 定期・手動再ビルド用 GitHub Actions
├── public/
│   └── header.png           # ポータル上部のヘッダーバナー画像
├── src/
│   ├── lib/
│   │   └── github.ts        # GitHub API からのリポジトリ取得ロジック
│   └── pages/
│       └── index.astro      # ギャラリーメインページ（レイアウト・振り分け・ソート）
├── astro.config.mjs
└── package.json
```

---

## 🏷 作品の分類・自動集計ルール

ビルド時に GitHub API を検索し、リポジトリに付与されている **Topic** に応じてカードを自動振り分けします。
（※GitHub の仕様上、大文字で入力された Topic も自動的に小文字に変換されます）

| 分類 | 指定 Topic | 表示位置・レイアウト |
| :--- | :--- | :--- |
| **全作品（対象リポジトリ）** | `digitalworks-2026`<br>*(入力は `DigitalWorks-2026` でも可)* | 「ALL PROJECTS」セクションに表示（1〜4列グリッド） |
| **優秀作品（ピックアップ）** | `featured-2026` | ページ最上部の「FEATURED WORKS」セクションに強調表示（1〜2列グリッド） |

---

## 🚀 ポータル運営者向けセットアップ・運用手順

### 1. Cloudflare Pages のビルド設定
* **Framework preset**: `Astro`
* **Build command**: `npm run build`
* **Build output directory**: `dist`
* **Deploy command**: （空欄）

### 2. 環境変数の設定 (`GITHUB_TOKEN`)
GitHub API のレート制限（未認証: 60回/時 ➔ 認証済: 5,000回/時）を回避するため、GitHub の Personal Access Token (PAT) を設定します。

1. GitHub の `Settings > Developer Settings > Personal access tokens` からトークンを発行します（パブリックリポジトリの読み取り権限のみで可）。
2. Cloudflare Pages のプロジェクト設定（`Settings > Environment variables`）に以下を追加します：
   * **Variable name**: `GITHUB_TOKEN`
   * **Value**: 発行した Personal Access Token

### 3. 定期再ビルド設定 (`CLOUDFLARE_DEPLOY_HOOK`)
1. Cloudflare Pages の `Settings > Builds & deployments > Deploy hooks` から Webhook URL を生成します。
2. 本リポジトリの `Settings > Secrets and variables > Actions` の **Repository secrets** に以下を登録します：
   * **Name**: `CLOUDFLARE_DEPLOY_HOOK`
   * **Secret**: 生成した Deploy Hook URL
3. 毎日深夜（JST 0:00）に `.github/workflows/rebuild.yml` が自動実行され、最新の作品一覧へ更新されます。手動で即時反映したい場合は、GitHub の `Actions` タブからワークフローを手動実行（`Run workflow`）してください。

### 4. ヘッダー画像の変更方法
* `public/header.png` を差し替えることでトップバナーを変更できます。
* **推奨サイズ**: `1280 × 384 px`（高解像度ディスプレイ向けには2倍の `2560 × 768 px`）、アスペクト比約 `10:3`、容量 500KB 以下推奨。

---

## 👨‍💻 ローカル開発手順

```bash
# 依存関係のインストール
npm install

# 開発サーバーの起動 (http://localhost:4321)
npm run dev

# 本番ビルドの動作確認
npm run build
npm run preview
```

---

## 📖 学生向け設定手順（概要）

学生は自身の成果物リポジトリ（**Public** 設定）で以下の作業を行います：

1. **Cloudflare Pages** にリポジトリを連携して Web サイトを公開し、公開 URL（`https://xxxx.pages.dev`）を取得する。
2. GitHub リポジトリ画面右側の **About (⚙️アイコン)** を開き、以下を設定して保存する：
   * **Description**: 作品のキャッチコピーや概要（ポータルのカードに大きく表示されます）
   * **Website**: 公開した Cloudflare Pages の URL
   * **Topics**: **`DigitalWorks-2026`** を入力して確定（自動的に `digitalworks-2026` になります）
   * *(※優秀作品に選出された場合は、追加で **`featured-2026`** も登録)*