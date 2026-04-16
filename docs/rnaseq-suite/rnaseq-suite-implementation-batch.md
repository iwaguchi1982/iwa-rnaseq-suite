# RNA-Seq Suite Phase-to-Batch Breakdown
Version: draft-0.1
Target: v1.0.0
Policy: v1.0.0 では LLM 補助を入れない
Core philosophy: 「賢い補助」ではなく「壊れない受け渡し」を完成条件に置く

---

## 0. この文書の位置づけ

この文書は、Phase Gate 判定表をさらに実装単位へ落としたものとする。  
各バッチは、以下を満たす粒度で切る。

- 1つの主責務に集中している
- commit review で確認点が明確
- source of truth が1つまたは少数に絞れる
- done / hold / reject 判定がしやすい
- 次バッチへの handoff が明確

---

## 1. バッチ設計の基本ルール

### 1.1 バッチの切り方
- spec 固定バッチ
- validator / fixture バッチ
- 実装本体バッチ
- integration バッチ
- export / handoff バッチ
- hardening / regression バッチ

### 1.2 バッチごとの最低記録
各バッチで最低限、以下を残す。

- ゴール
- 触る契約
- commit review で何を見るか
- source of truth
- 未確定事項
- 何を確認して決めるか
- 非対象
- done 条件

### 1.3 repo との対応
suite milestone と repo version は一致しなくてよい。  
この文書では、以下の感覚で割り当てる。

- counter 主体バッチ
- reporter 主体バッチ
- shared / suite バッチ
- adapter 主体バッチ
- release 主体バッチ

### 1.4 Source artifact identity policy
v1.0.0 では、治験用アプリのような厳密な「原本性」管理そのものは主目的にしない。  
ただし、将来の rerun / replay / provenance 確認に備えて、
**source artifact の同一性・完全性・由来追跡性**
を保持する軽量な仕組みを入れる。

ここで扱うのは、法的な原本性そのものではなく、以下の実務要件である。

- この run / report がどの入力ファイルに基づくか追える
- 受領時点のファイル識別情報を保持できる
- checksum により同一性確認ができる
- rerun packet から必要な入力参照へ戻れる
- 後で厳格運用に発展させても壊れにくい

### 1.5 軽量 record の扱い
v1.0.0 では、独立した重い監査 spec とせず、
`SourceArtifactRecord`
という軽量 record を provenance / sidecar / packet の一部として扱う。

### 1.6 SourceArtifactRecord の最小項目
- `artifact_id`
- `artifact_role`
- `original_filename`
- `received_at`
- `received_from` optional
- `storage_locator`
- `byte_size`
- `checksum_type`
- `checksum_value`
- `media_type`
- `format`
- `capture_mode`
- `metadata`

### 1.7 v1.0.0 ではやらないこと
- 電子的署名
- 承認ワークフロー
- WORM 保存前提の厳密運用
- 監査証跡の完全制度化
- 法的原本管理そのもの

---

# 2. Phase v0.20 - v0.29
# テーマ
Suite foundation / contract freeze

## このphaseの目的
- Suite 構成要素（Counter, Reporter, Adapter）の責務境界と 6 core spec の役割を固定する
- スキーマ規約（schema_name, version, metadata 等）を確立し、将来の拡張性と互換性の土台を作る
- 実装を開始するための最小限のワークフロー、レビュー形式、原本性記録（SourceArtifactRecord）の運用を固める

## バッチ一覧

### B20-01
**名称:** Suite skeleton / boundary fixed  
**主担当:** shared / suite

- 重要バッチ
- ゴール:
  - counter / reporter / adapter の責務境界を文章で固定する
- 触る契約:
  - app boundary 定義
  - suite mission
- commit review で何を見るか:
  - counter が execution、reporter が analytics、adapter が normalization になっているか
  - vendor 差異が reporter 本体へ入らない前提になっているか
- source of truth:
  - devmap
  - suite overview
- 未確定事項:
  - adapter の入口粒度
- 何を確認して決めるか:
  - 想定 vendor 納品物パターン
  - counter -> reporter handoff 想定
- 非対象:
  - spec の最小 field 定義の詳細化
  - vendor 個別 adapter 実装
  - validator / integration test の本実装
  - UI / export 設計
- done 条件:
  - app ごとの主責務 / 非責務が1文で言える
  - 責務逆流がない

---

### B20-02
**名称:** Core spec names fixed  
**主担当:** shared / suite

- 重要バッチ
- ゴール:
  - 6 core spec の名称と役割を固定する
- 触る契約:
  - AssaySpec
  - MatrixSpec
  - ExecutionRunSpec
  - ComparisonSpec
  - ResultSpec
  - ReportPayloadSpec
- commit review で何を見るか:
  - spec 名が揺れていないか
  - 責務が被っていないか
- source of truth:
  - core spec document
- 未確定事項:
  - 各 spec の最小必須 field
- 何を確認して決めるか:
  - round-trip 最小 use-case
- 非対象:
  - 各 spec の全 field 完全定義
  - validator 実装
  - app 内部リファクタ
  - 疾患固有語彙の導入
- done 条件:
  - 各 spec の責務が1文で固定されている

---

### B20-03
**名称:** Common schema conventions  
**主担当:** shared / suite

- 重要バッチ
- ゴール:
  - schema_name / schema_version / metadata / overlay / provenance の基本ルールを固定する
- 触る契約:
  - 全 spec 共通ヘッダ
- commit review で何を見るか:
  - 拡張逃がし先があるか
  - 破壊的変更時の version 扱いが定義されているか
- source of truth:
  - schema convention 文書
- 未確定事項:
  - provenance の最小粒度
- 何を確認して決めるか:
  - 実行ログと report consumer の必要粒度
- 非対象:
  - app ごとの個別例外処理
  - migration ツール実装
  - execution / report ロジック実装
  - 監査特化機能
- done 条件:
  - 全 spec に共通ルールが適用できる

---

### B20-04
**名称:** Minimal workflow contract map  
**主担当:** shared / suite

- 軽量バッチ
- ゴール:
  - `FASTQ -> counter -> MatrixSpec -> reporter -> ResultSpec / ReportPayloadSpec`
    を図として固定する
- 触る契約:
  - workflow / handoff 定義
- commit review で何を見るか:
  - 中間 artifact の役割が曖昧でないか
  - app をまたぐ truth が増えていないか
- source of truth:
  - representative workflow 図
- 未確定事項:
  - ExecutionRunSpec をどこまで両 app で持つか
- 何を確認して決めるか:
  - 現行 counter / reporter 実装の中間成果物
- 非対象:
  - backend 詳細実装
  - HTML / PDF / JSON export 設計
  - UI 画面設計
  - vendor 個別分岐
- done 条件:
  - handoff の中身を図で説明できる

---

### B20-05
**名称:** Phase 0 acceptance seed  
**主担当:** shared / suite

- 軽量バッチ
- ゴール:
  - この後の Phase で使う最低限の acceptance review 形式を作る
