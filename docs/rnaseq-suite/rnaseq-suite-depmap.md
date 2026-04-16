# RNA-Seq Suite devmap
Version: draft-0.1
Owner: iwa-bio-analysis-orchestra
Scope: iwa-rnaseq-counter / iwa-rnaseq-reporter を中核とした RNA-Seq Suite 全体設計

---

## 1. 目的

RNA-Seq Suite は、RNA-Seq 解析における以下を標準化する。

1. FASTQ から count matrix 生成までの実行系ワークフロー
2. 生成された count / expression 系データの標準形式への統一
3. 統計解析・可視化・レポート出力
4. 社内実行結果と外部委託納品物の差異吸収
5. 将来の比較解析・シグネチャ解析・AI補助への接続基盤化

---

## 2. Suite 全体像

RNA-Seq Suite は、以下の3層で整理する。

### 2.1 Execution Layer
FASTQ から count matrix を生成する層。  
主担当アプリ: `iwa-rnaseq-counter`

責務:
- QC
- trimming
- mapping / pseudoalignment / quantification
- 実行条件、参照、バージョン、ログの保持
- matrix 出力の標準化

### 2.2 Normalization / Ingestion Layer
社内生成物と外部委託納品物を、suite 内の標準入力へ正規化する層。  
主担当: `iwa-rnaseq-reporter` の前段 adapter 群

責務:
- vendor ごとの差異吸収
- 入力ファイル検査
- 列名、ID、group 情報、比較情報の正規化
- 不足情報の検知
- 正規 spec への変換

### 2.3 Analytics / Report Layer
正規化済みデータに対し、比較・可視化・レポート生成を行う層。  
主担当アプリ: `iwa-rnaseq-reporter`

責務:
- QC summary 表示
- PCA
- correlation heatmap
- DEG
- volcano plot
- heatmap
- enrichment
- gene search
- HTML / PDF 等への出力
- machine-readable payload 出力

---

## 3. 中核アプリの責務

## 3.1 iwa-rnaseq-counter

### 役割
RNA-Seq の一次〜二次解析前段における実行系アプリ。  
FASTQ から count matrix の生成までを担う。

### 主な処理
- input validation
- QC
- trimming
- reference selection
- alignment / quantification
- count matrix 生成
- execution provenance 保存

### サポート対象
- STAR
- HISAT2
- kallisto
- salmon

### 原則
- 可視化ロジックを持ち込みすぎない
- 解釈や意思決定をしない
- 実行結果と provenance を正しく残す
- matrix を reporter が読める標準契約で返す

---

## 3.2 iwa-rnaseq-reporter

### 役割
RNA-Seq の統計解析・可視化・レポート生成アプリ。  
counter の出力だけでなく、外部委託納品物も受け止める標準化ハブを兼ねる。

### 主な処理
- normalized input loading
- metadata / group / contrast validation
- exploratory analysis
- DEG analysis
- enrichment
- report generation
- structured payload export

### 原則
- vendor 固有ロジックを本体に埋め込まない
- 可視化本体は正規化済み入力のみを扱う
- 外部委託差異は adapter で吸収する
- result と narrative を混ぜない

---

## 4. Non-goals

この suite では現時点で以下を主責務にしない。

- 臨床意思決定そのもの
- target prioritization そのもの
- AI による全自動結論生成
- 疾患固有ロジックの core 埋め込み
- vendor ごとの例外処理を reporter 本体へ蓄積すること
- raw PDF や画像を source of truth とした解析

---

## 5. 設計原則

## 5.1 Contract First
機能追加より先に入出力契約を固定する。

## 5.2 Adapter Isolation
外部委託企業ごとの差異は adapter に隔離する。

## 5.3 Provenance Required
生成物には provenance を持たせる。

## 5.4 Disease-agnostic Core
tumor-normal 固定、onco 固定語彙を core に埋め込まない。

## 5.5 Result / Narrative Separation
統計結果と説明文を混ぜない。

## 5.6 Minimal Public Contract Change
公開 contract をむやみに変更しない。

---

## 6. Canonical Spec

RNA-Seq Suite で優先して固定する spec は以下。

## 6.1 今すぐ実装に直結する spec
- `AssaySpec`
- `MatrixSpec`
- `ExecutionRunSpec`
- `ComparisonSpec`
- `ResultSpec`
- `ReportPayloadSpec`

