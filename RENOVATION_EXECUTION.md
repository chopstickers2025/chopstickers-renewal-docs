# RENOVATION_EXECUTION.md
Chopstickers Renewal 実行正本 / Execution Single Source of Truth

---

## 1. Document status / authority

- **本ファイルの位置づけ:** Chopstickers Renewal を完成させるための **現在地・統合順序・制御方法** の正本。「今どこまで出来ていて、次に何をどうするか」を定義する。
- **対になる文書:** 事業・システムの「あるべき仕様」は `CHOPSTICKERS_CANONICAL.md`。本ファイルには business rule を書かない（参照のみ）。
- **自動化制御の詳細:** `docs/AUTOMATION_CONTRACT.md`（delivery-core）。本ファイル §7 はその要約。
- **更新日:** 2026-09-11（初版）。
- **authority:**
  - business / system 事実 → `CHOPSTICKERS_CANONICAL.md` が優先。
  - execution / progress / integration 順序 → 本ファイルが優先。
  - **GitHub（Issue / PR / commit / comment）が canonical。** 本ファイルおよび `ops/renovation-status.json` は resume を速くするための便宜スナップショットであり、GitHub と食い違ったら GitHub を正とし、スナップショットを合わせる。
- **supersedes:**
  - `Chopstickers System Renovation Roadmap.md` の「現在の統合作業順序」部分（§0 2026-09-08 Strategy Revision を反映。Phase 0–7 は各コンポーネントの仕様・完了基準としては有効だが、直列の作業順序ではない）。
  - DailyLog 2026-09-08〜09-11 の統合方針・自動化再設計の記述（要約を本ファイルへ集約）。

---

## 2. Final goal

2026-09-08 owner 確定の運用モデル（Blue / Green Renewal 統合・検証）:

1. 現行本番 `chopstickers.jp` を **Blue（基準系）** として固定・稼働継続する。触らない。
2. 現行本番相当をベースに、非公開の **Green（Renewal 統合環境、project `chopstickers-project`）** を構築する。
3. 接続テストを本番から分離する（staging Firebase / staging GAS endpoint / Stripe Test Mode / 本番書き込みなし）。
4. PC 側の最新 WIP と GitHub 上の review 済み成果を安全に Green へ集約する。
5. 完成した部品（Delivery Core / Admin Products / Admin Delivery / Admin General / NG2 / Katakana / Green frontend）を **段階的に** Green へ統合し、各ステップ直後に実動テストする（最後にまとめてではない）。
6. storefront / index / 共通 JS / Gift 選択 / Live Preview を統合する。
7. モバイル / デスクトップで E2E テスト（注文 → 決済 → 管理 → NG1 → 無料再配達 → NG2 → 3rd Delivery）。
8. 不具合修正 → 回帰テスト。
9. 本番差分 diff・cutover 手順・rollback 手順を検証。
10. **production cutover decision** — owner が公開先を完成済み Green へ切り替える判断を下す。本番を現在地で作り替えない。cutover 後も rollback 経路を維持。

---

## 3. Current architecture / branches

### 3.1 リポジトリ
| repo | 役割 |
|---|---|
| `chopstickers2025/chopstickers-hp` | サイト本体（`public/` がリポジトリルート = Firebase web root）。storefront・admin・顧客向けページ・共通 JS |
| `chopstickers2025/chopstickers-delivery-core` | 配送・注文管理コア（GAS / clasp）。モジュール化されたコア + アダプタ + テスト。**doc SSOT も `docs/` に置く** |

### 3.2 hp repo の重要ブランチ
| branch | 最新 | 役割 |
|---|---|---|
| `main` | `82dd837` | 旧本番ベースライン。Renewal はここに載っていない。cutover まで merge しない |
| `renewal/wip-snapshot-20260909` | `aef5afd` + 未コミット | **Blue WIP snapshot**（PC 作業ツリーの凍結。Delivery 刷新 scaffold + Katakana WIP を含む） |
| `renewal/green-20260909` | `2154b8b` | **Green 統合先の基盤**。admin-products 専用ではない。ここへ全レーンを統合していく |
| `feature/katakana-r2-20260910` | `39bea3c` | **Katakana の Green 統合正本**（名称は r2 だが中身は R3 = 辞書 1,875 キー + iOS/キャッシュ修正込み） |
| `deploy/katakana-r3-20260910` | `b7fb1b6` | Blue 緊急 deploy 専用の監査証跡。**Green 統合には使わない** |
| `green/delivery-shape-integration-20260911` | `8bc4777` | PR #25。Delivery shape ページ（ng2 / admin-delivery / admin-general + shape-admin.css + core-api-shape.js）を Green へ。**READY-FOR-REVIEW** |
| `admin-products-green-storage-20260910` | `1eb4912` | admin-products の Green Firebase Storage / RTDB 統合（Auth ゲート + 画像永続化 + ルール案）。PR #21 |
| `claude/issue-22-20260910-1613` | — | automation control-plane contract scaffold（hp 側）。PR #23 |
| `claude/issue-17-20260908-2118` | `e5ca3b3` | Green pre-Firebase quarantine safeguards。PR #18 |

