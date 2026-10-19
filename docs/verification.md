# 驗證工具

`scripts/verify_snapshot.py` 需要 Python 3.10+，僅使用標準函式庫。它不執行備份／還原，也不終止程序。

## inventory

```text
python scripts/verify_snapshot.py inventory --root <離線快照> --manifest <快照外的清單.json>
```

記錄範圍內全部檔案的大小與 SHA256、目錄（含空目錄）及明列的排除項目。既有清單不會被覆蓋。來源應保持離線；單檔雜湊與樹狀清單變動檢查無法取代整體一致性快照。

遇到 junction、symlink 或其他 reparse point 會報錯。先確認連結目標是否要單獨備份；確定排除時加入一或多個 `--exclude <相對路徑>`。排除項目不屬於驗證成功的範圍。

## verify

```text
python scripts/verify_snapshot.py verify --root <待比對目錄> --manifest <清單.json> --report <目錄外的報告.json>
```

比對缺少、額外、雜湊不符的檔案與目錄。預設要求範圍內精確相符；合併還原可指定 `--allow-extras`，額外項目仍會列出。合併選項不能取代資料庫 sidecar 的人工範圍判斷。

清單是本工具的 `schema: 1` 格式，並非任意舊備份的元件映射 manifest。來源清單與報告都應保存在受驗證目錄外，報告不能覆蓋輸入清單。JSON 中的路徑會驗證為相對路徑。

## sqlite

```text
python scripts/verify_snapshot.py sqlite --root <離線資料目錄> --report <目錄外的報告.json>
```

辨識 SQLite 檔頭，以及副檔名為 `.sqlite`、`.sqlite3`、`.db` 的候選檔案。逐個複製主檔及存在的 WAL／SHM 到臨時目錄，再執行 `PRAGMA integrity_check`；不在原始資料庫上開啟 SQLite 連線。臨時副本會在檢查後清除。

`.db` 也可能屬於其他產品格式；無 SQLite 檔頭會明確失敗，請判斷類型並調整檢查範圍，不可把未知檔案當已驗證。找不到任何資料庫會回傳失敗，避免「零個檢查也成功」。

原資料須在複製和檢查期間保持穩定；雜湊檢查是輔助偵測，不是線上快照保證。資料庫結構完整不等於聊天完整、專案映射正確或 App 功能可用。

## 返回碼與報告

| 返回碼 | 意義 |
| --- | --- |
| 0 | 該指令在明列範圍內檢查通過。 |
| 1 | 已完成檢查，發現不符、損毀或沒有資料庫。 |
| 2 | 參數、權限、清單格式、連結、讀取或來源變動等操作錯誤。 |

成功產生的報告包含檢查時間、範圍、失敗項目與結果。檢查指令在操作錯誤時不一定會產生報告，應同時檢查返回碼及 stdout。這些產物可能包含個人路徑與檔名，僅供本機使用，請依 [隱私文件](privacy.md) 去識別化後再分享。
