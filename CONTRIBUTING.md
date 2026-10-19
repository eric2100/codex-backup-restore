# 貢獻方式

1. 使用合成資料重現問題；真實 Codex home、資料庫、憑證與日誌不應進入測試或 PR。
2. 調整後執行 `python -m unittest discover -s tests -v`。
3. 與 WAL、檔案複製或清單格式相關的改動，應加入能觀察實際結果的回歸測試。
4. 說明測試範圍；一般工具測試通過不等於實際 App 還原成功。
5. 審查 `git diff --cached` 與提交身分，確認沒有個資或憑證再送出。

請保留 [AI 撰寫揭露](AUTHORSHIP.md) 與 [MIT 授權](LICENSE)。
