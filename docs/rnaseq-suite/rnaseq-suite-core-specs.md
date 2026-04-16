# RNA-Seq Suite Core Specs
Version: draft-0.1

## 1. Purpose
この文書の目的は、RNA-Seq Suite を構成する 6 つの正式な Core Spec（契約）名称と、それぞれの「1 文責務」を固定することです。これにより、Counter, Reporter, Adapter, Export, Handoff 間の設計や議論において、**Spec 名称が揺れない状態**を作り出し、Spec 間の責務重複を未然に防ぎます。

## 2. Scope of This Document
この文書は、Suite アーキテクチャの根幹となるコア仕様の名前空間と責務境界の **Source of Truth** です。ここに含まれないシステムアーティファクト名に類義語を新設したり、派生語を利用して正式名称を増やすことは禁止されます。

## 3. Official Core Spec Names
以下の 6 つを正式名称として固定します。これ以外の別名や略称を正式な契約名として増やすことは厳禁です。

1. `AssaySpec`
2. `MatrixSpec`
3. `ExecutionRunSpec`
4. `ComparisonSpec`
5. `ResultSpec`
6. `ReportPayloadSpec`

---

## 4. Core Specs Definition

### 4.1 AssaySpec
#### 1 文責務 (Role)
解析対象 assay と、その入力条件・由来を定義する契約。
#### Primary Owner / Main Consumers
- **主担当 (Owner):** Counter intake / Execution 前段
- **主利用層 (Consumer):** Counter
#### In Scope
- 解析に使用される生データ（FASTQ 等）の参照。
- サンプルレベルのメタデータと状態設計。
#### Out of Scope / Forbidden Overlap
- **入れてはいけないもの:** 統計比較設計、DEG 結果、レポートの Summary。

### 4.2 MatrixSpec
#### 1 文責務 (Role)
count / expression matrix の標準表現を定義する契約。
#### Primary Owner / Main Consumers
- **主担当 (Owner):** Counter output / Reporter input
- **主利用層 (Consumer):** Reporter
#### In Scope
- 発現量等の定量マトリックスの構造と型。
#### Out of Scope / Forbidden Overlap
- **入れてはいけないもの:** 実行ログ本文、統計結果自体、人間向けの定性的な解釈文 (Narrative)。

### 4.3 ExecutionRunSpec
#### 1 文責務 (Role)
実行条件・実行結果・Provenance を記録する契約。
#### Primary Owner / Main Consumers
- **主担当 (Owner):** Counter / Reporter 共通 Provenance
- **主利用層 (Consumer):** Suite 全体、後続の Handoff/Rerun
#### In Scope
- 実行時の環境情報、入力パラメータ、完了状態、リファレンスのバージョン情報。
#### Out of Scope / Forbidden Overlap
- **入れてはいけないもの:** 生の Matrix 本体、DEG Table 本体、Report Narrative。

### 4.4 ComparisonSpec
#### 1 文責務 (Role)
比較設計と統計的比較対象を定義する契約。
#### Primary Owner / Main Consumers
- **主担当 (Owner):** Reporter / Adapter normalization
- **主利用層 (Consumer):** Reporter
#### In Scope
- 比較する群（Case/Control など）の関係性、因子、条件設定。
#### Out of Scope / Forbidden Overlap
- **入れてはいけないもの:** 実行ログ、Result Table、Figure や Report の Assembly（組み立て）情報。

### 4.5 ResultSpec
#### 1 文責務 (Role)
統計解析結果そのものを保持する契約。
#### Primary Owner / Main Consumers
- **主担当 (Owner):** Reporter analytics output
- **主利用層 (Consumer):** Reporter、Handoff Export
#### In Scope
- DEG などの統計的推定値、P 値、効果量などの解析計算結果の数値データ。
#### Out of Scope / Forbidden Overlap
- **入れてはいけないもの:** レポート作成上のテンプレート・構造情報、UI レイアウト定義、定性文 (Narrative) 主体の Summary。

### 4.6 ReportPayloadSpec
#### 1 文責務 (Role)
UI / HTML / PDF / handoff に共通利用するレポート用材料を束ねる契約。
#### Primary Owner / Main Consumers
- **主担当 (Owner):** Reporter UI / Export / Handoff
- **主利用層 (Consumer):** ユーザー、Downstream Consumer
#### In Scope
- レポートを構成する可視化情報、図表への参照、エンドユーザーに提示する Summary コンテンツ。
#### Out of Scope / Forbidden Overlap
- **入れてはいけないもの:** Raw Vendor 差異吸収ロジック、実行 Engine 固有の内部状態、統計計算結果（Source of Truth である ResultSpec）を無視した独自の Result 保持。

---

## 5. Cross-Spec Boundary Rules
1. **Source of Truth の分裂禁止**: Result の結果は `ResultSpec` から生み出され、`ReportPayloadSpec` で参照される形で接続されます。`ReportPayloadSpec` が独自に再計算された Result を保持してはいけません。
2. **Absolute Path / Environment Dependency**: どの Spec においても、自身の中に「ローカル環境絶対パス (Absolute Path)」や特定の Host に依存する Locator を **Core 責務として組み込むことは禁止** されます。常に Relative 参照か、安定した Identifier を使用すること。

---

## 6. Naming Guardrails
現時点 (v1.0.0 ターゲット) では、法的・監査的・治験的な確定性を過度に示唆する名称（Legal / Regulatory な重みを持つ語）を、Core Spec 名やシステム成果物の正式名称として採用することを **固く禁じます**。

**採用禁止ワード (v1.0.0 以降の拡張候補として保留):**
- `Audit` (例: AuditTrailSpec)
- `Compliance` (例: ComplianceRecord)
- `Certified`
- `Validated`
- `Approved`
- `Regulatory`
- `GxP`
- `Submission`

現行のシステムアーティファクトの命名にこれらの語が混入することは、機能スコープ以上の法的説明責任を招きかねないため承認できません。

---

## 7. Future Reserved but Unused Terms
機能拡張に合わせて以下のような概念が登場する可能性がありますが、これらはすべて **「将来候補 (Post-v1.0.0) / 現在未使用」** として厳格に隔離されます。本文書の定める主契約と混同させてはいけません。

- `AuditTrailSpec`：**将来候補** であり、v1.0.0 では未使用。
- `ComplianceRecord`：**将来の厳格運用向けメモ** であり、現行の system artifact 名ではない。

---

## 8. Follow-up Items After B20-02
以下の項目は、本バッチ (B20-02) の対象外であり、次以降の改善で決定します。
- 各 Spec の全 Field の完全な型定義と Schema Convention 実装。
- `ExecutionRunSpec` と上流の `JobRequestSpec` (仮) の厳密な境界線。
- Spec に Overlay される Metadata の最小 Required フィールドの決定。
- Source Artifact Identity をどのレイヤーの Spec に配置するかの決定。

---

## 9. B20-02 Done Definition
- [x] 6 Core Spec の正式名称が固定されていること。
- [x] 各 Spec の責務が 1 文で明快に固定されていること。
- [x] Spec 間の「入れてはいけないもの (Forbidden Overlap)」が明記され、責務重複が抑制されていること。
- [x] 強すぎる法務・監査系ワード (Audit/Compliance 等) が排除され、Naming Guardrails が設けられていること。
- [x] 未確定事項が Follow-up Item として次段へ送られていること。