### 3.3 delivery-core repo の重要ブランチ
| branch | 最新 | 役割 |
|---|---|---|
| `main` | `d156ec1` | Delivery Core の本流。Roadmap（§0 なし・旧）/ Delivery System Rules / CLAUDE / 設計書 を `docs/` に保持 |
| `claude/issue-52-20260909-1347` | `4881d6b` | **PR #56**。安全な admin reschedule contract（Issue #52）。third_delivery anchor / adjacent overlap / retry idempotency 修正済み |
| `green-gas-foundation` | `33475bb` | **PR #57**。Green GAS foundation（fail-closed）。482 tests passed |
| `claude/issue-53-20260908-1417` | `c9c7622` | **PR #55**。Roadmap へ 2026-09-08 Strategy Revision（§0）を追記 |
| `claude/issue-58-20260910-1603` | `433f87b` | **PR #59**。automation control-plane scaffold（`docs/AUTOMATION_CONTRACT.md` + `ops/renovation-status.json`） |
| `automation/watchdog-v1-20260911` | `9724756` | **PR #60**。stalled-lane recovery watchdog v1（`.github/workflows/renovation-watchdog.yml`） |

### 3.4 ローカル worktree（参考）
- hp: `chopstickers-workshop/public`（Blue）/ `chopstickers-project/public`（Green base）/ `Documents/katakana-deploy-tree` / `Documents/katakana-r1-baseline`
- delivery-core: `chopstickers-workshop/delivery-core`（PR #56 branch）

### 3.5 監督自動化
- 統合監督は **「Chopstickers Hourly Progress」1本**（旧「Chopstickers Delivery Hourly Progress」は停止）。Green / Delivery / Admin Products / Admin Delivery / NG2 / Katakana 統合をまとめて監督。

---

## 4. Workstream status

> 各レーン: status / source / done / remains / integration target / dependency / next_action。
> status 語彙: `review_ready`（レビュー可・未統合）/ `in_progress` / `blocked` / `integrated`（Green へ取り込み済み）/ `not_started`。
> **GitHub が正。** 下表と GitHub が食い違ったら GitHub を正としてここを更新する。

### 4.1 Delivery Core（配送・注文管理コアロジック）
- **status:** `review_ready`
- **source:** delivery-core `claude/issue-52-20260909-1347` @ `4881d6b` / PR #56（Issue #52）
- **done:** third_delivery reschedule anchor を `secondDeliveryDateTime + 90分` に確定（originalDeliveryDateTime fallback 不採用）。adjacent-footprint overlap 修正。retry idempotency 修正。`DeliveryCore.test.js` 252 passed / 0 failed。
- **remains:** PR #56 のレビュー・main マージ判断（owner）。残る adapter/integration テスト失敗は既存の clock drift（本 PR の範囲外・再実装しない）。
- **integration target:** delivery-core `main` → Green GAS。
- **dependencyःなし（レビュー待ち）。
- **next_action:** PR #56 を review する。**再実装しない。** 追加修正が必要なら既存ブランチから narrow に。

### 4.2 Green GAS（chopstickers-project の GAS 基盤）
- **status:** `review_ready`
- **source:** delivery-core `green-gas-foundation` @ `33475bb` / PR #57
- **done:** fail-closed Green GAS foundation。482 tests passed / 0 failed。Green project の Web App / RTDB / Hosting / Auth、Green 専用 Sheet / Drive / Calendar、Green GAS project を provision 済み。
- **remains:** service account 設定・clasp 設定・Green への deploy・Green Firebase への接続確認（owner boundary の手前まで）。PR #57 レビュー。
- **integration target:** Green GAS project（`clasp` deploy は owner 承認後）。
- **dependency:** Delivery Core（PR #56）の main 反映が前提だと整合が取りやすい。
- **next_action:** PR #57 を review。clasp / deploy / 外部書き込みは owner 承認まで実施しない。

