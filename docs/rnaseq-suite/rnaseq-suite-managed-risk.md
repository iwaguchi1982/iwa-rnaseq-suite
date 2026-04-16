## 現状の境界と既知リスク (B20-01 調査結果)

### 1. 物理的な結合 (Source-level Coupling)
- **Reporter -> Counter**: `iwa_rnaseq_reporter/src/iwa_rnaseq_reporter/io/bundle_loader.py` において、 `iwa_rnaseq_counter.io.read_analysis_bundle` を直接インポートしている。
- **リスク**: パッケージレベルでの分離ができていないため、Counter の内部変更が Reporter を破壊する恐れがある。
- **対応**: v0.30+ で `shared-spec` パッケージを抽出し、共通契約・データ構造をそこへ移管することで解消する。

### 2. テスト環境の結合 (Test Fixture Coupling)
- **Reporter Tests**: `iwa_rnaseq_reporter/tests/integration/test_counter_reporter_handoff.py` が、物理パスで `iwa_rnaseq_counter/tests/fixtures` を参照している。
- **リスク**: Suite 全体としてリポジトリを跨いだテスト実行が必要となり、疎結合な開発を阻害する。
- **対応**: 代表的な Handoff artifact を独立した `suite-test-fixtures` として定義・配置する。

### 3. 未開発の Ingestion 層 (Missing Adapter Layer)
- **現状**: 現在、外部納品物の読み込みは Reporter 本体の `bundle_loader` や `import_builder` に実装されつつある。
- **リスク**: 特定のベンダー差異が Reporter 本体に蓄積し、コードが汚染される。
- **対応**: v0.50+ で Adapter 層を論理的・物理的に分離し、Reporter は「Suite 標準契約」のみを信じるように徹底する。

### 4. B20-01 の結論
- 現状、ディレクトリレベルでは分離されているが、**「package-level の疎結合」は未達** である。
- この課題を B20-01 の成果として記録し、解消を Phase 3/4 (v0.30-v0.40) の重要課題として送る。

### 5. Counterのbackend kallistoの未実装
- kallisto は suite 想定 backend だが、現時点では未実装 / 未registry 接続

### 6. metadata への `feature_type` / `normalization` 追加
- `metadata` に `feature_type` と `normalization` も追加されている。
- これは B40-04 の必須要件ではなかったが、DEG core の責務を壊す変更ではないため許容とする。
- 将来の補強候補:
  - `thresholds` がまだ入っていないことを明示的に守るテスト
  - `figure ref` がまだ入っていないことを明示的に守るテスト
- ただし、これらは **B40-04 時点では非ブロッカー** とする。

### 7. B40-5 test_result_core.py の補強案
- volcano_section.source_refs == [result_spec.result_id] の完全一致まで見る
- heatmap / enrichment section がまだ存在しないことを守るテストを足す