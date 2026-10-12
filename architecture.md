# CALMO 會員儲值 × 店員點餐系統 — 架構方案
> 編製：HALOS 智日科技 ｜ 2026-10-12 ｜ 版本 v0.2（已加入客戶確認的麵包＋飲品半價規則、日結／月結報表、PWA 部署）（供 Michael 與 Claude Code 開發使用）
> 標示「【示例】」者為假設數據，須 CALMO 確認；標示「【待核實】」者為未經證實的外部資料。

---

## 0. 現況觀察（來自店內照片）

| 觀察 | 對系統設計的啟示 |
|---|---|
| 店員目前用 iPad 上的「微信小程序類」點餐系統（簡體介面、貨幣顯示「¥」、綠色主題），有「流水號快速開單／選擇桌號（1–5 號，2人桌）」、「訂單列表：商家備餐中／未支付」 | 需保留 **流水號開單 + 桌號** 兩種模式；介面改為繁體、貨幣統一 MOP |
| 現有系統分類：半價飲品、拿鐵系列、Open Sourdough Bread、EXCLUSIVE、經典咖啡 | 現系統以獨立「半價飲品」分類人手處理（CALMO ¥21.50 = 43÷2、焙雪 ¥22.50 = 45÷2），且**加配不會跟隨半價**；新系統改為自動促銷規則，飲品連加配一併半價 |
| 現有選項：濃縮咖啡「soe +¥3」；拿鐵（熱/凍）基價 33、「凍 +¥5」、「咖啡豆 soe +3」、「奶：米奶 +3／燕麥奶 +3」 | 冷熱以 **價格差 modifier** 處理（熱 33 → 凍 38 剛好 +5），與實體餐牌一致 |
| 酸種麵包「楓糖炒滑蛋」「藍莓酸种包」有「選項」按鈕 | 麵包類需「麵包底」選項組 |
| 照片中沒有套餐；**客戶已確認**：買一件開放式酸種麵包 ＋ 一杯飲品 → 飲品（連加配）半價 | 以「促銷規則」實作（見 §12），不建立固定套餐品項 |
| 視覺：米白紙張、黑色細體寬字距「C A L M O」、淺灰大型手寫「C」水印、淺木框 | demo 已採用此風格 |

---

## 1. 餐牌轉錄（MOP，忠實轉錄自 IMG_7311–7313）

### 經典咖啡 Coffee Classic（熱 / 凍）
濃縮咖啡 Espresso 23 / — ｜美式咖啡 Americano 28 / 32 ｜招牌冰滴 Signature Ice Drip — / 39 ｜拿鐵 Latte 33 / 38 ｜澳白 Flat White 38 / — ｜摩卡 Mocha 33 / 38 ｜卡布奇諾 Cappuccino 35 / —

### 拿鐵系列 Latte Selections
短笛 Piccolo Latte 36 / — ｜米乳拿鐵 Rice Milk Latte 36 / 41 ｜榛子拿鐵 Hazelnut 38 / 43 ｜海鹽焦糖拿鐵 Salty Caramel 38 / 43 ｜玫瑰拿鐵 Blush Rose 38 / 43 ｜香草拿鐵 Vanilla 38 / 43 ｜黑糖拿鐵 Brown Sugar 38 / 43

### 特調咖啡 Special Coffee
蘋果美式 Apple Americano — / 40 ｜香橙美式 Orange Americano — / 40 ｜檸檬美式 Lemon Americano — / 40 ｜流沙美式 Creamy Americano — / 40 ｜維也納 Viennese Coffee — / 43 ｜米奶髒咖啡 Rice Milk Dirty 36 / —

### 冷萃系列 Cold Brew Selections
冷萃咖啡 082 Cold Brew 082 — 42（風味：玫瑰花香、百香果、芒果、熱帶水果、水蜜桃、藍莓、荔枝）
冷萃咖啡 083 Cold Brew 083 — 42（風味：烏梅、花香、果汁、蕃茄）

**咖啡類餐牌註**：默認中深烘豆，如需淺烘 SOE +$3；可轉其他奶類（燕麥奶、米奶）+$3；可轉湯力 Tonic +$3。

### 獨家飲品 Exclusive Drinks（只供凍）
卡慕 CALMO 43（焦糖、咖啡、米奶）｜焙雪 Toasted Snow 45（茶香味冰滴、黑糖奶油）｜雲滴 Cloud Drip 43（茶香味冰滴、奶油牛奶）｜霧抹 Matcha Mist 43（蘋果汁、抹茶奶蓋）｜茉冰 Helena Sparkle 45（蘋果汁、茶感梳打）

### 無咖啡飲品 Non-Coffee（熱 / 凍）
蘋果梳打 — / 38 ｜柚子梳打 — / 38 ｜檸檬梳打 — / 38 ｜香橙梳打 — / 38 ｜焙茶鮮奶 Hojicha Latte 39 / 45 ｜焙茶朱古力 Hojicha Chocolate 39 / 45 ｜朱古力鮮奶 Chocolate Milk 38 / 42 ｜抹茶鮮奶 Matcha Latte 39 / 45 ｜抹茶湯力 Matcha Tonic — / 42 ｜草莓抹茶鮮奶 Strawberry Matcha Fresh Milk 42 / 48
**註**：可轉其他奶類（燕麥奶、米奶）+$3；可轉湯力 Tonic +$3。

