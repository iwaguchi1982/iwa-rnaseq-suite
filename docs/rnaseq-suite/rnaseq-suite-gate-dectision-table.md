# RNA-Seq Suite Phase Gate 判定表
Version: draft-0.1
Target: v1.0.0
Policy: v1.0.0 では LLM 補助を入れない
Core philosophy: 判定基準は「賢いか」ではなく「壊れない受け渡しか」

---

## 0. この文書の目的

この文書は、各 Phase のレビュー時に
**done / hold / reject**
を機械的に近い形で判断するための gate 定義である。

各 Phase について、最低限以下を固定する。

- その Phase のゴール
- done と言ってよい条件
- hold に留める条件
- reject すべき条件
- commit review で必ず見るもの
- source of truth
- 未確定事項が残る場合、何を確認して決めるか

---

## 1. 判定ラベルの定義

## DONE
次の Phase に進めてよい。  
局所的な軽微修正は残っていてもよいが、
**その Phase の主責務は満たしており、suite handoff を壊していない** 状態。

## HOLD
方向性は正しいが、done と言うには根拠が不足している。  
実装を破棄する必要はないが、
**spec / test / fixture / integration evidence の不足** により保留する状態。

## REJECT
その Phase の成果物として受け入れない。  
**責務逸脱、source of truth の破壊、public contract 破壊、暫定実装の恒久化、suite handoff 破壊**
のいずれかが発生している状態。

---

## 2. 全 Phase 共通の即時 REJECT 条件

以下のいずれかに該当した場合、その commit / PR / タスクは原則 REJECT とする。

- reporter 本体に vendor 固有分岐を書き込んだ
- raw vendor file / raw JSON 直読みに戻した
- public contract を無断で破壊した
- schema version を変えずに破壊的変更を入れた
- result と narrative を混在させた
- matrix と execution log / provenance を混在させた
- app 間 handoff を暫定 shim 前提にした
- 例外を握りつぶして成功扱いにした
- source of truth を増殖させた
- 今回の Phase の非対象スコープを主機能として持ち込んだ

---

## 3. 全 Phase 共通の HOLD 条件

以下は原則 HOLD とする。

- 方向性は正しいが、validator がない
- representative fixture がない
- integration evidence が不足している
- source of truth は正しいが、テストで担保されていない
- spec 文書と実装がズレているが、修正範囲が限定的
- 失敗時挙動が未定義
- 未確定事項が残っており、確認方法だけは定義されている

---

## 4. Phase Gate 判定表

---

## Phase v0.20 - v0.29
### テーマ
Suite foundation / contract freeze

### ゴール
RNA-Seq Suite を app 集合ではなく、
**spec でつながる suite**
として定義し直す。

| 判定 | 条件 |
|---|---|
| DONE | 6 core spec の名称と責務が固定されている。counter / reporter / adapter の責務境界が明文化されている。schema_name / schema_version / metadata / overlay / provenance の基本ルールがある。v1.0.0 で LLM を入れない方針が明文化されている。 |
| HOLD | spec 名称は見えているが最小責務が曖昧。counter / reporter 境界が文章ではあるが fixture や workflow 図に落ちていない。ExecutionRunSpec と JobRequestSpec の境界が未確定だが、確認方法は定義されている。 |
| REJECT | spec 名が揺れている。counter / reporter の責務が逆流している。vendor 差異吸収先が定義されていない。suite milestone と package version の扱いが混線している。 |

### commit review で必ず見るもの
- devmap
- core spec 草案
- app boundary 定義
- representative workflow 図
- 現行 repo の公開 API に関する整理

### source of truth
- RNA-Seq Suite devmap
- core spec 文書
- workflow 定義
- representative fixture 設計

### 未確定事項がある場合に何を確認して決めるか
- 最小 round-trip use-case
- 既存 handoff 実体
- 既存 validator / test が守っている暗黙 contract
- counter -> reporter の実データ受け渡し

---

