# CHOPSTICKERS_CANONICAL.md
Chopstickers 事業・システム 正本 / Business & System Single Source of Truth

---

## 1. Document status

- **本ファイルの位置づけ:** Chopstickers の「事業として何を提供しているか」「システムがどう振る舞うべきか」を定義する **business / system canonical source**。
- **対になる文書:** 改修の進行・統合順序・作業ルールは `RENOVATION_EXECUTION.md`（execution canonical）。本ファイルにはタスク状態を書かず、仕様の実装境界のみ §5.8・§3a.4・§8.3a/8.3b に記す。
- **更新日:** 2026-09-17（Storefront Order Builder / Cart / canonical Order Core / reservation ownership / Delivery Time UI / Important Rules 文言 / Temporary Coming Soon production mode の owner decision を反映。詳細は §3a・§4.13・§5.9・§8.3a・§8.3b・§9.4）
- **supersedes（本ファイルが正本を引き継ぐ対象。原本は参照資料として残す）:**
  - Google Drive『事業概要_最新版_20260908』のうち「現行サービス／事業コンセプト／配送ルール／価格／admin構成／データ構成」に関する記述
  - Google Drive『事業概要_Chopstickers_v2.md』（旧版・全体）
  - `Delivery System Rules.md`（逐条は KEEP-ACTIVE。要点は本ファイル §4 に集約）
  - `管理機能改修 仕様書＆工程表.md` / `admin-products.html_仕様書.md` / `再配達システム改修_設計書.md` の「事業ルール・画面責務」部分（詳細設計は各文書を KEEP-ACTIVE）
- **矛盾時の優先順位（高い順）:**
  1. 最新の明示的 owner decision（DailyLog / GitHub issue・PR コメントで確認できるもの）
  2. 最新の稼働方針 / confirmed Green integration policy（本ファイル §9、`RENOVATION_EXECUTION.md`）
  3. 最新の確定 business rule（本ファイル §4・§10、`Delivery System Rules.md`）
  4. merged / persisted かつ reviewed なコード
  5. 最新事業概要（Google Drive）
  6. `Chopstickers System Renovation Roadmap.md`
  7. `CLAUDE.md`
  8. 各詳細仕様書
  9. 旧事業概要・旧テスト指示・旧 WIP メモ
- **原則:** business rule をコードから推測しない。コードが新しいことは business rule がコード側で正しいことを意味しない。owner decision と矛盾する古い文書は **superseded** として扱う。

---

## 2. Business overview

### 2.1 現行サービス
- **事業名:** Chopstickers（ブランド名は継続）。
- **現行事業:** Chopstickers Delivery — 東京の宿泊者向けに、名入れ刻印した箸を宿泊先へ配達するサービス。
- **提供物:** 手頃な価格の既製品の箸（箱入り）に小型レーザーで名入れ刻印し、梱包して宿泊施設の前で手渡し。
- **キャッチフレーズ（現行）:** "Midnight Engraving Lab" / "Design before sleep, receive the next morning."
- **ロゴサブテキスト:** ENGRAVING & DELIVERY（全言語で英語表記に統一）。
- **ブランドトーン:** Honest & Minimal。「Premium」「Luxury」「最高の」「究極の」等の誇張表現は使わない。事実ベースで記述する。

### 2.2 Workshop 事業の扱い
- **Workshop（体験型・THE MIDNIGHT ENGRAVING LAB）は終了済み。** システム・HTML・GAS コードは削除済み。将来「VIP・法人向け高単価ワークショップ」へピボットする可能性はあるが、**現行仕様ではない**。
- 旧 Workshop の営業時間・価格・定員・「0〜5時の前日扱い」等の仕様を現行仕様として残さない。

### 2.3 ターゲット・地理
- **主ターゲット:** タビマエ層（訪日前・旅行計画中の欧米圏旅行者）。母国語ディレクトリ（/fr /de /it /es）経由の流入を主軸に、来日前の事前注文・帰国直前の買い忘れ対応を狙う。
- **対象外:** 箸文化圏（日中韓台ほかアジア圏）。
- **配達エリア:** 東京都東部6区（江戸川・江東・墨田・台東・千代田・中央）限定。港区・渋谷区・新宿区は将来の段階的緩和候補（未確定 / §11）。
- **ブラックリストエリア:** 銀座・御徒町・アメ横・神田・新橋等の繁華街はシステムでブロック（安全確保）。
- **引き渡し:** 宿泊施設の前で直接手渡し（施設内立入不可）。到着後最大10分待機。

### 2.4 独自性
- **NIGHT & EARLY MORNING DELIVERY**（20:00–08:00 に宿泊施設前で手渡し）。
- **Katakana Converter**（多言語音素変換エンジン + 固有名詞辞書。名前を即時カタカナ化してプレビュー表示）。

---

## 3. Customer journey

1. **Order** — 顧客が言語別サイトで商品（Classic / Gift）を選択、刻印文字・色・配達日時・宿泊先住所・連絡先を入力。100% 事前決済（現金不可）。
2. **Preview** — FV の簡易プレビュー（Quick Preview）と Live Preview 画面で刻印イメージを確認。Katakana Converter で名前のカタカナ表記を確認。FV の入力は Live Preview へ引き継がれる（英字 → 箸1 / カタカナ → 箸2）。
3. **Payment** — Stripe Checkout Session API で合計金額を動的に決済ページ生成 → 決済 → `success.html` へ遷移。
4. **Delivery（Normal）** — 予約枠に基づき宿泊先へ配達。到着15〜30分前に WhatsApp 連絡。成功で `status = delivered`。
5. **NG1（初回不在）** — 到着後10分待機して不在 → `status = NG1`、`originalDeliveryDateTime = 初回 scheduledDateTime` を保存。顧客へ無料再配達ページ（`redelivery.html`）URL を送信。
6. **Free Redelivery** — NG1 のみ・1回無料・初回住所固定。最短 = `originalDeliveryDateTime + 90分`。確定で `status = pending` / `deliveryStage = redelivery`。再変更・再予約不可。
7. **NG2（再配達も不在）** — `status = NG2`。無料再配達は終了。顧客へ NG2 専用ページ（`ng2.html`）URL を送信。この時点で `secondDeliveryDateTime`（= NG2 となった配達の scheduledDateTime）を保存。
8. **NG2 対応（`ng2.html` で3択）:**
   - **3rd Delivery — ¥1,000**（Stripe 事前決済）。最短 = `secondDeliveryDateTime + 90分`。住所変更可。決済成功で `status = pending` / `deliveryStage = third_delivery`。
   - **Domestic Shipping — ¥1,000**（Stripe 事前決済・国内のみ）。決済成功で `status = shipping`。
   - **Disposal — FREE**（明示同意必須・返金なし）。確定で `status = disposed`。