### 開放式酸種麵包 Open Sourdough Bread
炒滑蛋 32 ｜牛油果炒滑蛋 38 ｜楓糖炒滑蛋 38 ｜藍莓蜜糖奶酪 Blueberry Honey Cream Cheese 38 ｜黑松露炒滑蛋 40 ｜煙肉炒滑蛋 40 ｜燒奶酪芝士 Roasted Cream Cheese 40 ｜牛油果青醬 Avocado Pesto 40 ｜蘑菇炒滑蛋火箭菜 45 ｜意式火腿 Ricotta Prosciutto 45 ｜黑松露蘑菇芝士 45 ｜奶油肉蘑菇芝士 Creamy Mushroom & Cheese 45 ｜蝦仁炒滑蛋 45 ｜牛油果三文魚 Avocado & Salmon 55
**註**：可免費轉香濃厚方包，或 +$5 升級Q彈米麵包。

### 甜品 Dessert
焦糖熔岩芝士蛋糕 38 ｜原味巴斯克 34 ｜開心果巴斯克 39 ｜朱古力巴斯克 42 ｜合桃朱古力曲奇 18

### 轉錄不確定之處
- 部分中文小字因反光／解像度略模糊：「奶油肉蘑菇芝士」英文為 Creamy Mushroom & Cheese，中文「肉」字存疑；「意式火腿」對應英文 Ricotta Prosciutto。
- 「轉湯力 Tonic」是否適用所有咖啡（含奶咖）未明，demo 暫放於咖啡及無咖啡類。
- SOE 是否適用冷萃（冷萃本身是指定豆）— demo 不提供。
- 現有 POS「半價飲品」的適用時段／條件未知。
- 「藍莓酸种包」為現有 POS 名稱，餐牌為「藍莓蜜糖奶酪」，以餐牌為準。

---

## 2. 系統模組

```
┌─────────── 前台 (iPad PWA, 橫向) ───────────┐   ┌──── 後台 (Web) ────┐
│ 點餐 POS │ 會員查詢/充值 │ 訂單/出品(KDS) │   │ 餐牌/選項/套餐管理 │
└─────┬─────────────┬───────────────┬──────┘   │ 會員/儲值/報表    │
      │             │               │          │ 員工/權限/交班    │
┌─────▼─────────────▼───────────────▼──────────┴─────────┬──────────┘
│            核心服務（Next.js API / Supabase RPC）        │
│ Catalog │ Pricing Engine │ Orders │ Wallet Ledger │ Members │ Promotions │
└───────┬───────────────────────────────┬────────────────┘
        │ Payment Adapter               │ Channel Adapter
  現金／銀行卡終端／MPay、BOC Pay…       澳覓 Aomi／mFood…（Phase 3）
```

1. **Catalog 餐牌**：分類、品項、選項組（modifier groups）、套餐、售罄、時段。
2. **Pricing Engine 計價引擎**（純函數，前後端共用，必須有單元測試）：基價 + 選項差價 + 促銷（麵包＋飲品連加配半價，§12）+ 會員折扣 → 明細與總額。**伺服器端重新計價**，不信任前端金額。
3. **Orders 訂單**：開單（流水號／桌號／外賣自取）、狀態機（draft → placed → preparing → ready → completed / voided）、退單需權限。
4. **Members 會員**：以澳門手機號（+853 8 位）為主鍵，等級、標籤、同意條款紀錄。
5. **Wallet 儲值錢包**：**只追加（append-only）的雙欄流水帳**（本金 principal / 贈送 bonus 分開），餘額由流水推導；消費扣款先扣哪一欄須訂政策（建議先扣贈送或按比例，影響退款）。
6. **Promotions**：充值贈送方案、半價飲品、套餐優惠。
7. **KDS / 出品**：iPad 或第二屏；Supabase Realtime 推送；之後可接熱敏打印機（Epson TM-m30 / Star mC-Print，ePOS/WebPRNT 經瀏覽器直接打印）。
8. **報表與交班**：日結、付款方式分佈、儲值負債（未消費餘額）、充值/消費對帳。
9. **Adapters**：Payment、Delivery Channel、Printer —— 統一介面，按供應商實作。

---

## 3. 資料模型（Postgres）

