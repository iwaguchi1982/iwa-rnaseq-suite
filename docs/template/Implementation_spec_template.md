# 粗い設計 → 実装指示書 変換テンプレ
Version: draft-0.1
Purpose: 現場起点の粗い設計を、AI 実装可能な実装指示書へ渡すための入力テンプレ
batch: B30-1
---

## 作業フロー

1. あなたが大枠を書く
2. Geminiが事前調査メモを書く
3. ChatGPTが実装指示書に落とす
4. Geminiが実装
5. ChatGPTがレビューして done / hold / reject を返す

---

## 0. 今回の一言ゴール

### この作業で最終的に実現したいこと
- 

### 一言で言うと
- 例: reporter が review bundle を formal に export / import できるようにする

---

## 1. 背景と現場課題

### 誰が困っているか
- 

### 何に困っているか
- 

### なぜ困っているか
- 

### 今の運用で何が起きているか
- 

### 放置すると何がまずいか
- 

### 今回それをどう改善したいか
- 

---

## 2. 今回のスコープ

### 今回やること
- 
- 
- 

### 今回やらないこと
- 
- 
- 

### 触ってよい範囲
- 例: `src/.../adapter/` と validator のみ
- 例: review export builder と import validator のみ

### 触ってはいけない領域
- 例: app.py の大規模整理は禁止
- 例: 既存 `session_state` key は変更禁止
- 例: UI デザイン調整は禁止
- 例: v0.19 contract の破壊的変更は禁止

---

## 3. Source of Truth

### 今回の source of truth
- 
- 
- 

### source of truth ではないもの
- 
- 
- 

### 絶対に戻ってはいけない実装
- 例: raw vendor file 直読みに戻る
- 例: UI 層で design 情報を再解釈する
- 例: absolute path を contract に混ぜる
- 例: 一時的な shim を正式仕様にする

---

## 4. 期待する完成状態

### done になったと言える状態
- 
- 
- 

### hold にすべき状態
- 例: 方向性は正しいが validator が弱い
- 例: source of truth は守っているが fixture が不足
- 例: 正常系は通るが異常系の期待値が未固定

### reject にすべき状態
- 例: public contract を壊した
- 例: source of truth を増殖させた
- 例: adapter と本体の責務が混ざった
- 例: 例外を握りつぶして成功扱いにした

---

## 5. 現場制約・業務制約

### 社内フロー上の制約
- 

### 法的・監査・治験・記録上の制約
- 

### 運用上の制約
- 

### 技術上の制約
- 

### 納期・優先度
- 

### 今回最優先の観点
- 例: contract correctness
- 例: portability
- 例: rerun / provenance
- 例: suite handoff を壊さないこと

### 今回は未優先でよいもの
- 例: UI polish
- 例: helper 分割の美しさ
- 例: summary 再計算の高度化

---

## 6. 参考資料・参照コード

### 読むべき文書
- devmap:
- milestone:
- acceptance checklist:
- phase gate:
- implementation batch:

### 参照してほしい既存コード
- 例: `src/.../comparator_review_session.py`
- 例: `src/.../comparator_review_export_builder.py`
- 例: `src/.../adapter_base.py`

### 再利用してほしい既存関数・既存クラス
- 
- 
- 

### 似た実装があれば
- 例: v0.19.4 の handoff builder を参考
- 例: v0.20.1a の duplicate guard と同じ思想で
- 例: existing validator helper を流用

---

## 7. 今回ほしいデータ構造

### 新規 dataclass / model が必要か
- [ ] 必要
- [ ] 不要
- 理由:

### 必要なら、今わかっている範囲で書く
#### Dataclass 名
- 

#### 役割
- 

#### 必須フィールド
- `field_name: type`
- `field_name: type`
- `field_name: type`

#### Optional フィールド
- `field_name: type | None`
- `field_name: type | None`

#### これは入れない
- 
- 

### shape がまだ曖昧なら
- Gemini に既存コード探索で候補 shape を出してほしい
- ChatGPT に実装指示書で最終 shape を固定してほしい

---

## 8. 異常系の期待値

### 不一致や欠損が起きたときの期待挙動
- 例: `comparison_id` 不一致なら `ValueError`
- 例: required artifact 欠落なら invalid context + issues
- 例: schema mismatch は読み込み継続するが `is_valid=False`
- 例: fatal parse error は error context を返す

### fail-fast にしたいもの
- 
- 

### issues に積んで続行してよいもの
- 
- 

### silent drop 絶対禁止のもの
- 
- 

### absolute path / 環境依存情報の扱い
- 例: contract / handoff / export に absolute path 禁止
- 例: bundle-relative ref または stable name のみ許可

---

## 9. UI が絡む場合だけ書く

### UI 変更はあるか
- [ ] ある
- [ ] ない

### UI でやること
- 

### UI でやらないこと
- 

### 既存 session_state で守るべきもの
- 
- 

### 新規 session_state key を増やしてよいか
- [ ] よい
- [ ] 最小限のみ
- [ ] 禁止

---

## 10. テスト観点

### 正常系で最低限見たいこと
- 
- 

### 異常系で最低限見たいこと
- 
- 

### roundtrip / validator / import-export で見たいこと
- 
- 

### フル回帰まで必要か
- [ ] 必須
- [ ] 可能なら
- [ ] 今回は targeted のみ

### done 前に最低限通っていてほしいコマンド
```bash
# targeted