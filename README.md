# 桌面麻糬 (Desktop Mochi) 開發注意事項

開發桌寵類應用時，需要關注的功能細節與技術坑點筆記。

## 視窗行為

- **無邊框 + 透明背景**：必備。Electron 用 `frame: false, transparent: true`；Flutter / 原生需處理 alpha channel 與合成器設定
- **Always on top**：層級要選對。macOS 有 `NSWindow.level`（floating / statusBar / screenSaver…）；Windows 用 `SetWindowPos` 配 `HWND_TOPMOST`；要決定是否蓋過全螢幕應用（看遊戲時通常要讓開）
- **無焦點 (no focus / non-activating)**：寵物視窗不該搶鍵盤焦點，否則會打斷 IME、alt-tab、遊戲焦點。Electron `focusable: false`；Windows 用 `WS_EX_NOACTIVATE`；macOS `NSPanel` + `becomesKeyOnlyIfNeeded`
- **點擊穿透 (click-through)**：滑鼠事件穿過寵物視窗、操作底下視窗。Windows 需 `WS_EX_LAYERED | WS_EX_TRANSPARENT`，macOS 用 `ignoresMouseEvents = true`
- **動態切換穿透**：滑鼠在寵物 sprite 上時要能互動，離開時恢復穿透。需要先做 hit-test（透明像素 vs 不透明像素）
- **多螢幕 / DPI**：HiDPI 縮放、跨螢幕拖曳、解析度切換時的位置記憶
- **視窗尺寸**：是否跟著 sprite 動畫變化？貼齊螢幕邊緣？

## 互動

- 拖曳移動（按住寵物拖動）
- 點擊 / 雙擊 / 右鍵選單
- 滑鼠 hover 偵測（在點擊穿透模式下尤其麻煩）
- 系統托盤 / 選單列圖示（顯示/隱藏、設定、退出）
- 全域熱鍵（呼喚寵物回到游標旁、暫時隱藏）
- 防止寵物被拖到螢幕外消失
- **全域鍵鼠活動量監聽**（如 fluffy-desk 把它當成「精力值」來源）：macOS 需 Accessibility 權限、Windows 低階 hook (`SetWindowsHookEx`) 或 Raw Input、Linux X11 用 `XRecord` / Wayland 幾乎做不到。**屬於隱私敏感行為，必須揭露**
- **滑鼠軌跡 / 手勢辨識**（直線揮動、畫圈…）作為互動輸入，需設計取樣與雜訊容忍

## 動畫與渲染

- Sprite sheet / GIF / APNG / WebP / Live2D / Spine / Lottie 等格式取捨
- **行為驅動而非循環播放**：用狀態機 / 行為樹 / 簡易 AI 決定當下播什麼動畫（覓食、午睡、追游標、互相打鬧…），動畫只是 view 層
- 閒置動畫、互動動畫、狀態切換的過渡
- 幀率控制（閒置時降到 10–15 fps，互動時拉高）
- GPU 加速 vs CPU 繪製，避免不必要的合成
- Alpha 邊緣鋸齒、半透明像素在某些 compositor 上的閃爍

## 系統整合

- 開機自啟（macOS Login Items、Windows Startup folder / Registry、Linux autostart）
- 在 Mission Control / 工作切換 / 虛擬桌面的顯示行為
- 鎖定畫面、螢幕保護、睡眠喚醒後的狀態
- 全螢幕應用（遊戲、影片）時的避讓策略
- 通知中心 / 提醒整合（要不要讓寵物說話？）
- **系統指標讀取**（CPU / 記憶體 / 電池）作為 AI 輸入：Windows PDH、macOS `host_statistics` / `sysctl`、Linux `/proc/stat`；取樣頻率要低，避免桌寵自己拉高 CPU 形成正回饋

## 效能與資源

- 閒置時的 CPU / GPU / 記憶體佔用（桌寵會長時間開著，必須輕量）
- 筆電電池衝擊（背景動畫需可暫停或降頻）
- 視窗失焦 / 螢幕關閉時暫停渲染
- 啟動時間與安裝包大小