- 触る契約:
  - batch review template
  - phase review template
- commit review で何を見るか:
  - done / hold / reject の根拠が書ける形か
- source of truth:
  - review template
- 未確定事項:
  - どこまでを batch レベルで mandatory にするか
- 何を確認して決めるか:
  - 実際のレビュー運用
- 非対象:
  - 各バッチの個別 done 条件の最終確定
  - 自動 gate 実装
  - release checklist 本体
  - repo 固有運用ルールの固定
- done 条件:
  - 各バッチで同じ観点を使い回せる

---

### B20-06
**名称:** Source artifact identity convention  
**主担当:** shared / suite

- 重要バッチ
- ゴール:
  - source artifact をどう記録するかの共通ルールを固定する
- 触る契約:
  - SourceArtifactRecord
  - provenance convention
- commit review で何を見るか:
  - 「原本性」ではなく軽量な identity / integrity 記録に留まっているか
  - app ごとに別方言の artifact 記録を作っていないか
  - checksum / locator / received_at の扱いが共通化されているか
- source of truth:
  - source artifact identity policy
  - schema convention 文書
- 未確定事項:
  - `SourceArtifactRecord` を sidecar に置くか provenance subobject に置くか
  - `received_from` を必須にするか optional にするか
- 何を確認して決めるか:
  - 実際に受領する FASTQ / metadata / vendor files の運用
  - rerun / replay 時に最低限必要な識別情報
- 非対象:
  - 電子署名
  - 承認ワークフロー
  - WORM 前提保存
  - 法的原本管理そのもの
- done 条件:
  - source artifact 記録の最小項目が固定されている
  - counter / reporter / adapter が同じ前提で参照できる
  - 将来の厳格運用へ拡張可能な形になっている

---

# 3. Phase v0.30 - v0.39
# テーマ
Counter contract hardening

## このphaseの目的
- FASTQ と AssaySpec に基づく入力バリデーション（Intake）を Counter の境界で完結させる
- 実行痕跡（ExecutionRunSpec）と結果（MatrixSpec）を分離し、 provenance と入力 origin への追跡性を担保する
- 複数の定量化バックエンド（STAR, Salmon等）の差異を隠蔽し、一貫した MatrixSpec を生成する基盤を構築する

## バッチ一覧

### B30-01
**名称:** Counter intake contract  
**主担当:** counter

- 重要バッチ
- ゴール:
  - FASTQ 入力と assay 入力条件を intake で閉じる
- 触る契約:
  - AssaySpec
- commit review で何を見るか:
  - input validation が入口で完結しているか
  - 後段へ壊れた入力を流していないか
  - intake 時点で source artifact を記録できるか
  - FASTQ / sample sheet / metadata sheet などの入力識別情報が落ちていないか
  - checksum と locator を後段で辿れるか
- source of truth:
  - AssaySpec
  - intake fixture
  - source artifact capture rule
- 未確定事項:
  - 必須 field の最小集合
- 何を確認して決めるか:
  - 代表 run 条件
- 非対象:
  - QC / trimming の詳細実装
  - backend abstraction
  - MatrixSpec 生成
  - report / 可視化ロジック
- done 条件:
  - invalid input を入口で止められる
  - 入力 artifact の identity 情報を intake で固定できる
---

### B30-02
**名称:** QC / trimming boundary  
**主担当:** counter

- 重要バッチ
- ゴール:
  - QC と trimming を実行系責務として切り分ける
- 触る契約:
  - ExecutionRunSpec
- commit review で何を見るか:
  - QC 結果と最終 matrix を混ぜていないか
  - trimming 情報が provenance に残るか
- source of truth:
  - ExecutionRunSpec
  - QC fixture
- 未確定事項:
  - warning と fatal の境界
- 何を確認して決めるか:
  - 実運用で止めたい失敗条件
- 非対象:
  - MatrixSpec builder
  - backend 差異吸収の最終実装
  - reporter 向け warning narrative
  - export 用 summary
- done 条件:
  - QC / trimming の実行痕跡が追跡できる

---

### B30-03
**名称:** Backend abstraction base  
**主担当:** counter

- 重要バッチ
- ゴール:
  - STAR / HISAT2 / kallisto / salmon を共通抽象で扱う土台を作る
- 触る契約:
  - backend interface
- commit review で何を見るか:
  - backend ごとの差異を interface に閉じ込められているか
- source of truth:
  - backend abstraction 定義
- 未確定事項:
  - transcript / gene level 差分の表し方
- 何を確認して決めるか:
  - 4 backend の代表出力
- 非対象:
  - 4 backend の完全パリティ保証
  - matrix 生成ロジック
  - report / export 責務
  - rerun packet 組み立て
- done 条件:
  - backend 追加時の差し込み口が明確

---

### B30-04
**名称:** MatrixSpec minimal builder  
**主担当:** counter

- 軽量バッチ
- ゴール:
  - backend 出力を MatrixSpec に落とす最小 builder を作る
- 触る契約:
  - MatrixSpec
- commit review で何を見るか:
  - matrix に log や UI 用情報が混ざっていないか
- source of truth:
  - MatrixSpec
  - representative matrix fixture
- 未確定事項:
  - normalization_state の扱い
- 何を確認して決めるか:
  - reporter が実際に読む項目
- 非対象:
  - QC summary や report 用 payload 生成
  - backend 固有便利情報の混入
  - rerun packet 本体
  - UI / export 向け整形
- done 条件:
  - reporter へ渡す最小 matrix が組める

---

### B30-05
**名称:** ExecutionRunSpec recorder  
**主担当:** counter

- 重要バッチ
- ゴール:
  - 実行条件・ツール・ログ・status を ExecutionRunSpec として残す
- 触る契約:
  - ExecutionRunSpec
- commit review で何を見るか:
  - matrix と execution 情報が分離されているか
  - 失敗時にも log ref が残るか
  - rerun に必要な backend / reference / parameter / environment 情報が ExecutionRunSpec に残るか
  - run がどの source artifact を使ったか追跡できるか
- source of truth:
  - ExecutionRunSpec
  - failure fixture
  - source artifact refs
- 未確定事項:
  - resource usage の最小粒度
- 何を確認して決めるか:
  - 障害切り分けに必要な項目
- 非対象:
  - reporter 用 payload 生成
  - consumer handoff packet 生成
  - 法的監査証跡の制度化
  - スケジューラ抽象の全面設計
- done 条件:
  - success / failure 両方で追跡可能
  - 将来の rerun packet に必要な実行情報を欠かさない
  - source artifact ref と run record が接続される

---

### B30-06
**名称:** Counter validator / regression seed  
**主担当:** counter

- 重要バッチ
- ゴール:
  - counter contract の validator と回帰テストの土台を置く
- 触る契約:
  - AssaySpec
  - MatrixSpec
  - ExecutionRunSpec
- commit review で何を見るか:
  - validator が contract を実際に見ているか