```sql
-- 餐牌
categories(id, name_zh, name_en, sort, is_active)
items(id, category_id, name_zh, name_en, description, base_price numeric(10,2),
      image_url, is_active, sold_out, sort, tags text[])
modifier_groups(id, name_zh, min_select int, max_select int, sort)   -- min>=1 即必選
modifier_options(id, group_id, name_zh, price_delta numeric(10,2), is_default bool, sort, is_active)
item_modifier_groups(item_id, group_id, sort, override_min, override_max)
item_option_overrides(item_id, option_id, price_delta, is_available)   -- 例：濃縮無凍；某品項凍差價非 +5
--   冷熱方案：temp 作為必選 group，option「凍」price_delta 按品項覆寫（多數 +5，美式 +4、朱古力鮮奶 +4、焙茶 +6）

-- 促銷（取代固定套餐；詳見 §12）
promotions(id, code -- 'BREAD_DRINK_HALF', name_zh, type -- 'pair_discount', trigger_category_ids uuid[],
           target_category_ids uuid[], pct numeric -- 50, ratio int -- 1 (每件觸發品配 1 件), rounding_cents int -- 10,
           include_modifiers bool -- true, valid_from, valid_to, channels text[] -- 'pos','qr'（外賣待確認）, is_active)
order_line_discounts(id, order_line_id, promotion_id, units int, amount numeric(10,2),
           paired_line_id -- 配對的麵包行, overridden_by_staff_id NULL)
-- 店員關閉某行半價：order_lines.promo_opt_out bool + audit_log

-- 會員與儲值
members(id uuid, phone_e164 unique, name, tier, birthday, marketing_opt_in, created_at)
wallet_accounts(id, member_id unique, status)   -- 不存餘額或只作快取
wallet_entries(id bigserial, account_id, type -- 'topup'|'bonus'|'spend'|'refund'|'adjust'|'expire'
       , principal_delta numeric, bonus_delta numeric, order_id, topup_id,
       staff_id, idempotency_key unique, created_at)          -- 不可 UPDATE/DELETE（RLS + trigger）
topups(id, member_id, plan_id, paid_amount, bonus_amount, payment_id, created_at)
topup_plans(id, pay_amount, bonus_amount, valid_from, valid_to)   -- 【示例】300送20/500送50/1000送120/2000送300
-- 餘額 = SUM(principal_delta)+SUM(bonus_delta)，扣款於單一 transaction 內 SELECT … FOR UPDATE

-- 訂單
orders(id, store_id, seq_no, table_no, channel -- 'pos'|'aomi'|'mfood', type -- dine|takeaway|delivery,
       status, member_id, subtotal, discount_total, total, created_by, created_at, external_ref)
order_lines(id, order_id, item_id, promo_opt_out bool, name_snapshot, qty,
            unit_price, line_total, notes)                 -- 套餐子項以 parent_line_id 掛載
order_line_modifiers(line_id, option_id, name_snapshot, price_delta)
payments(id, order_id, method -- cash|card|wallet|mpay|bocpay|…, amount, status,
         provider, provider_ref, idempotency_key, created_at)
staff(id, name, pin_hash, role -- staff|manager|owner), audit_log(...)
```
要點：**名稱與價格快照**寫入訂單行（改餐牌不影響歷史）；金額一律 `numeric(10,2)`，不要用浮點；所有付款／扣款有 `idempotency_key` 防重複扣款。

---

## 4. 技術棧建議（適合 Claude Code 建置）

| 層 | 建議 | 原因 |
|---|---|---|
| 前端 | **Next.js 15 (App Router) + TypeScript + Tailwind + shadcn/ui**，PWA（加到 iPad 主畫面全螢幕） | Claude Code 最熟悉；一個 repo 搞掂前後台 |
| 狀態 | Zustand（購物車）+ TanStack Query | 簡單 |
| 後端/資料庫 | **Supabase**（Postgres + Auth + Realtime + RLS + Edge Functions），區域選 **Singapore/Tokyo** | 出品屏即時推送；RLS 保護流水帳 |
| 計價 | `packages/pricing` 純 TS 函數 + **Vitest** 測試 | 前端即時顯示、伺服器重算同一套邏輯 |
| 離線 | IndexedDB 暫存訂單佇列（Phase 2）；**儲值扣款必須在線**（避免透支／雙花） | |
| 部署 | Vercel（或 Cloudflare）；Sentry 監控 | |
| 打印 | Epson ePOS SDK for JavaScript / Star WebPRNT（區網） | 無需原生 App |
| 測試 | Playwright E2E（點餐→儲值支付→出品） | |

替代：若要極低維護，可考慮直接採用現成 POS（見 §8）+ 只自建會員儲值。這是值得向 CALMO 提出的「買 vs 建」選項。

---

## 5. 儲值合規重點（澳門）— ⚠️ 非法律意見，須律師／澳門金融管理局 (AMCM) 確認

