# RNA-Seq Suite 受け入れ基準チェックリスト
Version: draft-0.1
Target: v1.0.0
Policy: v1.0.0 では LLM 補助を入れない
Core philosophy: v1.0.0 の成功条件は「賢い補助」ではなく「壊れない受け渡し」に置く

---

## 0. この文書の使い方

この文書は、各フェーズごとに以下を固定するためのものとする。

1. ゴール
2. commit review 時に何を見るか
3. source of truth はどこか
4. まだ未確定なことは何か
5. それを何で確認して決定するか
6. 受け入れ基準チェックリスト

この文書の目的は、実装速度より先に
**「どこを見て done と言うか」**
を固定することにある。

---

## 1. Suite 共通原則

### 1.1 最重要原則
- 壊れない受け渡しを最優先する
- result より前に contract を守る
- app 単体最適ではなく suite 接続最適で判断する
- reporter 本体に vendor 例外処理を持ち込まない
- AI / narrative を result 本体へ混ぜない
- 実装都合で source of truth を増やさない

### 1.2 全フェーズ共通の commit review 観点
- source of truth を守っているか
- public contract を壊していないか
- adapter と本体の責務が混ざっていないか
- execution provenance が落ちていないか
- 例外を握りつぶしていないか
- 暫定実装が恒久仕様に昇格していないか
- 今回やらないことへ踏み込んでいないか
- suite handoff を壊していないか

### 1.3 全フェーズ共通の source of truth
- devmap
- core spec 文書
- fixture
- contract validator
- integration test
- release checklist

### 1.4 全フェーズ共通の未確定事項への原則
未確定なものは、会話や印象で決めず、以下で決める。

- spec 文書
- fixture
- validator
- representative input
- representative output
- failing test
- integration scenario

---

# 2. フェーズ別定義

---

## Phase v0.20 - v0.29
# テーマ
Suite foundation / contract freeze

### ゴール
RNA-Seq Suite を「2つのアプリ」ではなく、
**spec で接続される suite**
として定義し直す。

### commit review 時に何を見るか
- 6 core spec の名前と責務がぶれていないか
- counter / reporter の責務境界が曖昧になっていないか
- vendor 差異を adapter へ隔離する方針が明文化されているか
- schema versioning ルールが決まっているか
- metadata / overlay / provenance の逃がし先があるか
- 現時点で入れないものが明示されているか

### source of truth
- RNA-Seq Suite devmap
- core spec 草案
- app boundary 定義文書
- package / repo の現状構成
- representative workflow 図

### 現状ではまだ分からないこと
- 6 core spec の最小必須 field
- counter / reporter 間でどこまでを handoff に載せるか
- metadata をどこまで core に入れ、どこから overlay に逃がすか
- ExecutionRunSpec と JobRequestSpec の境界
- ComparisonSpec の最小責務

### 何を確認して決定するか
- 最小 round-trip use-case
- counter -> reporter の実際の受け渡し内容
- 既存 fixture / bundle 構造
- 破壊したくない公開 API
- 既存 commit / test が守っている事実上の contract

### 受け入れ基準チェックリスト
- [ ] 6 core spec の名称が固定されている
- [ ] 各 spec の責務が1文で定義されている
- [ ] counter の入力 / 出力が明文化されている
- [ ] reporter の入力 / 出力が明文化されている
- [ ] vendor adapter の存在理由が明文化されている
- [ ] reporter 本体が raw vendor file を source of truth にしない方針がある
- [ ] schema_name / schema_version の運用方針がある
- [ ] metadata / overlay / provenance の基本ルールがある
- [ ] LLM を v1.0.0 のスコープ外と明記している
- [ ] suite milestone を package version と分けて扱う方針がある

---

## Phase v0.30 - v0.39
# テーマ
Counter contract hardening

### ゴール
counter を
**FASTQ 実行器ではなく、標準 handoff 生成器**
として安定化する。

### commit review 時に何を見るか
- intake validation が入口で閉じているか
- QC / trimming / backend / matrix build / run record の責務が混ざっていないか
- backend 差異が標準出力へ正規化されているか
- backend 固有情報が ExecutionRunSpec 側へ退避できているか
- matrix と execution log が混在していないか
- エラー時にも suite handoff に必要な情報が残るか

### source of truth
- AssaySpec
- MatrixSpec
- ExecutionRunSpec
- counter の representative input
- backend ごとの fixture
- counter validator / regression test

### 現状ではまだ分からないこと
- STAR / HISAT2 / kallisto / salmon の最終共通 contract
- transcript-level と gene-level の最小共通表現
- strandedness / paired-end / reference 差異をどこまで MatrixSpec に入れるか
- log のどこまでを ExecutionRunSpec に持つか
- warning と failure の境界