- source of truth:
  - validator
  - regression fixtures
- 未確定事項:
  - 何を blocker にするか
- 何を確認して決めるか:
  - 過去の壊れ方
- 非対象:
  - full coverage の回帰網羅
  - reporter 側 validator 強化
  - release blocker の最終判定
  - docs 整備
- done 条件:
  - 少なくとも代表正常系 / 代表異常系がある

---

# 4. Phase v0.40 - v0.49
# テーマ
Reporter core analytics hardening

## このphaseの目的
- 統計コア（ResultSpec）とレポート事実（ReportPayloadSpec）の役割を分離し、解析の「事実の主語」を固定する
- UIや表示閾値に汚染されない、DEG（有意差発現解析）の純粋な統計結果としての core result を確立する
- レポートを再構成するためのペイロード基盤（Identity, Summary, Context, Narrative）を構築し、レポートの原本性を支える

## バッチ一覧

### B40-01
**名称:** Reporter normalized intake  
**主担当:** reporter

- 重要バッチ
- ゴール:
  - reporter が正規化済み MatrixSpec / ComparisonSpec のみを入口にする
- 触る契約:
  - MatrixSpec
  - ComparisonSpec
- commit review で何を見るか:
  - raw vendor file 直読みに戻っていないか
- source of truth:
  - normalized intake definition
- 未確定事項:
  - ComparisonSpec 最小 design_type
- 何を確認して決めるか:
  - 代表 use-case
- 非対象:
  - vendor 正規化
  - UI 層の設計修正
  - export 実装
  - AI / narrative 補助
- done 条件:
  - intake が spec ベースで閉じる

---

### B40-02
**名称:** Comparison validation core  
**主担当:** reporter

- 重要バッチ
- ゴール:
  - group / contrast / covariate の検証を UI 手前で終わらせる
- 触る契約:
  - ComparisonSpec
- commit review で何を見るか:
  - UI 層で設計を再解釈していないか
- source of truth:
  - ComparisonSpec
  - validation fixture
- 未確定事項:
  - invalid contrast の扱い
- 何を確認して決めるか:
  - 現場で必要な比較設計
- 非対象:
  - DEG 本体実装
  - payload assembly
  - vendor 由来不足情報の自動補完
  - export 用レイアウト設計
- done 条件:
  - 不正設計を前段で止められる

---

### B40-03
**名称:** Exploratory analysis core  
**主担当:** reporter

- 軽量バッチ
- ゴール:
  - PCA / correlation を正規入力だけで再現する
- 触る契約:
  - ResultSpec
  - ReportPayloadSpec
- commit review で何を見るか:
  - plot ごとに独自入力を要求していないか
- source of truth:
  - representative exploratory outputs
- 未確定事項:
  - figure_refs 粒度
- 何を確認して決めるか:
  - 現場で最低限必要な exploratory 可視化
- 非対象:
  - DEG / enrichment 実装
  - HTML / PDF 出力
  - narrative 生成
  - vendor 個別分岐対応
- done 条件:
  - PCA / correlation が spec から再現できる

---

### B40-04
**名称:** DEG result core  
**主担当:** reporter
**バッチ種別:** 重要バッチ

- ゴール:
  - DEG を ResultSpec に格納する最小実装を固める
- 触る契約:
  - ResultSpec
- commit review で何を見るか:
  - DEG 表と figure 参照が切り分けられているか
- source of truth:
  - ResultSpec
  - DEG fixture
- 未確定事項:
  - thresholds の持ち方
- 何を確認して決めるか:
  - 現行 DEG 出力
- 関連する既決事項:
  - B40-01: reporter intake boundary 確定済み
  - B40-03: plot の SOT 整理済み
- 影響するレイヤー:
  - Core / Interface / Test
- 非対象:
  - exploratory 可視化の拡張
  - export レイアウト調整
  - signature / comparator 接続
  - AI コメント生成
- done 条件:
  - DEG 結果が result object として保持できる

---

### B40-05
**名称:** Volcano / heatmap / enrichment linkage  
**主担当:** reporter
**バッチ種別:** 軽量バッチ

- ゴール:
  - DEG 系可視化と enrichment を ResultSpec / PayloadSpec に接続する
- 触る契約:
  - ResultSpec
  - ReportPayloadSpec
- commit review で何を見るか:
  - 可視化の truth が result/payload に乗っているか
- source of truth:
  - representative result-output mapping
- 未確定事項:
  - enrichment の共通化範囲
- 何を確認して決めるか:
  - 現場の report 構成
- 非対象:
  - public comparator 統合
  - signature scorer 統合
  - AI 解釈補助
  - consumer handoff 設計
- done 条件:
  - 主要解析の参照関係が追える

---

### B40-06
**名称:** Payload assembly base  
**主担当:** reporter
**バッチ種別:** 重要バッチ

- ゴール:
  - ReportPayloadSpec を report source of truth として組み立てる
- 触る契約:
  - ReportPayloadSpec
- commit review で何を見るか:
  - UI / export / summary の truth が payload に寄っているか
- source of truth:
  - ReportPayloadSpec
  - 現行 UI summary / export preview の共通要素
  - B40-04 の ResultSpec 出力
- 未確定事項:
  - narrative_slots の最小形
- 何を確認して決めるか:
  - HTML / PDF / UI で共通に使う要素
- 関連する既決事項:
  - B40-01: Boundary / Context 基盤
  - B40-04: ResultSpec (DEG)
- 影響するレイヤー:
  - Interface / UI / Docs
- 非対象:
  - HTML / PDF レンダリング最終化
  - LLM narrative
  - downstream packet 設計
  - 監査用 rerun 情報の組み込み
- done 条件:
  - payload から report を再構成できる見通しが立つ

---

# 5. Phase v0.50 - v0.59
# テーマ
Vendor adapter / normalization

## このphaseの目的
- 外部納品物（Vendor data）を Suite 共通契約（MatrixSpec, ComparisonSpec）へ落とす Adapter インターフェースを固定する
- IDの揺れやメタデータの欠損に対する正規化・検査ポリシー（Warning/Reject）を確立し、入力の信頼性を担保する
- 主要外部データソースを Reporter の正規 Intake へ接続可能にすることで、外部データの統合解析パスを実証する

**B50-01～09のバッチはv1.0.0で対応予定 B50-0a-0cとは切り分けて考える**

## バッチ一覧

### B50-0a-0cの目的/前提
- **B50 本体停止、B50-0a〜0c は adapter skeleton 契約整備 phase**と再定義** 26.04.16(確定)
- 実装可能成果物は docs / dataclass / validation skeleton / test seed まで、非対象は importer 本体・vendor 個別 parser・manual import UI
- B50-0a〜0c の最小保証は count matrix 前提。TPM / FPKM 一般対応は B50 本体再開時の論点（現場の要求を再整理。研究本部間で要求が跨いでいるため）
- B50-0c は reporter 新入口にしない