### 4.3 Admin Products
- **status:** `review_ready`（Green Storage 統合まで実装済み・未統合）
- **source:** hp `admin-products-green-storage-20260910` @ `1eb4912` / PR #21（base `admin-products-pc-checkpoint-20260910` / PR #19）。関連: PR #15、closed PR #4（baseline）。
- **done:** admin-products の UI 6th-round リファクタ（PR #19）。Green Firebase module + 管理者 Auth ゲート（`js/admin-products-green.js` / `js/admin-products-green-config.js`）。Green RTDB load/save 層（`/products` `/productSeries` `/heroImages`）。Green Storage への画像アップロード（Daily Use / Gift box / Gift ギャラリー / FV・Hero）。Storage / RTDB ルール案（`public/docs/green-*.rules(.json)` + `green-storage-integration.md`）。
- **remains:** Green Web API key はローカル設定済み（`admin-products-green-config.js`）。ルールは review 済みだが **Firebase Console への適用は未実施**。実 Firebase write / rules apply / deploy は未実施（owner boundary）。`renewal/green-20260909` への統合 PR チェーンの整理・マージ判断。
- **integration target:** `renewal/green-20260909`。
- **dependency:** Green Firebase project（provision 済み）。
- **next_action:** PR #19 → #21 のチェーンを `renewal/green-20260909` へ統合する順序を確定。**Green 版 admin-products ファイルを Blue 版で上書きしない**（Green が新しい）。rules apply / write は owner 承認まで待つ。

### 4.4 Admin Delivery
- **status:** `review_ready`（Green shape 統合 PR 提出済み・未マージ）
- **source:** UI = hp `renewal/wip-snapshot-20260909` @ `aef5afd` の `admin-delivery.html`（+ `shape-admin.css` + `js/core-api-shape.js`）。過去の UI 検討: `origin/admin-delivery-ui-*` 群、PR #12。Green 統合 = `green/delivery-shape-integration-20260911` @ `8bc4777` / **PR #25**。
- **done:** Delivery operations queue UI（Delivery List / Today・Tomorrow・Calendar / order detail / モバイル操作 UI / 配達アクション）。Core API クライアント（`core-api-shape.js`、endpoint は in-browser 設定・埋め込みなし）。PR #25 で ng2 / admin-delivery / admin-general を Green へ file-scoped 統合（前タスク完了、READY-FOR-REVIEW）。
- **remains:** PR #25 レビュー・`renewal/green-20260909` へマージ判断。Core API endpoint（Green GAS / Delivery Core）の配線（Phase 1-G 相当）。Delivery List を実 RTDB / GAS に接続して動作確認。
- **integration target:** `renewal/green-20260909`。
- **dependency:** Green GAS（4.2）/ Delivery Core（4.1）。
- **next_action:** PR #25 を review。マージ後に Core API endpoint を Green 向けに設定して接続テスト。

### 4.5 Admin General
- **status:** `review_ready`
- **source:** UI = hp `aef5afd` の `admin-general.html`（過去: `origin/frontend-shape-20260905` @ `003f6f8` "Admin General: finalize role split from legacy admin"、delivery-core PR #29 "admin-general repository-side contract"）。Green 統合 = PR #25（`admin-general.html` を含む）。
- **done:** admin.html から Global Status / 営業時間 / deliverySlots / 60日カレンダー / Ticker / 季節演出 / production capacity / Save All を分離。PR #25 で Green へ統合し、inline Firebase config を Blue（`chopstickers-workshop`）→ Green（`chopstickers-project`）へ repoint（Green-safe 化）。現行の Green RTDB ルール案では `/status` 未許可のため fail-closed。
- **remains:** PR #25 レビュー。admin-general を Green で実際に機能させるなら、Green RTDB ルールへ `status` ノードを追加するか owner が判断（Roadmap Phase 6 相当・別 PR）。Extra Slots 削除・在庫を Products へ・capacity を General へ の最終確認。
- **integration target:** `renewal/green-20260909`。
- **dependency:** Green Firebase project。
- **next_action:** PR #25 を review。Green `/status` ルール拡張の要否を owner に確認（真の business/boundary 判断のみ `OWNER_DECISION_REQUIRED`）。

