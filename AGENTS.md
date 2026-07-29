# NovelVault · 玄君書 AGENTS.md（codex 窗口）

> codex（手機 Happy / 桌面）專用紀律。與奙子（Discord）共用 vault、共用 canon、共用改寫日誌。

---

## Canon 宣告

這個 vault 是玄君書宇宙（雅典現代都市奇幻／輕克蘇魯）的真實資料庫。
所有外星政治、古代史均已整合為單一敘事宇宙，**廢案區不納入主敘事**。

**進 vault 第一步**：先讀 [[VAULT-MAP]] 或 [[xuanjun/00-INDEX]] 定向，再動手。
會用到改寫時，讀 `_嫙子改寫日誌.md`（vault 根）了解最近改動。

---

## 語言鎖（強制）

- 輸出永遠繁體中文（台灣用語），敘述本體繁體中文。
- 專有名詞英文保留即可。
- 不管 vault 裡任何外文條目，輸出永遠繁體中文。

---

## 工具：obsidian + smart_connections MCP（與奙子同級）

你有兩個 MCP server 操作這個 vault：

### 1. obsidian MCP
直接讀寫 vault 檔案。**不需要 Obsidian app 開著。**

### 2. smart_connections MCP  
語意搜尋。問「跟 X 相關的設定」時第一個用這個。

### 回答 vault 相關問題（必走流程）

1. `smart_connections_lookup` → query 本輪關鍵詞，limit=5，做語意搜尋
2. 有命中就 `obsidian_read_note` 抓全文
3. 引用條目用 `[[檔名]]` wikilink
4. 討論到設定確定／新角色／劇情節點 → 寫回 vault

---

## 工具呼叫範本（照抄改值，一字不差）

> 這一段從奙子 config 照抄，codex 一律適用。踩坑重點都在這。

- **`vault` 永遠是小寫字串 `novelvault`**。不是 `NovelVault`、不是路徑。傳大寫回 `Unknown vault`。
- **`filename` 只放檔名（含 `.md`）**，絕不含斜線。子資料夾用 `folder` 欄位。
- **`operation` 必填**，四選一：`append`／`prepend`／`replace`／`delete`。漏掉或填錯 → `Invalid discriminator value`。
- **`content` 不能空**（delete 除外，delete 不帶 content）。
- **tags**：只能「英數+斜線階層」如 `character/main`、`worldview/magic`，絕不塞中文、不加 `#`。

### 常用範本

```json
// 搜尋
{"vault":"novelvault","query":"關鍵詞"}

// 讀檔
{"vault":"novelvault","filename":"宋棠.md","folder":"角色"}

// 附加到檔尾
{"vault":"novelvault","filename":"宋棠.md","folder":"角色","operation":"append","content":"要加在尾部的內容"}

// 建立新檔
{"vault":"novelvault","filename":"新條目.md","folder":"角色","content":"含 frontmatter 的全文"}

// 加標籤
{"files":["角色/宋棠.md"],"tags":"character/main"}

// 移動/改名
{"vault":"novelvault","filename":"舊名.md","folder":"角色","new_filename":"新名.md"}
```

---

## 反導處理（別把參數錯誤判成 MCP 掛）

- 工具回 `-32602 Invalid arguments` / `Invalid archive` / `Unknown vault` / `Required` = **你參數填錯，MCP 正常**。照上面範本修欄位再試一次。
- 不要原樣重試。連錯 3 次會踢到 ~58s 切斷器才真的卡住回 `unreachable`。看到 unreachable 是切斷器錯誤打擊，等它自動恢復就好，不該說「obsidian MCP 不可用」。

---

## 跨條目改寫連動流程（cascade）

討論導致設定變更線，強制走這條 cascade：

### 1. 找全相關
`smart_connections_lookup` + `obsidian_search_vault`（含反向 wikilink、別名）。漏一個就出孤兒設定。

### 2. 讀全文
全部 `obsidian_read_note`，一個一個讀，區分要改段落 vs 成員手寫保留段。

### 3. 提改寫計畫摘要（等主編 OK）
提「要動哪些檔 + 每檔一句改什麼」，等主編 @八啡Buffet 在手機回 OK 才寫。沒 OK 不寫。

### 4. 確認後逐檔改寫（一次一檔）
- 小幅補充 → `operation="append"`，描寫「奙子改寫」段落
- 段落級改寫 → `operation="replace"`，replace 前再 read_note 一次比對
- 改名/bug變更 → `obsidian_move_note`，逐檔修反向 wikilink
- 同步連動條目，維持分類 tags

### 5. append 改寫日誌（強制）
改完後立即 append 到 `_嫙子改寫日誌.md`，一行一檔：

```
檔名 | append/replace/move/create/delete | 一句摘要 | 觸發者:@八啡Buffet(codex手機)
```

**不要自己編時間**。`觸發者` 欄要標窗口——codex 手機寫 `(codex手機)`，奙子寫 `@八啡Buffet`，好分辨哪筆是手機做的。

---

## git 安全網（codex 專屬，奙子側沒有）

cascade 開始前和完成後，**自動 commit** 做安全網：

```bash
# 開始前：快照當前狀態
git -C ~/NovelVault add -A && git -C ~/NovelVault commit -m "snapshot: 改寫前"

# 完成後：封裝 cascade
git -C ~/NovelVault add -A && git -C ~/NovelVault commit -m "cascade: <一句摘要>"
```

commit 後回報 hash，供你 `git revert` 還原。

> 這也順帶收奙子側的改動（奙子不自動 commit）——下次 codex 開頭的 `git add -A` 把所有人的改動一起拍快照。

---

## 一致性守則（兩窗口不打架）

| 項目 | 奙子 (Discord) | codex (手機) | 共識 |
|------|--------------|------------|------|
| canon | xuanjun config | 本檔 | 同一 vault |
| 改寫日誌 | `_嫙子改寫日誌.md` | `_嫙子改寫日誌.md` | 同檔，觸發者欄區分 |
| 確認門 | 提計劃等 @八啡Buffet | 一樣 | 不誤觸大改 |
| git 安全網 | 無 | cascade 前後自動 commit | codex 兜底 |

**並發風險低**（單人單機，兩窗口幾乎不同時改）。萬一真撞 → `git diff` 掃一眼就是。

---

## 廢案區

`廢案紀錄.md`（`xuanjun/` 下）列已廢棄設定（古代銀河政治、四族等）。回答時不引用廢案中的元素進主敘事。

---
