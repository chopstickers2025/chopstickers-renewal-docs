# CANONICAL_MIGRATION_AUDIT.md
正本再構成 監査レポート（一時ファイル / Phase D で削除候補）

- 作成日: 2026-09-11
- 目的: `CHOPSTICKERS_CANONICAL.md` / `RENOVATION_EXECUTION.md` の2正本が、既存資料の情報を漏れ・矛盾なく吸収しているかを検証する。
- **本ファイルは最終正本ではない。** Phase D（旧資料の archive / delete 実施）完了後に削除してよい。
- 削除判定の原則: 「古そうだから」で決めない。unique information が新正本へ吸収済みかを確認する。

---

## 1. Coverage matrix

吸収先: **C** = `CHOPSTICKERS_CANONICAL.md`、**E** = `RENOVATION_EXECUTION.md`、**AC** = `AUTOMATION_CONTRACT.md`（既存・維持）。

| 旧ファイル | 役割 | 新正本への吸収先 | 吸収率 | 残存 unique 情報 | 矛盾 | 最終候補 |
|---|---|---|---|---|---|---|
| `事業概要_最新版_20260908`（Drive） | 事業仕様の最新版 | C §2–§11（事業・価格・配送・admin・データ）／ E（改修進捗） | partially absorbed（約75%） | 機材（K8 接続手順・刻印設定値・素材別テンポ/出力）、消耗品・在庫リスト、固定費・収支、広告・集客、同梱物・引き渡しスクリプト、変更履歴（§15） | あり（§2 参照。Katakana 件数 / 改修戦略 / GAS 構成 / Green / admin-products 進捗） | **KEEP-ACTIVE**（運用・機材・収支のSSOT）＋ Drive に修正版を別途作成（後述） |
| `事業概要_Chopstickers_v2.md`（Drive / delivery-core docs） | 旧事業概要（2026-09-01改定） | C（要点は最新版経由で吸収済み） | fully absorbed（最新版 20260908 に置換済み） | なし（最新版が上位互換） | あり（旧配達時間表現・旧価格の途中状態） | **ARCHIVE**（比較用に Drive 保持、delivery-core docs からは削除候補） |
| `事業概要.md` ×2（Drive・2025-12 / 2026-01） | 初期事業概要 | — | obsolete | なし（歴史的） | あり（Workshop 前提・旧コンセプト全般） | **ARCHIVE** |
| `仕様サマリー_Chopstickers_20260903` / `Chopstickers_Specifications_Summary`（Drive） | 2026-09-03 時点の仕様サマリー | C §2–§8 | fully absorbed（C が上位互換・より新しい） | なし | あり（2026-09-03 時点のため Green / Delivery Core / Katakana R3 未反映） | **ARCHIVE** |
| `Chopstickers System Renovation Roadmap.md`（main版・§0 なし） | 改修ロードマップ | E §2, §5（統合順序）／ Phase 0–7 は各コンポーネント仕様として維持 | partially absorbed | Phase 2–7 の各コンポーネント完了条件の逐条（/products フィールド、Select Material 動的化、可変 Series Tab 等） | あり（§0 なし版は「直列順序」を示唆 → Strategy Revision で否定） | **KEEP-ACTIVE**（PR #55 で §0 追記版へ更新。Phase 詳細は残す。「現在の統合順序」は E が正） |
| `Chopstickers System Renovation Roadmap.md`（PR #55 / §0 追記版） | 改修戦略の最上位（2026-09-08） | E §2（final goal）に §0.2 の12ステップを反映 | partially absorbed（戦略は E、Phase 詳細は Roadmap） | Phase 0–7 詳細、Phase 1-A〜1-H の細目 | なし（E と整合） | **KEEP-ACTIVE**（PR #55 マージ推奨。E が「現在の順序」、Roadmap が「コンポーネント仕様」） |
| `Delivery System Rules.md`（2026-09-03・逐条） | 配送業務ルールの正本（逐条） | C §4（要点）＋ C §3（journey）＋ C §10（invariants） | partially absorbed（要点は C、逐条の例・文言テンプレは Rules） | §16 Domestic Shipping 文言テンプレ、§24–25 分析データ項目の全リスト、§25 Web 表示文言の3段階、状態遷移図（§27）の詳細 | 軽微（§11/§15 の originalDeliveryDateTime 表現 → owner decision で 3rd は secondDeliveryDateTime に確定。C §4.6 で明示） | **KEEP-ACTIVE**（配送の逐条 SSOT。C §4.6 の owner decision を追記する軽微修正を推奨） |
| `管理機能改修 仕様書＆工程表.md`（2026-09-03） | admin 3分離・商品動的化の設計 | C §6（責務境界）／ E §4.3–§4.5 | partially absorbed | /orders 基本方針（§4）、/products マスター案の全フィールド定義（§5）、Series 設定（§6）、/shopStatus 運行設定（§7）、商品UI連動（§9）、責務分離の詳細（§12） | あり（admin-products「雛形のみ」→ 実際は Green Storage 実装済み。E §4.3 で最新化） | **KEEP-ACTIVE**（admin データ設計の SSOT。進捗記述は E が正） |
| `admin-products.html_仕様書.md`（2026-09-03・§1–50） | 商品 CMS 詳細仕様 | C §6.3, §6.5（要約） | partially absorbed（要約のみ） | Series Manager / 商品カード入力項目 / 刻印位置調整 GUI / Live Preview 反映 / FV・Hero Manager / Spreadsheet 同期方向 / 入力チェック / Firebase Storage パス / 多言語商品テキスト（§39–50） の全詳細 | なし（C は要約なので矛盾なし。実装は Green で先行） | **KEEP-ACTIVE**（admin-products 実装の詳細 SSOT） |
| `再配達システム改修_設計書.md`（2026-09-03） | Phase 1 実装設計 | C §3, §4／ E §4.1, §4.4, §4.9 | partially absorbed | `reserveDeliverySlot(...)` の責務10項目（§17）、GAS_redelivery_revised.gs の関数分類基準（§5）、テスト方針の Slot/Flow/Side-effect リスト（§22）、実装順序 1-A〜1-H（§23） | 軽微（§9 Free Redelivery / §11 3rd は originalDeliveryDateTime 基準の余地を残す表現 → C §4.6 で 3rd は secondDeliveryDateTime・fallback 禁止・fail closed に確定。§10「仮名 delivery-options.html」→ 実ファイル `ng2.html`） | **KEEP-ACTIVE**（Phase 1 実装設計の SSOT。§9–11 と §10 の軽微修正を推奨） |
| `GASのコード.md`（2026-09-03） | 本番 GAS コードのスナップショット/注釈 | C §8.3（概要） | partially absorbed | 本番 GAS の実コード断片・fetchFirebase ヘルパー・doPost ルーティングの注釈 | 要確認（delivery-core repo の `コード.js` / モジュール群がどこまで置換したか） | **KEEP-AS-REFERENCE**（delivery-core repo との対応関係を確認するまで保持。置換確認後 ARCHIVE 候補） |
| `GAS_redelivery_revised.gs`（2026-09-03・draft） | 再配達改修 draft GAS | — | obsolete（delivery-core repo が置換） | LockService / 日跨ぎ計算 / ±30分占有 / multi-location PATCH の実装（delivery-core に取り込み済みか要確認） | あり（redeliveryDate/redeliveryKey を正式モデル扱い等の旧設計） | **KEEP-AS-REFERENCE**（delivery-core への取り込み完了を確認するまで。確認後 DELETE 候補） |
| `GAS_redelivery_TEST.gs`（2026-09-03・draft） | 再配達 TEST 版 GAS | — | obsolete | なし（TEST 版・本番正本ではない） | — | **KEEP-AS-REFERENCE**（Phase 1-A 監査完了まで。確認後 DELETE 候補） |
| `再配達_実環境TEST_セットアップ手順.md`（2026-09-03） | 実環境 TEST のユーザー側手順 | E §8（testing strategy） | partially absorbed | TEST 用 Apps Script プロジェクト作成・シークレット設定の手順、安全装置の一覧 | あり（delivery-core repo のテスト基盤が別途整備済み・482 tests 等） | **ARCHIVE**（delivery-core のテスト基盤が置換） |
| `再配達_実環境TEST手順書.md`（2026-09-03・方法A） | 実環境 TEST 手順書 T1〜T8 | E §8 | partially absorbed | RTDB を使いつつ実業務データへ影響を与えない検証手順の詳細 | あり（同上） | **ARCHIVE** |
| `Chopstickers-workshop プロジェクト調査報告.md`（2026-09-02） | 初回フォルダ構成調査 | C §7, §8（一部） | mostly absorbed | なし（当時のスナップショット） | あり（入れ子 Git・buckup/ 等、現在は解消済みの構成） | **ARCHIVE** |
| `CLAUDE.md`（2026-09-02） | 開発安全ルール | 短縮版 CLAUDE.md（§4 提案）＋ C §9, §10 ＋ E §7, §9 | partially absorbed | §1–§11 の逐条（Phase 単位実装 / 調査フェーズ / 秘密情報 / Git / 判断に迷った場合） | なし | **KEEP-ACTIVE**（§4 の短縮版へ置換提案。今回は変更しない） |
| `事業概要 セクション15（変更履歴）` | 事業・システムの変遷履歴 | — | not absorbed（意図的） | 全変遷履歴（コンセプト・価格・商品・インフラ・廃止サービス） | — | **KEEP-ACTIVE**（Drive 事業概要内。歴史は正本に入れず原本で保持） |
| 月次 `YYYY-MM_DailyLog`（Drive） | 日次意思決定ログ | C / E（意思決定の結論のみ） | partially absorbed（結論のみ） | 意思決定の一次記録・経緯・理由 | — | **KEEP-ACTIVE**（意思決定の一次ソース。正本は結論を吸収、原本は残す） |
| `docs/AUTOMATION_CONTRACT.md`（delivery-core PR #59） | 自動化制御プレーン | E §7（要約）／ 本体は維持 | referenced（重複記載しない） | terminal classes の逐条、snapshot スキーマ | なし | **KEEP-ACTIVE**（execution governance の SSOT。E は要約のみ） |
| `ops/renovation-status.json`（delivery-core PR #59/#60） | レーン状態スナップショット | E §4（workstream status） | partially absorbed | 機械可読な lane snapshot（watchdog 用） | なし（GitHub が正・snapshot は便宜） | **KEEP-ACTIVE**（機械可読 snapshot。E §4 は人間可読版） |
| `public/docs/green-storage-integration.md` ほか green rules（hp） | Green admin-products の Storage/RTDB 設計 | C §8.1, §8.2（概要）／ E §4.3 | partially absorbed | Green RTDB スキーマ全フィールド、Storage パス、save policy、`window.APGreen` API | なし | **KEEP-ACTIVE**（Green admin-products 実装の SSOT） |
| `public/.github/katakana/RELEASE1.md` + `dictionary-audit.json`（hp） | Katakana R1 ハンドオフ + 監査 | E §4.7（要約） | partially absorbed | R1 の diff 詳細、preserved interfaces、Blue release checklist、Targeted Green integration 手順、辞書監査の全エントリ | 軽微（R1 = 837 キー、現在は R3 = 1,875 キー。E §4.7 で最新化） | **KEEP-ACTIVE**（Katakana 統合の実務ハンドオフ。件数は E が正） |