### 4.6 NG2 flow（`ng2.html`）
- **status:** `review_ready`（Green へ file-scoped 統合済み・未マージ）
- **source:** hp `aef5afd` の `ng2.html`（+ `shape-admin.css` + `js/core-api-shape.js`）。Green 統合 = PR #25。バックエンド = delivery-core `DeliverySideEffects.js` / `PaymentAdapter.js`（NG2 の third_delivery / shipping バリデーション）。
- **done:** NG2 解決ページ UI（3rd Delivery / Domestic Shipping / Disposal のラジオ選択、選択項目のみ展開、Disposal 二重確認、デモモード）。`DeliverySideEffects.js` が NG2 時に `https://chopstickers.jp/ng2.html?id=<orderId>` を送信。PR #25 で Green へ統合。
- **2026/09/13 確定・PR #31（`feature/ng2-3rd-delivery-domestic-shipping-20260913`・Green レビュー可・未マージ）：** NG2 / 3rd Delivery / Domestic Shipping のルールと法務文言を正式サービス条件として確定・全面同期。
  - `ng2.html`: 3rd Delivery 選択可能時刻を `secondDeliveryDateTime+90分` のみでフィルタ（fallback 禁止、Demo Mode で確認可）。住所再利用を「Use the address from my first delivery」に変更し初回配達先を実データ表示。3rd Delivery 不在時の同意チェックボックス・Domestic Shipping の Japan-only/tracking-email/carrier責任分界の同意チェックボックスを追加（未チェックの間 ¥1,000 CTA 無効）。
  - `terms.html` / `commerce.html`（Green）へ同内容の法務文言を反映。`index.html` の Important Notice / FAQ / 注文直前 Important Rules へも簡潔な要約を追加。`js/translations.json` の fr/de/es/it へ同内容を翻訳・反映済み（English fallback 漏れなしを確認）。
  - **Blue 側（法務文言のみ）：** PR #32（`legal/3rd-delivery-domestic-shipping-terms-20260913`、`renewal/wip-snapshot-20260909` 起点）で `terms.html` / `commerce.html` / `index.html`（Important Notice・FAQ・Important Rules）+ 同 fr/de/es/it 翻訳を同期。ロジック・Firebase・Stripe・GAS・Admin・Katakana は無変更。Blue 側は「Email/WhatsApp 手配」の既存メカニズムを維持した文言（ng2.html 相当の自動 Web セルフサービスは Blue には存在しない）。
- **remains:** PR #25 / PR #31 レビュー。Core API endpoint 配線。Stripe（Test Mode）連携の接続テスト。現状 Blue 本番で `ng2.html` は 404（未 live）。
- **integration target:** `renewal/green-20260909`。
- **dependency:** Delivery Core（4.1）/ Green GAS（4.2）/ Payment（Stripe Test）。
- **next_action:** PR #25 / PR #31 を review。マージ後に endpoint 設定 → NG2 3択の E2E（Test Mode）。

### 4.7 Katakana
- **status:** `review_ready`（Blue 本番へは緊急 deploy 済み。Green へは未統合）
- **source（Green 統合正本）:** hp `feature/katakana-r2-20260910` @ `39bea3cf83a68d524c10958981330a0f67f7e701`（名称 r2 / 中身 R3）。
- **監査証跡（Green で使わない）:** `deploy/katakana-r3-20260910` @ `b7fb1b6`（Blue 緊急 deploy 専用）。R1 = `hotfix/katakana-r1-20260909` @ `1bafd7f`。ハンドオフ: `public/.github/katakana/RELEASE1.md` + `dictionary-audit.json`。
- **done:** 辞書 458 → **1,875 キー**（公式人名頻度 + D級AI推定読み + ChatGPT クロスレビュー + 目視）。新エンジン `js/katakana-engine.js` / ウィジェット `js/katakana-widget.js` / `css/katakana-widget.css`（EN/FR/DE/IT/ES 発音切替、10文字上限、stale-request 拒否）。`js/ui.js` は既存実装を残しブリッジ約20行追加。5 HTML エントリポイントに engine→widget→ui の defer script + `?v=katakana-r3-20260910b` cache-buster。2026-09-10 に Blue 本番へ `hosting:clone` で昇格（本番 RTDB / GAS / Stripe / Hosting 設定は不変）。
- **remains:** `feature/katakana-r2-20260910` を `renewal/green-20260909` へ統合。**Green は order 入力 / モーダル markup が異なる可能性があるため、エンジン・辞書は unit で取り込み、ui.js / HTML のブリッジ hunk は Green の現行実装へ手作業で適用する**（Green の完全ファイルを上書きしない。RELEASE1.md「Targeted Green integration」参照）。R2 本格版（source-backed 20,000–30,000 readings、発音レビュー、対象言語拡大）は将来。
- **integration target:** `renewal/green-20260909`。
- **dependency:** なし（storefront レーンと調整）。
- **next_action:** `39bea3c` の engine/dict を Green へ取り込み、ブリッジ差分を Green の ui.js/HTML へ適用。統合後にテスト再実行し実 SHA を記録。**別パイプライン成果を再実装しない。**

