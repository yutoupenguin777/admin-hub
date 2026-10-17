# App 統一後台（admin-hub）

所有 app 的用戶數據儀表板，一頁看完。打開 `index.html`、輸入密語（與靈耀賣家後台同一組）即可。

## 架構

- 純靜態 HTML（`index.html`），不需伺服器，直接 `open index.html`。
- 每個 app 連自己的 Supabase 專案，走「密語 RPC」模式（同 lingyo 026_app_stats.sql）：
  - `stats_config`：bcrypt 雜湊存密語，RLS 無 policy，外部讀不到。
  - `stats_check(p_secret)`：內部驗證，不開放 anon。
  - `stats_set_secret(p_secret)`：**first-set-wins** — 只有還沒設定過才有效。
    admin-hub 首次解鎖時會自動把你輸入的密語綁進去，之後就鎖死。
  - `app_stats(p_secret)`：聚合統計（JSON）。
  - `app_users(p_secret)`：名單（用戶/家庭層級，一列一個）。
- 密語存 localStorage（`adminhub_secret`），按「鎖定」清除。

## 各 app 狀態

| App | Supabase | SQL 檔 | 狀態 |
|---|---|---|---|
| 靈耀 lingyo | xgipwimgthjzgovpazsr | `lingyo/supabase/migrations/026_app_stats.sql` | ✅ 原本就有（密語走 sales_check） |
| 菜蟲 caichong | xbmknfwetpndrjxegdfg | `caichong/supabase/migrations/002_app_stats.sql` | ✅ 已自動套用（Management API） |
| 卷卷 juanjuan | jgbglzyykkeoaiwankek | `juanjuan/supabase/app_stats.sql` | ⚠️ 要手動貼一次 SQL Editor |
| 明盾 mingdun | fzxdlmfkqgdmthphzlee | `mingdun/supabase/app_stats.sql` | ⚠️ 要手動貼一次 SQL Editor |
| Rally | ifzutbbooczubxvcvukq | `rally/supabase/migrations/0003_app_stats.sql` | ⚠️ 要手動貼一次 SQL Editor |
| 明目 mingmu | 無後端 | — | 純本地 App，看 App Store Connect |

⚠️ 的三個專案在不同 Supabase 帳號下（CLI 只登入了 Caichong 那個帳號），
所以要各自到 Dashboard > SQL Editor 貼上對應 SQL 檔執行一次。
儀表板裡對應分頁也會直接顯示要貼哪個檔＋SQL Editor 連結。
貼完回儀表板按「重新整理」，密語自動綁定，完成。

## 密語重設

某個 app 密語綁錯了：到該專案 SQL Editor 跑
`delete from stats_config where id = 1;`，再回儀表板重新整理即可重綁。

## 安全備註

- 只暴露密語保護的聚合 RPC 給 anon；密語以 bcrypt 雜湊存放。
- `stats_set_secret` 是 first-set-wins：SQL 剛裝好、還沒人解鎖前，理論上誰先呼叫誰綁定。
  裝完盡快打開儀表板解鎖一次即可（與 lingyo sales-hub 同等級的個人工具威脅模型）。
- 明盾的 RPC 是 SECURITY DEFINER，不受 x-family-code RLS 影響，但只回傳聚合數與
  家庭摘要（守護者名／計數），不回傳電話號碼。
