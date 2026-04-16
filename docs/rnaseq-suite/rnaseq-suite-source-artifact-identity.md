# RNA-Seq Suite Source Artifact Identity
Version: draft-0.1

## 1. Purpose
この文書は、RNA-Seq Suite において入力データ（Source Artifact: FASTQファイル、メタデータシート等）をどのように識別・記録するかに関する共通ルール（Convention）を定義するものです。
アプリごと（Counter, Reporter, Adapter等）で記録形式・方言が乱立するのを防ぎ、Provenance（来歴情報）や Rerun（再実行）時に必要な同一性の共通語彙を提供することを目的とします。

## 2. Scope of This Document
本文書では、入力ファイルを識別・保護するための **「Lightweight (軽量な) Identity / Integrity」** に焦点を絞ります。
法的・監査的・治験対応に求められるような「Heavy Governance（重厚な監査基準対応）」機能の設計は本文書のスコープ外（v1.x 以降の拡張）とします。

## 3. Lightweight Identity / Integrity Principles

#### Lightweight Identity（軽量な識別性）
「この run または report が、過去のどの Source Artifact（入力ファイル）に基づいて生成されたか」を、Suite 内で安定して追跡・説明できる最小限の識別情報の記録を指します。

#### Lightweight Integrity（軽量な完全性）
受領元が提供する checksum、軽量に取得可能な検査情報、または size / format / capture context などの補助情報により、
受領時点の Source Artifact に対する同一性確認や破損検知を支援するための技術的記録を指します。
v1.0.0 では、全 artifact に対する重い完全性計算や厳格な原本性担保は要求しません。

## 4. What v1.0.0 Includes
v1.0.0 のスコープにおける Source Artifact の記録（`SourceArtifactRecord`）では、以下を満たすことを目指します。
- Counter / Reporter / Adapter 等が、共通の構造で Source Artifact の Identity を保持できること
- Integrity に関する技術的補助情報（例: 受領元が提供する checksum、軽量に取得可能な検査情報）を、必要に応じて記録できること
- 意図せぬファイル破損や取り違え（Anomaly）を、後工程または再実行時に論理的手段で検知できること

## 5. What v1.0.0 Explicitly Does Not Include
逆に、重いガバナンス要件が混入しないよう、**以下は明確に v1.0.0 の要件から除外・禁止とします。**

- **法的真正性・法的原本性の担保** (Legal/Regulatory Audit requirements)
- **電子的署名 (Electronic Signatures) による真正性担保**
- **承認ワークフロー (Approval Workflows)**
- **WORM (Write Once Read Many) ストレージを前提とした物理的保護**
- **法的原本管理そのものの制度化**

> [!WARNING]
> 将来的な追加（Strict Mode / Governance Extension）の余地は残しますが、現行の中核契約（Core Spec）の名前や実態に「Audit」「Approved」等といった法的監査を彷彿とさせるフィールド・機能を混入させることは禁止します（Heavy Governance 非介在の原則）。

---

## 6. SourceArtifactRecord

### 6.1 Role
`SourceArtifactRecord` は、単一の Source Artifact につき一つの記録構造を持ちます。
記録をアプリ個別の辞書や JSON 仕様に散らさず、本レコード形式に集約することで、全モジュールが同一の前提でデータの出処を参照できるようにします（方言の排除）。

### 6.2 Minimal Fields
最低限、以下の項目を固定フィールドとして持ちます（Dataclass化等の実装は本段階では不要）。
- `artifact_id`
- `artifact_role`
- `original_filename`
- `received_at`
- `received_from` (Optional)
- `storage_locator`
- `byte_size`
- `checksum_type` (Optional)
- `checksum_value` (Optional)
- `media_type`
- `format`
- `capture_mode`
- `metadata`

### 6.3 Field Semantics
各フィールドのセマンティクス（意味付け）は以下の通りです。

- **`artifact_id` (必須)**: Source Artifact を Suite 内で一意に識別する安定した ID (Stable Identity)。人間向けの便利名称 (Convenience Name) とは区別されます。
- **`artifact_role` (必須)**: FASTQ, sample_sheet, vendor_result 等の役割分類。アプリごとに独自用語（方言）を作ってはいけません。
- **`original_filename` (必須)**: 受領時に付与されていた元々のファイル名。Official Identity ではなく、あくまで参考・付帯情報としての役割です。
- **`received_at` (必須)**: ファイルを受領・認識した時刻。B20-03 で定めた ISO 8601 (Timezone必須) フォーマットに従います。
- **`received_from` (Optional)**: 受領元を示す記録（外注ベンダー名やシステム名など）。意味は固定ですが、入力の有無は任意 (Optional) とします。
- **`storage_locator` (必須)**: Source Artifact が置かれている参照識別子（Official Ref）。**※本項目は後述の「Portability 原則と Locator ルール」の強い制約を受けます。**
- **`byte_size` (必須)**: 受領時点のファイルサイズ（バイト数）。取り回しの良い軽量な補助情報として含めます。
- **`checksum_type` (Optional)**: 受領元が提供する、または軽量に取得可能な場合に記録する Checksum 算出アルゴリズム種別。
- **`checksum_value` (Optional)**: 受領元が提供する、または軽量に取得可能な場合に記録する Checksum 値。v1.0.0 では全 Source Artifact に対する算出を必須としない。
- **`media_type` (必須)**: MIME に相当するメディア種別（例: `text/plain`, `application/gzip`）。必要以上に Vendor 固有化しないようにします。
- **`format` (必須)**: ファイル内のデータフォーマット（FASTQ, CSV, TSV 等）。`media_type` とは役割が異なることを明示します。
- **`capture_mode` (必須)**: 受領の形態（`direct_upload`, `copied_from_vendor`, `generated_sidecar_reference` 等）。列挙の詳細決定は後続になりますが、役割は固定です。
- **`metadata` (Optional)**: その他の補助情報。ただし、上位要素で規定された Canonical なフィールドを入れる逃げ道（何でも箱）としては使用しないでください。