**B50-01～06のバッチはv1.0.0で対応予定 B50-0a-0cとは別で分けておく**

### B50-0a
**名称:** Adapter intake contract seed  
**主担当:** adapter / shared  
**バッチ種別:** 重要バッチ

- ゴール:
  - 外部取り込み本体を止めたまま、最小受け入れ条件と adapter の入口契約を先に固定する
  - 「このデータ（count matrix と sample id 入り metadata）があれば最低限の解析表示が可能」と明示できる状態にする
  - B50-01 をそのまま再開するのではなく、**B50-01 の前段として scope を絞った contract seed** を整備する
- 触る契約:
  - adapter intake contract
  - minimal intake requirement
- commit review で何を見るか:
  - importer を作り始めていないか
  - contract-first の入口定義に徹しているか
  - core spec 側へ責務が漏れていないか
  - adapter skeleton の責務 / 非責務が曖昧になっていないか
- source of truth:
  - minimal intake requirement v0.1
  - count matrix / metadata の受け入れ条件
  - B50 一時停止方針
- 未確定事項:
  - required columns の最終一覧
  - alias 許容範囲
- 何を確認して決めるか:
  - 現場ユーザーが実際に渡してくる最小データ構成
  - metadata の実運用上の列名ゆれ
- 関連する既決事項:
  - B40-01: Intake 構造の汎用化
  - B50 一時停止方針: 外部取り込み本体は v1.0.0 以降へ移動
- 影響するレイヤー:
  - Core / Interface / Docs
- 非対象:
  - vendor 個別 parser 実装
  - 実ファイル読込ロジック
  - manual import UI
  - comparison 推定
  - metadata 意味推論
- done 条件:
  - count matrix + sample id 入り metadata を最小受け入れ条件として文書化できる
  - csv / tsv / excel を許容形式として明示できる
  - 必須列 / 許容別名 / 変換名方針を manual に記載できる
  - **adapter skeleton の責務 / 非責務を文章で定義できる**

---

### B50-0b
**名称:** Adapter preflight validation skeleton  
**主担当:** adapter  
**バッチ種別:** 重要バッチ

- ゴール:
  - core に渡す前に、外部入力の曖昧さを止める preflight / validation skeleton を定義する
  - 読める / 読めない / 通せる / 通せない の境界を説明可能にする
  - comparison や semantics に踏み込まず、**external input preflight** に責務を限定する
- 触る契約:
  - import preflight contract
  - validation result skeleton
- commit review で何を見るか:
  - validation が解析ロジック化していないか
  - warning / reject の入口整理に留まっているか
  - comparison や cohort semantics を勝手に補っていないか
  - file format / required columns / sample id 一致確認の preflight に留まっているか
- source of truth:
  - minimal intake requirement v0.1
  - representative metadata patterns
  - B40-02 の comparison validation 境界
- 未確定事項:
  - warning と reject の最終境界
  - 欠損 / 重複 / 型崩れの fatal 条件
- 何を確認して決めるか:
  - sample id 欠損・重複の発生パターン
  - metadata 必須列不足の頻度
- 関連する既決事項:
  - B50-0a: Adapter intake contract seed
  - B40-02: Comparison validation
- 影響するレイヤー:
  - Core / Test / Docs
- 非対象:
  - sample / gene 正規化本体
  - comparison 自動生成
  - clinical semantics 解釈
  - vendor 個別最適化
  - reporter 向け解釈ロジック
  - design の意味解釈
  - comparison の semantic 妥当性判定
- done 条件:
  - sample id 照合、必須列確認、欠損 / 重複チェックの preflight 項目が定義されている
  - warning / reject 候補の入口境界を記述できる
  - 「読める」と「安全に解釈できる」を分けて説明できる
  - **provenance 最低限保持の preflight 観点を定義できる**

---

### B50-0c
**名称:** Adapter handoff payload skeleton  
**主担当:** adapter / shared  
**バッチ種別:** 重要バッチ

- ゴール:
  - adapter が core に渡す前の handoff-ready payload の骨格だけを定義する
  - adapter と core の責務境界を固定し、外部入力の曖昧さを core spec に持ち込まないようにする
  - adapter 境界の内部契約を定義しつつ、**最終 handoff は `MatrixSpec / ComparisonSpec` に収束する** 前提を明示する
- 触る契約:
  - AdapterPayloadSpec
  - ImportCandidateSpec
- commit review で何を見るか:
  - adapter が core spec を直接拡張していないか
  - payload が normalization / validation 前提の中間契約に留まっているか
  - disease-specific semantics が混入していないか
  - **B50-0c が reporter の新しい入口になっていないか**
  - **`AdapterPayloadSpec` / `ImportCandidateSpec` が canonical core spec として扱われていないか**
- source of truth:
  - adapter intake contract
  - validation skeleton
  - core handoff boundary
  - B40-01 で固定した reporter intake 境界
- 未確定事項:
  - payload に含める provenance 詳細度
  - normalization readiness の最終表現
- 何を確認して決めるか:
  - core 側が最低限必要とする handoff 情報
  - adapter 側で止めるべき曖昧さの範囲
- 関連する既決事項:
  - B50-0a: Adapter intake contract seed
  - B50-0b: Adapter preflight validation skeleton
  - B50-01: Adapter interface fixed
  - B40-01: reporter intake は `MatrixSpec + ComparisonSpec`
- 影響するレイヤー:
  - Core / Interface
- 非対象:
  - 本番 import pipeline
  - vendor 固有 parser
  - comparison の自動発明
  - disease / cohort 解釈の注入
  - report / DEG / plot ロジック
  - **reporter の新しい intake 契約の導入**
  - **`AdapterPayloadSpec` / `ImportCandidateSpec` の canonical core spec 化**
- done 条件:
  - adapter → core の handoff-ready payload 骨格が定義されている
  - adapter の責務と非責務を明示できる
  - **`AdapterPayloadSpec` / `ImportCandidateSpec` は adapter 境界の内部契約であり、suite の canonical core spec ではないと明記できる**
  - **最終 handoff は `MatrixSpec / ComparisonSpec` に収束すると明記できる**
  - 本実装再開時に B50-01 以降へ安全に接続できる

> [!WARNING]
> **B50-0c の最大リスクは、便利さのために `AdapterPayloadSpec` / `ImportCandidateSpec` がそのまま reporter の新しい入口として使われてしまうことにある。**
>
> これは B40-01 で固定した  
> **「reporter の入口は `MatrixSpec + ComparisonSpec`」**  
> という責務境界に逆行し、adapter 境界の内部契約を suite 全体の新しい source of truth に変質させる。
>
> したがって B50-0c では、  
> - `AdapterPayloadSpec` / `ImportCandidateSpec` は **adapter 内部の中間契約** に留める  
> - reporter は **これらを直接 intake しない**  
> - 最終的に reporter へ渡る契約は **`MatrixSpec / ComparisonSpec` に収束** させる  
>
> ことを強く固定する。
>
> **B50-0c は handoff 境界の整理であって、reporter の新入口追加ではない。**