9. **Completion** — 3rd Delivery 成功で `status = delivered`。3rd Delivery も未完了なら以降は Web で自動提供せず Email / WhatsApp 個別対応。

---

## 3a. Storefront Order Builder & Cart canonical（2026-09-17 owner decision）

> 上記 §3 step 1「Order」の canonical 画面構造。旧「Select Category 専用画面 → Category → 注文画面」は**廃止**。§5.0 の旧記述（Select Series → Select Category）はこの節へ superseded。

### 3a.1 Single Order Builder
- 顧客の入口は **単一の Order Builder 画面**。専用の Select Category 画面は持たない。
- 画面上部に **常設** の `[ Daily Use ] [ Gift ]` カテゴリ切替を置く。
- カテゴリ切替は **注文入力の途中でも可能**。切替しても既存の Cart 内容・入力済みの制作情報（engraving text / 色 / 素材選択等）を消さない。

### 3a.2 Daily Use フロー
Category → Series → Product / Material → Engraving → Cart / Review

### 3a.3 Gift フロー
Category → Matching Pair → Gift Product → Gift engraving → Cart / Review

### 3a.4 Cart / Review canonical
- **共通 Cart を canonical とする。** Daily Use と Gift を同じ Cart 内に混載可能。
- Line の Product identity は `product_id`。
- Daily Use は同一 Product を **Line 集約**する（数量表示）。個別の engraving・向き・選択内容は **Unit 単位で保持**する（集約後も Unit ごとの製作情報は失わない）。
- **全 Line review gate:** Cart 内の全 Line が reviewed 済みでない限り checkout へ進めない。
- Review Modal は **category 別 renderer**（Daily Use renderer / Gift renderer）を持つ共通シェル。
- 製作情報を編集した Line は `reviewed = false` に戻る（再レビュー必須）。
- Pricing snapshot は `schema_version: 1`（詳細 §5.9）。
- **旧仕様（廃止方向）:** Pair ごとの固定状態表示（旧 Gift の Pair 単位 UI）は廃止方向とする。
- Cart は Order Builder 画面下部の **常時確認 UI** として統合する（別画面へ遷移しない）。

### 3a.5 実装境界（2026-09-17 時点）
- hp PR #73（`storefront-cart-order-builder-20260917`）: 共有 Cart（category tabs・category 別 subtotal・Order Total）、Daily Use の Line 集約 + Unit 保持、共通 Review Modal シェル + category 別 renderer、編集時 `reviewed=false` を実装。Daily Use/Gift 混載の **表示・レビューは可能**だが、既存 submit payload が両カテゴリを同時に表現できないため **混載 checkout は意図的にブロック**（storefront 側で `core_create_order` 未接続のため）。既存の Daily Use 単体 / Gift 単体 submit payload は無変更。
- hp PR #74（`storefront-order-pricing-snapshot-20260917`）: reviewed 済み Cart から `schema_version: 1` の pricing snapshot を生成（詳細 §5.9）。混載 Cart の snapshot 生成はブロック。
- **未実装:** Order Builder 入口（上部 `[ Daily Use ] [ Gift ]` 常設トグルによる旧 Select Category 画面の置換）、混載 Order の storefront → `core_create_order` 接続、Pair 固定状態表示の廃止（UI 側）。PR #73/#74 は `main` 未merge・production 未deploy。

---

## 4. Delivery rules

> 逐条の正本は `Delivery System Rules.md`（2026-09-03）。本節はその要点と、それ以降の owner decision による確定事項を集約する。矛盾時は「最新 owner decision > 本節 > Delivery System Rules.md 逐条」。

### 4.1 Delivery window / 表示
- **顧客が予約できる配達時間:** 20:00–08:00、**30分刻み**、最終枠 **08:00**。
- **運営上の営業時間:** 20:00–08:30（待機・終了処理を含む）。
- **Web 表示:**
  - 短い表示: `NIGHT & EARLY MORNING DELIVERY / 8:00 PM – 8:00 AM / Last delivery slot: 8:00 AM`
  - 詳細・法務: `Operating hours: 8:00 PM–8:30 AM / Last available delivery slot: 8:00 AM`
- 「20:00–08:00 = 予約可能枠」「20:00–08:30 = 運営営業時間」は矛盾ではなく役割の違い。

### 4.2 Slot unit / footprint
- 配達枠は **1つ**。Standard / Express Overnight / Free Redelivery / 3rd Delivery が **同じ `deliverySlots` を共有**。専用の `redeliverySlots` は作らない。
- 予約1件成立で、選択時刻**とその前後30分**を占有（±1枠 footprint）。日跨ぎにも同じルール。
- **先着順。** 同一枠を同時選択した場合、最初に確定した注文のみ成立。
- 予約種別で変わるのは「選択可能開始日時」「料金の有無」「住所変更可否」のみ。

### 4.3 予約タイミング（注文種別ごとの選択可能開始）
| 種別 | 料金 | 選択可能開始 | 住所変更 |
|---|---|---|---|
| Standard Delivery | 商品価格に含む | 注文日の**翌々日 00:00 以降**（カレンダー日基準。特殊な「営業日」概念は作らない） | — |
| Express Overnight Delivery | **+¥3,000** | 23:59 締切。翌日 **04:00–08:00 / 20:00–24:00** の空き枠 | — |
| Free Redelivery | 無料 | `originalDeliveryDateTime + 90分` 以降の営業時間内空き枠 | 不可（初回住所固定） |
| 3rd Delivery | **¥1,000**（Stripe 事前決済） | `secondDeliveryDateTime + 90分` 以降の営業時間内空き枠 | 可（初回住所ワンタップ再利用 or 新住所。新住所も配達エリア制限に従う） |
| Domestic Shipping | **¥1,000**（Stripe 事前決済） | 顧客が希望日時を指定（配送業者へ伝えるのみ・到着保証なし） | — |

- グレースピリオド 0:00〜0:10 は既存運用として維持。

### 4.4 status / deliveryStage
- **status（現在この注文をどう処理するか）:** `pending` / `NG1` / `NG2` / `delivered` / `shipping` / `disposed`
- **deliveryStage（何回目の配達か・補助情報）:** `normal` / `redelivery` / `third_delivery`
- status と deliveryStage は役割を分ける。redelivery を status として扱わない。

