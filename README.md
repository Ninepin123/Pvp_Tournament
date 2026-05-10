# PvpTournament

> Minecraft Paper / Purpur 伺服器的單敗淘汰賽 PVP 比賽插件

支援單人賽 / 雙人賽、自動產生對戰表、玩家對戰表 GUI、管理員一鍵控制面板、隊友邀請、自動套裝、Vault 金錢與物品(完整保留 NBT)獎勵,並深度整合 Residence、PvPManager、DungeonSystem 避免比賽期間互相干擾。

- **API**: Paper 1.21.4
- **Java**: 21
- **必要**: 無(獎勵金錢需要 Vault)
- **軟相依**: Vault / Residence / PvPManager / DungeonSystem

---

## 目錄

- [安裝](#安裝)
- [第一次設定 (5 分鐘)](#第一次設定-5-分鐘)
- [辦一場比賽](#辦一場比賽)
- [玩家操作](#玩家操作)
- [雙人賽 (DUO) 流程](#雙人賽-duo-流程)
- [編輯冠軍獎勵](#編輯冠軍獎勵)
- [指令速查](#指令速查)
- [權限](#權限)
- [常見問題](#常見問題)
- [進階:設定檔](#進階設定檔)
- [第三方插件整合](#第三方插件整合)

---

## 安裝

1. 下載 `PvpTournament-x.x.x-SNAPSHOT.jar`
2. 丟進伺服器的 `plugins/` 資料夾
3. (選用)安裝 [Vault](https://www.spigotmc.org/resources/vault.34315/) 才能發金錢獎勵
4. 啟動伺服器,會自動生成 `plugins/PvpTournament/config.yml`、`arenas.yml` 等
5. 給管理員 `tournament.admin.*` 權限(OP 預設有)

---

## 第一次設定 (5 分鐘)

開啟管理員控制面板:

```
/tn panel
```

控制面板第三列有四顆按鈕:

| 按鈕 | 用途 |
|---|---|
| 🟡 **設定大廳傳送點** | 站在你想要當大廳的位置 → 點擊。所有玩家 `/tn join` 後會被傳到這裡 |
| 🔴 **設定紅隊出生點** | 站在競技場一側 → 點擊。比賽時紅隊在這裡開打 |
| 🔵 **設定藍隊出生點** | 站在競技場另一側 → 點擊 |
| 🟨 **編輯冠軍獎勵** | 開啟獎勵編輯介面(下面會說明) |

> **系統永遠只用一個場地**,不需要設定場地名稱或多場地。

(可選)用指令儲存玩家套裝(穿好裝備、放好背包後):

```
/tn kit save single        # 儲存單人賽套裝
/tn kit save duo           # 儲存雙人賽套裝
```

比賽進場時,系統會自動把玩家背包換成這套裝;比賽結束會還原。

---

## 辦一場比賽

```
/tn panel                         # 開啟控制面板,點擊操作
```

或用指令:

```
/tn create single                 # 建立單人賽 (或 duo 雙人賽)
/tn open                          # 開放報名 → 全頻道廣播
                                  # → 玩家用 /tn join 報名 (自動傳送到大廳並鎖定指令)
                                  # → 人數到了之後...
/tn start                         # 開始比賽,系統自動產生對戰表,各場開打
                                  # → 比賽進行中,玩家死亡會被傳到觀戰席
                                  # → 一場結束後會自動推進下一輪
                                  # → 最後決出冠軍,自動發獎勵
```

中途若要強制中止:

```
/tn stop                          # 取消比賽,所有玩家傳回原位、背包還原
```

---

## 玩家操作

| 指令 | 用途 |
|---|---|
| `/tn join` | 報名比賽。**會被自動傳送到大廳,且禁止使用其他指令、丟末影珍珠、傳送等** |
| `/tn leave` | 退出比賽,**傳回 `/tn join` 之前的位置**,解除指令鎖 |
| `/tn bracket` | 開啟對戰表 GUI,可看每一場的玩家、勝者、進度;每秒自動刷新 |
| `/tn status` | 文字版比賽狀態 |
| `/tn surrender` | 比賽進行中投降(整隊判輸) |
| `/tn spectate` | 傳送到觀戰席 |

> **報名後鎖定**:玩家 `/tn join` 後,任何非 `/tn` 開頭的指令都會被擋,末影珍珠 / 紫頌果 / `/tp` / `/spawn` / `/back` 全部失效。要解鎖:報名階段 `/tn leave`、比賽中 `/tn surrender`,或等比賽結束。

> **斷線即棄權**:比賽中斷線,該玩家的隊伍直接判輸;報名階段斷線等同 `/tn leave`。

---

## 雙人賽 (DUO) 流程

雙人賽會兩兩配對。預設**隨機配對**,你也可以自己邀請隊友:

```
玩家 A:  /tn join                  # 兩人都先報名
玩家 B:  /tn join

玩家 A:  /tn invite B               # A 邀請 B 當隊友(60 秒有效)
玩家 B:  /tn accept A               # B 接受 (或 /tn deny A 拒絕)

玩家 A 或 B:  /tn partner            # 查看自己的隊友
玩家 A 或 B:  /tn unpair             # 解除目前配對
```

**奇數人怎麼辦?** 假設 5 人報名 DUO:
- 配對 2 隊(4 人)
- 第 5 個人成為「**1 人種子隊伍**」 — 不會被踢掉,系統會把他排到較高 seed,首輪較容易輪空
- 若必須對到雙人隊則為 1v2,辛苦但能參賽

---

## 編輯冠軍獎勵

在 `/tn panel` 點擊金錠按鈕「編輯冠軍獎勵」,進入 6 列箱子介面:

```
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│   │   │   │   │   │   │   │   │   │  ← 上 5 列 (45 格)
│   │   │   │   │   │   │   │   │   │     拖你想送的物品進來
│   │   │   │   │   │   │   │   │   │     ✅ 完整保留附魔 / 命名 / lore / NBT
│   │   │   │   │   │   │   │   │   │
│   │   │   │   │   │   │   │   │   │
├───┼───┼───┼───┼───┼───┼───┼───┼───┤
│███│███│███│███│ 💰│███│███│███│ ✅│  ← 控制列
└───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

- **上 5 列(45 格)**:把任何物品拖進來、shift-click 也行。所有 NBT(附魔、書本內容、命名、custom data)會被完整保留。
- **金錢按鈕(中央太陽花)**:
  - 左鍵 +1000 / 右鍵 -1000
  - Shift+左鍵 +10000 / Shift+右鍵 -10000
  - 需要 Vault,沒裝會在 lore 顯示提示
- **儲存按鈕(右下綠色混凝土)**:存到 `plugins/PvpTournament/rewards.yml` 並關閉介面
- 直接按 Esc 關閉視窗也會自動儲存

之後比賽冠軍隊每位成員都會自動收到這些物品(各自拿一份)+ Vault 金錢入帳。

---

## 指令速查

### 玩家(任何人)

| 指令 | 用途 |
|---|---|
| `/tn join` | 報名 |
| `/tn leave` | 退出報名 |
| `/tn bracket` | 對戰表 GUI |
| `/tn status` | 文字狀態 |
| `/tn surrender` | 投降 |
| `/tn spectate` | 觀戰 |
| `/tn invite <玩家>` | 邀請隊友(DUO) |
| `/tn accept [玩家]` | 接受邀請 |
| `/tn deny [玩家]` | 拒絕邀請 |
| `/tn unpair` | 解除配對 |
| `/tn partner` | 查看隊友 |

### 管理員

| 指令 | 用途 |
|---|---|
| `/tn panel` 或 `/tn admin` | 開啟控制面板 |
| `/tn create <single\|duo>` | 建立比賽 |
| `/tn open` | 開放報名 |
| `/tn start` | 開始比賽 |
| `/tn stop` | 強制停止 |
| `/tn reload` | 重載 config / arenas / kit |
| `/tn setlobby` | 設大廳座標(panel 也可) |
| `/tn forcewin <matchId> <teamId>` | 強制判勝 |
| `/tn restore <玩家>` | 還原玩家背包快照 |
| `/tn kit save\|clear <single\|duo>` | 儲存 / 清除套裝 |
| `/tn testfill <count>` | 加 N 個假玩家測試對戰表 |
| `/tn arena ...` | 進階場地管理(panel 已含基本) |

> 別名:`/tournament`、`/tn`、`/pvpt` 都是同一個指令

---

## 權限

| 權限 | 預設 | 用途 |
|---|---|---|
| `tournament.player.*` | `true`(每個玩家) | 玩家所有指令 |
| `tournament.admin.*` | `op` | 管理員所有指令 |

細項:

```
tournament.player.join
tournament.player.leave
tournament.player.bracket
tournament.player.status
tournament.player.spectate
tournament.player.surrender
tournament.player.invite

tournament.admin.create
tournament.admin.open
tournament.admin.start
tournament.admin.stop
tournament.admin.reload
tournament.admin.arena       # 設場地 / 大廳
tournament.admin.kit
tournament.admin.forcewin
tournament.admin.restore
tournament.admin.testfill
tournament.admin.panel
```

---

## 常見問題

**Q: 玩家 `/tn join` 後傳不過去 / 留在原地?**  
A: 還沒設定大廳座標。在大廳位置打 `/tn setlobby` 或用 panel 按鈕。

**Q: 比賽進場後玩家在 Residence 領地裡互砍不會掉血?**  
A: 確認 `PvPManager` 在啟用清單。本插件會自動覆寫 Residence + PvPManager 的 PVP 阻擋。如果還是不行,確認伺服器啟動順序(本插件 `softdepend` 它們)。

**Q: 比賽結束後玩家還能在領地內互砍?**  
A: PvPManager 的「戰鬥標籤」會繞過領地保護。本插件已在 `endMatch` / `stop` 時自動 `untag`,如果失效請看 server log 是否有 reflection 警告。

**Q: 玩家不夠 2 個人就 `/tn start`?**  
A: 會回 `NOT_ENOUGH_PLAYERS`。可改 `config.yml` 的 `min-players-single` / `min-teams-duo`。

**Q: 想用更少的真人測試完整對戰表?**  
A: `/tn testfill 7` 加 7 個 Bot,跟你一起填滿對戰表。Bot vs 真人時,真人會被傳進場,你 `/tn forcewin <matchId> <teamId>` 強制判贏即可推進。

**Q: 比賽中可以 `/tn bracket`、`/tn status`、`/tn surrender` 嗎?**  
A: 可以,所有 `/tn ...` 子指令都不在鎖定範圍。

**Q: 玩家斷線重連會自動回比賽嗎?**  
A: 不會。**斷線即判棄權**。

**Q: 獎勵物品的附魔 / 名稱會保留嗎?**  
A: 會。獎勵編輯 GUI 用 Bukkit `ItemStack` 序列化,完整保留所有 NBT(附魔、display name、lore、custom data components 等)。

---

## 進階:設定檔

`plugins/PvpTournament/config.yml` 主要參數:

```yaml
tournament:
  max-players-single: 32       # 單人賽最大人數
  max-teams-duo: 16            # 雙人賽最大隊數
  min-players-single: 2
  min-teams-duo: 2
  final-use-own-items: true    # 冠軍賽用玩家自己背包(否則套用 kit)
  normal-round-use-kit: true   # 一般場套用 kit

match:
  countdown-seconds: 10        # 開賽倒數
  max-duration-seconds: 600    # 單場最長秒數,逾時隨機判勝

gui:
  bracket-refresh-ticks: 20    # 對戰表 GUI 刷新間隔 (tick;最低 5)

duo:
  invite-expire-seconds: 60    # /tn invite 邀請有效秒數

messages:
  prefix: "&6[比賽系統]&r "
```

獎勵存在 `rewards.yml`(由 GUI 寫入),不要手動編輯。

場地 / 大廳座標存在 `arenas.yml`(由 GUI / 指令寫入)。

---

## 第三方插件整合

### Residence
比賽中在領地內,Residence 會把 PVP `cancel`。本插件在 `EventPriority.HIGH` 的 listener 反向 `setCancelled(false)`,讓比賽傷害不被擋。

### PvPManager
- 比賽中:同上,本插件在 `HIGH` 反向放行同場異隊攻擊
- **比賽結束後**:呼叫 `CombatPlayer.untag(UntagReason.PLUGIN_API)` 強制清掉戰鬥標籤,避免領地保護被持續繞過

### DungeonSystem
玩家進場前若在 DungeonSystem 組隊中,會被自動踢出 / 解散,避免 DungeonSystem listener 把比賽傷害當隊友傷害過濾。

整體 listener priority 鏈:

```
LOWEST  Residence            → 可能 cancel
NORMAL  PvPManager           → 可能 cancel
HIGH    PvpDamageOverride    → 比賽同場異隊 → setCancelled(false)
HIGHEST DamageListener       → 致死攔截 / 同隊友傷 / 同場驗證
```

---

## 開發者:建構

```bash
mvn -DskipTests package
```

產出在 `target/PvpTournament-x.x.x-SNAPSHOT.jar`,需 JDK 21+。

---

## 授權

MIT

## 作者

pinp · [github.com/ninepin1234](https://github.com/ninepin1234)