---

### B50-01
**名称:** Adapter interface fixed  
**主担当:** adapter / shared
**バッチ種別:** 重要バッチ

- ゴール:
  - vendor adapter の共通入口 / 共通出口を固定する
- 触る契約:
  - adapter interface
- commit review で何を見るか:
  - adapter が normalization 専用責務になっているか
- source of truth:
  - adapter interface 定義
- 未確定事項:
  - manual import の入口位置
- 何を確認して決めるか:
  - 実納品物パターン
- 関連する既決事項:
  - B40-01: Intake 構造の汎用化
- 影響するレイヤー:
  - Core / Interface
- 非対象:
  - vendor 個別 parser 実装
  - DEG / plot / report ロジック
  - manual import UI
  - reject policy の最終確定
- done 条件:
  - vendor ごとの差異を同じ口で受けられる

---

### B50-02
**名称:** Vendor file inspection  
**主担当:** adapter
**バッチ種別:** 重要バッチ

- ゴール:
  - ファイル存在、シート、列、必須項目の検査を作る
- 触る契約:
  - import validation report
- commit review で何を見るか:
  - 解析ロジックが混ざっていないか
- source of truth:
  - representative vendor files
- 未確定事項:
  - reject にする欠損条件
- 何を確認して決めるか:
  - 納品物の不足頻度
- 関連する既決事項:
  - B50-01: Adapter interface
- 影響するレイヤー:
  - Core / Test
- 非対象:
  - sample / gene 正規化本体
  - comparison 推定
  - reporter 向け解釈ロジック
  - 欠損自動補完ルールの固定
- done 条件:
  - 読める / 読めない理由を説明できる

---

### B50-03
**名称:** Sample / gene normalization  
**主担当:** adapter
**バッチ種別:** 重要バッチ

- ゴール:
  - sample ID / gene ID の揺れを正規化する
- 触る契約:
  - MatrixSpec
- commit review で何を見るか:
  - mapping 記録が残るか
- source of truth:
  - normalization rules
  - mapping fixture
- 未確定事項:
  - どこまで自動補正するか
- 何を確認して決めるか:
  - 実データの揺れパターン
- 関連する既決事項:
  - B50-01: Adapter interface
- 影響するレイヤー:
  - Core / Interface
- 非対象:
  - 解析ロジック
  - 過剰な自動補正
  - reject policy の最終確定
  - report 用要約生成
- done 条件:
  - 正規化痕跡が追える

---

### B50-04
**名称:** Group / comparison normalization  
**主担当:** adapter
**バッチ種別:** 重要バッチ

- ゴール:
  - vendor 納品物から ComparisonSpec を作る
- 触る契約:
  - ComparisonSpec
- commit review で何を見るか:
  - comparison 不足時に黙って補完していないか
- source of truth:
  - ComparisonSpec
  - vendor mapping fixture
- 未確定事項:
  - 不足比較情報の扱い
- 何を確認して決めるか:
  - 現場が最低限必要とする group 情報
- 関連する既決事項:
  - B50-01: Adapter interface
  - B40-02: Comparison validation
- 影響するレイヤー:
  - Core / Interface
- 非対象:
  - DEG / report / export 生成
  - unsupported design の全面対応
  - silent な比較補完
  - 監査用メタデータ設計
- done 条件:
  - vendor data を正規 comparison へ落とせる

---

### B50-05
**名称:** Warning / reject policy  
**主担当:** adapter / shared
**バッチ種別:** 重要バッチ

- ゴール:
  - import 不備を warning で通すか reject するかの基準を固定する
- 触る契約:
  - import validation policy
- commit review で何を見るか:
  - 境界が一貫しているか
- source of truth:
  - warning / reject policy
- 未確定事項:
  - manual override の可否
- 何を確認して決めるか:
  - 実際の欠損ケース
- 関連する既決事項:
  - B50-01 〜 B50-04 一連の正規化フロー
- 影響するレイヤー:
  - Interface / Docs
- 非対象:
  - vendor parser 実装
  - UI override
  - 法務・監査向け分類制度
  - reporter 本体の例外分岐追加
- done 条件:
  - 境界が説明できる

---

### B50-06
**名称:** First vendor adapters  
**主担当:** adapter
**バッチ種別:** 軽量バッチ

- ゴール:
  - 主要 1〜2 系統の vendor 形式を実際に通す
- 触る契約:
  - MatrixSpec
  - ComparisonSpec
- commit review で何を見るか:
  - adapter 実装が reporter 本体へ漏れていないか
- source of truth:
  - vendor fixtures
  - normalized output fixtures
- 未確定事項:
  - 対応優先 vendor 順
- 何を確認して決めるか:
  - 実運用で頻出する委託先
- 非対象:
  - 広範な vendor 網羅
  - OCR / PDF 取り込み
  - reporter 本体への special case 追加
  - manual import の無制限拡張
- done 条件:
  - 正規化後に internal と同じ入口へ流せる

---

# 6. Phase v0.60 - v0.69
# テーマ
Counter-Reporter integration

## このphaseの目的
- Counter の出力と Reporter の入力を直接つなぐ Native Round-trip を正常系・異常系含めて完遂し、Suite 統合を実証する
- 入力原本（Source Artifact）から最終成果物までの lineage を手作業なしで追跡可能にする
- Spec Version 互換性チェック（compatibility check）により、アプリ間のバージョン不整合を検知・防御する

## バッチ一覧

### B60-01
**名称:** Native round-trip happy path  
**主担当:** shared / counter / reporter
**バッチ種別:** 重要バッチ

- ゴール:
  - counter native 生成物を reporter がそのまま読む happy path を通す
- 触る契約:
  - MatrixSpec
  - ComparisonSpec
  - ResultSpec
  - ReportPayloadSpec
- commit review で何を見るか:
  - 中間 shim が必要になっていないか
- source of truth:
  - round-trip integration fixture
- 未確定事項:
  - 最小代表 run
- 何を確認して決めるか:
  - 代表 FASTQ case
- 関連する既決事項:
  - B40 (Reporter), v0.3 (Counter) 各自の安定化
- 影響するレイヤー:
  - Core / Interface / Test
- 非対象:
  - export polishing
  - vendor normalized 系統の統合
  - archived rerun / replay 試験
  - deliverable 最終化
- done 条件:
  - end-to-end 正常系が1本通る

---

### B60-02
**名称:** Handoff artifact lineage  
**主担当:** shared
**バッチ種別:** 重要バッチ

- ゴール:
  - counter から reporter へ渡る artifact lineage を辿れるようにする
  - source artifact -> run -> matrix -> result -> payload の lineage が辿れるか
  - 中間生成物だけでなく入力 origin まで戻れるか
- 触る契約:
  - lineage definition
  - source artifact refs
  - representative handoff artifact