- **第13/2023號法律《金融體系法律制度》**（2023-11-01 生效）第二條把「**發行及管理支付工具，尤其是…電子儲值支付工具**」列為金融業務，**只有獲許可的金融機構才可經營**。來源：<https://bo.dsaj.gov.mo/bo/i/2023/33/lei13_cn.asp>
- 法律本文未見「單一商戶／封閉式預付卡」豁免條款【待核實】。實務上咖啡店、餅店的「店內儲值／會員卡」在澳門普遍存在，一般被視為只能在發行商戶本身消費的**預付消費（閉環）**，與 MPay 這類可在多商戶使用的電子儲值支付工具不同；但這個界線**沒有找到 AMCM 的明文指引**，須確認。可查 AMCM 指引頁：<https://www.amcm.gov.mo/en/other-institution/other-institution-regulatory-guidelines?type=rules-and-guidelines>
- **降低風險的設計（建議一律採用）**：
  1. **閉環**：只可在 CALMO（同一法人旗下門店）消費；**不可提現、不可轉賬給他人、不可在第三方使用**。
  2. **非生息、無利息**；贈送額以「優惠」名義，與本金分帳。
  3. 設合理**上限**（例如單一會員餘額上限 MOP 3,000【示例】）。
  4. 清楚的**條款**：有效期、退款政策（建議只退未使用本金）、結業／轉手處理；顧客書面／電子同意並存檔。
  5. **消費者保障**：澳門《消費者權益保護法》（第9/2021號法律）對預付消費有規定【待核實具體條文】，消委會曾多次提醒預付消費風險 → 保持條款透明、可隨時查詢餘額與流水。
  6. **個人資料**：第8/2005號法律《個人資料保護法》— 收集電話／生日須告知目的，取得同意；如資料庫放境外（Supabase 新加坡）屬資料跨境轉移，須評估／按個人資料保護局 (DSPDP) 要求處理【待核實】。
  7. 會計上未消費餘額為**負債**（合約負債），系統須能出「儲值負債報表」。

---

## 6. 支付與外賣平台整合（Adapter 層）

```ts
interface PaymentProvider {
  id: 'cash'|'wallet'|'card_terminal'|'mpay'|'bocpay'|'aggregator'
  createPayment(o:{orderId, amount, idempotencyKey}): Promise<{status:'pending'|'succeeded'|'failed', ref?:string, qr?:string}>
  queryPayment(ref): Promise<Status>; refund(ref, amount): Promise<Status>
}
interface DeliveryChannel {
  id:'aomi'|'mfood'|...
  pushMenu(catalog): Promise<void>; setItemAvailability(itemId, available): Promise<void>
  onOrderWebhook(payload) -> NormalizedOrder; acceptOrder(ref); rejectOrder(ref, reason); markReady(ref)
}
```

**澳門支付格局（【待核實】，請向收單銀行查詢最新）**
- **聚合支付（「澳門聚合支付」/ 簡單聚合碼 Simple Pay）**：澳門金管局推動，一個碼／一部終端可收 MPay、BOC Pay、中銀/工銀/澳門通、支付寶（澳門）、微信支付等。商戶一般經**收單銀行（中國銀行澳門、工銀澳門、大豐、澳門通等）**申請。
- **MPay（澳門通旗下）**、**BOC Pay（中銀澳門）**、**澳門通卡（Macau Pass）**、銀聯／Visa／Master 實體卡 → 多數經銀行提供的**智能 POS 終端**處理。
- **整合路徑**（由易至難）：
  1. **Phase 1「半整合」**：系統記錄付款方式，金額由店員在獨立終端輸入；日結時對帳。零技術風險。
  2. **Phase 2 終端聯動**：向收單銀行查詢其智能 POS（多為 Android，如 PAX / Sunmi）是否開放 **ECR / 本地 API / App-to-App intent**；或銀行的線上聚合支付 API（動態 QR + webhook）。
  3. 顧客自助：門店 QR 點餐 + 線上支付（需商戶線上收單協議）。

**外賣平台（【待核實】）**
- 澳門主要平台：**澳覓 Aomi**、**mFood（澳門通旗下）**。上述平台**公開商戶 API 文件很少**，多數需商務洽談或透過其合作 POS／平板接單。
- Phase 3 方案：先用平台商家平板接單 + 手動入單（或平台打印單）；同步洽談 API；如不開放，可評估 **聚合接單 SaaS** 是否支援澳門平台。
- 外賣價可能不同（平台抽佣）→ `channel_prices` 表以渠道設定價格。

---

## 7. 分階段路線圖

| 階段 | 時長（估） | 範圍 | 驗收 |
|---|---|---|---|
| **MVP（Phase 1）** | 3–5 週 | iPad 點餐（全餐牌、選項組、冷熱差價、麵包底、麵包＋飲品半價促銷）；流水號／桌號；出品屏；會員（電話）；儲值充值+贈送+儲值支付；現金/卡/MPay 記錄（手動終端）；日結對數報表、月結儲值負債表；餐牌後台；員工 PIN | 與現行系統並行 1 週，對帳無差異 |
| **Phase 2** | 4–6 週 | 熱敏打印（廚房單/收據）；離線佇列；會員等級、生日禮、積分；促銷時段；退款／作廢審批；收單銀行終端聯動或聚合支付 API；會員自助查餘額（WhatsApp/網頁 OTP） | 收款自動回寫、無手輸 |
| **Phase 3** | 視平台 | 澳覓 / mFood 接單整合、餐牌同步、渠道價；顧客 QR 自助點餐；多門店；會計匯出 | 外賣單自動入 KDS |

---

## 8. 參考產品

| 產品 | 參考點 | 連結 |
|---|---|---|
| Square for Restaurants | Modifier 設計、KDS、禮品卡/儲值 | <https://squareup.com/us/en/point-of-sale/restaurants> |
| Loyverse | 免費 POS、modifiers、會員積分 | <https://loyverse.com> |
| Eats365（香港） | 港澳在地化、套餐、外賣平台整合 | <https://www.eats365pos.com/hk/> |
| iCHEF（台灣） | 咖啡店流程、儲值金、繁中介面 | <https://www.ichefpos.com> |
| Toast | 餐廳 POS、Menu/Modifier API 文件 | <https://doc.toasttab.com> |
| Square Catalog API | ModifierList / ItemVariation 資料模型可直接借鏡 | <https://developer.squareup.com/docs/catalog-api/what-it-does> |

