# 2026-03-02 gog Calendar 強制使用規則與對照表
**記錄者**：阿莎
**記錄時間**：2026-03-02 18:23
**核心目的**：徹底解決 Calendar 操作低級錯誤，建立無彈性的強制規則。

---

## 🚫 嚴格禁止事項 (違反即導致 400 badRequest)

### 1. 參數順序與選項絕對原則
**第一個參數必須是 Account ID**：
- ✅ 正確：`gog calendar add "jetannku@gmail.com" ...`
- ❌ 錯誤：`gog calendar add --summary "..."` (缺少 Account ID)

### 2. 時間格式絕對原則
**必須使用完整的 RFC3339 格式**：
- ✅ 正確：`2026-03-04T09:00:00+08:00` (帶時區)
- ❌ 錯誤：`2026-03-04` (純日期，必錯)
- ❌ 錯誤：`2026-03-04 09:00` (缺少時區，可能錯)

### 3. 必須驗證原則
**任何 Calendar 操作後，必須使用 Search 命令驗證**：
- ✅ 正確：`add` 後執行 `gog calendar search "關鍵字" --account jetannku@gmail.com`
- ❌ 錯誤：只發送郵件通知，不驗證是否真的寫入 Calendar

### 4. 禁止命令混用
**Calendar 指令專用，不能混用其他指令**：
- ✅ 正確：只執行 `gog calendar add` 和 `gog calendar search`
- ❌ 錯誤：在執行 `gog` 的同時添加其他 Shell 操作

---

## 📋 命令對照表

### 新增行程 (必須執行並驗證)

#### 單一提醒 (當天彈出)
```bash
gog calendar add "jetannku@gmail.com" \
  --summary "3月4日石家莊之旅" \
  --from "2026-03-04T09:00:00+08:00" \
  --to "2026-03-04T18:00:00+08:00" \
  --reminder "popup:1440m"
```

#### 雙重提醒 (彈出 + 郵件)
```bash
gog calendar add "jetannku@gmail.com" \
  --summary "3月8日北京回台北" \
  --from "2026-03-08T13:40:00+08:00" \
  --to "2026-03-08T14:00:00+08:00" \
  --reminder "popup:315m,email:1440m" \
  --description "3月8日13:40從北京回台北"
```

#### 驗證 (新增行程後必須執行)
```bash
gog calendar search "北京" --account jetannku@gmail.com
```

### 列出行程

#### 列出所有行程 (無時間範圍)
```bash
gog calendar list --account jetannku@gmail.com
```

#### 列出指定範圍行程 (支持 --from/--to)
```bash
gog calendar list --account jetannku@gmail.com --from "2026-03-01T00:00:00+08:00" --to "2026-03-31T23:59:59+08:00"
```

### 事件操作 (推薦使用)

#### 查詢指定日期範圍的事件
```bash
gog events "jetannku@gmail.com" \
  --from "$(date +%Y-%m-%dT%H:%M:%S+08:00)" \
  --to "$(date -d '+30 days' +%Y-%m-%dT%H:%M:%S+08:00)"
```

#### 刪除特定事件 (需要 Event ID)
```bash
gog delete "jetannku@gmail.com" <eventId>
```

#### 按日期範圍刪除 (使用 delete 命令)
```bash
# 注意：不同版本 gog 可能支持不同參數，需驗證
gog calendar delete "jetannku@gmail.com" --date-range "2026-03-28T00:00:00+08:00/2026-03-28T23:59:59+08:00"
```

### 搜索與查詢

#### 搜索行程 (支持多種關鍵字)
```bash
gog calendar search "北京" --account jetannku@gmail.com
gog calendar search "上海" --account jetannku@gmail.com
gog calendar search "T3" --account jetannku@gmail.com
```

---

## 🚫 常見錯誤與對照表

| 錯誤訊息 | 錯誤原因 | 正確做法 |
|----------|----------|----------|
| `unexpected argument` | 參數順序錯誤 | Account ID 必須是第一個參數 |
| `badRequest` | 時間格式不完整 | 必須包含時區 `+08:00` |
| `badRequest` | 使用了純日期 `2026-03-04` | 必須使用完整時間格式 `2026-03-04T09:00:00+08:00` |
| `no TTY available` | 在非交互式環境執行 | 使用 `gog calendar add` 而非 `gog calendar interactive` |
| `events requires eventId` | 沒有提供 Event ID | 先用 `gog events` 查詢 ID，再刪除 |
| `list does not support --from/--to` | 工具版本差異 | 使用 `gog calendar list` 無參數或使用 `gog events` 查詢 |

---

## 🎯 強制執行流程

### 步驟 1: 執行新增命令
```bash
gog calendar add "jetannku@gmail.com" \
  --summary "3月28日上海回台北" \
  --from "2026-03-28T09:00:00+08:00" \
  --to "2026-03-28T17:00:00+08:00" \
  --reminder "popup:1440m,email:2880m"
```

### 步驟 2: 強制驗證 (不驗證視為失敗)
```bash
gog calendar search "上海回台北" --account jetannku@gmail.com
```

### 步驟 3: 確認輸出 (必須包含 ID 和 Summary)
- 必須看到：`Event ID` 和 `Summary`
- 如果看不到，視為失敗，必須重新執行

---

## 💾 永久記憶關鍵字

1. **gog calendar add**：第一個參數永遠是 Account ID
2. **RFC3339 格式**：`YYYY-MM-DDTHH:mm:ss+08:00` (時區固定為台北時間)
3. **驗證原則**：`add` 後必須 `search`
4. **禁止純日期**：嚴禁使用 `2026-03-04`，必須包含時間
5. **提醒格式**：`--reminder "popup:15m,email:1440m"`

---

## 🚨 禁止行為 (一經發現，嚴格修正)

1. **禁止猜測**：不要猜測命令是否成功，必須驗證
2. **禁止假裝**：不要假裝看到了輸出，必須如實反饋
3. **禁止跳過驗證**：不要跳過 `search` 步驟
4. **禁止簡化格式**：不要簡化時間格式，必須完整

---

**記錄時間**：2026-03-02 18:23
**記錄者**：阿莎
**狀態**：已永久記憶，將嚴格遵守