- commit review で何を見るか:
  - どの run からどの result が来たか追えるか
- source of truth:
  - lineage definition
- 未確定事項:
  - 表示粒度
- 何を確認して決めるか:
  - 障害調査の導線
- 関連する既決事項:
  - B60-01: Native round-trip
- 影響するレイヤー:
  - Core / Interface / UI
- 非対象:
  - UI 上の lineage 表示完成
  - rerun packet 組み立て
  - deliverable traceability 実装
  - 監査制度化
- done 条件:
  - lineage が手作業なしで追える
  - 少なくとも代表ケースで入力 origin まで戻れる

---

### B60-03
**名称:** Version compatibility check  
**主担当:** shared
**バッチ種別:** 重要バッチ

- ゴール:
  - spec version の整合チェックを入れる
- 触る契約:
  - schema_version compatibility
- commit review で何を見るか:
  - version 不整合が黙って通らないか
- source of truth:
  - compatibility rule
  - validator
- 未確定事項:
  - minor / patch の互換性ポリシー
- 何を確認して決めるか:
  - 既存 change pattern
- 関連する既決事項:
  - B60-01 (Happy path integration)
- 影響するレイヤー:
  - Core / Interface
- 非対象:
  - schema の再設計
  - migration tooling
  - round-trip 外の validator 強化
  - release version policy の最終化
- done 条件:
  - 互換性 break を検知できる

---

### B60-04
**名称:** Integration negative cases  
**主担当:** shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - 欠損 / 不整合 / invalid contrast の統合異常系を確認する
- 触る契約:
  - integration error handling
- commit review で何を見るか:
  - 失敗時に原因が追えるか
- source of truth:
  - negative integration fixtures
- 未確定事項:
  - blocker にする異常系の優先順位
- 何を確認して決めるか:
  - 過去に起こりそうな破綻
- 非対象:
  - 全 failure taxonomy の完成
  - release blocker 定義
  - UI エラーメッセージの最終調整
  - docs 整備
- done 条件:
  - 少なくとも代表異常系が定義される

---

### B60-05
**名称:** Suite minimal acceptance path  
**主担当:** shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - suite と呼べる最小 acceptance path を固定する
- 触る契約:
  - end-to-end acceptance scenario
- commit review で何を見るか:
  - この path が今後の基準として使えるか
- source of truth:
  - acceptance scenario
- 未確定事項:
  - 代表ケース数
- 何を確認して決めるか:
  - 正常系 / 異常系の代表性
- 非対象:
  - full release scenario
  - README / demo data 整備
  - vendor 個別 acceptance 拡張
  - export 最終仕様の固定
- done 条件:
  - 毎回通す最小統合ルートが決まる

---

# 7. Phase v0.70 - v0.79
# テーマ
Report / Export / Handoff stabilization

## このphaseの目的
- ReportPayloadSpec を起点に、HTML/PDFレポートと機械向けの JSON Export (Handoff) を一貫した事実から生成する
- 「1年後の再解析を保証する」ための Rerun/Replay 用パケット定義を確立し、再現性情報の網羅性を担保する
- 成果物、プロット、元の数値データの対応関係（Traceability）を確定させ、レビュー・監査可能な納品物品質に引き上げる

## バッチ一覧

### B70-01
**名称:** Payload-to-HTML path  
**主担当:** reporter
**バッチ種別:** 軽量バッチ

- ゴール:
  - HTML を ReportPayloadSpec から構成できるようにする
- 触る契約:
  - ReportPayloadSpec
- commit review で何を見るか:
  - HTML 専用 truth が生えていないか
- source of truth:
  - ReportPayloadSpec
  - representative HTML
- 未確定事項:
  - セクション最小構成
- 何を確認して決めるか:
  - 現場が必要とする report 構成
- 非対象:
  - PDF 生成
  - machine-readable export
  - rerun 用メタデータ同梱
  - report template 高度化
- done 条件:
  - HTML が payload 依存で成立する

---

### B70-02
**名称:** Payload-to-PDF path  
**主担当:** reporter
**バッチ種別:** 軽量バッチ

- ゴール:
  - PDF を payload から構成できるようにする
- 触る契約:
  - ReportPayloadSpec
- commit review で何を見るか:
  - PDF 専用の別 truth が増えていないか
- source of truth:
  - representative PDF
- 未確定事項:
  - PDF 固有表現の許容範囲
- 何を確認して決めるか:
  - 納品物イメージ
- 非対象:
  - HTML レンダリング再設計
  - machine-readable handoff
  - rerun packet
  - consumer packet 定義
- done 条件:
  - PDF と payload のトレースが取れる

---

### B70-03
**名称:** Machine-readable export  
**主担当:** reporter / shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - downstream 向け JSON / handoff を payload / result から生成する
- 触る契約:
  - ResultSpec
  - ReportPayloadSpec
- commit review で何を見るか:
  - export 用の暫定整形が core contract を壊していないか
- source of truth:
  - export definition
- 未確定事項:
  - downstream consumer の最小要件
- 何を確認して決めるか:
  - 後工程の使用用途
- 非対象:
  - 人向け summary デザイン
  - rerun / replay 実証
  - UI 固有 export
  - 監査用 packet 設計
- done 条件:
  - 機械向け handoff が作れる

---

## B70-04
**名称:** Consumer handoff packet minimal definition  
**主担当:** shared
**バッチ種別:** 重要バッチ

- ゴール:
  - downstream consumer や後工程へ渡すための handoff packet の最小構成を定義する
- 触る契約:
  - handoff packet definition
- commit review で何を見るか:
  - downstream が読むのに必要な情報が欠けていないか
  - report 用 summary と machine-readable handoff の責務が混ざっていないか
  - rerun 専用情報を無理に consumer packet へ押し込みすぎていないか
- source of truth:
  - handoff packet document
  - downstream use-case
- 未確定事項:
  - appendix 的情報の要否
  - consumer ごとの最小必須 field
- 何を確認して決めるか:
  - downstream reviewer
  - 実際の受け渡しユースケース
- 関連する既決事項:
  - B60 シリーズ (Integration)
  - B70-03 (Machine-readable export)
- 影響するレイヤー:
  - Interface / Docs
- done 条件:
  - 手渡し成果物としての最小セットが固定される
  - downstream が読める packet の責務が明確になる

---

### B70-05
**名称:** Rerun reproducibility packet  
**主担当:** shared / counter / reporter
**バッチ種別:** 重要バッチ

- ゴール:
  - 1年後の自分が受け取っても、同じ条件で rerun / replay できるだけの再現性情報を束ねる
- 触る契約:
  - rerun packet definition
  - ExecutionRunSpec
  - AssaySpec
  - MatrixSpec
  - ComparisonSpec
  - ReportPayloadSpec
  - SourceArtifactRecord
