# Flask Server 後端說明

## 專案目標

本 Flask Server 作為中介層，負責接收前端的 HTTP 請求，並透過 TCP/IP 協定與 Dobot 機械臂進行通訊，實現對機械臂的遠端控制。

## 核心功能需求

### 1. Dobot 連線管理

#### 功能說明
- 建立與 Dobot 機械臂的 TCP/IP 連線
- 管理三個端口的連線狀態：
  - **Dashboard Port (29999)**: 控制端口
  - **Move Port (30003)**: 運動端口
  - **Feed Port (30004)**: 回饋端口
- 實現連線池管理，避免重複建立連線
- 處理連線異常與自動重連機制

#### 實作要點
- 使用 Dobot API (`dobot_api.py`) 中的類別
- 在伺服器啟動時建立連線
- 提供連線狀態檢查功能

### 2. Web 控制介面

#### 功能需求：簡易控制頁面
建立一個 HTML 頁面，提供基本的機械臂控制功能：

**頁面元素：**
- **標題**: 顯示 "Dobot 機械臂控制面板"
- **連線狀態指示器**: 顯示與 Dobot 的連線狀態（已連線/未連線）
- **Enable Button (使能按鈕)**:
  - 功能: 使能機械臂（啟動馬達電源）
  - 按下後呼叫後端 API `/api/robot/enable`
  - 顯示操作結果（成功/失敗訊息）

- **Disable Button (去使能按鈕)**:
  - 功能: 去使能機械臂（關閉馬達電源）
  - 按下後呼叫後端 API `/api/robot/disable`
  - 顯示操作結果（成功/失敗訊息）

**互動流程：**
1. 使用者開啟網頁
2. 頁面自動檢查 Dobot 連線狀態
3. 使用者點擊 Enable 按鈕
4. 前端發送 POST 請求至後端 API
5. 後端透過 Dobot API 呼叫 `dashboard.EnableRobot()`
6. 回傳執行結果至前端
7. 前端顯示操作成功或失敗訊息

**技術要求：**
- 使用 Flask 渲染 HTML 模板
- 使用 JavaScript (Fetch API 或 AJAX) 進行非同步請求
- 實現即時回饋，不需重新整理頁面

### 3. RESTful API 設計

#### 必要 API 端點

**機器人控制類**

```
POST   /api/robot/enable
功能: 使能機械臂
請求: 無需參數
回應: {"status": "success"/"error", "message": "..."}
```

```
POST   /api/robot/disable
功能: 去使能機械臂
請求: 無需參數
回應: {"status": "success"/"error", "message": "..."}
```

```
GET    /api/robot/status
功能: 取得機械臂當前狀態
回應: {
  "connected": true/false,
  "enabled": true/false,
  "position": {"x": 0, "y": 0, "z": 0, "r": 0},
  "error": null/"錯誤訊息"
}
```

**系統狀態類**

```
GET    /api/health
功能: 健康檢查，確認伺服器運作正常
回應: {"status": "ok", "timestamp": "..."}
```

```
GET    /api/connection/check
功能: 檢查與 Dobot 的連線狀態
回應: {
  "dashboard": true/false,
  "move": true/false,
  "feed": true/false
}
```

### 4. 錯誤處理機制

#### 需處理的錯誤情境

**連線錯誤**
- Dobot 未開機或網路不通
- IP 位址設定錯誤
- TCP 連線逾時

**操作錯誤**
- 機械臂已使能狀態下重複 Enable
- 機械臂未使能狀態下執行運動指令
- 機械臂發生報警或錯誤

**回應格式**
- 所有錯誤都應回傳標準格式的 JSON
- 包含錯誤代碼、錯誤訊息、建議處理方式

### 5. 日誌記錄

#### 記錄內容
- 所有 API 請求與回應
- Dobot 連線建立與中斷事件
- 機械臂狀態變更（Enable/Disable）
- 錯誤與異常事件

#### 記錄格式
- 時間戳記
- 請求來源 IP
- 操作類型
- 執行結果
- 錯誤訊息（若有）

## 專案結構規劃