### 何を確認して決定するか
- 4 backend の代表 run
- 各 backend の出力差分
- reporter が実際に必要とする最小入力
- 運用時に人が見るログ項目
- 失敗時の切り分けに必要な情報

### 受け入れ基準チェックリスト
- [ ] FASTQ から標準 MatrixSpec を返せる
- [ ] 4 backend の違いが標準 contract に収束する
- [ ] backend 固有事情を ExecutionRunSpec へ逃がせる
- [ ] 失敗時に status / error summary / log ref が残る
- [ ] sample / assay / reference 情報の由来が追跡できる
- [ ] matrix と log / resource usage が分離されている
- [ ] reporter が追加変換なしで読める最低限 handoff が成立する
- [ ] CLI / batch entrypoint が contract を壊さない
- [ ] validator と regression test がある
- [ ] 「通る run だけ通る」ではなく失敗時挙動も定義されている

---

## Phase v0.40 - v0.49
# テーマ
Reporter core analytics hardening

### ゴール
reporter を
**画面の寄せ集めではなく、標準入力から結果契約を返す解析機**
として安定化する。

### commit review 時に何を見るか
- reporter 本体が正規化済み入力だけを扱っているか
- UI 層で comparison や group を再解釈していないか
- exploratory / DEG / enrichment / export の責務が混ざっていないか
- result と figure / table / payload の関係が明確か
- narrative が result に混じっていないか
- raw JSON / raw file の直読みに戻っていないか

### source of truth
- MatrixSpec
- ComparisonSpec
- ResultSpec
- ReportPayloadSpec
- reporter fixture
- representative report output
- report integration test

### 現状ではまだ分からないこと
- ComparisonSpec の design_type の最小集合
- DEG や enrichment の result をどこまで共通化するか
- figure_refs / table_refs の最小責務
- gene search を ResultSpec で持つか payload 側で持つか
- report summary と analysis summary の境界

### 何を確認して決定するか
- 現場が最低限必要とする report 構成
- 既存 UI / export の実装形
- 実データでの plot 再現パターン
- report consumer が何を source of truth として使うか
- PDF / HTML / UI の差分

### 受け入れ基準チェックリスト
- [ ] MatrixSpec + ComparisonSpec から ResultSpec を生成できる
- [ ] ReportPayloadSpec が report の source of truth になっている
- [ ] PCA が正規入力だけで再現できる
- [ ] correlation heatmap が正規入力だけで再現できる
- [ ] DEG 結果が ResultSpec に格納できる
- [ ] volcano / heatmap / enrichment の参照先が追跡できる
- [ ] figure / table / summary の責務が分離されている
- [ ] UI と export が payload 中心で動いている
- [ ] vendor 固有解釈が reporter 本体に入っていない
- [ ] raw vendor file 直読みに戻っていない

---

## Phase v0.50 - v0.59
# テーマ
Vendor adapter / normalization

### ゴール
外部委託納品物の差異を
**reporter 本体でなく adapter 層で吸収**
できるようにする。

### commit review 時に何を見るか
- adapter が正規化責務に限定されているか
- adapter に解析ロジックが入り込んでいないか
- column mapping / sample mapping / group mapping が追跡可能か
- metadata 不足時の warning 方針があるか
- vendor ごとの差異が adapter 実装に閉じているか
- manual import が例外処理の穴になっていないか

### source of truth
- adapter interface
- vendor mapping table
- normalization rules
- representative vendor files
- import validation result
- normalized MatrixSpec / ComparisonSpec

### 現状ではまだ分からないこと
- どの vendor 形式から先に対応するか
- comparison 情報が不足した納品物への対応
- gene ID や sample ID のゆれへの対処範囲
- manual import をどこまで許すか
- warning 止まりにする条件と reject にする条件

### 何を確認して決定するか
- 実際に受領する納品物サンプル
- 列名、シート構成、ID の揺れ
- 最低限現場で必要な comparison 情報
- 不足 metadata の頻度
- import failure 事例

### 受け入れ基準チェックリスト
- [ ] adapter interface が定義されている
- [ ] vendor 形式ごとの差異が adapter に閉じている
- [ ] adapter が MatrixSpec / ComparisonSpec を返せる
- [ ] reporter 本体に vendor 分岐が追加されていない
- [ ] sample ID / gene ID 正規化の記録が残る
- [ ] 不足 metadata が warning か reject か判定できる
- [ ] manual import の責務範囲が定義されている
- [ ] import validation report が出せる
- [ ] 主要 1〜2 系統の納品形式で通る
- [ ] 正規化後は internal data と同じ入口に乗る

---