### 4.8 Green frontend（storefront / index / 共通 JS）
- **status:** `not_started`（統合工程）
- **source:** hp `renewal/green-20260909` の現行 storefront + Blue `renewal/wip-snapshot` の WIP（配達文言・Katakana）。
- **done:** Green base に旧本番相当の storefront が存在。
- **remains:** 各レーン（Products / Delivery / General / NG2 / Katakana）の WIP を集約・整合（reconciliation）した後、Renewal 統合レーンで storefront / index / 共通 JS / Gift 選択 / Live Preview を接続。配達ポリシー文言グループ（`redelivery.html` / `commerce.html` / `terms.html` / `success.html` / `tokyo-souvenir.html` / `admin.html` / `js/translations.json`）を `CHOPSTICKERS_CANONICAL.md` §4 と整合させて統合（Blue → Green 方向、`Delivery System Rules.md` と突合）。
- **integration target:** `renewal/green-20260909`。
- **dependency:** 他レーンの WIP 集約完了。共有ファイルは各レーン reconciliation 後に Renewal 統合レーンでのみ触る。
- **next_action:** 他レーンの Green 統合 PR（#25 / #21 系 / Katakana）がマージされた後に着手。単独で先行しない。

### 4.9 Delivery customer-facing pages（`redelivery.html` 再構築）
- **status:** `in_progress`（設計確定・実装は Renewal 内）
- **source:** `Delivery System Rules.md` + `再配達システム改修_設計書.md`。現行 `redelivery.html` は Green base = 旧「Redelivery Booking」版、Blue snapshot = 別バージョン（要 reconciliation）。
- **done:** 新モデル（status / deliveryStage、共通 deliverySlots、+90分バッファ）の設計確定。
- **remains:** `redelivery.html` を新 order モデルへ再構築し Green へ統合。`ng2.html` との一貫性確保。保存モデルを新 order モデルへ寄せる。
- **integration target:** `renewal/green-20260909`。
- **dependency:** Delivery Core（4.1）/ Green GAS（4.2）。
- **next_action:** Delivery Core / Green GAS 統合後、`redelivery.html` の Blue/Green 版を突合し新モデルへ再構築。

### 4.10 Automation / watchdog
- **status:** `review_ready`
- **source:** delivery-core PR #59（`claude/issue-58-…` @ `433f87b`、control-plane scaffold）+ PR #60（`automation/watchdog-v1-…` @ `9724756`、watchdog v1）。hp PR #23（`claude/issue-22-20260910-1613`、control-plane contract scaffold）+ PR #24（Green watchdog）。
- **done:** `docs/AUTOMATION_CONTRACT.md`（terminal classes / next_action / stale detection / strategy switching / OWNER_WAIT）。`ops/renovation-status.json`（レーン snapshot スキーマ）。`.github/workflows/renovation-watchdog.yml`（stalled-lane recovery）。統合監督「Chopstickers Hourly Progress」1本化。
- **remains:** PR #59 → #60、PR #23 → #24 のレビュー・マージ。2026-09-11 owner 指示の「仕組みによる停止検知 + strategy switch」フロー（§7）の実運用への反映。
- **integration target:** delivery-core `main` / hp `renewal/green-20260909`。
- **next_action:** PR #59/#60/#23/#24 を review。watchdog を有効化して stalled-lane 検知を機械化。

### 4.11 E2E
- **status:** `not_started`
- **remains:** Green 環境（staging Firebase / GAS / Stripe Test）で、モバイル/デスクトップ両方で 注文 → 決済 → admin → NG1 → 無料再配達 → NG2 → 3rd Delivery / Domestic Shipping / Disposal を通し。日跨ぎ / 08:00 / ±30分 / Normal vs Redelivery 競合 / 多言語 も。
- **dependency:** 4.1〜4.9 の Green 統合完了。
- **next_action:** 全レーン統合後に E2E スイートを Green で実行。pre-existing failure を separation。