### 4.5 Order モデル
- **基本:** `orderId` / `status` / `deliveryStage` / `scheduledDateTime`
- **必要時:** `originalDeliveryDateTime` / `secondDeliveryDateTime` / `paymentStatus` / `resolution`（`domestic_shipping` | `disposal`）/ `shippingAddress` / `shippingPreferredDateTime` / `adminNotes` / `evidencePhoto`
- **旧フィールド**（`timeSlot` / `type` / `redeliveryDate` / `redeliveryKey` / `sleep_in` / `Today_*` / `Tomorrow_*`）は移行期間の**読み取りフォールバックのみ**。新規注文・新規再配達の正式モデルに書き戻さない。Renewal 完了後に削除。

### 4.6 third_delivery timing（owner decision 2026-09-10・確定）
- **3rd Delivery の基準時刻 = `secondDeliveryDateTime`（NG2 となった2回目配達の scheduledDateTime）+ 90分 のみ。**
- **`originalDeliveryDateTime` への fallback は禁止。**
- NG2 時に `secondDeliveryDateTime = order.scheduledDateTime` を**凍結保存**する。
- 復旧不能な legacy データ（該当タイムスタンプ欠損）は **fail closed**（3rd Delivery 予約を通さない）。
- `originalDeliveryDateTime` は admin 表示の参考情報としてのみ保持（分類・予約判定に使わない）。
- 実装: `chopstickers2025/chopstickers-delivery-core` PR #56（`4881d6b`）。third_delivery reschedule anchor / adjacent-footprint overlap / retry idempotency を修正済み。

### 4.6a NG2 UI 要件（owner decision 2026-09-13・確定、`ng2.html`）
- 住所再利用 UI 文言: 「Use the address from my first delivery」（旧「Use my original delivery address」から変更）。選択時、その直下に初回配達時の住所（`originalAddress.address` / `.postal`、将来 `facilityName` 追加時はそれも含む）を実データで表示する。legacy 注文で一部フィールドが欠損している場合は、存在するフィールドのみ表示する。
- 3rd Delivery 申込み時、以下2点の同意チェックボックスを**両方**必須とする（未チェックの間 ¥1,000 payment CTA を無効化）:
  1. 追加料金¥1,000の支払い完了後に配達日時が正式確定する旨。
  2. 3rd Delivery でも不在の場合、以降のWeb再配達は提供されず、返金なしで廃棄される旨（文言は `terms.html` §1 と一致させる）。
- Domestic Shipping 申込み時、Japan-only / tracking number email 通知 / carrier引渡し後の責任分界（当店の発送手配ミスは免責しない）を明示し、同意チェックボックスを必須とする（未チェックの間 ¥1,000 payment CTA を無効化）。
- Demo Mode は `secondDeliveryDateTime` のデモ値を持ち、実際に+90分フィルタが機能することを確認できる（originalDeliveryDateTime へのfallbackが発生しないことも含む）。
- **同期範囲（owner decision 2026-09-13・確定）：** 上記ルールは `ng2.html` / `terms.html` / `commerce.html` に加え、`index.html` の Important Notice・FAQ・注文直前の Important Rules（`del_rules_list`）へも簡潔な要約を反映する。初回注文画面の conversion を害さないよう、詳細は Terms/FAQ/ng2.html に譲り、注文直前の注意文は一文程度に留める。fr/de/es/it（`js/translations.json`）へも同内容を翻訳し、English fallback 漏れがないことを確認する。Green（PR #31）・Blue（PR #32、法務文言のみ）双方で同期済み。
- **表記統一（owner decision 2026-09-17・確定）：** 顧客向け表示文言の "hotel" 固定表現は **"Accommodation"** へ統一する（`del_rules_list` / Important Notice / FAQ / terms / commerce 含む、英語表記の統一。日本語側の「宿泊施設」は変更なし）。
- **Important Rules 追記（owner decision 2026-09-17・確定）：** 到着後10分待機ルールの記述の後に、以下を明記する:
  - 最短90分後から無料再配達可能（§3 step 6 の `originalDeliveryDateTime + 90分` と同一）。
  - 無料は1回のみ（§4.8 と同一）。
  - 営業時間内の空き枠から選択（§4.3 の「営業時間内空き枠」と同一）。
  - **旧記述「再配達希望時間120分前まで無料」は削除する。** 本ファイルには元々この記述はない（`RENOVATION_EXECUTION.md` §10.3 の旧フィールド削除計画に含まれる「旧120分ルール」はコード側の legacy 参照であり、本節とは別管理）。もし顧客向け UI 文言に同種の旧表現が残っていれば、この owner decision をもって置き換える。
  - 実装状況: 上記の文言統一・追記は canonical 決定のみで、`index.html` 等コード側への反映は未実施（コード repo 変更は本更新の対象外）。

### 4.7 Slot / reschedule の不変則
- 各予約は ±1枠の footprint を占有する。
- 隣接予約が存在しても、自分の footprint 内での reschedule を誤ってブロックしない（old/new footprint の可用性判定は preview release に対して行い、共有する隣接枠は closed のまま維持）。
- キャンセルは、別の予約がまだ必要としている枠を解放しない。
- 有料注文（3rd Delivery / Domestic Shipping）の顧客側キャンセルは提供しない。

### 4.8 Retry / redelivery の制約
- Free Redelivery は NG1 のみ・1回限り・住所変更不可・確定後の再予約不可。
- 3rd Delivery も未完了の場合、Web で自動的な追加予約は提供せず Email / WhatsApp 個別対応。

### 4.9 保管期限
- 基本 = **初回配達予定日 + 7 暦日 の 23:59 JST**。表示は具体的な日時を JST で明示（「1 week」等の曖昧表現は使わない）。
- 期限までに正式なアクション（Free Redelivery 予約完了 / 3rd Delivery 予約+決済完了 / Domestic Shipping 申込+決済完了 / Disposal 選択 / 個別承認対応）が無ければ、翌 00:00 JST 以降に自動廃棄対象。
- ページ閲覧・フォーム途中はアクション完了とみなさない。
- 期限内に正式アクションを完了していれば、指定日が期限を超えても期限超過のみを理由に廃棄しない。

