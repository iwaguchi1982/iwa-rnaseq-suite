# RNA-Seq Suite Milestone Plan
Version: draft-0.1
Target: v1.0.0
Policy: v1.0.0 では LLM 補助を入れない

---

## 0. 前提整理

### 現在の認識
- `iwa-rnaseq-counter` と `iwa-rnaseq-reporter` を RNA-Seq Suite の中核とする
- counter は execution 系
- reporter は analytics / report 系
- 外部委託納品物の差異吸収は reporter 本体ではなく adapter 層で扱う
- v1.0.0 では AI / LLM 補助は入れない
- suite の完成条件は「実行・正規化・可視化・レポート化」が一貫して成立すること

### 重要な方針
- repo 個別 version と suite milestone を分けて考える
- 当面の真の進捗管理単位は suite milestone とする
- package version は rc 以降で整理する

---

## 1. v1.0.0 の定義

## 1.1 v1.0.0 で達成すること
RNA-Seq Suite v1.0.0 は、以下を満たした状態とする。

1. FASTQ から count matrix 生成までが標準 contract で実行できる
2. reporter が標準 contract を直接読める
3. reporter が主要な RNA-Seq 可視化・統計出力を再現できる
4. 外部委託納品物を adapter 経由で正規入力へ変換できる
5. レポート出力と machine-readable handoff が成立する
6. provenance と execution record が追跡できる
7. LLM なしでも現場運用できる

## 1.2 v1.0.0 で入れないもの
- LLM による要約補助
- AI warning narrative
- EvidenceRiskSpec 系
- ReviewDecisionSpec 系
- No-go / hold 判定 AI
- Claim audit
- public comparator の本格統合
- signature scorer の本格統合

---

## 2. マイルストン全体像

- v0.20 - v0.29: Suite 基盤固定フェーズ
- v0.30 - v0.39: Counter contract hardening フェーズ
- v0.40 - v0.49: Reporter core analytics hardening フェーズ
- v0.50 - v0.59: Vendor adapter / normalization フェーズ
- v0.60 - v0.69: Counter-Reporter 統合フェーズ
- v0.70 - v0.79: Report / Export / Handoff 安定化フェーズ
- v0.80 - v0.89: 運用性・例外系・回帰防止フェーズ
- v0.90 - v0.99: Release candidate フェーズ
- v1.0.0: RNA-Seq Suite Core Release

---

## 3. 各マイルストン詳細

## v0.20 - v0.29
### テーマ
Suite foundation / contract freeze

### 目的
RNA-Seq Suite を app 集合ではなく、契約でつながる suite として固定する。

### 主対象
- devmap 固定
- app boundary 固定
- 6 core spec の最小形固定
- versioning ルール固定
- schema naming ルール固定
- metadata / overlay / provenance の基本ルール固定

### 完了条件
- `AssaySpec`
- `MatrixSpec`
- `ExecutionRunSpec`
- `ComparisonSpec`
- `ResultSpec`
- `ReportPayloadSpec`

の最小 contract が文書化されている

- counter / reporter の責務境界が明文化されている
- vendor 差異は adapter で吸収する方針が固定されている
- suite milestone と package version を分離運用することが決まっている

### このフェーズの到達点
「何を作るか」ではなく「何を壊してはいけないか」が明確になる

---

## v0.30 - v0.39
### テーマ
Counter contract hardening

### 目的
counter を execution app として安定化する

### 主対象
- intake validation
- QC contract
- trimming contract
- backend abstraction
- matrix builder
- run recorder
- CLI / batch entrypoint の責務整理
- error / warning / log の標準化

### backend 対象
- STAR
- HISAT2
- kallisto
- salmon

### 完了条件
- 4 backend の出力が最終的に `MatrixSpec` に正規化される
- 実行結果が `ExecutionRunSpec` に残る
- 失敗時に status / log ref / error summary が追跡できる
- reporter が追加変換なしで読める最低限 bundle が成立する

### このフェーズの到達点
counter が「FASTQ 実行器」ではなく「標準 handoff 生成器」になる

---

## v0.40 - v0.49
### テーマ
Reporter core analytics hardening

### 目的
reporter の標準入力から主要解析を安定再現できるようにする

### 主対象
- normalized input loading
- metadata validation
- group / contrast validation
- exploratory analysis
- DEG
- volcano
- PCA
- correlation heatmap
- top DEG heatmap
- enrichment
- gene search
- result assembly
- payload assembly

### 完了条件
- `MatrixSpec + ComparisonSpec` から `ResultSpec + ReportPayloadSpec` が生成できる
- reporter 本体が raw vendor file を直接読まない
- UI / export が payload 中心で再構成可能になる
- figure / table / summary の責務境界が安定する

### このフェーズの到達点
reporter が「画面の寄せ集め」から「結果契約を返す解析機」へ変わる

---

## v0.50 - v0.59
### テーマ
Vendor adapter / normalization

### 目的
外部委託納品物を reporter 本体へ混入させずに吸収する

### 主対象
- vendor adapter interface
- file inspection
- column mapping
- sample ID normalization
- gene ID normalization
- group / contrast normalization
- warning rules
- manual import 補助
- import validation report