### 4.12 Production cutover
- **status:** `not_started`（owner decision）
- **remains:** 本番差分 diff / cutover 手順 / rollback 手順の最終検証 → owner が公開先を Green へ切り替え → 監視 → 必要時 Blue へ rollback。
- **dependency:** E2E（4.11）green。
- **next_actionःなし（E2E 完了まで）。**merge to main / deploy / production Firebase・GAS・Stripe・Hosting・DNS は owner 承認まで実施しない。**

---

## 5. Confirmed integration order

> 2026-09-08 Strategy Revision（Roadmap §0）に基づく。Roadmap Phase 0–7 は各コンポーネントの仕様・完了基準としては有効だが「順番に完成させてから統合」ではない。以下が現在の統合順序。

1. **Delivery Core PR #56 の completion / review**（4.1）— third_delivery 契約を確定させる。
2. **Admin Products Green Storage**（4.3）— PR #19 → #21 を `renewal/green-20260909` へ。rules apply / write は owner 承認まで保留。
3. **Green GAS verification**（4.2）— PR #57 review。Green GAS project との接続確認（deploy は owner 承認）。
4. **Green frontend connection**（4.8 の一部）— storefront を Green base で健全化。共有ファイルは reconciliation 後。
5. **Admin Delivery integration**（4.4 / PR #25）— ng2 / admin-delivery / admin-general を `renewal/green-20260909` へ。Core API endpoint を Green 向けに配線。
6. **Katakana / NG2 / Admin feature integration**（4.6 / 4.7 / 残 admin）— Katakana は engine/dict を unit 取り込み + ブリッジ手適用。NG2 は endpoint 配線後に Test Mode で確認。
7. **E2E**（4.11）— Green で全フロー通し。
8. **Production cutover decision**（4.12）— owner。

各ステップの直後に実動テスト（統合完了後にまとめてではない）。順序は実情に合わせて最小修正してよいが、全面再設計しない。

---

## 6. Green targeted integration rules

- **Blue（`public/`）を丸ごと Green へコピーしない。** whole-tree の pull / reset / checkout 禁止。
- **file-level / commit-level の targeted integration** のみ。混成スナップショット commit（例 hp `aef5afd`）の cherry-pick は禁止 — 必要なパスだけ取り出す。
- **Green が新しいファイルを Blue 版で上書きしない**（特に admin-products。Green 版が先行）。
- **dependency set 単位で統合**（相互依存で閉じた塊をまとめて）。
- **現在判明している Delivery の欠落セット（Blue にあり Green に無い、統合必須）:**
  - `ng2.html`
  - `admin-delivery.html`
  - `admin-general.html`
  - `shape-admin.css`
  - `js/core-api-shape.js`
  - → PR #25（`green/delivery-shape-integration-20260911`）で file-scoped 統合済み・**READY-FOR-REVIEW**。
- **`admin-general.html` は Blue Firebase 接続（`chopstickers-workshop` / `/status` read-update）を Green-safe 化してから統合する。** PR #25 では inline `firebaseConfig` を `chopstickers-project` へ repoint 済み（値は `js/admin-products-green-config.js` の `GREEN_FIREBASE_CONFIG` と一致）。現行 Green RTDB ルール案では `/status` 未許可 → fail-closed。
- storefront / index / 共通 JS など複数レーンにまたがる共有ファイルは、各レーンの WIP を reconciliation してから Renewal 統合レーンでのみ扱う。
- lane ownership を維持する（Admin Products / Admin General / Admin Delivery / Delivery Core は互いに上書きしない）。
- 統合作業は dedicated branch + PR まで（可逆 repo-only 作業）は owner 確認待ちで止めない。merge は owner。

---

## 7. Automation control plane

> 正本: delivery-core `docs/AUTOMATION_CONTRACT.md`（PR #59）。以下は要約 + 2026-09-11 owner 指示の反映。

### 7.1 原則
- **GitHub が canonical。** `ops/renovation-status.json` は resume 用スナップショット（非 authoritative）。食い違ったら GitHub を正としスナップショットを合わせる。
- **1 run = 1 next_action。** run 終了時、触れたレーンに対し「次の run（自動 or 人間）が何をすべきか」を1つの具体的な命令として記録する（ナラティブ要約ではない）。
- **task result と GitHub workflow の conclusion は別物。** workflow failure ＝ 実装失敗ではない。

### 7.2 Terminal classes（`failureClass`）
`SUCCESS_PROGRESS` / `FAILED_BUT_PROGRESS` / `FAILED_NO_PROGRESS` / `MAX_TURNS` / `ENVIRONMENT_ERROR` / `TEST_FAILURE` / `OWNER_WAIT` / `DUPLICATE_RETRY`