```
server/
├── app.py                      # Flask 主應用程式入口
├── config.py                   # 設定檔（Dobot IP、Port 等）
├── requirements.txt            # Python 相依套件清單
│
├── routes/                     # API 路由
│   ├── __init__.py
│   ├── robot.py               # 機器人控制相關 API
│   └── system.py              # 系統狀態相關 API
│
├── controllers/                # 業務邏輯控制器
│   ├── __init__.py
│   └── robot_controller.py    # 機器人控制邏輯
│
├── services/                   # Dobot 通訊服務
│   ├── __init__.py
│   ├── dobot_connection.py    # Dobot 連線管理
│   └── dobot_service.py       # Dobot 操作封裝
│
├── templates/                  # HTML 模板
│   └── index.html             # 控制介面頁面
│
├── static/                     # 靜態資源
│   ├── css/
│   │   └── style.css          # 樣式表
│   └── js/
│       └── control.js         # 前端互動邏輯
│
├── utils/                      # 工具函式
│   ├── __init__.py
│   ├── logger.py              # 日誌工具
│   └── response.py            # API 回應格式化
│
└── README.md                   # 本說明文件
```

## 技術堆疊

- **Web 框架**: Flask 2.x
- **Dobot API**: 使用專案中的 `Dobot/TCP-IP-4Axis-Python-main/dobot_api.py`
- **前端**: HTML5, CSS3, JavaScript (Vanilla JS 或 jQuery)
- **日誌**: Python logging 模組
- **設定管理**: python-dotenv (可選)

## 環境設定

### 必要套件
```
Flask>=2.0.0
numpy>=1.20.0
python-dotenv>=0.19.0
```

### Dobot 連線資訊
- **IP 位址**: 192.168.1.6 (LAN1)
- **Dashboard Port**: 29999
- **Move Port**: 30003
- **Feed Port**: 30004

### 網路需求
- 伺服器與 Dobot 必須在同一區域網路 (192.168.1.x)
- 確保防火牆允許 TCP 連線至上述端口
- 建議伺服器監聽 `0.0.0.0:5000` 以允許區域網路存取

## 執行流程

### 啟動步驟
1. 確認 Dobot 已開機並連線至 LAN1 (192.168.1.6)
2. 設定電腦 IP 為 192.168.1.x 網段
3. 安裝 Python 相依套件
4. 啟動 Flask 伺服器
5. 開啟瀏覽器存取控制介面

### 操作流程
1. 使用者開啟 Web 介面 (http://伺服器IP:5000)
2. 系統自動檢查 Dobot 連線狀態
3. 使用者點擊 Enable 按鈕啟動機械臂
4. 系統回傳執行結果
5. 使用者可繼續進行其他操作
6. 完成後點擊 Disable 按鈕關閉機械臂

## 安全性考量

### 存取控制
- 建議實作簡易的認證機制（基本認證或 Token）
- 限制只允許區域網路存取
- 記錄所有操作日誌以供稽核

### 操作安全
- Enable/Disable 操作需要確認機制
- 提供緊急停止功能
- 操作前檢查機械臂狀態
- 防止重複或衝突的指令

### 錯誤處理
- 完善的例外處理機制
- 清楚的錯誤訊息回饋
- 自動重連機制（連線中斷時）

## 測試需求

### 功能測試
- [ ] Dobot 連線建立成功
- [ ] Enable 功能正常運作
- [ ] Disable 功能正常運作
- [ ] 狀態查詢正確回傳
- [ ] 錯誤處理正確執行

### 整合測試
- [ ] 前端按鈕與後端 API 整合
- [ ] Dobot API 呼叫正常
- [ ] 日誌記錄完整

### 壓力測試
- [ ] 快速連續點擊按鈕的處理
- [ ] 多使用者同時存取（若需要）
- [ ] 長時間運行穩定性

## 後續擴充功能

### 運動控制
- 座標輸入介面（X, Y, Z, R）
- MovJ / MovL 運動指令
- 圓弧與整圓運動控制
- 點動控制（Jog）

### 監控功能
- 即時位置顯示
- 關節角度監控
- 錯誤與報警即時通知
- 運動軌跡視覺化

### 進階功能
- 路徑規劃與執行
- 多點位儲存與播放
- 速度與加速度調整
- 工具座標系設定

## 開發注意事項

1. **先實現基本功能**: 優先完成 Enable/Disable 功能，確保穩定後再擴充
2. **模組化設計**: 保持程式碼結構清晰，方便後續維護
3. **錯誤處理**: 每個 API 都需要完善的例外處理
4. **日誌記錄**: 充足的日誌有助於除錯
5. **安全優先**: 操作機械臂前務必確保安全
6. **測試驗證**: 每個功能都需經過測試才能部署

## 參考資源

- Dobot API 文件: `../Dobot/TCP-IP-4Axis-Python-main/README.md`
- Dobot API 範例: `../Dobot/TCP-IP-4Axis-Python-main/PythonExample.py`
- Flask 官方文件: https://flask.palletsprojects.com/
- Dobot 官方協定: https://github.com/Dobot-Arm/TCP-IP-Protocol