## 6.2 名前だけでも早期固定しておく spec
- `SubjectSpec`
- `VisitSpec`
- `SpecimenSpec`
- `ReferenceDatasetSpec`
- `SignatureSpec`
- `JobRequestSpec`

---

## 7. 各 spec の責務

## 7.1 AssaySpec
解析対象 assay の定義。

最低限含めたい項目:
- assay_id
- assay_type
- species
- read_layout
- strandedness
- reference_build
- annotation_version
- input_files
- sample_refs
- metadata
- overlay

## 7.2 MatrixSpec
count / expression matrix の標準契約。

最低限含めたい項目:
- matrix_id
- matrix_kind
- value_type
- gene_id_type
- gene_annotation_version
- sample_ids
- feature_ids
- file_refs
- normalization_state
- provenance
- metadata
- overlay

## 7.3 ExecutionRunSpec
実行条件と実行結果の provenance 契約。

最低限含めたい項目:
- run_id
- app_name
- app_version
- toolchain
- command_summary
- started_at
- finished_at
- status
- input_refs
- output_refs
- resource_usage
- log_refs
- environment
- metadata

## 7.4 ComparisonSpec
比較設計の契約。

最低限含めたい項目:
- comparison_id
- design_type
- sample_grouping
- contrasts
- covariates
- exclusion_rules
- statistical_method
- metadata
- overlay

## 7.5 ResultSpec
統計解析結果の標準契約。

最低限含めたい項目:
- result_id
- result_kind
- comparison_ref
- matrix_ref
- method
- thresholds
- table_refs
- summary_metrics
- provenance
- metadata

## 7.6 ReportPayloadSpec
UI / HTML / PDF / handoff に共通利用できるレポート用 payload 契約。

最低限含めたい項目:
- payload_id
- project_summary
- qc_summary
- analysis_summary
- figure_refs
- table_refs
- result_refs
- warnings
- narrative_slots
- provenance
- metadata

---

## 8. 正規データフロー

## 8.1 Counter 主導フロー
`FASTQ -> AssaySpec -> counter execution -> MatrixSpec + ExecutionRunSpec`

## 8.2 Reporter 主導フロー（内部生成物）
`MatrixSpec + ComparisonSpec -> reporter -> ResultSpec + ReportPayloadSpec + ExecutionRunSpec`

## 8.3 Reporter 主導フロー（外部委託納品物）
`Vendor files -> VendorAdapter -> normalized MatrixSpec / ComparisonSpec -> reporter -> ResultSpec + ReportPayloadSpec`

---

## 9. 外部委託納品物対応方針

外部委託先ごとに納品形式が異なることを前提とし、以下の方針を採る。

### 9.1 Reporter 本体に vendor 個別分岐を持ち込まない
例:
- A社専用 CSV
- B社専用 Excel
- C社専用 TSV
- 特定列名の揺れ
- 独自 group 命名

これらは adapter 層で吸収する。

### 9.2 Adapter の責務
- required file existence check
- header / column mapping
- sample ID normalization
- gene ID normalization
- comparison candidate extraction
- metadata補完
- 欠損項目の warning 化
- spec 変換

### 9.3 Adapter の非責務
- DEG 本体実装
- plot 本体実装
- report template 実装
- 疾患解釈
- narrative 生成

### 9.4 入力クラス
- `counter_native`
- `vendor_normalized`
- `manual_import`

---

## 10. Counter の設計方針

## 10.1 モジュール分割
- intake
- QC
- trimming
- reference resolver
- quant backend
- matrix builder
- execution recorder
- export

## 10.2 backend abstraction
各 backend は共通抽象で扱う。

対象:
- STAR
- HISAT2
- kallisto
- salmon

共通化ポイント:
- backend name
- backend version
- input requirements
- output contract
- strand / paired-end compatibility
- transcript / gene level distinction
- reference dependency
- log extraction point

## 10.3 Counter の done 条件
- FASTQ から標準 `MatrixSpec` を返せる
- 実行記録が `ExecutionRunSpec` で残る
- 失敗時に status / error summary / log ref が残る
- reporter が追加変換なしで読める最小契約が成立する

---

## 11. Reporter の設計方針