---

## 9. 風險

1. **儲值監管**（§5）—最高優先，開發前先拿法律意見；設計保持閉環。
2. **錢包一致性**：雙重扣款／併發 → DB transaction + row lock + idempotency key + 只追加流水。
3. **網絡中斷**：點餐可離線、儲值不可 → 顯示清楚狀態；備用 4G 熱點。
4. **支付/外賣 API 可得性**：澳門本地 API 文件不公開，可能只能半整合 → 預期管理。
5. **餐牌複雜度**：冷熱差價不一致（+4/+5/+6）、只熱/只凍、SOE 適用範圍 → 用 override 表 + 測試覆蓋所有品項。
6. **資料遷移**：現有系統的會員／餘額（如有）需匯入並由店主簽核。
7. **iPad Safari PWA 限制**：Web Push、背景同步有限；打印需區網設備。
8. **個資跨境**（Supabase 境外區域）。
9. **單人開發／AI 生成代碼品質** → 強制測試、Code review、Staging 環境。

---

## 10. 給 Claude Code 的 CLAUDE.md（直接貼到 repo 根目錄）

````markdown
# CLAUDE.md — CALMO POS & Wallet

## Project
Staff-facing ordering (iPad landscape PWA) + member stored-value wallet for Café CALMO, Macau.
Built by HALOS 智日科技. UI language: Traditional Chinese (Macau usage, e.g. 朱古力, 外賣自取, 澳白). Currency: MOP, 2 decimals.

## Stack
- Next.js 15 App Router, TypeScript strict, Tailwind, shadcn/ui, Zustand, TanStack Query
- Supabase (Postgres, Auth, Realtime, RLS, Edge Functions); migrations in `supabase/migrations`
- Vitest (unit), Playwright (e2e). pnpm workspaces: `apps/web`, `packages/pricing`, `packages/adapters`

## Non-negotiable rules
1. Money = `numeric(10,2)` in DB, integer cents in TS (`type Cents = number`). Never floats.
2. `packages/pricing` is pure and shared; the server ALWAYS re-prices orders. Every menu item must have a pricing test.
3. Wallet is an append-only ledger (`wallet_entries`, principal & bonus columns). No UPDATE/DELETE (enforce with RLS + trigger). Balance = SUM. Spend happens in one SQL function `wallet_spend(account, cents, order_id, idempotency_key)` using `FOR UPDATE`; reject if insufficient. Wallet payments require online connection.
4. Wallet is closed-loop: no cash-out, no P2P transfer, balance cap configurable. Refunds return principal only (configurable).
5. Order lines snapshot names and prices. Promo discounts stored per line in `order_line_discounts`.
6. All payment/integration calls go through adapter interfaces in `packages/adapters` (PaymentProvider, DeliveryChannel, Printer). Phase 1 implements `cash`, `wallet`, `manual_terminal` only.
7. Every mutation that moves money writes `audit_log` with staff_id.
8. Phone numbers stored E.164; Macau default +853, 8 digits.

## Domain
- Modifier groups: min/max select (min>=1 → 必選). Temperature (熱/凍) is a required group; 凍 price delta overridden per item (most +5; Americano +4; Chocolate Milk +4; Hojicha/Matcha +6; Strawberry Matcha +6). Items with only one temperature expose only that option.
- Coffee add-ons: 淺烘 SOE +3; 燕麥奶/米奶 +3 (milk drinks); 轉湯力 Tonic +3. Bread base: 酸種(default) / 厚方包 +0 / Q彈米麵包 +5.
- Promotion BREAD_DRINK_HALF (client-confirmed): each Open Sourdough bread unit unlocks 50% off ONE drink unit, drink price INCLUDING modifiers. Auto-pair to the most expensive eligible drink units; staff can opt a line out (`promo_opt_out`), then next eligible drink is used. Drink categories = classic, latte, special, cold, excl, non (待確認). Round each discounted unit to MOP 0.1 (待確認). No fixed combos.
- Wallet spend is split principal:bonus proportionally to current balances; principal part = revenue, bonus part = discount. Reports: daily reconciliation & monthly liability (see docs/architecture.md §13).
- Seed menu: see `supabase/seed/menu.sql` generated from docs/architecture.md §1.

## Commands
pnpm dev | pnpm test | pnpm e2e | supabase db reset