### 4.10 Domestic Shipping の責任範囲
- 国内のみ・海外発送不可。入力: Recipient Name / Postal Code / Address / Preferred Delivery Date / Preferred Delivery Time。
- 希望日時は配送業者へ伝えるのみで到着保証なし。配送業者へ引き渡した時点で Chopstickers の発送業務は完了（遅延・紛失・破損・誤配・持戻り等は直接管理外。ただし当店の手配ミス・発送前の当店責は除く）。
- **発送後、追跡番号（tracking number）を顧客へ Email で通知する（owner decision 2026-09-13・確定）。** 絶対免責表現（「発送後は一切責任を負わない」等）は使わない。配送業者引渡し後の配送状況・日時調整・不在再配達は、顧客が追跡情報を用いて配送業者と直接やり取りする旨を明示する。

### 4.11 返金
- いかなる時点での不在でも返金不可（特商法ページに明文化）。
- 製造上の欠陥・刻印ミスは代替品再送または全額返金。
- **機材トラブル等 Chopstickers 側の事情による履行不能（owner decision 2026-09-13・確定）：** 基本ルールは**全額返金・注文キャンセル**。故障時点では修理可否・代替機調達・到着時期・再制作から配送完了までの所要期間が不明であり、固定的な部分返金・無償再配送の保証はできないため。復旧後の無償再制作・無償配送・割引等の任意対応は owner 裁量による個別対応（Email/WhatsApp 経由）であり、顧客への保証・規約上の権利ではない。旧「50%返金＋無料配送（国内滞在7日以上）」ルールは撤回済み（採用しない）。

### 4.12 fail-closed legacy behavior
- 新モデルに必要なタイムスタンプ・状態が復旧不能な legacy 注文は、危険側に倒すのではなく **fail closed**（該当機能を提供しない）とし、admin での個別対応に委ねる。

### 4.13 Delivery Time UI canonical（2026-09-17 owner decision）
- **Other / Special Request は廃止。** 配達時間選択の選択肢は「Night & Early Morning」→「Next-day Express Delivery」の順のみとする。
- Delivery time 選択 UI は **AM / PM タブ**へ変更する（時刻を1本のリストで見せる現行方式から変更）。
- Cutoff 表示は **"Order by midnight"** を基本文言とする。**00:00–00:10 のグレースピリオド**（§4.3 既存運用）は基本文言と混在させず、別文として明記する。
- **実装状況:** UI変更は未実装（canonical 決定のみ）。既存の Express Overnight（§5.3, 23:59 締切）・グレースピリオド運用（§4.3）とは矛盾しない — 表示文言と UI 構造のみの変更であり、締切時刻・料金・対応時間帯（翌日 04:00–08:00 / 20:00–24:00）は変更しない。

---

## 5. Product / pricing

> 「現行」= 本番で稼働している価格。「方針確定」= owner decision だが実装・表示は別途確認する。未確定は推測しない。

### 5.0 Product pricing / hierarchy canonical（2026-09-16 owner decision）
- 階層は **Category → Series → Product**。`daily_use` の Series は Classic / Sakura。`gift` の Series は Matching Pair、Product は Engimon / Happy Life / 将来追加する夫婦箸 Gift 商品。
- Series canonical は `series_id` / `category_id` / `label` / `sort_order` / `status`。Series は `price` / `base_price` / `additional_price` / `currency` を持たない。**価格 SSOT は Product** の `price` / `additional_price` / `currency`（`JPY`）。Series pricing 方針は撤回済み。
- `additional_price` は**同じ `product_id` の2個目以降、1個あたり**の価格。合計は数量1なら `price`、数量2なら `price + additional_price`、数量3なら `price + 2 × additional_price`。異なる `product_id` 間で共有しない。
- 現HPコードの Gift Series ID は `giftbox`。Matching Pair は business / display 名であり、`giftbox` → `matching_pair` の ID migration は未実施。
- ~~将来の Green Storefront 入口は Select Series → Select Category。~~ **superseded（2026-09-17）:** Storefront 入口は Select Category 専用画面を廃止し、単一の Order Builder + 上部常設 `[ Daily Use ] [ Gift ]` トグルへ変更。詳細・フロー全体は §3a。
- 全商品30% OFF・クーポン・合計金額条件割引などは、将来の **Cart-level discount** として Product pricing と分離する。Cart-level discount engine は未実装（Cart 自体は §3a・PR #73 で表示・レビューまで実装済み）。

### 5.1 Classic (Daily Use) — 現行
- 刻印対象: 箸のみ。着色: 金・銀・無色（Burn）。「Silver」は廃止済み。
- 使用可能文字: 英数字・ひらがな・カタカナのみ。**最大10文字**（内部スペース・句読点含む。入力・刻印出力の双方に10文字上限。超過は切り詰めず拒否）。
- 価格（現行）: 1膳 **¥3,500** / 追加1膳ごと **+¥2,000** / まとめ買い最大 **5膳（¥11,500）**。
- Product pricing canonical: `price=3500` / `additional_price=2000` / `currency=JPY`。
- 表記: "From ¥3,500 (Includes Engraving & Standard Delivery)"。

### 5.2 Matching Pair Gift Series — 現行
- 刻印対象: 桐箱のみ（箸本体への刻印なし）。使用可能文字: **英語のみ**（カタカナ非対応）。
- Engimon (Gift Box)（旧・梅）: **¥5,800**。
- Happy Life (Gift Box)（旧・竹）: **¥7,800**。
- Product pricing canonical: Engimon `price=5800`、Happy Life `price=7800`、いずれも `currency=JPY`。Gift Product の `additional_price` は未決定（`0` や通常価格と同額を設定しない）。
- 「松（Pine）」プラン・「松竹梅」プラン名は**完全廃止**。
- 文字数/行数: Engimon 最大18文字 × 8行 / Happy Life 最大18文字 × 12行。上限 20文字 / 15行（DailyLog 2026-09-08）。フォント: Times New Roman（Bold + Italic）。センター寄せ。

### 5.3 Express Overnight Delivery — 現行
- 追加料金 **+¥3,000**。翌日配達（朝・夜いずれも）を選んだ場合に自動加算＋同意チェックボックス。23:59 締切。対応時間 翌日 04:00–08:00 / 20:00–24:00。
- **canonical Order Core 上の扱い（2026-09-17 owner decision）：** Express の +¥3,000 は **Product price へ混ぜない**。`order-level charge` として Product Line 群とは別に扱う（詳細・実装 §5.9・§8.3a）。

### 5.4 NG2 対応の料金 — 現行
- 3rd Delivery ¥1,000 / Domestic Shipping ¥1,000 / Disposal 無料。