---

### 6.4 Checksum Handling Policy
- 受領元（vendor 等）が checksum を提供している場合は、それを記録してよい。
- ローカルで軽量に取得可能な場合は、checksum を記録してよい。
- checksum 算出が重い場合、v1.0.0 ではその場で無理に全件算出することを標準必須としない。
- 利用ユーザーが明示的に求める場合は、全件 checksum 取得を optional に実施してよい。
- その場合は、「この処理は重い」ことを明示する。
- 治験系や高管理運用では、後続の strict mode / governance extension 側で全件取得前提として扱ってよい。

---

## 7. Official Locator Rules

ここでの「Locator（識別子・参照値）」は、B20-03 から続く Portability（可搬性）を守るための最重要項目です。  
現場での誤読「URI が入っているから、アプリが勝手に別サーバーのデータをダウンロードしに行って良い」を防ぐため、強い制約を設けます。

### 7.1 Relative Ref / Stable Identifier / Host-bounded Locator
Official Locator として許容される表現は、以下に限られます。
1. **Relative Ref** (相対参照)
2. **Environment-independent Stable Identifier** (環境に依存しない安定した識別子)
3. **Application-boundary local reference / Host-bounded Locator** (アプリがデプロイされている同一ホスト・運用境界の内側でのみ解決される参照)

### 7.2 Remote Fetch Is Forbidden
Official Locator（例: URI や URL 様の文字列）は **「リモートデータを取得・フェッチするための命令 (Command/Privilege)」ではありません。**  
あくまで **「識別やアクセス権の記録 (Identity Record)」** です。  
URI の記載をもって「他ホスト / 他サーバーのデータを自動的に取りに行く（Remote Fetch）」運用を正当化することは固く禁じます。

### 7.3 Absolute Path Is Forbidden
B20-03 でも宣言された通り、実行環境に過度に依存する **「Absolute Path (絶対パス)」** や **「Host-specific Local Absolute Path」** を Official Locator として登録することは禁止します。

---

## 8. Relationship to Provenance / Sidecar / Packet
`SourceArtifactRecord` の「物理的な置き場所」については、現時点（B20-06）において最終確定させません（sidecar ファイル, Provenance subobject, 納品 Packet 内の payload のいずれになるかは柔軟に後送りとします）。
重要なのは、**「位置がどこであれ、論理的に同じレコードであることをアプリごとに別の方言で持たない（単一の構造で記述される）」** ことです。
また、本構造は独立した重監査用 Spec ではありません。

---

## 9. Heavy Governance Non-Inclusion
セクション5の内容を強調します。
Suite v1.0.0 は、あくまで**「Lightweight Identity」** の維持を至上命題とします。
電子的原本管理機能（電子署名、WORM、法的承認）を用いた監査証跡の制度化は、v1.x 以降の「Governance Extension」として取り扱うべきであり、現段階で `SourceArtifactRecord` を過剰な法務用途に複雑化させないことを宣言します。

---

## 10. Forbidden Patterns
これまでの定義に基づき、以下のアンチパターン (NG設計) を明示して固く禁じます。

- ❌ 電子署名や法的原本管理そのものを v1.0.0 の実装要件に混ぜる
- ❌ Counter / Reporter / Adapter 等で別々の Artifact Record 仕様（方言）を独自に運用する
- ❌ Official Locator に Absolute Path (フル絶対パス) を混入させる
- ❌ Official Locator の URI をもって「外部ホスト・別ネットワーク層のデータをダウンロードする命令」として解釈する

---

## 11. Follow-up Items After B20-06
本バッチで方針が固定された後、実装レベルで決定すべき残件 (Follow-up) は以下の通りです。

- `SourceArtifactRecord` の最終的な物理配置場所（Sidecar vs Provenance Subobject 等）の決定
- `capture_mode` でサポートする列挙型 (Enum) の詳細な洗い出しと決定
- `received_from` を本当に Required に格上げするかどうかの最終的な要件確認
- strict mode での checksum full capture や、治験系 / 高管理運用時の full checksum policy の実現
- 以降の Batch (B30+, B80+) に向けた Dataclass / Validator 実装への落とし込み

---

## 12. B20-06 Done Definition
- [x] Source Artifact 記録に関する最小項目（`SourceArtifactRecord` の13フィールド）が固定されている。
- [x] Lightweight Identity と Heavy Governance （法的原本・承認等）の境界線と「v1.x へ後送りする方針」が明確である。
- [x] Official Locator における Absolute Path 禁止および「Remote Fetch を意味しない」原則が明記され、Portability が守られている。
- [x] これら原則により Counter / Reporter / Adapter 間での方言乱立リスクが閉ざされている。
- [x] B20-03 (Schema Convention), B20-04 (Handoff Boundary) との内容矛盾が存在しない。
