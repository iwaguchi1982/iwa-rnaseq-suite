# RNA-Seq Suite Schema Conventions
Version: draft-0.1

## 1. Purpose
この文書の目的は、RNA-Seq Suite 全体で利用されるすべての Core Spec（契約）に対して、**共通で適用される Schema ルール (Schema Conventions)** を定義し固定することです。
Spec ごとの「方言 (format drift)」を徹底的に排除し、将来のデータベースへの取り込みや社内ナレッジ検索において、パースエラーやメタ情報汚染（Downstream 汚染）が発生することを防ぎます。

## 2. Scope of This Document
この Conventions は、スイートを構成する **6 つの Core Spec（AssaySpec, MatrixSpec, ExecutionRunSpec, ComparisonSpec, ResultSpec, ReportPayloadSpec）全て** に強制適用されます。
一時的な内部状態や UI 表示用の Convenience な変換には適用されませんが、System Artifact（出力結果として永続化される JSON/Handoff 等）には厳守されなければなりません。

## 3. Global Convention Principles
- **Drift (ブレ) は許容しない:** 「この Spec だけ特別に別フォーマットを許す」などの例外的な Schema 実装を禁止します。
- **ダウンストリーム任せにしない:** 「あとで DB 側や検索基盤側で表記揺れを吸収する」といった甘えた設計を認めません。基盤側が単一フォーマットを信頼できるように、データ生成元 (Counter / Reporter 等) で固定します。

---

## 4. Common Header Rules
全ての Spec は、ルートレベルに以下の 4 つの共通ヘッダ要素を必ず持ちます。これらはどの Spec でも同じ意味・同じ役割を果たします。

### 4.1 schema_name
- **ルール:** 各 Spec の公式な識別子です。Suite 全体で一意でなければなりません。
- **原則:** B20-02 文書で定めた正式名称 (`AssaySpec`, `ExecutionRunSpec` 等) に直結する Stable Name を使用すること。略称や類義語への変更は禁止します。

### 4.2 schema_version
- **ルール:** 全 Spec にて必須のフィールドです。
- **原則:** バージョン表記形式は Suite 全体で統一されなければなりません。
- **フォーマット:** SemVer (Semantic Versioning) 形式 (`x.y.z` の書式。例: `1.0.0`) のみを正とします。「バージョンなし」や「日付ベース(20260412)」といった Spec ごとに異なる版管理を禁止します。

### 4.3 metadata
- **ルール:** 補助的な記述情報やシステムトラッキングに必要な付加情報を保持する領域です。
- **原則:** 対象 Spec の「Core 責務 (主データ)」を置き換えたり、本来 Canonical なフィールドに置くべき設定情報を逃がし続けたりする場所として不正利用してはいけません。

### 4.4 overlay
- **ルール:** 将来拡張や、プロジェクト局所的な情報の「逃がし先」として機能する拡張領域です。
- **原則:** Core フィールドを汚さないために、一時的なフラグ追加等はこの `overlay` 内部に留めること。Spec ごとに `overlay` の意味（目的）を変えないこと。

---

## 5. Provenance Positioning
- **位置づけ:** Provenance (由来・実行履歴) の記録は、Suite 全体の最重要原則です。特に `ExecutionRunSpec`, `ResultSpec`, `ReportPayloadSpec` において強く要求されます。
- **ルール:** Provenance は、人間向けの「解釈文 (Narrative)」ではなく、計算機的に追跡可能な「履歴・関連情報 (Traceability)」として構造化されなければなりません。Spec ごとに Provenance を別概念（例えばただの説明文フィールド）として再定義することは禁止です。

---

## 6. ID Rules
- **一意性の確保:** 各システムアーティファクトの ID は、スイートを通して一意性を保持します。
- **ルール:** 人間からの見栄え (Human convenience) よりも、安定した識別子 (Stable Identity) を優先します。
- **命名原則 (Prefix-based ID):** Spec 種別を視認可能にし、衝突を防ぐため、必ず以下の Prefix に従った命名規則を使用します。例外は認めません。
  - `ASSAY_XXXX`
  - `MAT_XXXX`
  - `RUN_XXXX`
  - `COMP_XXXX`
  - `RES_XXXX`
  - `PAYLOAD_XXXX`

---

## 7. Date / Time Rules
本ルールの遵守は**最優先**です。日付・時刻のフォーマットのブレは Downstream を最も汚染します。