### 5.5 Sakura Series — 価格方針確定・フロント未実装
- 商品は入荷済み（花きらり 桜 22.5cm 金/黒、18cm ピンク/ブルー）。刻印テスト完了。
- owner decision: Classic と同価格。Product pricing canonical は `price=3500` / `additional_price=2000` / `currency=JPY`。HP フロントへの追加と for kids（18cm）表示は未確定。

### 5.6 決済手段 — 現行
- クレジット/デビット: VISA / Mastercard / AMEX / JCB（Discover / Diners は Stripe 未サポートのため非表示）。Google Pay / Apple Pay 有効。現金不可・100% 事前決済。
- Stripe Checkout Session API（動的生成方式）。「価格ごとの固定決済 URL」方式は廃止済み。手数料 3.6%。
- 為替: frankfurter.dev で自動取得しドル・ユーロ表示。

### 5.7 価格変更時の同期先（ハードコード）
`js/products.json` / `js/forms.js` / `js/ui.js` / `js/firebase.js` / `index.html` / `js/translations.json` / `commerce.html`（`:root` の `--delivery-basic-jpy`）/ `tokyo-souvenir.html`。Stripe 決済リンクは価格変更で URL が変わらない（動的生成のため）。

### 5.8 実装境界
- Delivery Core PR #89（HEAD `75aa594aff71e56558bb3a98c67b4d735554384a`）で Series canonical と Product の `price` / `additional_price` / `currency` は実装済み（254/254 tests pass）。既存 Product の `additional_price` / `currency` 欠損は自動migrationしない。
- Admin Products `additional_price` UI、Storefront Product 価格表示、pricing calculation、Cart-level discount は未実装（pricing calculation・Cart は hp PR #73/#74 で表示・レビューまで実装済み。Order pricing snapshot は §5.9・PR #74。server-authoritative pricing の Order 接続は §8.3a・delivery-core PR #90）。

### 5.9 Server-authoritative pricing / Order pricing snapshot（2026-09-17 owner decision）
- **server-authoritative pricing を canonical Order で採用する。** クライアント側の合計金額表示は参考値であり、Order 確定金額の正本ではない。
- Order create 成功時、**server が確認済みの pricing snapshot を固定保存**する（`schema_version: 1`）。保存後は Cart/Product の以後の価格変更の影響を受けない。
- Express の +¥3,000 は `order-level charge` として Product Line 群と分離して保持する（§5.3）。
- **実装境界:**
  - Storefront 側（hp PR #74）: reviewed 済み Cart から `schema_version: 1` の pricing snapshot を都度再生成（キャッシュ・永続化しない、in-memory のみ）。Daily Use は `product_id` / `category_id` / 必須 `series_id` / Unit 単位の製作情報を保持し、pricing source を `canonical_product`（Product SSOT 準拠）と `legacy_daily_use` に区別する。Gift は `legacy_gift` pricing を使用し、`series_id` は canonical 識別子が無い場合 `null`（推測で埋めない）。Gift `additional_price` は生成しない（§5.2・§11 の未決定を尊重）。混載 Cart の snapshot 生成はブロック。Express 手数料は Line とは別に order-level charge として再計算。
  - Server 側（delivery-core PR #90）: `core_create_order` が現在の server Product `price` / `additional_price` から各 Product Line を再計算し、server-authoritative pricing を Line 単位・category 別 subtotal・JPY 通貨・Express order-level charge・配達日時とともに Order へ確定保存する。クライアント送信の合計額（`client_total`）は **一致検証のみ**に使用し、金額の正本としない。
  - どちらも production 未接続（storefront → `core_create_order` の配線は未実装。§3a.5・§8.3a 参照）。

---

## 6. Admin architecture

最終的に管理機能を3画面へ分離する。各画面の責務境界は以下で固定する。

### 6.1 admin-general.html — 全体・運行管理
- Global Status / Delivery Status
- seasonal effects（季節演出）/ ticker（電光掲示板）
- base delivery time / break time / base slot count
- 60-day calendar / daily slots
- **production capacity**（新規製作の作業時間。deliverySlots とは別概念）
- Save All（60日分一括保存。ループ対象は画面に見えている明日・明後日のみに厳格制限）

### 6.2 admin-delivery.html — 配送・注文管理（現場操作重視・スマホ/タブレット）
- Delivery List（`scheduledDateTime` をメイン表示。normal は現在日時のみ、redelivery / third_delivery は現在日時メイン + `Original: 初回日時` を小さく補助表示。NG1 / NG2 は次回配達日時未確定として識別）
- order detail / delivery execution
- NG1 / redelivery / NG2 / complete のアクション
- Google Maps / Admin Notes / 配送履歴 / 証拠写真

### 6.3 admin-products.html — 商品マスター管理（PC 編集重視）
- inventory（在庫）
- product info / product master（Series 追加・削除・並び替え、商品 CRUD、価格、表示/非表示、サイズ、対応文字種、series_tag、表示順）
- image / engraving（商品画像 Drag & Drop、Firebase Storage、Live Preview の刻印位置 GUI 調整、固定 Sample 文字、FV/Hero 画像管理）
- Spreadsheet 在庫同期（Firebase を正本、Spreadsheet は棚卸台帳として片方向: admin-products → Firebase → Spreadsheet）
- 詳細仕様: `admin-products.html_仕様書.md`（KEEP-ACTIVE）

### 6.4 責務境界（確定）
- **Extra Slots（ゲリラ枠）は削除。**
- **inventory は Products へ。**（旧 admin.html の在庫管理から移動）
- **Delivery List は Delivery へ。**
- **production capacity は General へ。**
- 「商品追加が配送ロジックを壊さない」「配送仕様変更が商品一覧を壊さない」「営業状態変更が商品マスターを壊さない」構造を維持する。

### 6.5 商品固有テキストの多言語
- Product Name / Description 等の商品固有テキストは `translations.json` に追加せず、Firebase 商品データ内に言語別（en/fr/de/it/es）で保存。EN が canonical。Auto Translate（登録・更新時のみ）＋手動修正＋Preview 言語切替。閲覧言語の翻訳がなければ英語 Fallback。

---

## 7. Customer-facing pages

