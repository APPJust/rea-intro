# 本片實測紀錄：REA analyze-javascript-application

- 日期：2026-10-17
- 版本：rea-agents 6.5.0（`npx -y rea-agents@latest --version`）
- 環境：Linux 雲端容器，Node.js 22.22.0，未使用任何原生反編譯引擎
- 指令：`npx -y rea-agents@latest analyze-javascript-application ./demoapp --json`
- 結果：exit 0，輸出 255,927 bytes 的 JSON

## 測試對象（自製 4 檔 Electron App）

- `package.json`：`{"name":"demo-notes","main":"main.js"}`
- `main.js`：`ipcMain.handle('notes:search', (e, q) => search.find(q))`，建立 BrowserWindow 並指定 preload
- `preload.js`：`contextBridge.exposeInMainWorld('notes', { search: (q) => ipcRenderer.invoke('notes:search', q) })`
- `search.js`：`exports.find = (q) => [...]`

## REA 找到的節點（節錄）

context-bridge-api `notes` → electron-preload `preload.js`（`ipcRenderer.invoke`）→ ipc-channel `notes:search` → ipc-handler `notes:search`（`main.js`）→ javascript-module `search.js:find`；另有 electron-main、browser-window 節點。每個節點附 artifact sha256、位置與信心等級。

## REA 自行附上的限制（原文節錄）

- "JavaScript and HTML were parsed as inert text; bundle bootstrap code was never executed."
- "Electron relationships are derived from inert syntax; runtime registration, reachability, defaults, and enforcement remain unproven."
- "… missing findings do not establish absence."