## Workflow
Small PRs; write tests first for pricing & wallet; run `pnpm test` before finishing; update this file when conventions change.
Reference UI: `docs/demo.html` (visual style: off-white paper #F1EDE6, ink #2A2826, wood #D8C8B0, letter-spaced "C A L M O").
````

### Kickoff prompt（第一次對 Claude Code 說的話）

```
閱讀 CLAUDE.md、docs/architecture.md 與 docs/demo.html。
任務：建立 Phase 1 MVP 骨架。
1) 初始化 pnpm monorepo（apps/web Next.js 15、packages/pricing、packages/adapters），設定 Tailwind、shadcn/ui、Vitest、Playwright。
2) 撰寫 Supabase migration：§3 所有表、RLS、wallet_entries 禁止 UPDATE/DELETE 的 trigger、wallet_spend / wallet_topup SQL function（含 idempotency）。
3) 依 §1 生成 seed（全部品項、冷熱價格、選項組與 override）。
4) 實作 packages/pricing 及測試：每個品項熱/凍價格與餐牌一致、SOE/換奶/Tonic/麵包加價、§12 麵包＋飲品半價全部測試案例。
6) 之後：§13 日結／月結報表 SQL view 與頁面，並以 §13.5 對數不變式寫測試。
5) iPad 橫向點餐頁（分類 / 品項格 / 選項彈窗 / 購物車），樣式參考 demo.html。
先列出計劃與檔案結構給我確認，再開始寫碼。每完成一步執行測試。
```

---

## 11. Demo 說明
`demo.html`（單一檔案，無外部依賴）：點餐（分類、搜尋、選項彈窗必選/可選、價格差）、麵包＋飲品自動半價（可逐行關閉）、會員電話查詢（試 66123456）、儲值充值+贈送【示例】、儲值扣款、出品看板（點擊推進狀態）、會員管理與流水帳／儲值負債。日結對數、月結儲值負債（含對數檢查）、手機版面。資料只存於記憶體，重新整理即重置。另有 `mobile.html`（顧客會員頁）與 `preview.html`（iPad＋iPhone 裝置預覽）。


---

## 12. 套餐優惠：麵包 ＋ 飲品，飲品連加配半價（客戶已確認）

### 12.1 規則
- 訂單中每 **1 件**「開放式酸種麵包」可令 **1 杯**飲品半價；半價基數 = 飲品**單價連所有加配**（凍、SOE、換奶、Tonic）。
- 引擎**自動配對最貴的合資格飲品單位**（對顧客最有利、店員最少爭議）。
- 店員可在購物車逐行**關閉**半價（例如顧客要分單、飲品另一人付）；關閉後自動改配下一杯合資格飲品。每次關閉寫入 audit_log。
- 購物車每杯飲品下方顯示「麵包套餐 · N 杯半價 −MOP x」或「未配對（麵包數量不足）」，總額區顯示「麵包＋飲品半價（已用／麵包數）」。
- 麵包本身的加配（例如 Q彈米麵包 +5）**不**半價。

### 12.2 假設（全部 待確認）
| # | 假設 | 備選 |
|---|---|---|
| A1 | 「飲品」= 經典咖啡、拿鐵系列、特調咖啡、冷萃、獨家飲品、無咖啡飲品全部 | 排除冷萃／獨家？ |
| A2 | 1:1 配對：2 件麵包 → 最多 2 杯半價 | 每單只限 1 組 |
| A3 | 自動配最貴飲品 | 配最便宜／按點單次序 |
| A4 | 每件半價後捨入至 MOP 0.1（四捨五入）。現價全為整數，半價最多出現 .5，實際不受影響 | 捨入至 0.5／1 |
| A5 | 適用堂食、外賣自取；外賣平台訂單不適用 | 視平台定價 |
| A6 | 與會員儲值支付可同時使用 | — |
| A7 | 不限時段（現系統「半價飲品」是否有時段未知） | 只限早午餐時段 |

### 12.3 演算法（packages/pricing）
```ts
function applyBreadDrinkHalf(lines: CartLine[], cfg): Discount[] {
  const breadUnits = sum(lines.filter(isBread).map(l => l.qty))
  const units = lines.filter(l => isDrink(l) && !l.promoOptOut)
     .flatMap(l => Array(l.qty).fill({ lineId: l.id, unit: l.unitPriceCents })) // unit 已含加配
     .sort((a, b) => b.unit - a.unit)
  return units.slice(0, breadUnits * cfg.ratio)
     .map(u => ({ lineId: u.lineId, amount: u.unit - roundTo(u.unit * (1 - cfg.pct/100), cfg.roundingCents) }))
}
```

### 12.4 必寫測試（Vitest）
1. 炒滑蛋 32 + 熱拿鐵 33 → 折扣 16.5，應付 48.5。
2. 牛油果三文魚 55 + 凍海鹽焦糖拿鐵連燕麥奶 46 + 熱美式 28 → 只半價 46（−23），應付 106。
3. 同上，店員關閉拿鐵 → 改半價美式（−14），應付 115。
4. 2 件麵包 + 3 杯飲品 → 最貴 2 杯半價。
5. 只有麵包、或只有飲品 → 無折扣。
6. 飲品 qty=2、麵包 qty=1 → 該行只 1 杯半價，顯示「1 杯半價」。
7. 麵包加 Q彈米麵包 +5 → 麵包價不受影響。
8. 甜品不觸發、不被半價。
9. 捨入：單價 47 → 23.5；若改為 0.5／1 捨入規則，參數化測試。
10. 伺服器重算結果與前端一致（同一函數 snapshot test）。

---

## 13. 會計處理、儲值流水帳與報表

### 13.1 會計處理（建議，須會計師確認）
| 事件 | 會計分錄（概念） |
|---|---|
| 充值 MOP 500（贈送 50） | 借：現金／銀行 500　貸：**合約負債（儲值）500**。贈送 50 **不入帳**，只在系統另冊記錄。 |
| 儲值消費 MOP 100（餘額本金 500：贈送 50） | 按餘額比例分攤：本金 90.91、贈送 9.09。借：合約負債 90.91　貸：**銷售收入 90.91**；贈送 9.09 列為**銷售折扣**（即該單實際收入 90.91）。 |
| 現金／卡／QR 銷售 | 借：現金／應收收單機構　貸：銷售收入（扣除半價優惠後淨額）。 |
| 作廢／退款（儲值支付） | 按原單本金／贈送比例退回錢包；沖回收入。 |
| 退還未用本金 | 借：合約負債　貸：現金。贈送不退。 |
| 長期未用（breakage） | **暫不確認**。是否按 IFRS 15 預期 breakage 比例確認收入、或設有效期，須會計師及律師確認（涉消費者保障）。 |

> 比例分攤（pro-rata）比「先扣贈送」更貼近「贈送 = 折扣」的經濟實質，亦令退款時本金計算清楚；「先扣本金／先扣贈送」作為可配置選項【待確認】。

### 13.2 流水帳資料模型（每筆交易本金／贈送分欄）
```sql
create type wallet_entry_type as enum ('topup','bonus','spend','refund','adjust','cashout','expire');
create table wallet_entries (
  id bigserial primary key,
  account_id uuid not null references wallet_accounts,
  type wallet_entry_type not null,
  principal_delta numeric(12,2) not null default 0,   -- 本金變動（負債）
  bonus_delta     numeric(12,2) not null default 0,   -- 贈送變動（備查，非負債）
  principal_after numeric(12,2) not null,             -- 方便稽核
  bonus_after     numeric(12,2) not null,
  order_id uuid, topup_id uuid, payment_id uuid,
  reason text,                                        -- 調整必填
  staff_id uuid not null, idempotency_key text unique not null,
  business_date date not null,                        -- 營業日（日結用，非 UTC 日期）
  created_at timestamptz not null default now()
);
-- 只增不改
create trigger wallet_entries_immutable before update or delete on wallet_entries
  for each row execute function raise_immutable();
-- 期間鎖定：period_closings(month, closed_at, closed_by, principal_close, bonus_close)
```
`wallet_spend(account, amount, order_id, key)`：`SELECT … FOR UPDATE` 鎖帳戶 → 計算 `bp = round(amount * bonus/(principal+bonus), 2)`、`pp = amount − bp` → 寫一筆 `spend`（兩欄皆負）。訂單 `payments` 同時記 `wallet_principal`、`wallet_bonus` 供報表。

### 13.3 日結對數報表（Daily reconciliation）
內容：
- 有效訂單數、作廢張數；銷售總額（折扣前）、**麵包＋飲品半價優惠金額及杯數**、淨銷售額。
- **按付款方式**（現金／銀行卡終端／MPay·聚合 QR／會員儲值）分列：**銷售收款**、**充值收款**、作廢退款、實收合計。
- **會員儲值消費不計入收款合計**（只列示，並拆本金收入／贈送折扣）。
- **現金櫃**：開櫃備用金 ＋ 現金銷售 ＋ 現金充值 − 現金退款 ＝ 應有現金；輸入實點現金 → 差額（長款／短款），簽核後鎖定。
- 卡／QR 應與收單銀行結算單對數（Phase 2 自動匯入）。

```sql
-- 收款按方式（銷售 vs 充值）
select method,
  sum(amount) filter (where kind='sale')   as sales,
  sum(amount) filter (where kind='topup')  as topups,
  sum(amount) filter (where kind='refund') as refunds
from payments_v                      -- union：orders.payments（kind=sale/refund）+ topups.payments（kind=topup）
where business_date = :d and store_id = :s
group by method;

-- 銷售與優惠
select count(*) filter (where status<>'voided') orders,
       count(*) filter (where status='voided')  voids,
       sum(gross) filter (where status<>'voided') gross,
       sum(discount_total) filter (where status<>'voided') discounts,
       sum(total) filter (where status<>'voided') net
from orders where business_date=:d and store_id=:s;

-- 應有現金
select :float + coalesce(sum(case when kind in ('sale','topup') then amount when kind='refund' then -amount end),0)
from payments_v where business_date=:d and method='cash';
```
存檔：`cash_counts(business_date, store_id, float, expected, counted, variance, counted_by, approved_by)`。

### 13.4 月結儲值負債表（Monthly stored-value liability）
**本金**：期初餘額 ＋ 本月充值本金 − 本月消費（本金部分）± 退款回存／調整 − 退還本金 ＝ 期末餘額。
**檢查**：期末餘額 **必須等於** 全體會員本金餘額合計（Σ principal_after 最後一筆）；不符即紅色警示，禁止結帳。
**贈送**：同一條式另冊列示（期初＋贈送−使用±調整＝期末未用贈送），不屬負債。
**收入確認**：本月確認收入 = 消費本金部分 − 退款沖回；贈送使用額 = 銷售折扣。
```sql
with p as (select date_trunc('month', :m::date) s, date_trunc('month', :m::date) + interval '1 month' e)
select
  sum(principal_delta) filter (where business_date <  p.s)                               as opening,
  sum(principal_delta) filter (where business_date >= p.s and business_date < p.e and type='topup') as topups,
 -sum(principal_delta) filter (where business_date >= p.s and business_date < p.e and type='spend') as consumed_revenue,
  sum(principal_delta) filter (where business_date >= p.s and business_date < p.e and type in ('refund','adjust','cashout')) as other,
  sum(principal_delta) filter (where business_date <  p.e)                               as closing
from wallet_entries, p;
-- 對數：closing = (select sum(principal_after) from latest_entry_per_account where business_date < p.e)
-- 贈送：同上換 bonus_delta
```
另附每位會員明細（期初／充值／消費／其他／期末／系統餘額 ✓）。

### 13.5 對數不變式（必須寫成自動測試）
1. Σ 所有 wallet_entries.principal_delta ＝ Σ 各帳戶最新 principal_after。
2. 日結：實收合計 ＝ 現金＋卡＋QR 的（銷售＋充值−退款）；儲值消費不入。
3. 淨銷售 ＝ 各付款方式銷售（含儲值）− 作廢。
4. 月結期末 ＝ 下月期初。
5. 每筆 spend：|principal_delta| ＋ |bonus_delta| ＝ 訂單儲值付款額。

---

## 14. 部署：PWA（iPad 店員端 ＋ 手機）

| 面向 | 方案 |
|---|---|
| 形態 | 一個 Next.js PWA，兩個入口：`/pos`（店員；iPad 橫向＋手機響應式）與 `/m`（顧客會員頁：會員 QR、餘額、充值、紀錄、條款） |
| 安裝 | iPad／iPhone Safari →「加至主畫面」；`manifest.webmanifest`（display: standalone）、`apple-mobile-web-app-capable`、`apple-touch-icon`、`theme-color`、`viewport-fit=cover`＋safe-area |
| 版面 | ≥ 821px：側欄＋分類＋餐牌＋購物車三欄；≤ 820px：底部導航、分類橫向捲動、購物車改為底部抽屜＋浮動結帳列 |
| 離線 | Service Worker（Serwist / next-pwa）快取餐牌與介面；訂單離線佇列；**儲值扣款、充值必須在線** |
| 顧客 QR | 動態碼（會員 ID ＋ 時間戳 HMAC，60 秒有效），店員掃描綁定會員；防截圖重用 |
| 顧客充值 | Phase 2：經收單銀行線上 MPay／BOC Pay／卡支付 API，webhook 成功後才入帳（`idempotency_key` = 支付單號） |
| 登入 | 店員：門店裝置登入＋4 位 PIN；顧客：電話 OTP（SMS 成本待評估，可考慮 WhatsApp） |
| 網域 | `pos.calmo.mo` / `m.calmo.mo`（示例） |

Demo 檔案：`demo.html`（店員端，響應式）、`mobile.html`（顧客會員頁）、`preview.html`（iPad＋iPhone 並排預覽）、`manifest.webmanifest`、`icon.svg`。


---

## 15. 店員點餐 UX 原則（v3，按 Michael 回饋重新設計）

1. **點餐畫面只有三樣**：分類標籤（首個為「常用」）、大品項格（只顯示名稱＋價錢，無描述）、購物車。出品、充值、會員、日結、月結全部收在右上「☰ 更多」，不與點餐爭位置。
2. **一按即加**：點品項即以預設加入（熱／鮮奶／中深烘／酸種麵包）；同款同選項自動合併數量；品項格右上顯示已點數量。
3. **冷熱一按選定**：可熱可凍的品項格分為左「熱 33」右「凍 38」兩半。理由：冷熱是最常見、而且必須的選擇，雙半格令它變成**一下點擊**（彈窗要點 2–3 下，長按又不易被發現）；只得一種溫度的品項是整格一按。如此現有餐牌**沒有品項需要彈窗**。
4. **選項後補**：換奶、SOE、Tonic、麵包底、半價開關等，**點購物車該行**開啟精簡底部面板（大 chip，≥ 52px，一個「確定」）；長按品項格可先選再加。
5. **購物車簡潔**：一行名稱、小字加配、一行「麵包套餐半價 −$23」、數量 −/＋（44–48px）、金額；唯一大按鈕「結帳 MOP xx」。
6. **結帳獨立頁**：大額應付金額、四個大付款鍵（現金／銀行卡／MPay·QR／會員餘額）＋「先充值」；**會員電話只在結帳頁輸入**（只為用餘額付款）。
7. **減少視覺雜訊**：單一墨色強調、凍價用低飽和藍字區分、統一 12px 間距、觸控目標 ≥ 48px、不用密集徽章。
8. 手機：同一套版面，分類橫向捲動、兩欄品項格、底部浮動「查看訂單」列 → 購物車抽屜 → 結帳頁。
9. 業務邏輯不變：計價引擎、半價促銷、錢包分帳、日結／月結報表全部沿用同一資料流（已回歸測試 106 案例）。