## Phase v0.60 - v0.69
# テーマ
Counter-Reporter integration

### ゴール
counter と reporter を
**本当に suite として接続**
する。

### commit review 時に何を見るか
- counter output を reporter がそのまま読めるか
- 受け渡し時に spec version が崩れていないか
- artifact lineage が追跡できるか
- round-trip fixture があるか
- integration test が bundle の実態を見ているか
- app 間の暫定変換コードが発生していないか

### source of truth
- round-trip integration test
- handoff artifact
- MatrixSpec
- ResultSpec
- ReportPayloadSpec
- ExecutionRunSpec
- fixture chain

### 現状ではまだ分からないこと
- suite 最小統合ケース
- どこまでを integration test に含めるか
- artifact lineage の表示方法
- spec 互換性 break の検知ルール
- app 境界をまたぐ warning の表現方法

### 何を確認して決定するか
- 最小入力 FASTQ ケース
- reporter が必要とする handoff 最小セット
- 実際の round-trip 実行結果
- 互換性 break 事例
- release 前に毎回通す統合シナリオ

### 受け入れ基準チェックリスト
- [ ] FASTQ -> counter -> MatrixSpec -> reporter が通る
- [ ] reporter が counter 生成物を追加変換なしで処理できる
- [ ] artifact lineage が追跡できる
- [ ] app 間の spec version を確認できる
- [ ] round-trip fixture が存在する
- [ ] integration test が最低1本ではなく代表複数系統ある
- [ ] counter だけ / reporter だけ最適のコードが統合を壊していない
- [ ] temporary shim が常設化していない
- [ ] failure case の統合挙動も確認されている
- [ ] suite と呼べる最小接続が成立している

---

## Phase v0.70 - v0.79
# テーマ
Report / Export / Handoff stabilization

### ゴール
解析結果を
**現場に渡せる成果物**
として安定化する。

### commit review 時に何を見るか
- HTML / PDF / JSON export が別々の truth を持っていないか
- payload から成果物が再構成できるか
- report template が解析結果に依存しすぎていないか
- handoff packet が後工程に必要な情報を満たすか
- export 専用の暫定整形が本体 contract を壊していないか
- report summary と raw result の関係が追跡可能か

### source of truth
- ReportPayloadSpec
- export fixtures
- representative HTML / PDF / machine-readable output
- handoff packet definition
- acceptance scenario

### 現状ではまだ分からないこと
- PDF と HTML の差分の扱い
- handoff packet の最小構成
- downstream consumer が必要とする field
- export template slot の固定範囲
- 図表を payload で持つ粒度

### 何を確認して決定するか
- 実際の納品イメージ
- 現場で必要な summary / appendix / tables
- downstream で読む JSON の用途
- reanalysis に必要な再利用情報
- export consumer からのレビュー

### 受け入れ基準チェックリスト
- [ ] HTML を payload から再構成できる
- [ ] PDF を payload から再構成できる
- [ ] machine-readable JSON を payload から再構成できる
- [ ] handoff packet の最小構成が定義されている
- [ ] export ごとに別 truth が増えていない
- [ ] figure / table / summary の参照関係が追える
- [ ] downstream へ渡す情報が不足していない
- [ ] summary と raw result が矛盾しない
- [ ] report template 変更が core contract を壊さない
- [ ] 現場に渡せる最小成果物が成立している

---

## Phase v0.80 - v0.89
# テーマ
Operational hardening / regression defense

### ゴール
v1.0.0 前に
**壊れにくさ**
を担保する。

### commit review 時に何を見るか
- regression test が増えているか
- 既知の壊れ方に対する防御があるか
- malformed input / missing metadata / invalid contrast への挙動が定義されているか
- 互換性 break が検知できるか
- 失敗時に調査できるログが残るか
- 例外系の仕様がコメントでなくテストで表現されているか

### source of truth
- regression tests
- negative fixtures
- validator
- migration notes
- release checklist
- known limitations

### 現状ではまだ分からないこと
- 何を必須回帰テストに入れるか
- malformed input の拒否基準
- missing metadata を warning で許す範囲
- backward compatibility の閾値
- どの障害を blocker とするか

### 何を確認して決定するか
- 過去のバグ履歴
- 失敗しやすい入力パターン
- 現場で実際に起こる運用エラー
- 既存 validator の守備範囲
- release 前の総合試験結果

### 受け入れ基準チェックリスト
- [ ] 主要 use-case が regression test で覆われている
- [ ] 主要 failure case が regression test で覆われている
- [ ] malformed input への挙動が定義されている
- [ ] missing metadata の挙動が定義されている
- [ ] invalid contrast の挙動が定義されている
- [ ] compatibility check がある
- [ ] validator が release blocker になりうる
- [ ] known limitations が記録されている
- [ ] migration note が必要な変更は記録されている
- [ ] v1.0.0 前に壊れ方が見えている