| ページ | 役割 |
|---|---|
| `index.html`（+ `/fr` `/de` `/it` `/es` の各 `index.html`） | メインの商品選択・プレビュー・注文。多言語は物理ディレクトリ方式（`<html lang>` / canonical / `PAGE_LANG`）。共通 JS/CSS/JSON はルート `/js/` に集約し絶対パス参照 |
| `redelivery.html` | Free Redelivery 予約ポータル（NG1 顧客向け・URL 共有型）。`orderId` で顧客名を自動表示、10分待機・ペナルティ同意チェック必須、二重予約ロック。**Delivery System Rules の新モデルへ再構築対象**（`RENOVATION_EXECUTION.md`） |
| `ng2.html` | NG2 専用対応ページ。3rd Delivery / Domestic Shipping / Disposal をラジオボタンで1つ選択し、選択項目のみ詳細 UI を展開。Disposal は二重確認。3rd Delivery は選択可能時刻を `secondDeliveryDateTime+90分` でフィルタし、初回配達先住所を実データ表示、不在時廃棄への同意チェックボックス必須。Domestic Shipping は Japan-only / tracking email 通知 / carrier責任分界の同意チェックボックス必須（詳細: §4.6a）。バックエンド endpoint は埋め込まず in-browser 設定（Core API） |
| `success.html` | 決済完了ページ |
| `terms.html` / `commerce.html`（特商法）/ `privacy.html` | 法務3ページ。配送ポリシー文言は §4 と整合させる |
| `matsuya.html` | 松屋ガイド（無料公開コンテンツ・SEO） |
| `tokyo-souvenir.html` | AEO 特化型ランディングページ |
| `maintenance.html` / `404.html` | 補助ページ |
| Katakana preview（`index.html` 内 + `js/katakana-engine.js` / `js/katakana-widget.js` / `css/katakana-widget.css`） | 名前のカタカナ表記を即時プレビュー。閲覧言語（/fr /de 等）で発音を自動判別。EN/FR/DE/IT/ES 発音切替ウィジェット。入力・出力とも最大10文字 |

- URL 表記: Firebase Hosting の `cleanUrls: true` / `trailingSlash: false` に合わせ「.html なし・末尾スラッシュなし」に統一（法務3ページは Search Console 登録の都合で `.html` 付き）。
- 画像・リンクは常に絶対パス（`/` 始まり）。

---

## 8. Data / system architecture

### 8.1 Firebase Realtime Database
- **Blue（本番）:** project `chopstickers-workshop`、RTDB `https://chopstickers-workshop-default-rtdb.asia-southeast1.firebasedatabase.app`。
- **Green（Renewal 統合）:** project `chopstickers-project`、RTDB `https://chopstickers-project-default-rtdb.asia-southeast1.firebasedatabase.app`。Web App / Hosting / Auth / Storage、Green 専用 Sheet / Drive / Calendar を provision 済み。
- **主なノード（現行）:** `/status/*`（mainStatus / deliverySlots* / activeHours / usedMinutes / defaultCapacity / tickerMessage 等）、`/orders/`（個人情報。外部 read 遮断）、`/dailyData/`。
- **改修目標構成:** `/products`（商品マスター: product_id / category_id / series_id / name / name_i18n_key / size / price / additional_price / currency / available_characters / image_url / is_active / sort_order / stock_count）、Series（§5.0 の非価格フィールド）、`/orders`（注文・配送）、`/shopStatus` または既存 `status` 配下（営業・運行設定・deliverySlots）。既存構造を一度に全面置換しない。
- Green admin-products の RTDB は `/products` `/productSeries` `/heroImages`（ルール案: root `.read/.write=false`、この3ノードのみ許可。詳細: hp `public/docs/green-rtdb.rules.json`）。
- `production capacity`（usedMinutes）と `deliverySlots` は別概念。再配達・3rd Delivery で新規製作 capacity を再消費しない。

### 8.2 Firebase Storage
- Green admin-products の画像永続化。パス: `products/<product_id>/main.<ext>` / `box.<ext>` / `gallery/<photo_id>.<ext>` / `hero/<hero_id>.<ext>`。上限 8MB。RTDB には URL/メタデータのみ書き、画像バイトは入れない。詳細: hp `public/docs/green-storage-integration.md` / `green-storage.rules`。

### 8.3 GAS / Delivery Core
- **本番 GAS（現行 Blue）:** 注文処理・メール送信・Spreadsheet 記帳・Calendar 連携・Stripe Checkout Session 生成。`FIREBASE_SECRET` / Stripe Secret Key は Script Properties に格納（コード直書きは廃止）。ゲスト宛メール送信は独立 try-catch で隔離（決済 URL 消失リスク排除。この構造を壊さない）。
- **Delivery Core（独立リポジトリ `chopstickers2025/chopstickers-delivery-core`）:** 配送・注文管理ロジックをモジュール化したコアとアダプタ群 — `DeliveryCore.js` / `DeliveryCoreAdapter.js` / `DeliverySideEffects.js` / `PaymentAdapter.js` / `ProductCore.js` / `ProductCoreAdapter.js` / `ShopStatusCore.js` / `ShopStatusCoreAdapter.js` / `コード.js`（doPost エントリ）。各 `*.test.js` あり。`DeliverySideEffects.js` が NG2 時に顧客へ `https://chopstickers.jp/ng2.html?id=<orderId>` を送る。
- **Green GAS foundation:** delivery-core PR #57（`green-gas-foundation` @ `33475bb`）。fail-closed 実装、482 tests passed。service account 設定・clasp・deploy・外部書き込みは未実施。
- **共通 slot 予約処理（設計）:** `reserveDeliverySlot(...)` に集約 — 入力検証 → Script Lock → fresh read → 枠確認 → 前後±30分算出（日跨ぎ）→ atomic write → order 状態更新 → Lock release → Mail/Sheet/Calendar 副作用。副作用は slot 確保と分離し、失敗時に予約成立を壊さない。

### 8.3a canonical Order Core contract（2026-09-17 owner decision）
- 新 action **`core_create_order`**（`schema_version: 1`）を Green canonical Order 作成契約とする。
- **必須:** `idempotency_key`。
- **structured payload:** `lines` / `units` / `delivery` / `customer` / `accommodation` / `contact` / pricing snapshot（§5.9）。
- **Daily Use + Gift の混載 Order に対応する**（backend 契約として。storefront 接続は §3a.5 の通り未実装）。
- **旧 `core_new_order` は Blue legacy route として残すが、Green canonical contract には使用しない。** `core_new_order` 自体への変更は行わない。
- **実装境界:** delivery-core PR #90（`feature/canonical-order-contract-20260917`）。server Product `price`/`additional_price` から Line 単位で再計算（同一 Product のみ additional_price 適用）。Daily Use + Gift 混載対応。Gift は Product に `additional_price` が無い場合、数量2以上を拒否。`client_total` は一致検証のみに使用。ローカル mock テストのみ（16/16 pass）。production/backend 呼び出し 0 件。Stripe 連携・Storefront endpoint 接続は本 PR の範囲外。`main` 未merge・production 未deploy。