### 7.3 クラス別ルール
- **MAX_TURNS / transient tooling failure:** 次の run は persisted findings（branch commits / PR description / `lastEvidence` / `task`）から **narrow に resume**。問題をゼロから再発見しない。
- **同一原因で2回連続失敗（同レーン・同 failureClass）:** 同一の generic task を再実行しない。scope を縮小して targeted repair、または具体的な質問を添えて `OWNER_WAIT` へエスカレート。
- **TEST_FAILURE:** 次の run は特定の失敗テストとその根本原因を狙う（全体の generic retry ではない）。pre-existing failure と新規 failure を切り分ける。
- **dependency wait（別 PR / lane / CI / service の解決待ち）:** `next_action` を「その依存を監視 / ポーリング」に設定。これは正常な状態であり、`OWNER_WAIT` ではない。
- **OWNER_WAIT:** 意図的に停止して待てる唯一のクラス。必要な決定を、owner が直接答えられる形（可能なら選択肢付き）で述べる。

### 7.4 2026-09-11 owner 指示: strategy switch フロー
```
作業開始 → 進捗あり？
  ├─ Yes → 次工程
  └─ No → 原因分類
       → 1回目: scope を縮小して再試行
       → 同一原因で2回失敗？
            ├─ No → 継続
            └─ Yes → 実行手段を変更（mandatory strategy switch）:
                 ├─ GitHub Claude Action → ローカル / persistent Claude Code
                 ├─ Claude → Codex
                 ├─ AI 実装 → ChatGPT 直接 GitHub 修正
                 ├─ 大きな task → 小さな task に分割
                 └─ 実装と検証を別 run 化
```
- 停止時間をなくすため、stall 検知は AI 監視ではなく**仕組み（watchdog workflow）で機械検知**する。
- alternate route を使っても **GitHub が canonical**（成果は必ず GitHub に載せる）。

### 7.5 Checkpoint
- turn budget が尽きる前に、coherent で reversible な repo-only 作業の checkpoint commit / push を優先する（黙って失うより良い）。
- テスト / status が支持しない限り `SUCCESS_PROGRESS`（完了）を報告しない。未完だが commit 済みは `FAILED_BUT_PROGRESS` / `MAX_TURNS` として、`next_action` に resume 地点を明記。

### 7.6 Reporting channels
- Email / Slack 等の通知は **reporting-only**。lane 進行や依存解決を gate / start / stop しない。owner email は安心・可視化の面として維持してよい。

### 7.7 OWNER_WAIT の条件
- 実質的な business rule の曖昧さ、または hard production boundary（§9）に到達した場合のみ。
- それ以外の reversible な repo-only 作業（コード編集・docs・テスト・branch commit/push・PR 作成（merge なし））は owner 承認を待たず継続する。

---

## 8. Testing / review strategy

- **static:** 変更ファイルの diff 確認、対象パスのみが変わっていることの検証、参照整合（`shape-admin.css` / `core-api-shape.js` 等）、config が Blue を向いていないことの確認。
- **unit:** delivery-core の Node スイート（`node --test`）。関連スイートを run（全体 generic retry ではなく対象を狙う）。
- **integration:** アダプタ / フロー結合テスト（`IntegrationFlow.test.js` 等）。
- **E2E:** Green 環境（staging Firebase / GAS / Stripe Test）でモバイル/デスクトップ両方。注文 → 決済 → admin → NG1 → 無料再配達 → NG2 → 3rd Delivery / Shipping / Disposal。日跨ぎ / 08:00 / ±30分 / 競合 / 多言語。
- **pre-existing failures separation:** 既存の clock drift 等による失敗は新規リグレッションと明確に切り分ける。「remaining adapter/integration failures are pre-existing clock drift」（PR #56）のように記録する。
- **Green-safe testing:** 本番 Firebase / GAS / Stripe へ接続しない。production endpoint への POST 禁止。実 Firebase write / rules apply 禁止。Green Firebase の RTDB ルールは現状のまま（変更は owner）。

---

## 9. Production boundaries

以下は **owner の明示的承認なしに実施しない**（到達したら `OWNER_WAIT` で停止し、必要な承認を具体的に述べる）:

- `main` へのマージ / auto-merge / PR merge
- production deploy（Firebase Hosting deploy / `firebase deploy`）
- `clasp push` / `clasp deploy`
- 本番 Firebase への書き込み / Firebase Security Rules の適用・変更 / 本番データ削除
- Stripe production アクション
- 本番 GAS / Script Properties の変更
- Hosting deploy / DNS / ドメイン変更
- その他 irreversible な production action