## 跨平台坑點

- **Windows**：DWM 合成、`WS_EX_LAYERED` 與 `transparent` 互斥的歷史問題；高 DPI 下座標換算
- **macOS**：notarization、沙盒限制、Apple Silicon 與 Intel 雙架構打包（universal binary 或分開發佈）；視窗 level 與 Stage Manager 互動；Accessibility 權限對話框 UX
- **macOS 自動更新陷阱**：每次自動更新後，系統會重新對 app bundle 套用 quarantine 屬性，使用者必須再次「在系統設定中允許」。fluffy-desk 已踩過這個雷；可考慮用 Sparkle 或在更新流程裡 `xattr -dr com.apple.quarantine` 處理（需有寫入權限）
- **Linux**：X11 vs Wayland 行為差異巨大；GNOME / KDE / Xfce 的 compositor 對透明與穿透支援不一；Wayland 下「always on top」需 layer-shell 協定；全域輸入監聽在 Wayland 幾乎無解

## 狀態與設定

- 位置 / 大小 / 顯示器記憶
- 寵物狀態（心情、飢餓、互動次數…）的持久化
- 使用者偏好（透明度、縮放、主題、語音開關）
- 多寵物 / 多實例：要決定是「同一程序開多視窗」還是「多個獨立 process」；多隻寵物之間的互動（追逐、避讓、繁殖）需要共用世界座標與訊息匯流

## 無障礙與 UX

- 不擋住關鍵 UI（工作列、選單列、系統提示）
- 寵物「迷路」時的快速召回機制
- 暫時隱藏熱鍵
- 色弱 / 動態敏感使用者的減少動畫選項

## 發佈與更新

- 程式碼簽章（macOS notarization、Windows SmartScreen / EV cert）
- **未簽章發佈的 UX**：fluffy-desk 直接走 GitHub Releases 不簽章，使用者第一次開啟會撞上 SmartScreen / Gatekeeper 警告，安裝指引必須清楚教學「右鍵開啟」、「仍要執行」等步驟
- 自動更新（electron-updater、Sparkle、Squirrel）；Windows 可背景靜默安裝，macOS 通常用通知 + 引導下載
- 安裝包格式：`.dmg` / `.pkg` / `.exe` / `.msi` / `.AppImage` / `.deb` / `.rpm`
- macOS 需考慮 universal binary，或對 Apple Silicon / Intel 各自打包並在更新通知中導向正確架構
- 資源熱更新（換皮、語音包）是否獨立於主程式

## 資產 pipeline

- 動畫格式選型（決定後續美術與工程成本）
- 音效播放（互動回饋、語音）的延遲與多平台後端
- 多語系 / 文案外掛

## 行為與 AI

- 狀態機 / 行為樹 / utility AI 的選型；狀態切換條件與優先序
- 輸入訊號：使用者鍵鼠活動量、CPU 負載、時間、寵物自身屬性（飢餓、體力）
- 多寵物互動規則（避免互相卡住、繁殖冷卻、群聚行為）
- 行為的可調參數應外部化，方便日後微調而不發版

## 隱私與權限揭露

- 全域鍵鼠監聽、CPU 取樣屬於敏感行為，必須在以下位置揭露：
  - 應用首次啟動的權限說明對話框
  - README / 官網 / 商店描述
  - 隱私政策（即使資料只在本機使用，也要明說「不上傳」）
- macOS 會強制跳出 Accessibility / Input Monitoring 權限對話框，UX 要事先鋪陳
- 預設關閉、由使用者主動開啟，是較安全的做法

## 參考案例

- [Codfisher/fluffy-desk](https://github.com/Codfisher/fluffy-desk)：跨平台桌寵（Windows / macOS）。觀察重點：「全透明、無焦點、滑鼠穿透、永遠最上層但不擋畫面」四件套；以鍵鼠活動量 + CPU 負載作為精力來源；macOS 自動更新後 quarantine 屬性會被重新套用導致需重新授權；未簽章透過 GitHub Releases 發佈；Apple Silicon / Intel 分包
