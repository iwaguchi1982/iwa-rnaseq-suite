# RNA-Seq Suite Review Template
Version: draft-0.1

## Quick Summary
- 今回の結論:
- 最大のリスク:
- 次に送ること:

## 用語ミニ辞書
- Source of Truth: 今回の判断で最優先で従う文書・定義
- Managed Risk: 今回見つかったが、記録したうえで次へ送る管理対象リスク
- Hidden Debt: 記録されずに埋もれる負債
- Wording Hardening: 今は意味が通るが、後で誤読防止のために磨くべき表現
- Suite Handoff 影響: 他 batch / phase / app への受け渡しに与える影響
- Follow-up: 今回は未着手だが、次以降で必ず扱うべき論点


## 1. Purpose
この文書は、RNA-Seq Suite の開発において「Batch ごと」および「Phase ごと」の合否判定（Gate Decision）を行う際の**公式レビューテンプレート**を定義するものです。
 レビューの視点が担当者によってブレることや、長期開発における「背景・現場課題」「残存リスク (managed risk)」「残件項目 (wording 宿題など)」が忘れ去られるのを防ぐため、必須フォーマットとして運用します。

## 2. Design Principles
本テンプレートは以下の3つの設計原則に基づいて構成されています。
1. **可読性 (Readability)**: 見出し階層を浅く保ち、必須項目と任意項目（補助メモ等）を明確に区分します。
2. **記入性 (Fillability)**: ゼロからの自由記述ではなく、穴埋め回答しやすい箇条書きの枠組を提供し、レビュー時の「何を書けば伝わるか」の迷いをなくします。
3. **管理性 (Manageability)**: 過去の Gate 記録を見返した際、同じ項目が同じ順序で並んでいる状態を担保し、次の Batch/Phase への「引継ぎ事項（Follow-up）」を絶対に漏らさない構造とします。

---

## 3. Batch Review Template
各実装バッチ（例: B20-04, B30-01）の完了判断時に使用するテンプレートです。「必須」と付記された項目は削除しないでください。

```markdown
# 完了報告: [Batch ID] [Batch 名]

## A. 基本情報 (必須)
- **Phase**: 
- **Batch ID**: 
- **Batch 名**: 
- **判定**: [DONE / HOLD / REJECT]

## B. 今回のゴール再確認 (必須)
- **この batch のゴール**: 
- **今回の Source of Truth**: 
- **今回やらないこと (Non-goals)**: 

## C. 背景 / 現場課題の再確認 (必須)
- **誰が困っているか**: 
- **何に困っているか**: 
- **今回の成果がその課題にどう効くか**: 

## D. 判定根拠 (必須)
- **何が固定されたか / 追加されたか**: 
- **何が確認・テストできたか**: 
- **なぜこの判定 (DONE/HOLD/REJECT) にするのか**: 

## E. Managed Risk / Known Conflict (必須)  
*(※なければ「特になし」で明記し、項目自体は消さない)*
- **今回見つかった新規リスク・境界違反の実態**: 
- **既知リスクの変化**: 
- **Hidden Debt (見えない負債) にしていないか**: 

## F. Wording / Naming / Documentation 宿題 (必須)
- **Wording Hardening 候補 (後で洗練させるべき文言)**: 
- **命名の保留事項 (Reserved Terms)**: 
- **将来誤読されやすいかも知れない表現**: 

## G. Suite Handoff 影響 (必須)
- **Suite 全体の受け渡し (Handoff) へのプラス影響**: 
- **残る留意点 / Downstream への汚染リスク**: 

## H. 次バッチ送り (必須)
- **次 Batch 候補**: 
- **今回やらなかったが今後必ず必要な Follow-up 要素**: 

## I. 非変更範囲 (必須)
- **何を触っていないか**: 
- **コード編集有無**: 
- **Scope 逸脱の有無**: 

## J. 補助メモ (任意)
- **参考 Commit / PR**: 
- **参考文書**: 
- **Reviewer コメント**: 
```

---

## 4. Phase Review Template
マイルストーン (Phase) 全体の完了時（例: Phase v0.20-v0.29 から v0.30 への移行）に使用する、大局観点のテンプレートです。

```markdown
# Phase Gate Review: [Phase 範囲 (例: v0.20-v0.29)]

## A. 基本情報 (必須)
- **Phase 範囲**: 
- **対象 Batch 群**: 
- **判定**: [Go / Hold / Re-plan]

## B. Phase の目的 (必須)
- **この Phase で何を固定する段階だったか**: 
- **何が終わっていれば次 Phase へ進める条件だったか**: 

## C. Phase 全体の成果 (必須)
- **この Phase で固定できたもの**: 
- **まだ未固定だが次へ進んでよいと判断したもの**: 
- **Phase 内で一貫して守れた原則**: 

## D. Managed Risk 集約 (必須)
- **Phase 全体で見えた顕著な Managed Risk / 技術的負債**: 
- **次 Phase へ送る重要課題**: 
- **見て見ぬふりをした Hidden Debt になっていないか**: 

## E. Wording / Naming 集約 (必須)
- **Phase 全体で残った Wording Hardening 候補一覧**: 
- **Naming の決定事項と保留事項**: 

## F. Architecture / Suite Handoff 影響 (必須)
- **Suite 全体の一貫性にどう効いたか**: 
- **App Boundary / Core Specs / Schema Conventions への整合性**: 
- **次 Phase へ渡すべき Architectural Note**: 

## G. 次 Phase 送り (必須)
- **次 Phase (vX.XX) で主に解くべきこと**: 
- **今 Phase ではあえてやらなかったこと**: 
- **Re-plan (計画見直し) が必要な論点**: 
```

---

## 5. Template Usage Rules
1. **項目の削除禁止**: `(必須)` となっている A〜I（Phase は A〜G）の項目は、たとえ「特になし」であっても見出し自体を削除してはいけません。見出しを消す行為は「チェック観点の欠落・引継ぎ漏れ」と同義とみなされ、レビュー差し戻しの対象となります。
2. **自由記述による暴走防止**: 各項目は長文や Narrative (物語風の散文) ではなく、事実ベースの箇条書きを推奨します。
3. **Source of Truth の厳守**: レビューは常に `devmap` または事前の `Implementation Batch` ドキュメント等と照らし合わせ、その場の雰囲気で完了にしないこと。

## 6. B20-05 Done Definition
- [x] Batch Review Template および Phase Review Template が明記されている。
- [x] 必須と任意が分離され、要求項目（背景再確認、Managed Risk、Wording 保留、Suite影響、次バッチ送り等）が漏れなく枠組みに落とし込まれている。
- [x] 可読性 / 記入性 / 管理性の設計原則に従い、Reviewer が迷わない形式になっている。