- commit review で何を見るか:
  - rerun に必要な情報が packet に揃っているか
  - 見た目の納品物だけで再解析不能になっていないか
  - app version / schema version / backend version / reference version が追えるか
  - input / intermediate / output の参照関係が辿れるか
  - container / lockfile / environment 情報が欠けていないか
  - source artifact identity が packet に入っているか
  - rerun packet が consumer handoff packet と混同されていないか
- source of truth:
  - rerun packet definition
  - ExecutionRunSpec
  - representative rerun scenario
  - archived run fixture
  - source artifact refs
- 未確定事項:
  - literal に同一環境を要求するか、論理的同等環境でよいか
  - container digest を必須にするか、environment lock まで許容するか
  - reference 実体を同梱するか、checksum + locator にするか
  - rerun packet の保存単位を run ごとにするか bundle ごとにするか
- 何を確認して決めるか:
  - 現場で想定する再解析運用
  - 1年後に必要な障害切り分け観点
  - オフライン環境 / 参照更新リスク
  - 実際の archived run を使った replay 試験
- 関連する既決事項:
  - B60-02 (Lineage)
  - B70-04 (Handoff packet)
- 影響するレイヤー:
  - Core / Interface / Docs
- 非対象:
  - 電子署名
  - immutable / WORM 保存
  - 承認ワークフロー
  - 法的原本管理そのもの
- done 条件:
  - rerun に必要な最小情報が文書化されている
  - representative run について rerun packet を生成できる
  - packet から必要な入力、参照、環境、パラメータを再構成できる
  - source artifact identity を packet から辿れる
  - 同じ入力から同じ条件で rerun 可能と説明できる
  - rerun packet と consumer handoff packet の責務が分離されている

### rerun packet に最低限含めたい項目
- `run_id`
- `schema_versions`
- `app_name`
- `app_version`
- `backend_name`
- `backend_version`
- `container_image_digest` または `environment_lock_ref`
- `input_artifact_refs`
- `input_checksums`
- `reference_genome_version`
- `reference_genome_checksum`
- `annotation_version`
- `annotation_checksum`
- `execution_parameters`
- `comparison_definition`
- `template_version`
- `output_artifact_refs`
- `provenance_chain`
- optional: `random_seed`
- optional: `resource_profile`
- optional: `host_context`

---

### B70-06
**名称:** Deliverable traceability  
**主担当:** shared / reporter
**バッチ種別:** 軽量バッチ

- ゴール:
  - summary / figure / table / raw result の対応を追跡可能にする
- 触る契約:
  - ReportPayloadSpec
  - ResultSpec
- commit review で何を見るか:
  - summary が raw result と矛盾していないか
- source of truth:
  - representative deliverables
- 未確定事項:
  - trace 粒度
- 何を確認して決めるか:
  - レビュー時に人が確認したい単位
- 非対象:
  - rerun packet 設計
  - consumer handoff packet 設計
  - 統計結果の再計算
  - UI / export の見た目調整
- done 条件:
  - 成果物から raw result へ戻れる

---

# 8. Phase v0.80 - v0.89
# テーマ
Operational hardening / regression defense

## このphaseの目的
- 回帰テスト行列（Regression Matrix）により、v1.0.0 リリースまで死守すべき正常動作範囲を固定する
- 意図しない入力、チェックサム不一致、ロケータ不整合などの運用異常系（Negative case）に対する防御を完成させる
- 実装上の限界（Known limitations）を明文化し、安定性ゲートを通過させる

## バッチ一覧

### B80-01
**名称:** Regression matrix  
**主担当:** shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - 回帰対象表を作り、何を毎回守るか固定する
- 触る契約:
  - regression matrix
- commit review で何を見るか:
  - 主要 use-case が抜けていないか
- source of truth:
  - regression matrix
- 未確定事項:
  - 必須回帰ケース数
- 何を確認して決めるか:
  - 過去の破綻履歴
- 非対象:
  - 新機能追加
  - vendor 対応拡張
  - docs / release note 整備
  - acceptance scenario 追加
- done 条件:
  - 守るべき正常系が一覧化される

---

### B80-02
**名称:** Negative fixture pack  
**主担当:** shared
**バッチ種別:** 重要バッチ

- ゴール:
  - malformed input / missing metadata / invalid contrast に加え、
    source artifact の欠損・checksum 不一致・locator 不整合も異常 fixture として持つ
- 触る契約:
  - negative fixtures
- commit review で何を見るか:
  - source artifact が欠けた場合の挙動が定義されているか
  - checksum 不一致が黙って通過しないか
  - artifact locator が解決不能な場合の扱いがあるか
- source of truth:
  - negative fixtures
  - source artifact anomaly fixtures
- 未確定事項:
  - どこまで網羅するか
- 何を確認して決めるか:
  - 実運用の失敗例
- 関連する既決事項:
  - B60-04 (Integration negative cases)
- 影響するレイヤー:
  - Test
- 非対象:
  - 異常系の完全網羅
  - release policy の最終決定
  - UI 文言の磨き込み
  - 監査制度化
- done 条件:
  - 代表異常系が揃う
  - source artifact 起因の異常系も少なくとも代表ケースで確認できる

---

### B80-03
**名称:** Validator hardening  
**主担当:** counter / reporter / shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - validator を release blocker 候補まで強化する
- 触る契約:
  - validators
- commit review で何を見るか:
  - validator が単なる型確認で終わっていないか
- source of truth:
  - validator suite
- 未確定事項:
  - blocker 化する範囲
- 何を確認して決めるか:
  - release 前の事故コスト
- 非対象:
  - feature redesign
  - exporter rewrite
  - 新規 spec 追加
  - release docs 更新だけの対応
- done 条件:
  - release gate に載せられる

---

### B80-04
**名称:** Compatibility / migration notes  
**主担当:** shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - 互換性ルールと migration note の運用を固める
- 触る契約:
  - compatibility rules
  - migration notes
- commit review で何を見るか:
  - 破壊的変更が記録されるか
- source of truth:
  - migration policy
- 未確定事項:
  - minor / patch 互換の扱い
- 何を確認して決めるか:
  - 今までの変更履歴
- 非対象:
  - schema 再設計
  - 自動 migration engine
  - package/release packaging
  - 新規 feature 導入
- done 条件:
  - 破壊的変更が追える

---

### B80-05
**名称:** Known limitations / non-goals register  
**主担当:** shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - v1.0.0 の限界と非対象を明文化する
- 触る契約:
  - known limitations
- commit review で何を見るか:
  - 暗黙の未対応が残っていないか
- source of truth:
  - limitations register
- 未確定事項:
  - どこまでを書くか
- 何を確認して決めるか:
  - 現場期待値
- 非対象:
  - 新機能の実装
  - workaround コードの追加
  - marketing / 説明資料
  - v1.1 以降の詳細ロードマップ設計
- done 条件:
  - できること / できないことを説明できる

---

### B80-06
**名称:** Pre-RC stability gate  
**主担当:** shared
**バッチ種別:** 重要バッチ