---

## 2. Contradiction audit

指定16項目の「旧仕様混入」チェック。**該当（旧記述が残っている）= ⚠️** / **非該当 = ✅**。

| # | チェック項目 | 判定 | 該当ファイル・記述（古いもの） | 正しい現行仕様 |
|---|---|---|---|---|
| 1 | Workshop 事業が現行扱い | ✅ | 最新資料はすべて Workshop 終了を明記。旧 `事業概要.md`（2025-12/2026-01）のみ Workshop 前提だが「旧版」と明示済み | Workshop 終了済み（C §2.2） |
| 2 | old delivery hours（AM4–8 等） | ⚠️（軽微） | `事業概要_最新版_20260908` は「4:00 AM–8:00 AM → 8:00 PM–8:00 AM（2026/09/01〜）」と取り消し線併記で残す。`事業概要_v2` は 2026-09-01 改定で移行途中の表現 | 20:00–08:00（顧客予約）/ 20:00–08:30（営業）（C §4.1） |
| 3 | old pricing | ⚠️（軽微） | `事業概要_最新版` §4 は価格変遷を取り消し線併記で保持（¥4,500 / ¥8,600 等）。`仕様サマリー_20260903` は当時価格 | Classic ¥3,500（+¥2,000/膳・最大5膳）、Engimon ¥5,800、Happy Life ¥7,800、Express +¥3,000（C §5） |
| 4 | same-night 限定 redelivery | ✅ | `Delivery System Rules.md` は same-night 限定を legacy として §29 で削除対象に明記。新資料に混入なし | Free Redelivery は `originalDeliveryDateTime + 90分` 以降の営業時間内空き枠。same-night 限定なし（C §4.3） |
| 5 | third_delivery が originalDeliveryDateTime 基準 | ⚠️ | `再配達システム改修_設計書.md` §8「3rd Delivery確定: originalDeliveryDateTime = 初回日時」§9/§11 の表現、`Delivery System Rules.md` §12 の originalDeliveryDateTime 言及。`事業概要` §3 の再配達運用記述 | **3rd Delivery の最短 = `secondDeliveryDateTime`（NG2 化した2回目配達）+ 90分 のみ。originalDeliveryDateTime fallback 禁止。legacy 欠損は fail closed**（owner decision 2026-09-10 / delivery-core PR #56。C §4.6） |
| 6 | NG2 page 不要扱い | ✅ | `Delivery System Rules.md` §14、Roadmap Phase 1-E、`再配達システム改修_設計書.md` §10 いずれも NG2 専用ページ必須。`ng2.html` 実装済み。Green 統合必須（PR #25） | NG2 専用ページ（`ng2.html`）は確定成果物（C §7、E §4.6） |
| 7 | admin-products 専用 Green 解釈 | ⚠️ | 過去チャット / 一部 PR 記述で `renewal/green-20260909` を admin-products 専用と誤解した経緯あり（`js/admin-products-green-config.js` のコメントは「admin-products 専用」だが、これは *その設定ファイル* が admin-products 専用の意） | `renewal/green-20260909` は **最終 Green 統合先の基盤**。admin-products は1レーン（E §3.2, §4、C §8.7） |
| 8 | Extra Slots（ゲリラ枠） | ⚠️ | `事業概要_最新版` §6 管理画面「本日用のゲリラ枠（Extra Slots）設定」、§14「Extra枠（ゲリラ枠 / 1+1モデル）のフロントエンド実装」 | **Extra Slots は削除**（C §6.4、E §4.5。DailyLog 2026-09-08） |
| 9 | inventory が General | ⚠️ | 旧 admin.html は「全日程共有のマスター在庫管理」を含む（`事業概要_最新版` §6） | **inventory は Products へ**（C §6.4） |
| 10 | Delivery List が General | ✅ | Roadmap / 管理機能改修 は Delivery List を admin-delivery に配置。旧 admin.html は単一画面なので「General」概念自体が無い | Delivery List は Delivery（C §6.2） |
| 11 | production capacity が Delivery | ✅ | 資料は production capacity（usedMinutes）を運行設定として扱い、admin-general 側。混入なし | production capacity は General（C §6.1） |
| 12 | Blue Firebase config を Green で使う | ⚠️（対処済み） | `admin-general.html`（Blue snapshot `aef5afd`）は inline で `chopstickers-workshop` config + `/status` read-update | PR #25 で `chopstickers-project` へ repoint 済み（Green-safe 化）。production-derived 接続は quarantine（C §8.7, §9、E §6） |
| 13 | old admin structure（単一 admin.html） | ⚠️（移行途中） | `事業概要_最新版` §6 は現行 admin.html（単一）を記述しつつ3分離を「改修中」と併記 | 3画面分離（general / delivery / products）が最終形（C §6） |
| 14 | old Katakana dictionary / version | ⚠️ | `事業概要_最新版` §6「katakana.json 458件」、`RELEASE1.md`「837 keys」、旧 `ui.js` の `convertAlgorithmicKatakana` のみ記述 | R3 = **1,875 キー** + `katakana-engine.js` / `katakana-widget.js` 新規。Green 統合正本 `feature/katakana-r2-20260910` @ `39bea3c`（E §4.7） |
| 15 | old Green integration status | ⚠️ | `事業概要_最新版` に Green（`chopstickers-project`）記述なし。admin-products「データ層未接続」 | Green project 全 provision 済み。admin-products は Green Storage 統合まで実装済み（E §4.2–§4.3、C §8.1） |
| 16 | completed work を not_started 扱い / failed workflow = 実装失敗 / same generic retry / merge=commit と同一工程 | ⚠️ | Roadmap（main版）は Phase 進捗が 2026-09-03 で停止（NG2 / admin-delivery / Delivery Core を未着手相当に見せる）。旧運用は failed GitHub workflow を実装失敗と混同、同一 generic retry を継続 | DailyLog 2026-09-08〜11 で大幅進捗。`AUTOMATION_CONTRACT.md` §2–§4 + 2026-09-11 strategy switch フロー（E §4, §7）。deploy は commit と別工程（Roadmap §6、C §9.3） |