### 8.3b Reservation ownership（2026-09-17 owner decision）
- **Green server が以下を所有する（single source of truth）:** reserved stock / delivery slot / production capacity。
- Order 作成 / idempotency / stock / delivery slot / production capacity hold は **同じ reservation 境界（1つの atomic ロック付き更新）**で処理する。
- **Blue の `usedMinutes` との dual-write は禁止。** Green の Order 作成経路は Blue `usedMinutes` を読み取らず、同期も書き込みも行わない。
- **実装境界:** delivery-core PR #90。Green は `dailyOperations/{date}/productionReservedMinutes` と `dailyOperationDefaults/productionCapacityMinutes` を使用（既存 server 側 `productionMinutesFor` の計算ルールを再利用）。Order・idempotency mapping・reserved stock・delivery slot・Green production minutes を1つの locked update で書き込む。重複 retry は新規 hold を作らず既存 Order を返す。有効期限切れの決済 hold は Green production minutes を解放する。

### 8.4 Stripe
- Checkout Session API（動的生成）。手数料 3.6%。100% 事前決済。
- **Green は Stripe Test Mode** を使用し、本番決済へ接続しない。
- 既知課題: Stripe 画面を開いた段階（未決済のカゴ落ち）でも Firebase キャパ減算・カレンダー登録・シート記帳が走る。Stripe Webhook で「決済100%成功時のみトリガー」する要塞化は未着手（§11）。

### 8.5 Hosting
- Firebase Hosting（`chopstickers.jp`）。`cleanUrls: true` / `trailingSlash: false`。immutable JS/CSS キャッシュ（バージョン文字列 `?v=...` で世代管理。変更時はバンプ必須）。
- `.github/` などの隠しディレクトリは Hosting ignore 対象。

### 8.6 client / server 責務
- **client（`public/`）:** UI・多言語制御・プレビュー・入力バリデーション・金額計算・エリア照合（`js/delivery_areas.js` ホワイトリスト、zipcloud API は廃止）。`js/firebase.js` の構成情報はクライアント公開前提の値。
- **server（GAS / Delivery Core）:** 注文の正式化・slot atomic 予約・決済セッション生成・状態遷移・副作用（Mail/Sheet/Calendar）・RTDB 書き込み（マスターキー認証）。
- システム時刻は全域 JST 固定（`firebase.js` の日時取得を Asia/Tokyo 固定）。GAS はフロントから渡る絶対日付を信頼（「0〜5時の前日扱い」は廃止、0:00〜0:10 グレースピリオド補正のみ）。

### 8.7 Blue vs Green
- **Blue** = 現行本番 `chopstickers.jp`（project `chopstickers-workshop`）。Renewal 完成・E2E 完了・cutover まで**維持**する。
- **Green** = 非公開の Renewal 統合環境（project `chopstickers-project`）。完成した部品を順次持ち込み、接続のたびに実動テスト。
- Green へ統合する際、**Blue の本番設定（Firebase config / RTDB URL / GAS endpoint / Stripe 本番キー）をそのまま持ち込まない。** production-derived な接続は Green-safe 化 / quarantine する。
- 最終アクションは「公開先を完成済み Green へ切り替える（cutover）」。本番を現在地で作り替えない。cutover 後も rollback 経路を維持。
- **capacity cutover 後の正本（2026-09-17 owner decision）：** production capacity カウンターは cutover 完了後、**Green counter のみを正本**とする。Blue `usedMinutes` との**恒久的な sync・compatibility counter は禁止**（§8.3b の dual-write 禁止と同じ原則を cutover 後も維持）。cutover 手順そのものは `RENOVATION_EXECUTION.md` §4.12。

---

## 9. Security / production boundaries

### 9.1 public / admin 分離
- 顧客向けページと管理画面（`admin*.html`）を分離。管理画面は Firebase Authentication（Google アカウントログイン）で `contact@chopstickers.jp`（`emailVerified`）のみ読み書き可。PIN コード方式は廃止済み。
- `/orders` の外部 read は遮断。`orderId` のみで個人情報へアクセスできないか（token 方式の要否）は本番前に確認。

### 9.2 secrets
- API キー・パスワード・アクセストークン・秘密鍵・サービスアカウント鍵を生成・出力・保存しない。
- `FIREBASE_SECRET` / Stripe Secret Key は GAS Script Properties。実際の値を要求・表示しない。
- Firebase Web config の `apiKey` はクライアント識別子（秘密鍵ではない）。アクセス制御は Auth 承認済みドメイン + Security Rules で行う。
- コード内に秘密情報を発見した場合、値を出力せず存在のみ報告する。

### 9.3 production boundaries（越えるには明示的 owner 承認が必要）
- `main` へのマージ
- 本番デプロイ（Firebase Hosting deploy / `clasp push` / `clasp deploy`）
- 本番 Firebase Security Rules の変更 / 本番 RTDB への書き込み / 本番データ削除
- Stripe 本番アクション
- 本番 GAS / Hosting の操作
- DNS / ドメイン変更
- これらに到達したら停止し、必要な承認を具体的に述べる（`RENOVATION_EXECUTION.md` §9、`AUTOMATION_CONTRACT.md` §7）。

### 9.4 Temporary production mode — Blue maintenance mode / Coming Soon frontend（2026-09-17）

