# 流程圖文件 (Flowchart)

根據 PRD 與 架構文件的規劃，本文件將系統互動轉化為視覺化的使用者流程與系統流程，以協助開發前確認邏輯順序無誤。

## 1. 使用者流程圖 (User Flow)

這張圖展示了使用者進入「CS2 賽事數據與飾品投資分析系統」後的可能操作路徑與頁面跳轉邏輯。

```mermaid
flowchart LR
    Start([使用者開啟網站]) --> Home[首頁 - 賽事與市場總覽]
    Home --> Nav{選擇主要功能}

    Nav -->|瀏覽賽事| ES[電競賽事中心]
    ES --> ES1[查詢戰隊與選手近期戰績]
    
    Nav -->|瀏覽飾品| Market[飾品市場分析]
    Market --> MK1[觀看各飾品漲跌幅排行]
    MK1 --> MK2[點入查看單一飾品歷史走勢圖]
    
    Nav -->|社群交流| Community[玩家社群討論版]
    Community --> CM1[參與即將到來的賽事預測投票]

    Nav -->|個人資產管理| Auth{是否已登入？}
    Auth -->|否| Log[註冊 / 登入頁面]
    Log -->|成功| Auth
    
    Auth -->|是| Profile[個人投資組合管理]
    Profile --> P1[新增持有飾品與買入價位]
    Profile --> P2[持續追蹤目前帳面市值與損益]
    Profile --> P3[設定飾品指定目標到價提醒]
```

---

## 2. 系統序列圖 (Sequence Diagram)

這裡以系統最核心且複雜的動作之一：「**登入狀態下的使用者，新增一筆個人投資資產**」為例，展示 MVC 架構中資料流動的順序。

```mermaid
sequenceDiagram
    actor User as 使用者
    participant Browser as 瀏覽器 (HTML/JS)
    participant Flask as Flask (Route)
    participant Model as Model (Items&User)
    participant DB as SQLite 資料庫

    User->>Browser: 在資產頁填寫飾品買入資訊並送出表單
    Browser->>Flask: POST /portfolio/add (傳遞表單 Data)
    
    Flask->>Flask: 驗證 Session 判斷是否為登入狀態<br/>並驗證表單欄位格式
    
    Flask->>Model: 呼叫 add_portfolio_item(userID, itemID, price)
    Model->>DB: 執行 INSERT INTO portfolios ...
    DB-->>Model: 回傳成功狀態與新增資料 ID
    Model-->>Flask: 回報資料寫入完成
    
    Flask-->>Browser: HTTP 302 重導向到 /portfolio (資產總覽)
    Browser->>Flask: GET /portfolio
    
    Flask->>Model: 查詢該使用者最新的投資組合與損益
    Model->>DB: SELECT * FROM portfolios WHERE user_id=?
    DB-->>Model: 回傳計算後的總資產資料
    Model-->>Flask: 準備渲染資料
    
    Flask->>Flask: 套用 Jinja2 模板渲染 HTML
    Flask-->>Browser: 回傳夾帶最新資產與損益的網頁畫面
```

---

## 3. 功能清單對照表

將上述操作路徑具體對應到未來的 Flask Router 規劃中，下表列出了應用程式預計開發的端點 (Endpoints) 以及各 HTTP 請求方法所做的事情。

| 功能名稱 | URL 路徑 (建議) | HTTP 方法 | 說明與用途 |
| -------- | --------------- | --------- | ---- |
| 首頁總覽 | `/` | GET | 顯示近期重點賽果與市場價格異動較大的頭條 |
| 註冊帳號 | `/auth/register` | GET/POST | 顯示註冊表單(GET) / 驗證並建立新會員資料(POST) |
| 會員登入 | `/auth/login` | GET/POST | 顯示登入表單(GET) / 驗證帳密並發配 Session(POST) |
| 會員登出 | `/auth/logout` | GET | 清除當前 Session，導回首頁 |
| 戰績查詢 | `/esports/teams` | GET | 列出職業戰隊及其近期相關的賽事戰績 |
| 飾品排行 | `/market` | GET | 呈現全站飾品價格漲跌幅排行榜 |
| 歷史圖表 | `/market/item/<id>` | GET | 顯示單一飾品詳細資料，並提供繪製成歷史走勢圖的數據 |
| 資產總覽 | `/portfolio` | GET | 檢視登入會員擁有的所有飾品總價值與當前帳面損益 |
| 新增資產 | `/portfolio/add` | POST | 在資料庫建立一筆使用者的飾品買入記錄（價格、數量等） |
| 到價提醒 | `/portfolio/alerts` | GET/POST | 列出既有提醒(GET) / 設定指定飾品到達特定價格的通知條件(POST) |
| 討論版列表 | `/community` | GET | 查看最新玩家討論文章與目前開放預測的賽事列表 |
| 參與預測 | `/community/predict/<match_id>` | POST | 送出使用者針對某一場賽事的勝負預測結果 |
