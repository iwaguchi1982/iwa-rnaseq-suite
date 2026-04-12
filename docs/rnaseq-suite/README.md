# RNA-Seq Suite Docs

このディレクトリは、`RNA-Seq Suite` の設計文書・運用文書の置き場です。  
`iwa-rnaseq-counter` と `iwa-rnaseq-reporter` を、**個別アプリではなく suite として整合的に開発するための source of truth** をまとめます。

## 位置づけ

RNA-Seq Suite は、主に以下の app / レイヤで構成します。

- `iwa-rnaseq-counter`
  - execution app
  - FASTQ から count matrix 相当の中間成果物生成までを担当
- `iwa-rnaseq-reporter`
  - analytics / report app
  - counter 生成物または正規化済み入力をもとに解析・可視化・出力を担当
- adapter layer
  - 外部委託納品物や vendor 差異の正規化を担当

この `docs/rnaseq-suite/` 配下には、上記を **contract / workflow / milestone / gate** の観点で固定する文書を置きます。

---

## まず読む順番

初見のときは、以下の順で読む想定です。

1. `rnaseq-suite-depmap.md`
   - 全体像と開発マップ
2. `rnaseq-suite-app-boundary.md`
   - counter / reporter / adapter の責務境界
3. `rnaseq-suite-core-specs.md`
   - core spec の定義
4. `rnaseq-suite-schema-conventions.md`
   - schema 共通ルール
5. `rnaseq-suite-minimal-workflow.md`
   - 最小 workflow / handoff
6. `rnaseq-suite-milestone.md`
   - suite milestone 全体像
7. `rnaseq-suite-gate-decision-table.md`
   - phase ごとの done / hold / reject 基準
8. `rnaseq-suite-implementation-batch.md`
   - batch 単位の分解
9. `rnaseq-suite-review-template.md`
   - レビューの共通テンプレ
10. `rnaseq-suite-inspection-checklist.md`
    - 実装・レビュー時の確認観点
11. `rnaseq-suite-managed-risk.md`
    - managed risk / 宿題
12. `rnaseq-suite-source-artifact-identity.md`
    - source artifact identity 方針

---

## 文書一覧

### 全体設計
- `rnaseq-suite-depmap.md`
- `rnaseq-suite-app-boundary.md`
- `rnaseq-suite-minimal-workflow.md`

### spec / contract
- `rnaseq-suite-core-specs.md`
- `rnaseq-suite-schema-conventions.md`
- `rnaseq-suite-source-artifact-identity.md`

### 開発計画 / gate
- `rnaseq-suite-milestone.md`
- `rnaseq-suite-gate-decision-table.md`
- `rnaseq-suite-implementation-batch.md`

### レビュー / 運用
- `rnaseq-suite-review-template.md`
- `rnaseq-suite-inspection-checklist.md`
- `rnaseq-suite-managed-risk.md`

---

## このディレクトリで管理するもの

- suite 全体の設計原則
- app boundary
- shared contract
- schema convention
- workflow / handoff
- milestone / phase / batch 計画
- gate review / inspection の基準
- managed risk

## このディレクトリで管理しないもの

- counter の実装詳細そのもの
- reporter の実装詳細そのもの
- app 固有の unit test
- vendor 個別実装コード
- UI の細部仕様だけを切り出した断片文書

---

## 運用メモ

- app 実装の変更は、原則として各 repo 側で行う
- suite に影響する変更は、この docs を source of truth として先に定義または更新する
- review は、実装差分だけでなく、この docs と矛盾していないかで判断する
- milestone と package version は一致しなくてよい
- v1.0.0 では LLM 補助を入れない

---

## 今後追加される可能性があるもの

- adapter interface 文書
- counter-reporter integration scenario
- acceptance scenario
- release checklist
- rerun packet / handoff packet 定義
- known limitations / non-goals

---

## 一言まとめ

このディレクトリは、RNA-Seq Suite を  
**「個別アプリの集合」ではなく「契約でつながる suite」**  
として維持するための文書置き場です。