- **現在の production 状態（owner による手動切替・2026-09-17）：** owner が既存 **Blue サイトをメンテナンス＋配達受付停止モードへ手動切替**した。新規注文受付は停止中。現在 chopstickers.jp（EN/DE/ES/FR/IT）に表示されている "major renewal is coming" / "Ordering is temporarily unavailable" の案内は、**この Blue 既存メンテナンスモードの手動切替によるもの**であり、下記 PR #75（Coming Soon frontend）の deploy によるものではない。PR #75 は下記の通り未deploy。
- **PR #75（Coming Soon frontend）の方針:** owner 承認済みの実装だが、**現時点では本番投入しない**（`main` 未merge・production 未deploy・deploy自体を見送り。上記の現行 Blue メンテナンスモードと役割が重複するため）。将来 deploy する場合に備え、以下は実装内容の記録として残す。
  - **想定される「残す」対象:** FV / Katakana Converter / Engraving Preview / Product Lineup（read-only）/ Brand・Service information / Legal / Contact。
  - **想定される「停止」対象:** Order UI / Delivery slot selection / Customer order form / Checkout / Stripe order flow / GAS order flow。
  - **EN / DE / ES / FR / IT すべて同じ扱い**（注文導線停止を全言語で統一する設計）。
  - **旧 Blue 注文コードは削除せず保持する。** frontend 表示・到達可否のみを変更し、production Firebase / GAS / Stripe / DNS / Admin は変更しない設計。
  - deploy された場合は **temporary production mode** として扱い、Renewal 完成後に Green Storefront（§3a）へ置換される。
- **実装境界:** hp PR #75（`renewal/coming-soon-mode-20260917`、base `reconciliation/blue-production-sync-20260914`、**state: OPEN**）。`window.RENEWAL_COMING_SOON` フラグで order-entry point（Classic/Gift select・sticky CTA・FV preview-modal Order・Lineup product-modal action・homepage delivery calendar）と `window.enterFocusMode()` をガード実装済み。`#delivery-section` は非表示。GAS/Stripe 通信 0 件をローカル + preview channel で確認済み。**production 未deploy・deploy 予定なし（現状の owner 方針）。** 方針変更時は、本節と `RENOVATION_EXECUTION.md` §4.13 の status を更新する。

---

## 10. Canonical business invariants

AI・実装者が **owner の明示的決定なしに変更してはならない** ルール（短縮版。詳細は §4）:

1. 現行事業は Chopstickers Delivery。Workshop は終了済みで現行仕様ではない。
2. Delivery window は顧客予約 20:00–08:00（30分刻み・最終枠 08:00）、運営 20:00–08:30。
3. 配達枠は1つ。全種別が同じ `deliverySlots` を共有し、予約は選択時刻 ± 30分（±1枠）を占有する。日跨ぎ同ルール。先着順。
4. Free Redelivery は NG1 のみ・1回無料・初回住所固定・最短 `originalDeliveryDateTime + 90分`・確定後変更不可。
5. NG2 は無料再配達を提供せず、`ng2.html` で 3rd Delivery ¥1,000 / Domestic Shipping ¥1,000 / Disposal 無料 の3択。
6. 3rd Delivery の最短時刻は `secondDeliveryDateTime + 90分` のみ。`originalDeliveryDateTime` fallback 禁止。legacy 欠損は fail closed。
7. status は `pending / NG1 / NG2 / delivered / shipping / disposed`。deliveryStage は `normal / redelivery / third_delivery`。両者の役割を混同しない。
8. 返金は不在では不可。製造欠陥・刻印ミスのみ再送 or 全額返金。
9. 保管期限 = 初回配達予定日 + 7 暦日 23:59 JST。曖昧表現を使わない。
10. Classic の刻印は英数字・ひらがな・カタカナのみ・最大10文字。Gift（桐箱）は英語のみ。
11. 現行価格: Classic ¥3,500（+¥2,000/膳・最大5膳）、Engimon ¥5,800、Happy Life ¥7,800、Express +¥3,000。
12. admin 責務境界: General = 運行・capacity・カレンダー、Delivery = Delivery List・配達実行・NG フロー、Products = 在庫・商品マスター・画像。Extra Slots は削除。
13. Blue 本番は cutover まで維持。Green へ Blue 本番設定を持ち込まない。production-derived 接続は quarantine する。
14. GitHub が canonical。production boundary（§9.3）を越える操作は owner 承認必須。
15. コピーライティングは Honest & Minimal。誇張表現を使わない。
16. Storefront 入口は単一 Order Builder（上部常設 `[ Daily Use ] [ Gift ]`）。専用 Select Category 画面は廃止（§3a）。
17. Cart は Daily Use / Gift 混載表示可能・共通 canonical。Product identity は `product_id`。全 Line review gate なしに checkout 不可（§3a.4）。
18. 価格 SSOT は Product の `price` / `additional_price` / `currency`。Order 確定金額は server-authoritative。Order create 成功時の pricing snapshot（`schema_version: 1`）は以後の価格変更の影響を受けない（§5.0・§5.9）。
19. Green の Order 作成は `idempotency_key` 必須の同一 reservation 境界で stock / delivery slot / production capacity を処理する。Blue `usedMinutes` との dual-write・恒久 sync counter は禁止（§8.3b・§8.7）。
20. Renewal 完成まで production は新規注文受付停止状態を維持する。現行は既存 Blue のメンテナンスモード（owner 手動切替）によるものであり、PR #75（Coming Soon frontend）は本番投入しない。いずれの場合も旧 Blue 注文コードは削除しない（§9.4）。

---

## 11. Known undecided items

真に未確定で、owner decision が必要 / 保留中のもの（既に決定済みのものをここに戻さない）:

| 項目 | 状態 |
|---|---|
| 3rd Delivery の「決済中の枠確保タイミング」（決済前の短時間 hold / 決済成功時 atomic 確保 / 既存 Stripe フロー活用 のどれか） | 未確定。Phase 1-A〜1-C で現行 Stripe/GAS を監査してから確定。確定前に 3rd Delivery 決済コードを実装しない（`再配達システム改修_設計書.md` §12） |
| Sakura Series のフロント追加・for kids（18cm）表示 | 未確定。価格方針は Classic と同額で確定済み（§5.5） |
| Gift Product の `additional_price` | 未決定。Engimon / Happy Life の通常価格は確定済み（§5.2） |
| 配達エリア拡大（港区・渋谷区・新宿区の段階的緩和） | 未確定（新橋周辺は厳格除外を維持） |
| Stripe Webhook 連携による在庫・記帳の完全自動化 | 未着手課題 |
| 複数種同時購入（"+ Add a Gift Box"） | 未着手課題 |
| 企業・ノベルティ向けロゴ刻印 | 課題化のみ（K8 精度テスト未実施） |
| Workshop の再開（VIP・法人向け高単価モデル） | 将来検討。現行仕様ではない |
| `redelivery.html` の最終ファイル名・ルーティング | 実装開始時に既存 URL を確認して確定（現行踏襲の想定） |

---

*本ファイルは事業・システムの「あるべき姿」を定義する。改修の現在地・順序・作業ルールは `RENOVATION_EXECUTION.md` を参照。*