## 11.1 モジュール分割
- intake
- adapter
- validation
- exploratory analysis
- differential analysis
- functional analysis
- report assembly
- export
- execution recorder

## 11.2 Reporter の source of truth
Reporter は以下のみを source of truth とする。
- 正規化済み `MatrixSpec`
- 正規化済み `ComparisonSpec`
- 関連 metadata
- `ExecutionRunSpec` 等の provenance

### 禁止
- vendor raw JSON の直読みに戻ること
- 可視化関数内で勝手に group 解釈すること
- UI 層で design 情報を再解釈すること

## 11.3 Reporter の done 条件
- 正規化済み入力から主要可視化が再現できる
- 結果が `ResultSpec` にまとまる
- レポート用材料が `ReportPayloadSpec` にまとまる
- HTML/PDF/UI が payload から再構成可能である

---

## 12. AI 補助機能の位置づけ

AI は RNA-Seq Suite の主結果を置き換えない。  
あくまで補助層とする。

### 初期フェーズで許可するもの
- QC warning summary
- 注意点の箇条書き補助
- レポート narrative の下書き
- レビュー用コメント草案

### 初期フェーズで禁止するもの
- DEG 結果そのものの決定
- group design の自動確定
- 自由文だけを唯一の成果物にすること
- provenance のない断定

### 将来接続先
- `EvidenceRiskSpec`
- `ReviewDecisionSpec`
- `LLMEvidenceBundleSpec`

---


## 13. Future Governance Extension (Post-v1.0.0 Memo)

RNA-Seq Suite は v1.0.0 では、軽量な provenance / source artifact identity / rerun traceability を主対象とし、
高管理運用（例: ヒト遺伝子取扱い、高管理系、治験系、厳格な承認・保持・監査証跡を要する運用）を
hard scope に含めない。

ただし、v1.x 以降の拡張として、より厳格な governance mode を追加できる設計余地は保持する。
将来的な拡張候補には、以下を含みうる。

- stricter retention / traceability
- approval-oriented workflow
- governance-specific sidecar / record
- rerun / replay / provenance を高管理運用向けに強化する optional layer

### 13.1 命名上の注意
- これは将来拡張に関する覚書であり、現時点の canonical spec ではない
- 現時点では正式 core spec 名 / system artifact 名として `Audit`, `Compliance`, `Validated`, `Approved`, `Regulatory`, `GxP`, `Submission` 等の強い語を採用しない
- `AuditTrailSpec` 等の概念が必要になっても、v1.0.0 時点では **未採用・未使用** とし、将来の optional governance extension として扱う
- 内部設計メモや将来検討の対象とすることは妨げないが、現行成果物名・現行 contract 名としては出さない

### 13.2 AI 補助との関係
- governance extension は AI 補助とは別の論点である
- AI は引き続き結果本体を置き換えず補助層とする
- 高管理運用においても、result / narrative / governance record は混在させない

---

## 14. Suite マイルストン

## Phase 1: Core Contract Fix
目的:
- counter / reporter の契約固定

対象:
- AssaySpec
- MatrixSpec
- ExecutionRunSpec
- ComparisonSpec
- ResultSpec
- ReportPayloadSpec

完了条件:
- counter -> reporter の標準受け渡しが成立
- 最小の解析結果と payload が出る

## Phase 2: Reporter Stabilization
目的:
- reporter 主機能の安定化

対象:
- PCA
- correlation
- DEG
- volcano
- heatmap
- enrichment
- gene search
- export

完了条件:
- 正規入力で主要レポート構成を生成可能

## Phase 3: Vendor Adapter Layer
目的:
- 外部委託納品物の差異吸収

対象:
- vendor adapter interface
- validation rules
- warning rules
- import normalization

完了条件:
- 少なくとも主要 1〜2 形式を正規 spec に変換できる

## Phase 4: Comparator / Signature 接続
目的:
- 比較・解釈補助の拡張

対象:
- public-data comparator
- signature scorer

完了条件:
- reporter 出力と comparator / signature の接続 contract が成立

## Phase 5: AI Safety-oriented Assist
目的:
- 結果本体を壊さない AI 補助導入

対象:
- risk summary
- evidence bundle
- review assist

完了条件:
- result / narrative / review state が分離されている

---
## Future Governance Extension (Post-v1.0.0 Memo)