**まとめ:** ✅ 5項目（#1, #4, #6, #10, #11）／ ⚠️ 11項目。⚠️ はいずれも新2正本（C / E）で正しい現行仕様を明示済み。旧記述は「取り消し線 + 新（日付）」形式で原本に履歴として残っているもの（#2, #3）と、正本更新で解消すべきもの（#5, #7, #8, #9, #13, #14, #15, #16）に分かれる。

**2026-09-16 owner decision（履歴追記）:** 旧 Series pricing 方針を撤回し、Product の `price` / `additional_price` / `currency` を価格 SSOT とした。Series は `series_id` / `category_id` / `label` / `sort_order` / `status` のみ。詳細と未実装境界は C §5、E §4.3a。旧方針の記録は歴史資料として維持する。

**2026-09-17 owner decision（履歴追記）:** 以下を canonical へ反映。旧記述は「取り消し線 + 新（日付）」形式または明示的 supersede 注記で歴史資料として残す。
- **Storefront 入口:** 旧「Select Category 専用画面 → Category → 注文画面」を廃止し、単一 Order Builder（上部常設 `[ Daily Use ] [ Gift ]`）へ統一。C §5.0 の旧「Select Series → Select Category」記述は C §3a へ supersede 済み。詳細 C §3a、E §4.14。
- **Cart / Review:** 共通 Cart canonical（`product_id` 識別・Daily Use Line 集約 + Unit 保持・全 Line review gate・category 別 renderer）。旧 Pair 固定状態表示は廃止方向。詳細 C §3a.4。
- **Pricing:** server-authoritative pricing・Order create 時の pricing snapshot 固定（`schema_version: 1`）・Express は order-level charge（Product price に混ぜない）。詳細 C §5.9。
- **canonical Order Core:** 新 action `core_create_order`（`schema_version: 1`、`idempotency_key` 必須）。旧 `core_new_order` は Blue legacy route のまま維持し Green canonical には使わない。詳細 C §8.3a、E §4.15。
- **Reservation ownership:** Green server が reserved stock / delivery slot / production capacity を所有。Blue `usedMinutes` との dual-write 禁止。詳細 C §8.3b。
- **Blue → Green capacity cutover:** 5ステップ手順（新規受付停止 → 予約監査 → one-time migration → 検証 → Green intake 開始）。以後 Green counter のみ正本、恒久 sync counter 禁止。詳細 C §8.7、E §4.12。
- **Delivery Time UI:** Other/Special Request 廃止。AM/PM タブへ変更。Cutoff は "Order by midnight" を基本文言とし、グレースピリオドは別文明記。詳細 C §4.13。
- **Important Rules 文言:** "hotel" → "Accommodation" 表記統一。到着後10分待機ルールの後に無料再配達3点（最短90分後・1回のみ・営業時間内空き枠）を明記。旧「120分前まで無料」表現は使わない。詳細 C §4.6a。
- **Temporary Coming Soon production mode（PR #75, hp）:** owner 承認済みの実装だが、**現時点では本番投入しない方針**（`main` 未merge・production 未deploy）。旧 Blue 注文コードは削除せず保持する設計。詳細 C §9.4、E §4.13。**現行 production の "Ordering unavailable" 表示は、この PR #75 の deploy ではなく、owner が既存 Blue サイトを手動でメンテナンス＋配達受付停止モードへ切り替えたことによるもの**（2026-09-17）。

