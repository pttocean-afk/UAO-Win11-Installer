# UAO 2.50 — Windows 11 離線安裝工具

> 透過 Windows 修復環境（WinRE）離線置換 C_950.NLS 字碼表，解決舊式 ERP 在 Windows 11 上中文字變成 `?` 的問題。

[![Windows](https://img.shields.io/badge/Windows-11-0078D6?logo=windows&logoColor=white)](https://github.com/pttocean-afk/UAO-Win11-Installer)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Stars](https://img.shields.io/github/stars/pttocean-afk/UAO-Win11-Installer?style=flat)

---

## 📋 目錄

- [為什麼需要這個工具](#為什麼需要這個工具)
- [運作原理](#運作原理)
- [安裝方式](#安裝方式)
- [解除安裝／還原](#解除安裝還原)
- [工具功能](#工具功能)
- [檔案結構](#檔案結構)
- [測試環境](#測試環境)
- [常見問題](#常見問題)
- [安全與風險說明](#安全與風險說明)
- [⚠️ 免責聲明](#️-免責聲明)
- [License](#license)

---

## 為什麼需要這個工具

臺灣仍有許多舊式 ERP 系統依賴 Big5／CP950 編碼。在 Windows 11 25H2 上，這些 ERP 輸入部分中文字（如「堃」、「湶」等）時會變成 `?`。

**傳統 UAO 安裝器**的問題：Windows 11 25H2 會阻止正在使用中的 `C_950.NLS` 被覆蓋，即使以系統管理員身分執行，舊安裝器也可能沒有實際生效。

**本工具**改採 Windows 修復環境（WinRE）離線置換——在 Windows 尚未載入 CP950 字碼表時完成安裝，確保字碼表確實被替換。

## 運作原理

```mermaid
flowchart LR
    A[執行安裝器] --> B[釋出 UAO 字碼表<br/>+ 建立 WinRE 腳本]
    B --> C[備份原始 C_950.NLS]
    C --> D[重新啟動至 WinRE]
    D --> E[離線置換字碼表]
    E --> F[重新進入 Windows]
    F --> G[ERP 可正常輸入<br/>原本缺的字]
```

## 安裝方式

### 需要的檔案

只需要 **UAO-Win11-Installer.exe**。字碼表與離線安裝腳本都已內嵌在 EXE 中。

### 安裝步驟

1. **儲存並關閉**正在進行的工作
2. 以系統管理員身分執行 `UAO-Win11-Installer.exe`
3. Windows 詢問是否允許變更裝置時，選擇「**是**」
4. 按「**1. 準備安裝檔與備份**」
5. 顯示準備完成後，按「**2. 重開到修復選單**」
6. 電腦重新啟動後，依序選擇：
   ```
   疑難排解 → 進階選項 → 命令提示字元
   ```
7. 在黑色命令提示字元視窗輸入：
   ```cmd
   C:\u
   ```
8. 如果顯示找不到檔案，改輸入：
   ```cmd
   D:\u
   ```
9. 畫面顯示 **`SUCCESS - UAO 2.50 installed.`** 代表安裝成功
10. 按任意鍵後輸入 `exit`，選擇繼續進入 Windows
11. 開啟 ERP，測試原本無法輸入的字（例如「堃」）

## 解除安裝／還原

1. 再次執行 `UAO-Win11-Installer.exe`
2. 按「**解除 UAO／準備還原原始字碼表**」
3. 按「**2. 重開到修復選單**」
4. 進入 WinRE 命令提示字元
5. 輸入：
   ```cmd
   C:\r
   ```
6. 如果找不到檔案，改用 `D:\r`
7. 顯示 **`SUCCESS - original Windows CP950 files restored.`** 代表還原成功
8. 按任意鍵後輸入 `exit`，選擇繼續進入 Windows

## 工具功能

安裝器具備以下安全機制：

| 功能 | 說明 |
|------|------|
| 🔐 系統管理員權限檢查 | 未以管理員執行時自動阻止 |
| ✅ 字碼表完整性驗證 | 安裝前比對內嵌 UAO 字碼表的 MD5 |
| 📦 原始檔案備份 | 自動備份 `System32` 與 `SysWOW64` 的原始 `C_950.NLS` |
| 🛡️ 自動還原保護 | 安裝失敗時自動嘗試恢復原始字碼表 |
| 🔄 重複執行防護 | 避免已安裝的 UAO 被誤存成「原始備份」 |
| 🔍 二進位驗證 | 離線安裝後比對來源與目標檔案是否一致 |

## 檔案結構

安裝後產生的檔案：

```
C:\
├── u.cmd                         離線安裝指令
├── r.cmd                         離線還原指令
└── UAO-Win11\
    ├── C_950.UAO.NLS              UAO 2.50 字碼表（196,642 bytes）
    └── Backup-YYYYMMDD-HHMMSS\
        ├── C_950.System32.original.NLS
        └── C_950.SysWOW64.original.NLS
```

## 測試環境

| 項目 | 內容 |
|------|------|
| 作業系統 | Windows 11 Pro 25H2 |
| OS Build | 26200.8655 |
| 架構 | x64 |
| 系統地區 | 中文（繁體，台灣）／CP950 |
| ERP 測試 | 可正常輸入「堃」（原顯示為 `?`） |

> ⚠️ 目前僅在 Windows 11 25H2 x64 實測。不應宣稱已支援所有 Windows 11 組建。

## 常見問題

**Q: 安裝時顯示 ERROR 怎麼辦？**
A: 拍下錯誤畫面，**不要重複執行安裝**，聯繫開發者。

**Q: 修復環境要求 BitLocker 復原金鑰？**
A: 必須先輸入金鑰解鎖系統碟才能繼續。

**Q: Windows 更新後又缺字了？**
A: Windows 大型版本更新可能還原 CP950 系統字碼表，重新安裝即可。

**Q: 已經變成 `?` 的資料能救回來嗎？**
A: 不行。已經被 ERP 儲存成 `?` 的資料無法藉由安裝 UAO 自動復原。

**Q: 工具會被防毒軟體擋嗎？**
A: 工具使用標準 C# 編譯，不會壓縮或混淆，已通過 Defender 掃描為「未發現威脅」。但未使用商業 Code Signing 憑證，SmartScreen 可能顯示「未知發行者」。

## 安全與風險說明

`C_950.NLS` 是 Windows 會載入的系統字碼表，修改存在實際風險：

- 覆蓋期間斷電或儲存裝置故障，可能使 Windows 無法正常開機
- BitLocker／裝置加密啟用時，WinRE 可能要求復原金鑰
- Windows 大型版本更新或系統修復可能還原微軟原始字碼表，使 UAO 失效
- UAO 使用 Big5 延伸映射，不同舊系統若使用不同外字映射，資料交換可能出現字義不一致
- 建議先在測試機驗證，再部署至正式 ERP 電腦

### 失敗保護

若離線畫面顯示：

```
CRITICAL - automatic restore failed. Do not restart.
```

**應停止重新啟動並進行人工復原。**

---

## ⚠️ 免責聲明

本工具會修改 Windows 系統核心字碼表檔案（`C_950.NLS`），屬於**高風險操作**。

**使用本工具所產生的任何風險由使用者自行承擔。** 包括但不限於：

- 系統無法開機或無法正常運作
- 資料遺失或損毀
- ERP 或其他應用程式相容性問題
- 違反企業 IT 安全政策或軟體授權條款
- BitLocker 或其他加密機制導致的資料無法存取

**建議：**
1. 在生產環境部署前，務必先在測試機上驗證
2. 操作前備份所有重要資料
3. 確保擁有 BitLocker 復原金鑰（如啟用加密）
4. 企業用戶應先諮詢 IT 管理人員

作者不對因使用本工具所造成的任何直接或間接損失負責。

---

## 檔案校驗

### UAO-Win11-Installer.exe

| 項目 | 值 |
|------|-----|
| 大小 | 210,944 bytes |
| SHA-256 | `BC7CFA260AC91A49E7F42F145229674F5F43BC9931AAA32EFED5E182CC35C845` |

### UAO 2.50 C_950.NLS（內嵌字碼表）

| 項目 | 值 |
|------|-----|
| 大小 | 196,642 bytes |
| MD5 | `7F69955CE72BA9F187A686A637B8FFDD` |
| SHA-256 | `D9BDEB71758FC3B459B48529DDF041756A0A464D61FB84EA961CD54AD6941D4F` |

---

## 相關文件

- [專案摘要 (PROJECT_SUMMARY.md)](UAO-Win11-PROJECT-SUMMARY.md) — 開發者用詳細資訊
- [安裝說明 (README.txt)](UAO-Win11-README.txt) — 純文字版操作說明

## License

MIT