**承認ポイント:**
- 各 Green 統合 PR のマージ = owner レビュー後。
- rules apply（Green Storage / RTDB）= owner。
- Green GAS deploy（clasp）= owner。
- production cutover = owner（E2E green が前提）。

reversible な repo-only 作業（コード / docs / テスト / branch / commit / push / PR 作成）は承認を待たず継続してよい。

---

## 10. Cleanup / retirement plan

> **今は何も削除しない。** 以下は cutover 後 / 各前提充足後の計画。

### 10.1 old docs
- `CANONICAL_MIGRATION_AUDIT.md` の coverage / contradiction 判定に基づき、Phase D で DELETE / ARCHIVE / KEEP を確定してから実施。
- 少なくとも新2正本（`CHOPSTICKERS_CANONICAL.md` / `RENOVATION_EXECUTION.md`）+ 短縮版 `CLAUDE.md` が coherent で、unique 情報の吸収が確認できるまで旧資料を削除しない。

### 10.2 old branches
- Katakana: `deploy/katakana-r3-20260910` は監査証跡として残す。統合後に整理判断。
- `claude/issue-3-*` 群（admin-products 実装履歴）、`admin-delivery-ui-*` 群は Green 統合完了後に整理候補。
- cutover まで `main` は触らない。

### 10.3 old files（コード）
- 旧フィールド（`timeSlot` / `type` / `redeliveryDate` / `redeliveryKey` / `sleep_in` / `Today_*` / `Tomorrow_*`）と旧 `redeliverySlots` / 旧120分ルール / same-night 限定処理は、新システム移行完了後（Roadmap Phase 7 相当）に段階削除。
- 旧 draft GAS（`GAS_redelivery_revised.gs` / `GAS_redelivery_TEST.gs` / `GASのコード.md`）は delivery-core repo が置換した範囲を確認してから retire。
- 未追跡素材・将来用ファイル（`img2/` 等）を勝手に削除しない。

### 10.4 Blue retirement
- cutover 完了 → 監視期間 → rollback 不要が確認できてから Blue 環境を retire。cutover 直後は rollback 経路を維持する。

### 10.5 premature deletion 禁止
- coverage 確認前の old docs 削除、Green 統合完了前の Blue 資産削除、E2E 前の legacy コード削除は行わない。

---

## 11. Immediate next actions

現時点の具体的な順序（曖昧なバックログを並べない）:

1. **本 PR（docs 正本再構成）を review** — `CHOPSTICKERS_CANONICAL.md` / `RENOVATION_EXECUTION.md` / `CANONICAL_MIGRATION_AUDIT.md`。coverage / contradiction 表を確認。
2. **delivery-core PR #56 を review**（Delivery Core / Issue #52）。再実装しない。
3. **delivery-core PR #55 を review**（Roadmap §0 Strategy Revision）→ main へ反映されれば Roadmap と本ファイルの整合が明確になる。
4. **delivery-core PR #57 を review**（Green GAS foundation）。
5. **delivery-core PR #59 → #60、hp PR #23 → #24 を review**（automation control-plane / watchdog）→ watchdog を有効化。
6. **hp PR #25 を review**（Delivery shape → `renewal/green-20260909`）。マージ後に Core API endpoint を Green 向けに配線。
7. **hp PR #19 → #21 を review**（admin-products Green Storage）。統合順序を確定。rules apply / write は owner 承認まで保留。
8. **Katakana**: `feature/katakana-r2-20260910` @ `39bea3c` の engine/dict を `renewal/green-20260909` へ unit 取り込み、ブリッジ差分を Green の `ui.js` / HTML へ手適用。
9. 上記マージ後に **Green frontend / storefront reconciliation** → **E2E**（Green・staging）。
10. E2E green 後に **production cutover decision**（owner）。

**禁止（再掲）:** main merge / PR merge / production deploy / clasp push・deploy / 本番 Firebase write / rules apply / Stripe production / Hosting deploy / DNS 変更 / PR #56 の再実装 / Katakana 成果の再実装 / Blue public 丸ごとコピー / Green admin-products の Blue 版上書き / coverage 確認前の old docs 削除。

---

*本ファイルは改修の「現在地と進め方」を定義する。事業・システムの仕様は `CHOPSTICKERS_CANONICAL.md` を参照。自動化制御の詳細は `docs/AUTOMATION_CONTRACT.md`。*