## Phase v0.30 - v0.39
### テーマ
Counter contract hardening

### ゴール
counter を
**標準 handoff 生成器**
として安定化する。

| 判定 | 条件 |
|---|---|
| DONE | 4 backend の差異が MatrixSpec / ExecutionRunSpec に正規化される。失敗時も status / error summary / log ref が残る。reporter が追加変換なしで読める最低限 handoff が成立する。backend 固有事情を MatrixSpec に無理に埋め込んでいない。 |
| HOLD | backend ごとの差異吸収は見えているが、1〜2 backend しか fixture がない。成功系は通るが失敗系が未定義。MatrixSpec と ExecutionRunSpec の責務は概ね守れているが validator が弱い。 |
| REJECT | backend ごとに出力 contract が分裂している。matrix に log や resource usage を混ぜている。失敗時に追跡情報が残らない。reporter 側の都合で counter に可視化用例外列を入れている。 |

### commit review で必ず見るもの
- AssaySpec
- MatrixSpec
- ExecutionRunSpec
- backend abstraction
- representative backend fixture
- negative / failure test

### source of truth
- MatrixSpec
- ExecutionRunSpec
- backend fixture
- counter validator
- regression test

### 未確定事項がある場合に何を確認して決めるか
- STAR / HISAT2 / kallisto / salmon の代表 run
- transcript-level と gene-level の差分
- strandedness / paired-end / reference の差分
- reporter が実際に必要とする最小入力

---

## Phase v0.40 - v0.49
### テーマ
Reporter core analytics hardening

### ゴール
reporter を
**標準入力から結果契約を返す解析機**
として安定化する。

| 判定 | 条件 |
|---|---|
| DONE | MatrixSpec + ComparisonSpec から ResultSpec + ReportPayloadSpec を生成できる。PCA / correlation / DEG / volcano / heatmap / enrichment の主要動線が正規入力で成立する。UI / export の truth が ReportPayloadSpec に寄っている。 |
| HOLD | 主解析は成立するが ResultSpec と ReportPayloadSpec の責務分離が弱い。UI は動くが payload が唯一の truth になっていない。代表 plot は出るが figure/table refs の設計が未完成。 |
| REJECT | reporter 本体が raw vendor file を直接読む。UI 層で design や group を再解釈している。result と narrative を混ぜた。plot ごとに独自入力を要求する。 |

### commit review で必ず見るもの
- MatrixSpec
- ComparisonSpec
- ResultSpec
- ReportPayloadSpec
- representative report fixture
- figure / table / summary の参照関係
- export 入口

### source of truth
- ResultSpec
- ReportPayloadSpec
- reporter fixture
- report integration test
- representative outputs

### 未確定事項がある場合に何を確認して決めるか
- 現場が最低限必要とする report 構成
- 既存 UI / export 実装
- DEG / enrichment の共通化可能範囲
- PDF / HTML / UI の差分

---

## Phase v0.50 - v0.59
### テーマ
Vendor adapter / normalization

### ゴール
外部委託納品物の差異を
**adapter 層で吸収**
する。

| 判定 | 条件 |
|---|---|
| DONE | adapter interface が定義され、主要 1〜2 系統の vendor 形式を MatrixSpec / ComparisonSpec に正規化できる。reporter 本体に vendor 分岐が入っていない。normalization の記録が追跡できる。 |
| HOLD | adapter の方向性は正しいが、sample ID / gene ID / comparison 不足への扱いが統一されていない。1 vendor しか通っていない。不足 metadata への warning / reject 境界が未確定。 |
| REJECT | vendor 差異を reporter 本体へ書いている。adapter に DEG / plot / report などの解析責務が混入している。manual import が無制限の例外入口になっている。 |

### commit review で必ず見るもの
- adapter interface
- mapping rules
- representative vendor files
- import validation result
- normalized MatrixSpec / ComparisonSpec
- warning / reject policy

### source of truth
- adapter interface
- normalization rules
- vendor fixtures
- import validation report
- normalized contract