---

## Phase v0.90 - v0.99
# テーマ
Release candidate

### ゴール
v1.0.0 を
**切れる状態**
にする。

### commit review 時に何を見るか
- package metadata と suite reality のズレを整理できているか
- README / install / usage が今の実態に合っているか
- representative workflow が再現可能か
- final bugfix が scope creep になっていないか
- release note に何を書くか説明できるか
- 手順書が人依存でないか

### source of truth
- release checklist
- README
- install guide
- usage guide
- demo data
- acceptance scenario
- final integration / regression results

### 現状ではまだ分からないこと
- package version の最終揃え方
- release note の最終粒度
- demo data の公開可能範囲
- 必須ドキュメントの下限
- RC 期間中の blocker 基準

### 何を確認して決定するか
- 実際の起動 / 実行 / export 手順
- 初見ユーザーの再現性
- release 対象ファイルの棚卸し
- blocker bug 一覧
- RC の総合試験結果

### 受け入れ基準チェックリスト
- [ ] README が現行実装に追随している
- [ ] install 手順が実行可能である
- [ ] representative workflow が再現できる
- [ ] demo data がある
- [ ] acceptance scenario がある
- [ ] blocker bug が整理されている
- [ ] package version の扱い方が決まっている
- [ ] release note 草案が書ける
- [ ] v1.0.0 に入れないものが再確認されている
- [ ] リリース判断を感覚ではなく checklist で行える

---

## Phase v1.0.0
# テーマ
RNA-Seq Suite Core Release

### ゴール
LLM を使わずに、
**FASTQ 実行・標準化・可視化・納品物生成が一貫して成立する core suite**
として完成させる。

### commit review 時に何を見るか
- suite として end-to-end が成立しているか
- handoff break が残っていないか
- reporter が vendor 分岐まみれになっていないか
- export / payload / result の truth が分裂していないか
- 既知制限が明文化されているか
- v1.1 以降へ送るものが整理されているか

### source of truth
- end-to-end acceptance scenario
- final release checklist
- final regression / integration test
- spec documents
- export artifacts
- handoff packet
- known limitations

### 現状ではまだ分からないこと
- どこまでを v1.0.0 完成条件とし、どこからを v1.1 に送るか
- 現場が「使える」と判断する最小セット
- vendor 対応範囲の最低ライン
- release 後の first maintenance scope

### 何を確認して決定するか
- final acceptance run
- representative vendor import
- representative counter native run
- representative report export
- 現場視点のレビュー
- release candidate の総合結果

### 受け入れ基準チェックリスト
- [ ] counter が標準 handoff を返せる
- [ ] reporter が標準 handoff を壊さず処理できる
- [ ] vendor adapter が本体を汚さず動作する
- [ ] ResultSpec / ReportPayloadSpec / ExecutionRunSpec がつながる
- [ ] HTML / PDF / machine-readable output が成立する
- [ ] end-to-end acceptance scenario が通る
- [ ] regression / integration の最低限が揃っている
- [ ] LLM なしで現場運用に耐える
- [ ] 既知制限が明文化されている
- [ ] v1.1 以降へ送る項目が分離されている

---

# 3. 最終リリース判定チェック

以下を全て満たしたとき、v1.0.0 を切ってよい。

- [ ] suite devmap が最新である
- [ ] 6 core spec が実装と矛盾していない
- [ ] counter / reporter / adapter の責務境界が保たれている
- [ ] handoff break がない
- [ ] source of truth が増殖していない
- [ ] export が payload から再構成できる
- [ ] representative workflow が再現できる
- [ ] 既知制限が説明できる
- [ ] LLM を入れなくても価値が成立する
- [ ] 「賢い」ではなく「壊れない」で勝てる状態になっている

---

# 4. レビュー時の短縮テンプレート

## フェーズレビュー要約
- フェーズ:
- ゴール達成状況:
- source of truth:
- 未確定事項:
- 何を確認して決めるか:
- handoff 影響:
- 今回の done 判定:
- 次フェーズへ送るもの:

## commit review 要約
- この commit はどの contract に触るか:
- public contract を壊したか:
- source of truth を増やしたか:
- adapter と本体の責務を混ぜたか:
- provenance は維持されたか:
- suite handoff への影響:
- accept / hold / reject:

---

# 5. この文書の一文定義

RNA-Seq Suite の受け入れ基準とは、
機能が増えたかどうかではなく、
**壊れない受け渡しが維持されたまま前進したかどうか**
を判定するための基準である。