RNA-Seq Suite は v1.0.0 では、軽量な provenance / source artifact identity / rerun traceability を主対象とし、
高管理運用（例: ヒト遺伝子取扱い、高管理系、治験系、厳格な承認・保持・監査証跡を要する運用）を
hard scope に含めない。

ただし、v1.x 以降の拡張として、より厳格な governance mode を追加できる設計余地は保持する。
将来的な拡張候補には、以下を含みうる。

- stricter retention / traceability
- approval-oriented workflow
- governance-specific sidecar / record
- rerun / replay / provenance を高管理運用向けに強化する optional layer

重要:
- これは将来拡張に関する覚書であり、現時点の canonical spec ではない
- 現時点では正式 core spec 名 / system artifact 名として `Audit`, `Compliance`, `Validated`, `Approved`, `Regulatory`, `GxP`, `Submission` 等の強い語を採用しない
- `AuditTrailSpec` 等の概念が必要になっても、v1.0.0 時点では **未採用・未使用** とし、将来の optional governance extension として扱う
- 内部設計メモや将来検討の対象とすることは妨げないが、現行成果物名・現行 contract 名としては出さない
- v1.0.0 の完成条件は、法的な厳格監査運用ではなく、LLM なしで FASTQ 実行・標準化・可視化・納品物生成が一貫して成立する core suite である

---

## 15. レビュー観点

以下を suite 共通レビュー観点とする。

- source of truth を守った
- raw vendor 直読みに戻っていない
- public contract を壊していない
- adapter と本体の責務が混ざっていない
- provenance が落ちていない
- 例外を握りつぶしていない
- 今回やらないことに踏み込んでいない
- 疾患固有語彙を core に埋め込んでいない

---

## 16. 受け入れ基準

## 16.1 Counter 側
- 4 backend の責務境界が整理されている
- backend 差異が標準出力に正規化される
- matrix と execution 情報が分離されている
- reporter への handoff が可能

## 16.2 Reporter 側
- 正規入力のみで主要解析が成立する
- 外部納品物差異を adapter 側で吸収できる
- figure / table / summary が payload 化される
- UI と export が payload 依存で動く

## 16.3 Suite 側
- counter / reporter / adapter の責務境界が明確
- spec 名称と versioning が導入されている
- app ごとの独自 JSON 乱立を防げている

---

## 17. 運用ルール

### 17.1 すべての spec に必須
- `schema_name`
- `schema_version`
- `metadata`
- `overlay`

### 17.2 ID の原則
一意であること。  
例:
- `ASSAY_XXXX`
- `MAT_XXXX`
- `RUN_XXXX`
- `COMP_XXXX`
- `RES_XXXX`
- `PAYLOAD_XXXX`

### 17.3 破壊的変更
- schema version を更新する
- migration ルールを記録する
- changelog に残す

### 17.4 例外的要求への対応
現場要望があっても、以下は原則拒否対象。
- reporter 本体への vendor 直書き分岐
- spec を壊す暫定実装
- result と narrative の混在
- 疾患固有列の core 追加

必要なら以下に逃がす。
- adapter
- overlay
- 小型 utility script
- 別アプリ切り出し

---

## 18. 技術的リスク

### 18.1 最大リスク
reporter が「可視化アプリ」ではなく「例外吸収アプリ」に崩れること。

### 18.2 リスク回避策
- adapter 分離
- contract first
- source of truth 固定
- payload 中心設計

### 18.3 追加リスク
- backend ごとの差異で matrix contract が崩れる
- 外部納品物に metadata が不足する
- design 情報不備で比較が成立しない
- AI コメントが result を侵食する

---

## 19. 開発優先順位

1. `iwa-rnaseq-counter`
2. `iwa-rnaseq-reporter`
3. vendor adapter layer
4. `iwa-public-data-comparator`
5. `iwa-signature-scorer`
6. AI 補助層

---

## 20. この devmap のひとことでの定義

RNA-Seq Suite とは、  
RNA-Seq の実行・正規化・可視化・レポート化を、  
counter と reporter を中核に、標準 spec と provenance に基づいて接続する suite である。

---

## 21. 実装時の最重要メッセージ

- counter は execution の標準化
- reporter は analytics の標準化
- vendor 差異は adapter で吸収
- result と narrative は分離
- AI は補助層
- spec を先に固定する