---

## 3. Old docs classification（Phase D 提案・未実施）

| 分類 | ファイル | 実施タイミング |
|---|---|---|
| **DELETE 候補** | `GAS_redelivery_revised.gs` / `GAS_redelivery_TEST.gs` | delivery-core repo への取り込み完了を関数単位で確認後 |
| **ARCHIVE 候補** | `事業概要_Chopstickers_v2.md`（delivery-core docs から。Drive は比較用に保持）／ `事業概要.md` ×2（Drive）／ `仕様サマリー_Chopstickers_20260903` + `Chopstickers_Specifications_Summary`（Drive）／ `再配達_実環境TEST_セットアップ手順.md` / `再配達_実環境TEST手順書.md` / `Chopstickers-workshop プロジェクト調査報告.md`（workshop root → `archive/` サブフォルダへ） | 新2正本の owner 承認後 |
| **KEEP-AS-REFERENCE** | `GASのコード.md`（delivery-core との対応確認まで） | — |
| **KEEP-ACTIVE** | `Delivery System Rules.md`（§4.6 追記の軽微修正推奨）／ `Chopstickers System Renovation Roadmap.md`（PR #55 §0 追記版）／ `管理機能改修 仕様書＆工程表.md` / `admin-products.html_仕様書.md`（詳細設計 SSOT）／ `再配達システム改修_設計書.md`（§9–11・§10 の軽微修正推奨）／ `AUTOMATION_CONTRACT.md` / `ops/renovation-status.json`（PR #59/#60）／ `public/docs/green-*`／ `public/.github/katakana/*`／ `事業概要_最新版`（Drive・運用/機材/収支 SSOT）／ 月次 DailyLog | — |
| **変更しない（今回）** | `CLAUDE.md`（短縮案は §4） | Phase D で coverage 確認後 |

