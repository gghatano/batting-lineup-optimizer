# 設計書（整形版）

---

## 0. 概要

| 項目            | 採用技術・ツール                                                    |
|-----------------|---------------------------------------------------------------------|
| **Frontend**    | React 18 + Vite + TypeScript 5                                      |
| **State/Data**  | TanStack Query（CSV fetch & cache） + PapaParse / Zustand（選手プール／履歴） |
| **Visualization** | Chart.js 4（箱ひげ・ヒストグラム）                                |
| **Simulation**  | Web Worker（TypeScript）で Monte-Carlo 実行                         |
| **Hosting/CI**  | GitHub Pages + GitHub Actions                                       |

---

## 1. アーキテクチャ

### 1.1 技術スタック

| レイヤ            | 採用技術                                              | 役割                                                                                 |
|-------------------|-------------------------------------------------------|--------------------------------------------------------------------------------------|
| データ取得        | **PapaParse**, **TanStack Query**                     | Google Spreadsheet またはローカル CSV を取得し JSON 化／キャッシュ                   |
| シミュレーション  | **Web Worker (TS)**                                   | Monte-Carlo（9 回裏まで）／ヒューリスティック探索（1 箇所入替）                     |
| UI                | **React 18**, **dnd-kit**, **Zustand**                | 選手選択・打順編集・履歴表示 GUI                                                     |
| 可視化            | **Chart.js 4**                                        | 平均・分布グラフ描画                                                                 |
| ホスティング      | **GitHub Pages**                                      | 静的 SPA 配信                                                                       |
| CI/CD             | **GitHub Actions** + *peaceiris/actions-gh-pages*     | `main` → build → `gh-pages` デプロイ                                                |

### 1.2 ディレクトリ構造

```text
root/
├─ public/
│   └─ sample_players.csv
├─ src/
│   ├─ config.ts
│   ├─ hooks/
│   │   └─ usePlayers.ts
│   ├─ worker/
│   │   └─ simulator.ts
│   ├─ components/
│   │   ├─ PlayerTable.tsx
│   │   ├─ LineupEditor.tsx
│   │   └─ HistoryGrid.tsx
│   └─ ...
├─ config/
│   ├─ tsconfig.json
│   ├─ tsconfig.app.json
│   ├─ tsconfig.node.json
│   ├─ eslint.config.js
│   └─ vite.config.ts
├─ data/
│   └─ sample_players.csv
├─ docs/
│   ├─ requirements.md
│   ├─ design.md
│   ├─ task.md
│   ├─ progress.md
│   └─ development_workflow.md
├─ reports/
│   ├─ README.md
│   ├─ daily-report-template.md
│   └─ daily-report-YYYY-MM-DD-HHmmSS.md
├─ logs/
│   └─ server.log
└─ .github/
    └─ workflows/deploy.yml
````

---

## 2. データソース

下記 Google Spreadsheet を CSV で取得して利用する。
[https://docs.google.com/spreadsheets/d/18OlCqMkP8gNGskMy-NWRXU8Cfb4F7K5g7FelTauhLn4/edit?usp=sharing](https://docs.google.com/spreadsheets/d/18OlCqMkP8gNGskMy-NWRXU8Cfb4F7K5g7FelTauhLn4/edit?usp=sharing)

---

## 3. シミュレーションエンジン

1. **確率テーブル作成**：各打席結果の累積分布を `Float32Array` にキャッシュ
2. **`atBat()`**：`Math.random()` → 2 分探索で打撃結果を決定
3. **走者状態**：ビット列 `0b000–0b111` で管理し、結果に応じてビットシフト
4. **ゲームループ**：9 回・3 アウトで終了し総得点を返却
5. **ヒューリスティック探索**

   * 1 打者 × 隔離距離を選び `swap` → 得点改善で採用
   * 改善なし 10 回連続で探索終了、履歴へ逐次追加

---

## 4. UI / UX 設計

### 4.1 デザインシステム

| 項目            | 要旨                                                                |
| ------------- | ----------------------------------------------------------------- |
| **参照**        | [Atlassian Design System](https://atlassian.design/components)    |
| **原則**        | 明確性・効率性・一貫性・アクセシビリティ                                              |
| **主要コンポーネント** | Button / Table / Select / Input / Badge / Modal / Loading / Toast |

#### カラーパレット

| 用途             | 色コード      |
| -------------- | --------- |
| Primary Blue   | `#0052CC` |
| Success Green  | `#00875A` |
| Warning Orange | `#FF8B00` |
| Danger Red     | `#DE350B` |
| Neutral        | `#42526E` |

### 4.2 画面構成

| 画面   | コンポーネント           | 主な操作                           |
| ---- | ----------------- | ------------------------------ |
| 選手選択 | `PlayerTable`     | チーム選択 → 行クリックで 9 名選択           |
| 打順編集 | `LineupEditor`    | dnd-kit で並び替え／削除               |
| シミュ  | `SimulationPanel` | 試合数入力（number / slider）→ ▶︎ ボタン |
| 履歴   | `HistoryGrid`     | 打順・平均得点を時系列テーブルで表示             |

### 4.3 レスポンシブ対応

| ビューポート幅     | レイアウト        |
| ----------- | ------------ |
| ≥ 1200 px   | 左右分割レイアウト    |
| 768–1199 px | 上下スタックレイアウト  |
| ≤ 767 px    | 単一カラム + タブ切替 |

---

## 5. CI / CD

| ステップ             | 内容                                                               |
| ---------------- | ---------------------------------------------------------------- |
| **lint & build** | `npm ci && npm run lint && npm run build` で静的チェック後 Vite ビルド      |
| **deploy**       | *peaceiris/actions-gh-pages* で `dist/` を `gh-pages` ブランチへ自動 push |

---

## 6. 開発・テスト方針

### 6.1 フロー概要

| フェーズ                   | 実施内容（自動／手動）                                                                                                 | 合格条件                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------- | -------------------------- |
| **ローカル検証**             | 1. `npm run dev` で起動しブラウザ表示を手動確認<br>2. **Claude Code** で自動 UI テスト<br>  • `curl` 200 だけでなく DOM 片を `grep` で検証 | UI 要素が想定どおり描画される           |
| **Feature → gh-pages** | 1. Feature ブランチを `gh-pages` ブランチへマージ & push<br>2. GitHub Pages に自動デプロイ<br>3. Claude Code + 人間で再度表示確認        | GitHub Pages でも UI/DOM が一致 |
| **develop 反映**         | gh-pages 上で OK を確認後、Feature ブランチを `develop` にマージ・push                                                       | develop へマージ完了             |
| **main リリース（任意）**      | ステージング OK 後 `develop` → `main` へマージし、本番ビルド & デプロイ                                                           | 本番 URL で UI/DOM が一致        |

### 6.2 テストガイドライン

1. **Claude Code スクリプト**

   * `curl -sL <url> | grep -q "<h1>Batting Lineup Optimizer</h1>"` のように、代表要素を検証
   * 主要ルート `/`, `/simulation`, `/history` をカバー
2. **人間による目視**

   * レイアウト崩れ・チャート描画・ドラッグ操作を確認
3. **失敗時の対応**

   * CI 失敗 → ブランチにフィードバック、修正後リラン
   * gh-pages 表示崩れ → develop へのマージを保留し、Feature ブランチで再修正



