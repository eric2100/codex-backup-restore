# Codex Backup & Restore

Windows Codex 資料備份、還原與驗證技能。以離線一致性、回復副本、逐檔 SHA256 及 SQLite 完整性檢查，降低「複製完成卻無法還原」的風險。

**由 OpenAI Codex（OpenAI 的 AI 程式助理）撰寫與整理。** 詳見 [AUTHORSHIP.md](AUTHORSHIP.md)。本專案採 [MIT License](LICENSE)，屬社群專案，並非 OpenAI 官方備份產品。

## 功能與範圍

- 備份／還原前盤點 Codex home、共用技能、專案、App 資料與選定的 SSH／認證資料。
- 先保存目標現況，處理主程序、外掛及 SQLite WAL 的檔案占用問題。
- 區分 `DATA_VERIFIED`（資料層）與 `APP_VERIFIED`（實際功能），避免把排程啟動或 Robocopy 成功當作完整復原。
- 提供標準函式庫實作的驗證工具與合成測試，不包含任何人的備份、憑證、聊天或還原執行紀錄。

這是一套 **Codex 技能＋唯讀驗證工具**。備份、複製、還原與程序管理由讀取技能的 agent 依環境執行；工具本身不會一鍵關閉 App、覆寫資料或修復資料庫。不能保證所有版本或機器均可無損遷移。

## 安裝技能

需要 Git；驗證工具需要 Python 3.10+，無第三方套件。

在 PowerShell 執行（若已安裝同名技能，先比較內容並保存原版本）：

```powershell
$skillRoot = if ($env:CODEX_HOME) { Join-Path $env:CODEX_HOME 'skills' } else { Join-Path $env:USERPROFILE '.codex/skills' }
New-Item -ItemType Directory -Path $skillRoot -Force | Out-Null
git clone https://github.com/eric2100/codex-backup-restore.git (Join-Path $skillRoot 'codex-backup-restore')
```

請在支援本機技能的 Codex 環境使用。若目前對話尚未載入新技能，可重新開啟對話並確認技能已列出。

可直接提出：

```text
使用 $codex-backup-restore 幫我備份 Codex，先確認來源與目的地，再驗證備份。
使用 $codex-backup-restore 從我指定的備份還原，保全現況並完成資料與 App 功能驗證。
```

## 手動驗證工具

先取得來源穩定、不再寫入的離線快照。以下路徑全部是相對於目前目錄的虛構範例；`audit` 必須在受驗證的快照／還原目錄之外。

```powershell
python scripts/verify_snapshot.py inventory --root ./demo/snapshot --manifest ./audit/manifest.json
python scripts/verify_snapshot.py verify --root ./demo/restored --manifest ./audit/manifest.json --report ./audit/files.json
python scripts/verify_snapshot.py sqlite --root ./demo/restored --report ./audit/databases.json
```

指令、返回碼與限制請見 [驗證工具文件](docs/verification.md)。舊備份缺來源清單時，替備份建立的清單只能證明還原與該備份相符，不能證明當初備份沒有漏資料。

## 文件

- [技能入口](SKILL.md)：agent 執行原則與成功判定。
- [Windows 備份／還原流程](references/windows-workflow.md)：範圍、離線工作、sidecar、排程、重試與回復。
- [匿名故障排除](references/troubleshooting.md)：已觀察到的失敗及處理方式。
- [隱私與資料處理](docs/privacy.md)：哪些資料絕對不應公開，以及回報問題時如何匿名化。
- [作者與 AI 揭露](AUTHORSHIP.md)。

## 開發與測試

```powershell
python -m unittest discover -s tests -v
```

測試只使用臨時目錄、合成文字及合成 SQLite 資料庫，不讀取真實 Codex 資料。CI 在 Windows／Linux 與 Python 3.10／3.12 執行。Linux 測試只驗證通用工具，不代表 Windows App 還原已實測。

提交 issue／PR 前請閱讀 [CONTRIBUTING.md](CONTRIBUTING.md)。不要附真實資料庫、完整 manifest、狀態檔或登入資料。