### 完了条件
- 少なくとも主要 2 系統の納品形式を adapter で取り込める
- adapter が `MatrixSpec / ComparisonSpec` を返せる
- reporter 本体に vendor 分岐を書かなくて済む
- metadata 不足時の warning 動線が決まる

### このフェーズの到達点
reporter が例外処理の墓場になるリスクを回避する

---

## v0.60 - v0.69
### テーマ
Counter-Reporter integration

### 目的
suite 内の往復を本当に成立させる

### 主対象
- counter native handoff
- reporter intake compatibility
- execution provenance chaining
- artifact lineage
- batch execution -> report generation の統合試験
- fixture による round-trip test

### 完了条件
- `FASTQ -> counter -> MatrixSpec -> reporter -> ResultSpec / ReportPayloadSpec`
  が通る
- suite 内の artifact lineage が追跡できる
- integration test が最小の代表ケースで通る
- compatibility break を検知できるテストがある

### このフェーズの到達点
suite と呼べるだけの連結が初めて成立する

---

## v0.70 - v0.79
### テーマ
Report / Export / Handoff stabilization

### 目的
現場で使える納品物と handoff を整える

### 主対象
- HTML export
- PDF export
- machine-readable JSON export
- summary table export
- handoff packet
- rerun / reanalysis に必要な最低情報の整備
- report slot / template slot の整理

### 完了条件
- 人向けレポートを出せる
- 機械向け payload を出せる
- 後工程へ渡せる handoff packet を出せる
- 出力の source of truth が `ReportPayloadSpec` に揃う

### このフェーズの到達点
「アプリで見る」だけでなく「成果物として渡せる」状態になる

---

## v0.80 - v0.89
### テーマ
Operational hardening / regression defense

### 目的
v1.0.0 前の事故を防ぐ

### 主対象
- regression tests
- fixture 強化
- exception policy
- malformed input handling
- missing metadata handling
- backend mismatch handling
- invalid contrast handling
- release checklist
- migration note 整備

### 完了条件
- よくある入力破綻に対する挙動が定義されている
- 回帰テストが主要 use-case を覆っている
- bundle / payload / result の互換性チェックがある
- 「通るときだけ通る」状態を脱している

### このフェーズの到達点
日常開発で壊れにくい

---

## v0.90 - v0.99
### テーマ
Release candidate

### 目的
v1.0.0 を切れる状態に仕上げる

### 主対象
- package metadata 整理
- version policy 整理
- README / docs / install / usage 更新
- minimal demo data
- acceptance scenario 固定
- final bug fix
- final compatibility verification

### 完了条件
- counter / reporter の起動・実行・受け渡し・出力の手順が文書化されている
- 代表 workflow が再現できる
- known limitations が明文化されている
- v1.0.0 release note 草案が書ける

### このフェーズの到達点
技術的に完成しているだけでなく、公開・引継ぎ可能な状態になる

---

## v1.0.0
### テーマ
RNA-Seq Suite Core Release

### 構成
- `iwa-rnaseq-counter`
- `iwa-rnaseq-reporter`
- vendor adapter 基盤
- standard handoff / export
- core specs
- round-trip integration

### v1.0.0 の受け入れ条件
1. counter が 4 backend を標準 contract へ収束できる
2. reporter が標準入力から主要解析を再現できる
3. vendor adapter が外部委託差異を吸収できる
4. result / payload / execution provenance がつながる
5. HTML / PDF / machine-readable handoff が成立する
6. integration / regression が最低限揃っている
7. LLM なしで現場運用に耐える

---

## 4. v1.0.0 以降へ送るもの

以下は v1.0.0 の hard scope から外す。

### v1.1 以降候補
- `iwa-public-data-comparator`
- `iwa-signature-scorer`
- report template manager 高度化
- reanalysis launcher 強化

### v1.2 以降候補
- evidence-risk 系
- review decision 系
- LLM narrative assist
- evidence bundle
- no-go / hold support
- audit trail

---

## 5. 実務上の補足

### 5.1 進捗管理の基本単位
今後は repo 単位でなく、以下で管理する。

- suite milestone
- spec maturity
- integration maturity
- release readiness

### 5.2 version の考え方
- package version は release に近づくまで無理に milestone と一致させない
- 真の進捗は `v0.20 ... v1.0.0` の suite milestone で見る
- **Batch ID と package version の同期ルール**:
    - バッチのフェーズとインデックスをマイナーバージョンに投影する
    - 例: `B30-03` (Phase v0.30, Batch 03) -> `v0.33.x` (x は開発段階)
- rc 以降で package version を整理する

### 5.3 最重要リスク
最大の失敗パターンは以下。

- counter と reporter が別々に進み、hand off が壊れる
- reporter に vendor 例外処理が蓄積する
- result と report と export の source of truth が分裂する
- comparator や AI を早く入れすぎて core が崩れる

---

## 6. このロードマップの一文定義

RNA-Seq Suite v1.0.0 は、
LLM を使わずとも、
FASTQ 実行・標準化・可視化・納品物生成が一貫して成立する
core suite の完成版である。