# GPS AR Prototype

一個**完全獨立**的實驗性單頁應用，用來測試「GPS 座標 + 手機羅盤方位」疊加在相機畫面上的可行性與精確度。

不依賴任何框架、不需要後端、不需要資料庫。所有登記的地標都儲存在瀏覽器的 `localStorage`，重新整理或重開瀏覽器仍會保留。

> 這是一個全新獨立的專案（獨立資料夾、獨立 git 歷史），與其他既有專案（例如 Vec、flag-system）完全無關，不會共用任何檔案或服務。

## 功能

- **登記模式**：按下按鈕取得目前 GPS 座標（`enableHighAccuracy: true`），輸入標籤文字後存成一筆地標，並可從列表中刪除。
- **檢視模式（AR 疊加）**：
  - 開啟後鏡頭作為全螢幕背景（`getUserMedia`）
  - 持續讀取 GPS（`watchPosition`）
  - 讀取裝置朝向：
    - iOS 用 `webkitCompassHeading`，權限請求（`DeviceOrientationEvent.requestPermission()`）綁在按鈕點擊事件中
    - Android 優先用 `deviceorientationabsolute`（`(360 - alpha) % 360`），若裝置不支援絕對方位則退回一般 `deviceorientation` 的 `alpha`，並在畫面上顯示警告
  - 依 haversine 公式計算距離、依大地座標公式計算方位角，只顯示落在可調式 FOV（預設 65°，建議 20–120°）視角範圍內的地標
  - 畫面下方有即時診斷資訊區塊（monospace、半透明黑底），列出目前 GPS 精度、羅盤角度與來源、每個地標的方位角/距離/相對夾角/是否在畫面內

## 本機測試（需要 HTTPS 或 localhost）

`getUserMedia`、`geolocation`、`DeviceOrientationEvent` 這些 API 在瀏覽器中只允許在 **HTTPS** 或 **localhost** 環境下使用，直接用 `file://` 開啟會被瀏覽器擋掉。

在專案資料夾內啟動一個靜態伺服器即可：

```bash
cd gps-ar-prototype
npx serve .
```

啟動後瀏覽器打開 `http://localhost:3000`（或終端機顯示的網址）即可測試登記模式。

在桌機瀏覽器用 `localhost` 就能測試登記模式的 UI 邏輯，但檢視模式（相機 + 羅盤）建議用實體手機測試：

- 手機和電腦在同一個 Wi-Fi 下，`npx serve .` 通常會同時印出一個區域網路網址（如 `http://192.168.x.x:3000`），但**手機瀏覽器對 camera / geolocation 的安全限制通常要求 HTTPS**（`localhost` 例外只對開發機本身有效）。
- 若要在手機上測試，建議用下方「部署到 GitHub Pages」的方式取得一個 HTTPS 網址，或使用 `npx serve . --ssl-cert ... --ssl-key ...` / [ngrok](https://ngrok.com/) 等工具建立本機 HTTPS 通道。

其他靜態伺服器一樣可用，例如：

```bash
python3 -m http.server 8000
```

## 部署（全新獨立服務，不會動到既有專案）

### 方式一：GitHub Pages

1. 將這個資料夾 push 到一個**全新的** GitHub repo（例如 `gps-ar-prototype`），不要 push 進既有 repo 的分支或子目錄。
2. 到該 repo 的 Settings → Pages，Source 選擇 `main` 分支、根目錄 `/`。
3. 幾分鐘後即可透過 `https://<你的帳號>.github.io/gps-ar-prototype/` 存取，這是獨立網址，不影響任何既有服務。

### 方式二：Railway（Static Site）

1. 在 Railway 建立一個**全新專案**（New Project → Deploy from GitHub repo），選擇這個獨立的 `gps-ar-prototype` repo。
2. 因為是純靜態檔案，可以用任意靜態伺服器 buildpack，或簡單加一個 `package.json` 搭配 `npx serve .` 作為 start command。
3. 部署完成後 Railway 會給一個獨立的網域（`*.up.railway.app`），與你其他既有的 Railway 服務完全分開，不會互相影響。

## 注意事項

- 這是實驗性原型，UI 走「戶外陽光下可操作」的高對比大按鈕風格，沒有做美術設計。
- 羅盤方位在不同裝置/瀏覽器上的支援程度差異很大，診斷區塊會明確標示目前使用的資料來源，方便記錄與比對誤差。
- 地標資料只存在單一裝置的 `localStorage` 裡，不會同步到雲端或其他裝置。
