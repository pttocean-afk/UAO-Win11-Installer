# UAO 2.50 Windows 11 Installer

## 專案目的

讓仍依賴 Big5／CP950 的舊式 ERP 在 Windows 11 上使用 Unicode 補完計畫（UAO）2.50，解決部分中文字輸入後變成 `?` 的問題，例如「堃」。

傳統 UAO 安裝器會在 Windows 執行期間直接取得 `C_950.NLS` 權限並覆蓋系統檔。Windows 11 25H2 會阻止正在使用中的 NLS 字碼表被覆蓋，因此舊安裝器即使以系統管理員身分執行，也可能沒有實際生效。

本工具改採 Windows 修復環境（WinRE）離線置換，在 Windows 尚未載入 CP950 字碼表時完成安裝。

## 發布檔案

- `UAO-Win11-Installer.exe`：Windows 11 圖形化準備工具。
- `UAO-Win11-README.txt`：一般使用者操作說明。
- `UAO-Win11-PROJECT-SUMMARY.md`：專案與發布資訊。

使用者只需要取得 EXE 與 README。UAO 字碼表及離線安裝／還原腳本均已內嵌在 EXE。

## 已確認的測試環境

- Windows 11 Pro 25H2
- OS Build：26200.8655
- x64
- 非 Unicode 程式系統地區：中文（繁體，台灣）／CP950
- 實測舊 ERP 可正常輸入原本會變成 `?` 的「堃」

## 字碼表資訊

工具使用 UAO 2.50 的 `C_950.NLS`：

- 檔案大小：196,642 bytes
- MD5：`7F69955CE72BA9F187A686A637B8FFDD`
- SHA-256：`D9BDEB71758FC3B459B48529DDF041756A0A464D61FB84EA961CD54AD6941D4F`

安裝前會驗證內嵌字碼表的 MD5，驗證失敗時停止操作。

## 安裝器功能

1. 要求 Windows 系統管理員權限。
2. 從 EXE 釋出並驗證 UAO 2.50 字碼表。
3. 備份以下微軟原始字碼表：
   - `Windows\System32\C_950.NLS`
   - `Windows\SysWOW64\C_950.NLS`
4. 建立簡短的 WinRE 離線命令：
   - `C:\u.cmd`：安裝 UAO。
   - `C:\r.cmd`：還原微軟原始字碼表。
5. 可將電腦重新啟動至 Windows 修復選單。
6. 離線安裝後使用二進位比較驗證來源與目標檔案。
7. 安裝失敗時自動嘗試恢復原始字碼表。
8. 防止重複執行時將已安裝的 UAO 誤存成「原始備份」。

## 使用方式

1. 執行 `UAO-Win11-Installer.exe`。
2. 接受 Windows 的系統管理員權限提示。
3. 按「1. 準備安裝檔與備份」。
4. 按「2. 重開到修復選單」。
5. 依序選擇「疑難排解 → 進階選項 → 命令提示字元」。
6. 輸入：

   ```cmd
   C:\u
   ```

7. 如果修復環境將 Windows 分割區指派為 `D:`，改輸入：

   ```cmd
   D:\u
   ```

8. 顯示以下訊息代表安裝成功：

   ```text
   SUCCESS - UAO 2.50 installed.
   ```

9. 輸入 `exit`，選擇繼續進入 Windows，再以 ERP 測試缺字。

## 還原方式

1. 再次執行安裝器。
2. 按「解除 UAO／準備還原原始字碼表」。
3. 按「2. 重開到修復選單」。
4. 進入 WinRE 命令提示字元。
5. 輸入：

   ```cmd
   C:\r
   ```

6. 如果找不到檔案，改用 `D:\r`。
7. 顯示以下訊息代表還原成功：

   ```text
   SUCCESS - original Windows CP950 files restored.
   ```

## 建立的檔案與目錄

```text
C:\u.cmd
C:\r.cmd
C:\UAO-Win11\C_950.UAO.NLS
C:\UAO-Win11\Backup-YYYYMMDD-HHMMSS\C_950.System32.original.NLS
C:\UAO-Win11\Backup-YYYYMMDD-HHMMSS\C_950.SysWOW64.original.NLS
```

## 安全與風險說明

`C_950.NLS` 是 Windows 會載入的系統字碼表，修改存在實際風險：

- 覆蓋期間斷電或儲存裝置故障，可能使 Windows 無法正常開機。
- BitLocker／裝置加密啟用時，WinRE 可能要求復原金鑰。
- Windows 大型版本更新或系統修復可能還原微軟原始字碼表，使 UAO 失效。
- UAO 使用 Big5 延伸映射，不同舊系統若使用不同外字映射，資料交換可能出現字義不一致。
- 已經被 ERP 儲存成 `?` 的資料無法藉由安裝 UAO 自動復原。
- 建議先在測試機驗證，再部署至正式 ERP 電腦。

若離線畫面顯示：

```text
CRITICAL - automatic restore failed. Do not restart.
```

應停止重新啟動並進行人工復原。

## 防毒與程式簽章

新版工具使用標準 C#／.NET Framework 編譯：

- 不使用 AutoIt。
- 不壓縮或混淆執行檔。
- 不停用 Defender。
- 不隱藏執行命令。

目前成品已使用本機 Microsoft Defender Platform `4.18.26060.3008-0` 進行指定檔案掃描，結果為「未發現威脅」。

目前 EXE 未使用可信任的商業 Code Signing 憑證，因此其他電腦仍可能顯示 SmartScreen「未知發行者」。SmartScreen 聲譽提示不等同於 Defender 判定為病毒。正式公開發布時，建議使用受信任的程式碼簽章憑證簽署 EXE 並加入時間戳記。

## 目前成品校驗值

`UAO-Win11-Installer.exe`

- 大小：210,944 bytes
- SHA-256：`BC7CFA260AC91A49E7F42F145229674F5F43BC9931AAA32EFED5E182CC35C845`

發布前若重新編譯或簽章，檔案雜湊一定會改變，必須重新計算並更新 Release 說明。

## GitHub 發布建議

- 將 EXE 放在 GitHub Releases，不建議只提交二進位檔而沒有原始碼與建置說明。
- Repository 應包含來源碼、內嵌資源來源說明、建置步驟、授權條款及風險警告。
- 公開發布前確認 UAO 2.50 字碼表及相關資源的再散布授權。
- Release 頁面提供 SHA-256，方便使用者驗證下載內容。
- 建議標示目前僅在 Windows 11 25H2 x64 實測，不應宣稱已支援所有 Windows 11 組建。
- 建議先發布為 Pre-release，收集不同 Win11 組建、BitLocker 與 ERP 環境的測試結果。

## 目前限制

- 使用者仍需進入 WinRE 命令提示字元並輸入 `C:\u` 或 `D:\u`。
- 尚未完成所有 Windows 11 組建、ARM64、企業端安全政策與 BitLocker 組態測試。
- 尚未取得可信任的程式碼簽章。
- 尚未自動偵測 ERP 是 32 位元或 64 位元；目前同時處理 `System32` 與 `SysWOW64`。

