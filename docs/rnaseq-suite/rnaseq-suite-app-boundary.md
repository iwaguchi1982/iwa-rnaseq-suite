# RNA-Seq Suite アプリ境界定義 (App Boundary)
Version: draft-0.1

## 1. 目的
この文書は、RNA-Seq Suite を構成する各アプリケーションの責務（Responsibility）と境界（Boundary）を正式に定義するものです。これにより、「責務の逆流」を防ぎ、クリーンでメンテナンス性の高いパイプラインを保証するためのアーキテクチャ上の Source of Truth となります。

## 2. 境界の原則
- **Contract-First**: アプリ間の相互作用は、実装の詳細ではなく、標準化された Spec（契約）を介して行われなければならない。
- **Single Source of Truth**: 各アーティファクトは明確な出自を持ち、決定論的な経路を辿らなければならない。
- **ポータビリティ (Portability)**: 契約（Contract）や出力において、絶対パスなどの環境依存情報を混ぜることは厳禁。
- **Minimal Workflow**: アプリ間のデータ受け渡しは [Minimal Workflow Contract Map](rnaseq-suite-minimal-workflow.md) に規定された「Middle Artifact の責務吸収層」を介すること。

## 3. Suite の 3 層構造

### 3.1 実行層 (Execution Layer)
生データの入り口。高パフォーマンスな処理と、実行履歴（Provenance）の記録を担う。
- **対象アプリ**: `iwa-rnaseq-counter`

### 3.2 標準化・取り込み層 (Normalization / Ingestion Layer)
外部の「混沌（ベンダーごとの差異）」と内部の「標準化」を繋ぐ架け橋。
- **対象レイヤー**: `Adapter Layer`

### 3.3 解析・レポート層 (Analytics / Report Layer)
標準化されたデータの消費者。統計的解釈と、人間が読めるレポート出力を担う。
- **対象アプリ**: `iwa-rnaseq-reporter`

---

## 4. iwa-rnaseq-counter (実行系)

### 役割
生データ (FASTQ) を標準化されたカウント/発現量マトリックスに変換し、詳細な実行履歴 (Provenance) を生成する実行エンジン。

### 責務 (In Scope)
- FASTQ およびサンプルシートの入力バリデーション。
- 前処理 (QC, トリミング)。
- 標準化されたバックエンド (STAR, Salmon 等) によるマッピング/定量。
- `MatrixSpec` および `ExecutionRunSpec` の生成。

### 非責務 (Out of Scope)
- 決定的な統計解析 (DEG)。
- 機能エンリッチメント解析。
- インタラクティブな可視化や叙述的なレポート作成。

### 厳格な非目標 (Hard Non-goals)
- 実験デザインに基づく統計的な意思決定を行わない。
- 人間向けのレポート (HTML/PDF) を出力せず、機械可読な契約（JSON）を出力主軸とする。

---

## 5. iwa-rnaseq-reporter (解析系)

### 役割
標準化されたデータを解釈し、統計結果、可視化、およびリサーチレポートを作成するための解析・レポート拠点。

### 責務 (In Scope)
- 標準化された `MatrixSpec` および `ComparisonSpec` の読み込み。
- 探索的データ解析 (PCA, 相関解析)。
- 統計解析 (DEG, エンリッチメント)。
- レポート作成 (HTML, PDF, Handoff バンドル)。

### 非責務 (Out of Scope)
- シーケンスのアラインメントや定量（STAR/Salmon等）の直接実行。
- 非標準的なベンダー固有ファイルの直接パース（Adapter 層に委譲）。

### 厳格な非目標 (Hard Non-goals)
- 生データ (FASTQ) を直接読み込まない。
- ベンダー固有のパースロジックを解析エンジン本体に持ち込まない。

---

## 6. Adapter Layer (標準化層)

### 役割
ベンダー固有の納品形式やレガシーなデータを、Suite の標準的な Spec（契約）へ正規化する層。

### 原則
- **隔離 (Isolation)**: ベンダーごとに異なる「煩雑な」ロジックを Reporter 本体から隔離する。
- **非解析的**: データの正規化に徹し、新たな統計量（例: DEseq2 の実行等）の算出は行わない。

---

## 7. 許可される境界 (Allowed Boundaries)
- Counter -> `MatrixSpec` + `ExecutionRunSpec` -> Reporter
- ベンダーデータ -> `Adapter` -> `MatrixSpec` -> Reporter
- Suite アプリ -> `標準 Handoff バンドル` -> 後続工程/AI

## 8. 禁止される境界 (Forbidden Boundaries)
- Reporter -> Counter 内部ロジックの直接インポート（将来的に Shared spec で解決）。
- Reporter -> ベンダー固有 CSV/JSON の直接読込。
- Counter -> 統計解析・レポート解釈ロジックの持ち込み。

---

## 9. 現状の実装実態と既知の結合リスク
B20-01 の監査により、以下の結合リスクが特定されました。
- **直接インポート**: `bundle_loader.py` が `iwa_rnaseq_counter` から直接クラスをインポートしている。
- **パッケージレベルの依存**: スイート全体で共有される `shared-spec` パッケージが欠如しており、相互参照が発生している。
- **テストの結合**: Reporter の統合テストが Counter の fixtures に物理パスで依存している。

## 10. B20-01 以降のフォローアップ
- [ ] パッケージレベルの疎結合を実現するための `iwa-rnaseq-spec` (仮) の抽出 (Phase v0.3x以降)。
- [ ] アドホックなパースを置き換えるための `AssaySpec` の正式化。

---

## 11. B20-01 Done 定義
スイートの論理アーキテクチャがこの文書で固定され、開発者向け README に反映された時点で B20-01 は完了。物理的なコードレベルの分離は後続バッチへ送る。
