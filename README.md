# 桌面麻糬 (Desktop Mochi) 技術探索筆記

## 專案目標

記錄「桌面麻糬」這類桌寵應用的跨平台實作可行性研究，目前處於前期技術調查階段，後續會進行實驗驗證。

## 背景：Electron 的做法

網路上常見的桌寵教學使用 Electron，關鍵 API：

- `BrowserWindow` 設定 `{ frame: false, transparent: true }` → 無框透明視窗
- `win.setIgnoreMouseEvents(true, { forward: true })` → 滑鼠事件穿透，不干擾使用者操作底下的視窗

優點：一行設定就能達成跨平台透明 + 點擊穿透。

## 主題：Flutter 是否做得到？

結論：**可以，但比 Electron 麻煩，且各平台支援度不一。**

### Windows

- 無邊框視窗：可用 `bitsdojo_window` 或 `window_manager` 套件
- 背景透明：需在 C++ runner 端處理 `WM_NCCALCSIZE`、啟用 DWM 合成，Flutter 端 `scaffoldBackgroundColor` 設為 `Colors.transparent`
- 滑鼠穿透：需呼叫 Win32 API `SetWindowLong`，加上 `WS_EX_LAYERED | WS_EX_TRANSPARENT` 旗標。沒有現成 plugin，需自寫或用 FFI

### macOS

- 修改 `MainFlutterWindow.swift`：
  - `isOpaque = false`
  - `backgroundColor = .clear`
  - `styleMask` 移除 `.titled`
- 滑鼠穿透：`ignoresMouseEvents = true`

### Linux

- 最不穩定，依賴 compositor，不同桌面環境（GNOME / KDE / Xfce）行為不一致，常見閃爍與殘影

## Flutter vs Electron 核心差異

Flutter desktop 的渲染管線（Skia / Impeller）直接畫到視窗 surface，因此「透明」「點擊穿透」這類視窗層級功能必須回到各平台原生 API，Flutter 官方沒有統一的跨平台 API。Electron 則因為底層是 Chromium + Node，視窗管理已經抽象化，一個設定全平台通用。

## 建議

- 只做 Windows：Flutter 完全可行
- 跨三平台一份程式碼：Electron 仍是輕鬆選擇
- 後續實驗計劃（待補）