- ゴール:
  - RC に進める前の安定性 gate を置く
- 触る契約:
  - pre-RC checklist
- commit review で何を見るか:
  - 回帰・異常系・互換性の根拠があるか
  - archived rerun packet を使った replay 試験の結果があるか
  - source artifact ref が解決できるか
  - checksum / locator / provenance chain が破綻していないか
- source of truth:
  - pre-RC checklist
  - archived rerun packet
  - replay test result
- 未確定事項:
  - blocker bug 基準
- 何を確認して決めるか:
  - 総合試験結果
- 関連する既決事項:
  - B80/B70 全般 (Stability / Rerun)
- 影響するレイヤー:
  - Test / Docs
- 非対象:
  - 新機能追加
  - 大規模リファクタ
  - release note 草案作成
  - post-v1.0 scope の繰り上げ
- done 条件:
  - RC 前に少なくとも代表 1 ケースで replay 性を確認している
  - rerun packet と source artifact identity の接続が実証されている

---

# 9. Phase v0.90 - v0.99
# テーマ
Release candidate

## このphaseの目的
- 実装と整合したドキュメント（README, Usage）を整備し、人依存を排除した初見再現性を確保する
- Suite 全体の価値（ counter native & vendor normalized 等）を証明する Final Acceptance Scenario を完遂する
- リリースを妨げる blocker bug を最終整理し、v1.0.0 Core Release のマイルストーンを完了する

## バッチ一覧

### B90-01
**名称:** README / install / usage alignment  
**主担当:** release / shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - docs を現行実装へ揃える
- 触る契約:
  - README
  - install guide
  - usage guide
- commit review で何を見るか:
  - 実装と docs がズレていないか
- source of truth:
  - current implementation
  - acceptance scenario
- 未確定事項:
  - どこまで docs を必須にするか
- 何を確認して決めるか:
  - 初見再現性
- 非対象:
  - 機能実装
  - schema 変更
  - 新規 vendor 対応
  - validator / regression の大改修
- done 条件:
  - 手順が人依存でなくなる

---

### B90-02
**名称:** Demo / representative data pack  
**主担当:** release
**バッチ種別:** 軽量バッチ

- ゴール:
  - representative workflow を示す最小 demo data を整える
- 触る契約:
  - demo dataset
- commit review で何を見るか:
  - 代表性があるか
- source of truth:
  - acceptance scenario
- 未確定事項:
  - 公開可能範囲
- 何を確認して決めるか:
  - 使用許諾 / 社内利用条件
- 非対象:
  - benchmark 的なデータ拡張
  - public comparator デモ
  - release note 文言調整
  - 新規 feature 検証
- done 条件:
  - 手順検証に使える demo がある

---

### B90-03
**名称:** Final acceptance scenario  
**主担当:** shared / release
**バッチ種別:** 重要バッチ

- ゴール:
  - v1.0.0 を切るための代表 workflow を固定する
- 触る契約:
  - final acceptance scenario
- commit review で何を見るか:
  - counter native と vendor normalized の両方を見ているか
  - final acceptance scenario に rerun / replay 観点が最低1つ入っているか
  - source artifact identity を辿る確認が入っているか
- source of truth:
  - final acceptance scenario
- 未確定事項:
  - ケース数
- 何を確認して決めるか:
  - 現場で一番説明しやすい流れ
- 関連する既決事項:
  - v1.0.0 までの全バッチ
- 影響するレイヤー:
  - Test / Docs
- 非対象:
  - 新機能の受け入れ
  - blocker bug 修正そのもの
  - docs 全面更新
  - v1.1 scope の先取り
- done 条件:
  - release 判定の基準として使える
  - source artifact 起点の再追跡が acceptance scenario に含まれる

---

### B90-04
**名称:** Blocker bug closure  
**主担当:** release / shared
**バッチ種別:** 軽量バッチ

- ゴール:
  - blocker bug を整理し、残さないか、残すなら明文化する
- 触る契約:
  - blocker bug list
- commit review で何を見るか:
  - blocker の定義が甘くなっていないか
- source of truth:
  - blocker list
- 未確定事項:
  - hold-to-release の境界
- 何を確認して決めるか:
  - 総合試験と現場影響
- 非対象:
  - 新機能追加
  - spec 再設計
  - docs だけの polishing
  - release 後運用の詳細設計
- done 条件:
  - blocker が閉じるか、既知制限へ格下げされる

---

### B90-05
**名称:** Release note / package reality sync  
**主担当:** release
**バッチ種別:** 軽量バッチ

- ゴール:
  - release note 草案と package version の現実整理を行う
- 触る契約:
  - release note
  - version policy
- commit review で何を見るか:
  - suite milestone と package version の説明ができるか
- source of truth:
  - release plan
- 未確定事項:
  - 最終 version 表記
- 何を確認して決めるか:
  - リリース時の配布形態
- 非対象:
  - core spec 再設計
  - 新 acceptance criteria 追加
  - 新機能の既成事実化
  - v1.1 backlog の詳細設計
- done 条件:
  - v1.0.0 を説明できる

---

# 10. v1.0.0 Release Gate
# テーマ
RNA-Seq Suite Core Release

## バッチではなく最終ゲートとして確認するもの

### RG-01 End-to-end
- counter native で通る
- vendor normalized で通る
- reporter が正規入力のみで処理できる

### RG-02 Contract integrity
- 6 core spec が実装と矛盾しない
- source of truth が増殖していない
- result / payload / execution が分裂していない
- source artifact identity が provenance chain に接続されている
- rerun packet から input origin を辿れる

### RG-03 Deliverables
- HTML が成立する
- PDF が成立する
- machine-readable handoff が成立する
- **1年後の再解析（Rerun）を保証するメタデータ・環境情報が同梱されている**

### RG-04 Operational readiness
- regression / negative / integration が通る
- known limitations がある
- non-goals が明記されている
- representative archived run で replay 性を確認している
- source artifact checksum / locator の最低限確認ができる
- **正常系サンプルにおける Rerun 再現性が確認されている**

### RG-05 Scope discipline
- LLM 補助が v1.0.0 に混入していない
- comparator / signature など後続項目が分離されている

---

# 11. バッチ運用の短縮テンプレート

## バッチ計画
- Batch ID:
- 名称:
- 主担当:
- ゴール:
- 触る契約:
- commit review で何を見るか:
- source of truth:
- 未確定事項:
- 何を確認して決めるか:
- 関連する既決事項:
- 影響するレイヤー:
- done 条件:

## バッチ完了レビュー
- Batch ID:
- 判定: DONE / HOLD / REJECT
- 守れた contract:
- source of truth 確認:
- 未確定事項の残り:
- suite handoff 影響:
- 次バッチへの引き継ぎ:

---

# 12. この文書の一文定義

このバッチ分解は、
Phase を細かくすること自体が目的ではなく、
**各実装単位で「壊れない受け渡し」を確認しながら前進するための作業単位表**
である。