- **統一フォーマット:** **ISO 8601 (RFC 3339 拡張)** のみを正式な Contract 表現として固定します。
- **要件:**
  - 必ず Timezone (UTC offset) を含めること。
  - **許可される例:** `2026-03-27T21:57:37+09:00`, `2026-03-27T12:57:37Z`
  - **禁止される例:** `2026/03/27`, `26-03-2026`, タイムゾーン不明の文字列、Epoch 整数。
- **原則:** Contract レベルでは上記を強制します。UI での「YYYY/MM/DD」等のローカル整形は UI 側の責務であり、Storage や Spec の内部状態には決して混ぜ込まないこと。

---

## 8. Unit / Quantity Rules
- **統一ルール:** 値 (Value) と単位 (Unit) の責務を曖昧にしてはいけません。
- **分離の原則:** 単位を持つ量は、必ず「数値としての Value」および「単位を表す Unit 文字列」に分離して保持すること。
- **禁止表現:** `"10GB"`, `"5.0 pmol"`, `"100 bp"` のような、「数値と単位が結合された一つの文字列」をフィールドとして定義することを固く禁じます。

---

## 9. Path / Locator / Reference Rules
- **Portability 原則:** どのように実行・移動しても壊れないシステム参照を目指します。
- **Absolute Path 禁止:** `/home/user/data/fastq/sample.fq.gz` のような絶対パス (Absolute Path) や、ローカルホストに固有の Locator 文字列を、Schema Core 契約の中に永続層として持たせることを禁止します。
- **解決策:** 常に Relative Ref (バンドルなど相対位置からの解決) または URI / Stable Identifier を優先します。Runtime の便宜上解決されたフルパスと、契約（Spec）として書き出される Official Ref は完全に分離してください。
- **Source Artifact Identity 連携:** これらの Locator ルールの具体的な `SourceArtifactRecord` における制約（Remote Fetch禁止等の厳格解釈）および軽量な運用原則については、[rnaseq-suite-source-artifact-identity.md](file:///home/manager/iwa_bio_analysis_orchestra/dev_docs/rnaseq-suite-source-artifact-identity.md) を強く参照および遵守してください。

---

## 10. Forbidden Exception Rules
以下の「例外を認めるような妥協」は、いかなる機能追加・暫定対応においても明確に**禁止 (Forbidden)** されます。例外処理は常に Downstream の破壊を招きます。

1. **“この Spec だけ別の schema_version 形式” を禁止** (SemVer以外の採用禁止)。
2. **“この Spec だけ別の Date Format” を禁止** (部分的に `yyyy/mm/dd` にする等)。
3. **“この Spec だけ単位付き文字列” を禁止** (特定の Adapter が吐くゴミフォーマットのそのままの受け入れ等)。
4. **“この Spec だけ Absolute Path 可” を禁止** (パスに依存した実装の禁止)。
5. **“今回だけの暫定表現” を正式 Contract に登録することを禁止** (実装都合の一時しのぎは `overlay` 等にも逃がさず、Convenience コード内で解決すること)。

---

## 11. Known Convention Conflicts / Open Questions
B20-03 時点での既知の衝突や未確定事項は以下の通りです。
- **Migration/Version Up:** Major/Minor/Patch をいつどのタイミングで（Breaking Changes発生時にどう）Increment するかの厳密なトリガー要件は未確定。
- **Provenance Coverage:** どこまでの深さの Provenance（使用コンテナの Digest まで持つか等）を各 Spec の必須フィールドとするかの粒度が未確定。

## 12. Follow-up Items After B20-03
- 各 Spec の厳密な Pydantic/JSON Schema Dataclass への落とし込み (B30+, B40+)
- Schema Drift に対する自動 Validator の実装
- `Source Artifact Identity` の配置箇所の決定

---

## 13. B20-03 Done Definition
- [x] 全 Spec に適用される共通の Schema Conventions が文書として確立されている。
- [x] Header (`schema_name`, `schema_version`, `metadata`, `overlay`) の利用ルールが固定されている。
- [x] 日付・時刻の単一フォーマット (ISO 8601) が固定され、例外が塞がれている。
- [x] Unit/Quantity および Path/Locator/ID のブレないルールが固定されている。
- [x] "DB 側で吸収する" という妥協や "Spec ごとの例外ルール" を決して容認しないことが明記されている。