### 未確定事項がある場合に何を確認して決めるか
- 実際の納品物サンプル
- 列名やシート構成の揺れ
- group / comparison 情報の欠損頻度
- reject にすべき欠損の実例

---

## Phase v0.60 - v0.69
### テーマ
Counter-Reporter integration

### ゴール
counter と reporter を
**本当に suite として接続**
する。

| 判定 | 条件 |
|---|---|
| DONE | FASTQ -> counter -> MatrixSpec -> reporter -> ResultSpec / ReportPayloadSpec の round-trip が成立する。artifact lineage が追跡できる。spec version の整合が確認できる。代表複数系統の integration test がある。 |
| HOLD | round-trip は1本通るが代表性が弱い。artifact lineage が一部曖昧。counter / reporter 間に軽微な shim が残るが撤去計画と期限がある。failure case の統合挙動が未確認。 |
| REJECT | app 間 handoff のために暫定変換コードを常設化した。spec version が不整合のまま通している。counter 単体最適または reporter 単体最適で suite 接続が壊れている。 |

### commit review で必ず見るもの
- round-trip integration test
- handoff artifact
- lineage 記録
- spec version 整合性
- failure case 統合挙動

### source of truth
- round-trip fixture
- handoff artifact
- integration test
- lineage definition
- version compatibility rule

### 未確定事項がある場合に何を確認して決めるか
- 最小統合ケース
- 代表入力 FASTQ
- reporter が必要とする最小 handoff
- 互換性 break の過去事例

---

## Phase v0.70 - v0.79
### テーマ
Report / Export / Handoff stabilization

### ゴール
解析結果を
**現場に渡せる成果物**
として安定化する。

| 判定 | 条件 |
|---|---|
| DONE | HTML / PDF / machine-readable JSON / handoff packet が ReportPayloadSpec を中心に再構成できる。summary と raw result の対応が追える。downstream に必要な最小情報が満たされている。 |
| HOLD | export は出るが HTML / PDF / JSON で truth が分かれている。handoff packet の最小構成が未確定。summary はあるが raw result とのトレースが一部弱い。 |
| REJECT | export ごとに別ロジック・別 truth が生えている。report 用の暫定整形が core contract を壊している。見た目優先で provenance が落ちる。 |

### commit review で必ず見るもの
- ReportPayloadSpec
- export fixtures
- HTML / PDF / JSON 出力
- handoff packet definition
- summary-to-result trace

### source of truth
- ReportPayloadSpec
- export fixture
- handoff packet definition
- representative deliverables

### 未確定事項がある場合に何を確認して決めるか
- 実際の納品イメージ
- downstream consumer の要求
- reanalysis に必要な情報
- 現場レビュー

---

## Phase v0.80 - v0.89
### テーマ
Operational hardening / regression defense

### ゴール
v1.0.0 前に
**壊れにくさ**
を担保する。

| 判定 | 条件 |
|---|---|
| DONE | 主要 use-case と主要 failure case が regression test で覆われている。malformed input / missing metadata / invalid contrast の挙動が定義されている。compatibility check がある。known limitations が整理されている。 |
| HOLD | 正常系回帰はあるが異常系が薄い。validator はあるが release blocker として弱い。known limitations はあるが migration note が不足している。 |
| REJECT | 壊れやすい箇所が把握されていない。過去バグの再発防止がない。失敗時挙動がテストではなく雰囲気で運用されている。 |

### commit review で必ず見るもの
- regression tests
- negative fixtures
- validator
- compatibility check
- known limitations
- migration notes

### source of truth
- regression suite
- negative fixtures
- validator
- release checklist
- known limitations

### 未確定事項がある場合に何を確認して決めるか
- 過去バグ履歴
- 現場で起きる入力破綻
- release 前総合試験結果
- backward compatibility の閾値

---

## Phase v0.90 - v0.99
### テーマ
Release candidate