**重要:** DELETE / ARCHIVE 候補は、対応する unique information が新正本または他の KEEP-ACTIVE 文書へ吸収済みであることを Phase D で1件ずつ確認してから実行する。

---

## 4. 提案: 短縮版 CLAUDE.md

> 今回は適用しない。新2正本の coverage 確認後に置き換え提案する。現行 `CLAUDE.md`（§1–11 逐条）の安全ルールは C §9 / §10 と E §7 / §9 へ吸収済み。

```markdown
# CLAUDE.md — Chopstickers

## READ FIRST
1. docs/CHOPSTICKERS_CANONICAL.md — 事業・システムの正本（何を作っているか / どう振る舞うべきか）
2. docs/RENOVATION_EXECUTION.md — 改修の現在地・統合順序・作業ルール・自動化制御

矛盾したら:
- business / system の事実 → CHOPSTICKERS_CANONICAL.md
- execution / progress / integration → RENOVATION_EXECUTION.md
- 自動化制御の詳細 → docs/AUTOMATION_CONTRACT.md
- GitHub（Issue / PR / commit）が最終的な canonical。
- 上記2正本で明示的に参照されない旧資料は non-canonical。

## 原則
- Simple / Honest / Minimal。business rule を発明しない・コードから推測しない。
- owner decision と矛盾する古い文書は superseded。
- コピーライティングは誇張表現を使わない。

## 変更する前に必ず報告
- 現状確認 / 変更内容 / 影響範囲 / テスト方法

## 続けてよい作業（owner 承認を待たない）
- reversible な repo-only 作業: コード編集・docs・テスト・branch・commit・push・PR 作成（merge なし）

## 停止する境界（owner の明示的承認が必要）
- main merge / PR merge / production deploy / clasp push・deploy
- 本番 Firebase write / Firebase Rules 適用・変更 / 本番データ削除
- Stripe production / 本番 GAS / Script Properties 変更 / Hosting deploy / DNS 変更
- その他 irreversible な production action → OWNER_WAIT で停止し、必要な承認を具体的に述べる

## 禁止
- 秘密情報（API キー・トークン・秘密鍵・サービスアカウント鍵）の生成・出力・保存
- whole-tree の pull / reset / checkout、Blue public の丸ごとコピー
- Green の新しい成果を Blue 版で上書き
- 他レーンの成果（Katakana / PR #56 等）の再実装
- coverage 確認前の old docs 削除
```

---

*本ファイルは監査用。Phase D 完了後に削除してよい。最終正本は `CHOPSTICKERS_CANONICAL.md` と `RENOVATION_EXECUTION.md` の2本 + 短縮版 `CLAUDE.md`。*