### ゴール
v1.0.0 を
**切れる状態**
にする。

| 判定 | 条件 |
|---|---|
| DONE | README / install / usage / demo data / acceptance scenario が整っている。representative workflow が再現できる。blocker bug が整理されている。release note 草案が書ける。package metadata と suite reality のズレ整理方針がある。 |
| HOLD | 実装はほぼ完成だが docs が追いついていない。workflow は再現できるが初見者向け手順が弱い。blocker bug は少数残るが閉じ方が決まっている。 |
| REJECT | 実行手順が人依存。README が実装と一致しない。RC なのに大規模仕様変更を入れている。blocker bug を整理せず release に進めようとしている。 |

### commit review で必ず見るもの
- release checklist
- README
- install / usage guide
- demo data
- acceptance scenario
- blocker bug list
- final integration / regression 結果

### source of truth
- release checklist
- docs
- demo data
- acceptance scenario
- blocker list

### 未確定事項がある場合に何を確認して決めるか
- 初見ユーザーの再現性
- RC 総合試験結果
- 公開対象の棚卸し
- final blocker 一覧

---

## Phase v1.0.0
### テーマ
RNA-Seq Suite Core Release

### ゴール
LLM なしで
**FASTQ 実行・標準化・可視化・納品物生成が一貫して成立する core suite**
として完成させる。

| 判定 | 条件 |
|---|---|
| DONE | counter / reporter / adapter / handoff / export が end-to-end で成立する。代表 vendor import と counter native run の両方が通る。既知制限が明文化されている。v1.1 以降へ送るものが分離されている。 |
| HOLD | core suite は成立しているが vendor 対応範囲や deliverable 定義に最終確認が必要。既知制限や first maintenance scope の記述が不足している。 |
| REJECT | end-to-end が成立していない。handoff break が残る。vendor 差異が reporter 本体へ侵食している。export / payload / result の truth が分裂している。 |

### commit review で必ず見るもの
- end-to-end acceptance scenario
- final regression / integration result
- final specs
- export artifacts
- handoff packet
- known limitations
- v1.1 backlog 分離

### source of truth
- acceptance scenario
- final specs
- final release checklist
- export artifacts
- handoff packet
- known limitations

### 未確定事項がある場合に何を確認して決めるか
- final acceptance run
- representative vendor import
- representative counter native run
- representative report export
- 現場レビュー

---

## 5. 判定運用ルール

## 5.1 DONE にするための最低条件
以下を全て満たすこと。

- ゴールに対する成果が source of truth 上で確認できる
- commit review で見るべき対象が揃っている
- suite handoff を壊していない
- 次 Phase に進めるだけの test / fixture / validator 根拠がある
- HOLD で残すべき未確定事項がない、または次 Phase へ送っても危険でない

## 5.2 HOLD にする場合の必須記録
HOLD の場合、必ず以下を残す。

- 何が不足しているか
- どこまでできているか
- source of truth は何か
- 何を確認すれば DONE に上げられるか
- 誰が / どの commit 群で詰めるか

## 5.3 REJECT にする場合の必須記録
REJECT の場合、必ず以下を残す。

- 何が原則違反か
- どの source of truth と矛盾したか
- rollback / rewrite / split のどれが必要か
- 別 app / adapter / utility script へ逃がすべきか
- 再提出時の条件は何か

---

## 6. Commit Review / Gate Review テンプレート

今後、各バッチ (Batch) および各フェーズ (Phase) のレビューにおいては、以下の公式テンプレートを使用してください。
「背景・現場課題」「Managed Risk」「Wording 宿題」を取りこぼさないための構造化されたテンプレートです。

📄 **詳細なテンプレート本体はこちら**: [RNA-Seq Suite Review Template](rnaseq-suite-review-template.md)

---

## 7. この文書の一文定義

Phase の判定とは、
機能が増えたかではなく、
**その Phase が守るべき受け渡し契約を壊さずに前進したか**
を done / hold / reject で判断するための gate である。