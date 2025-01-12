![CleanShot 2025-01-12 at 18.17.26](https://hackmd.io/_uploads/SkBa8zZPyg.png)

[Free Weather API](https://open-meteo.com/)

### 影片介紹與應用概述
- 目標是建置一個天氣應用程式，使用清晰的程式碼結構。
- 應用程式自動抓取使用者位置，顯示當地天氣資訊與數據，例如溫度、濕度、風速等。
- 擴展功能建議：地點搜尋、一週天氣預報等。
- 使用開放 API（不需認證）來抓取天氣數據。

---

### 主要架構與設計
- 採用清潔架構，分為三層：Presentation（UI）、Domain（核心邏輯）、Data（資料來源）。
- 不使用 Use Case 層，直接在 Repository 中處理邏輯，簡化範例流程。
- 使用 Retrofit 進行 API 通信，採用 Hilt 進行依賴注入。

---

### API 資訊與數據格式
- 使用的 API 提供各種天氣數據，包括溫度、濕度、壓力等。
- API 回傳的資料格式為 JSON，其中重要字段為 `hourly`，包含每小時天氣數據。
- 請求參數需提供經緯度（latitude、longitude）與所需的數據類型。

---

### DTO 與數據對應
- DTO（Data Transfer Object）用於定義 API 的 JSON 回應格式。
- 重點字段包括：`temperature_2m`（溫度）、`humidity_2m`（濕度）、`weather_code`（天氣代碼）。
- 使用 @Json 註解來對應 API 字段與 DTO 屬性名稱。

---

### Domain 模型與 Mapper
- Domain 模型為核心邏輯層，不應包含具體實現細節（如 Retrofit 或 Room）。
- 使用 Mapper 將 DTO 轉換為 Domain 模型，確保核心層的獨立性。
- Domain 模型定義的數據格式更易操作，例如將 JSON 時間字段轉換為 `LocalDateTime`。

---

### 資料層結構設計
- 將 API 介面放置於資料層的 `remote` 子包中。
- 使用 Retrofit 定義 API 請求函數，指定請求方法與參數。
- 資料層負責從 API 或其他資料來源獲取數據並轉換為 Domain 模型。

---

### Weather 資料模型設計
- `WeatherInfo`：包含每日天氣數據（以 Map 存儲）與當前小時的天氣數據。
- `WeatherData`：包含特定小時的詳細天氣數據，例如溫度、濕度與天氣類型。
- Domain 層的資料結構簡單易用，利於後續邏輯處理。

---

### 圖片與初始設置
- 提供圖標資源，用於展示不同天氣條件（如晴天、雷雨）。
- 初始化代碼可從 GitHub 倉庫下載，包括依賴項與基礎設置。

---
### 建立 DTO 與 Domain 模型的 Mapper
- 目標：將 DTO 資料物件轉換為 Domain 層物件，以利業務邏輯處理。
- Mapper 放置於 `data/mappers` 資料夾中，透過擴展函數實現物件間的映射。
- 第一個 Mapper 將 `WeatherDataDTO` 轉換為對應的 `WeatherData` 地圖，依據日期（鍵值）對應每小時的天氣數據。

---

### 處理時間與基本數據映射
- 從 DTO 提取溫度、壓力、風速、濕度等資訊，透過索引值對應各自的資料。
- 時間轉換為 `LocalDateTime` 格式，使用 ISO 時間格式解析。
- 提取天氣代碼，對應至天氣類型（`WeatherType`）以利後續應用。

---

### 建立索引型資料結構
- 定義輔助類別 `IndexedWeatherData`，包含索引與天氣資料，用於暫存與分組。
- 透過索引計算日期，將資料分組到對應天數。每個天數分為 24 個小時資料條目。
- 將索引型資料映射為普通的 `WeatherData`，以鍵值對方式分配。

---

### 生成每週天氣資料結構
- 映射生成的天氣數據地圖，鍵值範圍從 0 到 6，分別代表今天到未來第 6 天。
- 每日的天氣資料結構簡化了後續的業務邏輯操作。

---

### 映射 DTO 為 WeatherInfo
- `WeatherInfo` 包含每日天氣資料地圖與當前天氣資料。
- 計算當前時間的對應資料，將分鐘值小於 30 的時間向下取整，否則向上取整至下一小時。
- 找到與當前時間最接近的小時資料，並生成當前天氣物件。

---

### Repository 設計與抽象
- `WeatherRepository` 作為抽象接口，定義數據操作行為，支持未來輕鬆更換 API 或數據來源。
- 抽象接口位於 `domain/repository` 資料夾，實現位於 `data/repository` 資料夾。
- 方法 `getWeatherData` 接收經緯度參數，返回封裝的 `Resource<WeatherInfo>`。

---

### 錯誤處理與 API 呼叫
- 使用 `try-catch` 處理 API 呼叫過程中可能發生的例外情況。
- 捕獲異常後，返回錯誤資源（`Resource.Error`），包含錯誤訊息。
- 成功呼叫則將 API 結果映射為 `WeatherInfo`，包裹於成功資源（`Resource.Success`）。

---

### Dagger Hilt 依賴注入
- 透過 `@Inject` 註解於 Repository 實現類的構造函數，實現 API 依賴注入。
- 確保 `WeatherAPI` 實例能自動由 Dagger Hilt 提供與管理，簡化依賴管理流程。

### 定義 Location Tracker 介面
- 建立 `domain.location` 包中的 Location Tracker 介面
- 定義 `getCurrentLocation` 函式，並使用 `suspend` 關鍵字處理非同步操作
- 遵守 Clean Architecture 原則，避免直接使用 Android 框架的 `Location` 類別，應該定義自己的經緯度數據類

### 實作 DefaultLocationTracker
- 在 `data.location` 包中實作 `DefaultLocationTracker` 類別
- 依賴 `FusedLocationProviderClient` 和應用程式的 `Context`
- 檢查 `ACCESS_FINE_LOCATION` 和 `ACCESS_COARSE_LOCATION` 權限是否被授權
- 確認 GPS 或網路提供器是否啟用
- 使用 `suspendCancellableCoroutine` 將回調轉換為協程，取得當前位置

### 設置 Dagger Hilt DI
- 在根包下創建 `di` 包，並建立 Hilt 模組
- `AppModule`: 提供 Retrofit API 和 `FusedLocationProviderClient` 實例
- `LocationModule`: 使用 `@Binds` 提供 `DefaultLocationTracker` 實例
- `RepositoryModule`: 使用 `@Binds` 提供 `WeatherRepository` 實例
- 配置應用程式類別 `WeatherApp`，並添加 `@HiltAndroidApp` 註解
- 在 `AndroidManifest` 中設置應用程式類別

### 設置 ViewModel
- 在 `presentation` 包中建立 `WeatherViewModel`，注入 `WeatherRepository` 和 `LocationTracker`
- 定義 UI 狀態類別 `WeatherState`，包含 `weatherInfo`、`isLoading` 和 `error` 屬性
- 創建協程，通過 `LocationTracker` 獲取位置，再調用 Repository 取得天氣數據
- 根據 API 回應更新 UI 狀態，成功時更新天氣資訊，錯誤時更新錯誤訊息
- 處理位置取得失敗情況，提示用戶授權權限或啟用 GPS

### Compose 的 UI 狀態管理
- 使用 `MutableState` 管理 UI 狀態，並暴露為不可變的 `State`
- `loadWeatherInfo` 函式負責更新 UI 狀態為「加載中」，並根據結果更新 UI 顯示天氣或錯誤訊息

### 設計 UI 與組件
- UI 的結構很簡單，包含天氣卡片、今日時間、天氣圖示、溫度、天氣描述、氣壓、濕度、風速等資訊。
- 天氣卡片將顯示今日的天氣狀況，包括天氣圖示、溫度、描述、氣壓、濕度與風速。
- 使用 `Column` 和 `Row` 來布局各個元素，確保信息顯示清晰且整齊。

### Weather Card 組件設計
- 透過 `Card` 包裹所有天氣資訊，使用 `Modifier` 設置外觀與邊距。
- 若天氣資料為空，則不顯示此卡片。
- 顯示 "Today" 文本並顯示當前時間，使用特定的時間格式。
- 天氣圖示來自 `weatherType`，並且調整圖片的大小與對齊方式。
- 顯示溫度與天氣描述，並用不同字體大小與顏色進行區分。
- 使用自定義的 `WeatherDataDisplay` 組件顯示氣壓、濕度、風速等資訊。

### Weather Data Display 組件設計
- 此組件用於顯示重複出現的氣象資料，例如氣壓、濕度、風速等。
- 接受數值、單位、圖示、文字樣式與圖示顏色等參數。
- 使用 `Row` 布局顯示圖示和資料，並通過 `Modifier` 來調整大小與間距。

### Weather Forecast 組件設計
- 顯示全天氣預報資料，包含今日的天氣與未來幾小時的天氣資訊。
- 使用 `LazyRow` 來顯示每小時的天氣資料。
- 針對每小時資料使用 `HourlyWeatherDisplay` 組件來顯示時間、圖示和溫度。

### Hourly Weather Display 組件設計
- 顯示每小時的天氣資訊，包含時間、天氣圖示和溫度。
- 使用 `Column` 布局來顯示時間、圖示與溫度，並調整對齊方式。
- 確保每個小時的顯示內容高度一致，避免顯示不整齊。

### Main Activity 中的 UI 配置
- 在 `MainActivity` 中組合所有組件，顯示天氣卡片與天氣預報。
- 使用 `Box` 來包含所有 UI 元素，並處理加載與錯誤狀態。
- 當加載時顯示進度條，當發生錯誤時顯示錯誤訊息。
- 處理用戶授權請求並在授權後加載天氣資訊。

### 測試與錯誤處理
- 測試 UI 顯示，確認在未授權的情況下顯示錯誤訊息。
- 確保在獲取位置授權後，成功顯示天氣資料並處理相應的 UI 更新。


# Terminology
- Clean Architecture：一種分層軟體設計方法，將系統分為不同的層級（例如表示層、域層、數據層），以提升可維護性與可擴展性。
- Presentation Layer：負責應用程式的使用者界面和用戶交互的層級。
- Domain Layer：核心業務邏輯所在層級，與具體實現細節無關，專注於應用的業務規則。
- Data Layer：處理應用的數據來源，例如 API、數據庫等，負責數據的讀寫操作。
- DTO（Data Transfer Object）：數據傳輸對象，用於與外部系統（例如 API）交換數據的輕量級類。
- Mapper：負責將數據模型從一種形式轉換為另一種形式的工具，通常用於將 DTO 映射為域模型。
- Retrofit：一個流行的 Android HTTP 客戶端，用於與 REST API 通信。
- Coroutines：Kotlin 中的輕量級異步編程框架，用於管理非同步操作。
- ViewModel：Android 架構組件的一部分，負責處理與 UI 相關的數據邏輯。
- Sealed Class：Kotlin 中的一種特殊類型，用於表示一組有限的子類型。
- Dependency Injection：通過將對象的依賴項注入到其構造函數或屬性中的模式，用於提升可測試性和解耦。
- Hilt：Google 提供的 Android 專用依賴注入框架。
- Weather API：用於檢索天氣數據的外部服務接口。
- JSON（JavaScript Object Notation）：一種輕量級數據交換格式，用於在客戶端和服務端之間傳遞數據。
- Serialization：將對象轉換為字節流或文本格式以進行傳輸或存儲的過程。
- Deserialization：將序列化的數據恢復為對象的過程。
- LocalDateTime：Java 和 Kotlin 中用於處理本地日期和時間的類型。
- OpenWeather API：一種流行的天氣數據 API，提供實時和預測的天氣資訊。
- Resource Class：用於封裝數據加載狀態（成功、錯誤、加載中）的實用類。
- Weather Type：表示特定天氣類型（如晴天、雨天）的類型。
- Forecast：天氣預報，用於表示未來某段時間內的天氣數據。
- Nullable：在 Kotlin 中，允許變數值為空的類型。
- Repository：負責數據操作和源管理的類別，通常用於封裝數據源邏輯。
- API Endpoint：API 提供服務的具體 URL。
- Weather Code：API 返回的整數代碼，用於表示特定天氣情況。
- Weather Icon：用於可視化顯示天氣情況的圖標。
- MVVM（Model-View-ViewModel）：一種架構模式，用於將業務邏輯與 UI 分離。
- JSON Annotation：用於指示序列化/反序列化過程中字段對應的標註。
- Domain Model：域層中的核心數據模型，與數據實現細節隔離。
- Dependency Management：組織和管理應用程序所依賴的外部庫或框架的過程。
- Localization：使應用支援多語言和地區格式的能力。
- Query Parameter：URL 中用於傳遞額外信息的參數。
- Gson：Google 提供的一個 JSON 序列化/反序列化庫。
- Modularization：將應用拆分為多個獨立模組的過程，以提升代碼可復用性和可維護性。
- Unit Testing：針對代碼的最小單位（例如函數、類）進行測試。
- Integration Testing：測試多個模塊或系統之間的交互行為。
- Latency：在網絡中，數據從發送到接收的延遲時間。
- Error Handling：處理應用運行過程中可能出現的錯誤的機制。
- Configuration File：用於存儲應用設置的文件，例如 API 密鑰或環境變數。
- Singleton：一種設計模式，確保一個類在應用中僅有一個實例。
- Data Binding：將數據源與 UI 元素綁定的技術。
- Gradle：Android 項目中用於構建和依賴管理的工具。
- Lottie：用於在 Android 中渲染動畫的庫。
- Drawable：Android 中用於表示圖形元素的資源類型。
- Build Variant：Android 應用中用於區分不同版本（如開發版與發布版）的功能。
- Proguard：用於縮小、混淆和優化 Java 程序的工具。
- Lint：用於檢測代碼潛在問題的靜態分析工具。
- Hot Reload：允許開發人員在不重新啟動應用的情況下加載代碼更改的功能。
- **DTO（Data Transfer Object）**：用於傳輸數據的簡單對象，通常不包含業務邏輯。
- **Domain Object**：代表業務邏輯核心的對象，通常包含相關屬性和行為。
- **Mapper**：負責將一種對象映射到另一種對象的功能類或方法。
- **Extension Function**：Kotlin中的特性，允許為現有類型添加新功能，而無需繼承或使用裝飾器。
- **LocalDateTime**：Java時間API中的一個類，用於表示日期和時間，不帶時區信息。
- **ISO DateTime**：一種國際標準化的日期時間格式，易於解析和使用。
- **GroupBy**：集合操作，用於將元素根據指定的鍵分組。
- **Index**：集合中每個元素的整數標識符，從0開始。
- **Indexed Weather Data**：自定義數據類，用於暫存與索引相關的天氣數據。
- **Map**：一種鍵值對集合，鍵唯一且不可變。
- **MapValues**：用於映射Map集合的值，而不改變鍵的操作。
- **Resource**：封裝數據及其狀態（成功或失敗）的通用類型。
- **Weather Info**：業務層的核心對象，封裝每天的天氣數據及當前天氣數據。
- **Repository**：負責處理數據來源（如API、資料庫）及其邏輯的設計模式。
- **Interface**：定義類行為的抽象結構，用於實現多態和代碼隔離。
- **Dagger Hilt**：用於Android依賴注入的框架，簡化依賴管理和生命周期管理。
- **Inject Constructor**：Dagger Hilt的功能，允許標記構造函數以便自動注入依賴項。
- **Suspend Function**：Kotlin中的協程特性，用於非阻塞操作。
- **Try-Catch**：用於捕獲和處理運行時異常的結構。
- **StackTrace**：當異常發生時，提供調試信息的追蹤日誌。
- **Weather API**：提供天氣數據的服務接口。
- **Lat（Latitude）**：地理座標中的緯度，表示北南方向的角度。
- **Long（Longitude）**：地理座標中的經度，表示東西方向的角度。
- **Weather Data DTO**：從API獲取的天氣數據對象，僅用於數據傳輸。
- **Weather Type**：定義特定天氣代碼與其對應類型的枚舉類。
- **From WMO**：依據WMO天氣代碼生成WeatherType的靜態方法。
- **Local Database Cache**：本地存儲的數據緩存，用於提高性能和支持離線模式。
- **Hourly Data**：每小時更新的天氣數據，包含溫度、濕度等細節。
- **Day Key**：標識某天的唯一鍵，用於分組和檢索天氣數據。
- **Now Object**：代表當前時間的LocalDateTime實例。
- **Minute Threshold**：用於確定是向上還是向下舍入分鐘數的標準。
- **Find**：集合操作，用於查找符合條件的第一個元素。
- **Weather Data Map**：從天氣數據DTO生成的映射結構，鍵為日，值為小時數據。
- **Unknown Error**：在異常處理中，無法獲取具體錯誤信息時的默認消息。
- **Success Resource**：表示操作成功的資源類型，包含結果數據。
- **Error Resource**：表示操作失敗的資源類型，包含錯誤消息。
- **Data Package**：項目結構中的數據層，用於存儲和管理數據相關邏輯。
- **Domain Package**：項目結構中的業務層，用於處理核心業務邏輯。
- **Model-View-ViewModel（MVVM）**：Android應用中的架構設計模式。
- **Weather Repository Implementation**：接口的具體實現類，負責數據獲取和轉換。
- **API Call**：通過網絡請求數據的操作，通常返回異步結果。
- **Error Handling**：在代碼中應對運行時異常的過程和策略。
- **Unit Test**：對單一功能單元進行驗證的測試方法。
- **Forecast Extension**：在應用中添加多天預測功能的能力。
- **Homework**：開發者練習的作業或補充功能實現。
- **Print Debugging**：通過打印輸出來調試代碼的技術。
- **Data Structure**：組織和存儲數據的方式，例如List、Map等。
- **Key-Value Pair**：Map中的基礎元素，由唯一鍵和值組成。
- **Weather Data Per Day**：存儲每日天氣數據的數據結構。
- **Current Weather Data**：表示當前時刻天氣信息的對象。
- **ViewModel**：在MVVM架構中，用於保存和管理界面相關數據的組件。
- **Hourly Weather Details**：每小時的具體天氣數據，如溫度和濕度。
- **Data Transformation**：將一種數據格式轉換為另一種格式的過程。
- **抽象化 (Abstraction)**：將系統中的具體實現隱藏起來，只暴露出簡單的接口，使得高層次的邏輯不依賴具體的實現細節。
- **位置追蹤 (Location Tracking)**：利用設備的 GPS 或其他技術，獲取並跟踪使用者的地理位置。
- **數據層 (Data Layer)**：應用程式架構中的一層，負責與外部數據源交互，例如網絡 API 或資料庫。
- **清潔架構 (Clean Architecture)**：一種軟體架構模式，旨在將業務邏輯與框架細節分離，促進可測試性和維護性。
- **掛起函數 (Suspend Function)**：Kotlin 中的一種特殊函數，允許在協程中執行非同步操作，並能夠掛起直到結果返回。
- **協程 (Coroutine)**：Kotlin 提供的輕量級線程，能夠簡化非同步操作，並提升性能。
- **依賴注入 (Dependency Injection)**：一種設計模式，將對象的依賴關係從外部提供，而非在內部創建，增強模組間的解耦性。
- **Dagger Hilt**：Google 提供的一個依賴注入框架，用於簡化 Android 中的 DI 實現。
- **擴展函數 (Extension Function)**：Kotlin 中的功能，允許對已有類型添加新功能，而不需要修改其原始代碼。
- **異步 (Asynchronous)**：操作不會阻塞主線程，可以在後台執行並在完成後通知結果。
- **回呼 (Callback)**：將函數作為參數傳遞給另一函數，在該函數執行完畢後被呼叫。
- **資源類型 (Resource Class)**：用來封裝 API 調用的結果，通常包括成功的數據、錯誤訊息或加載狀態。
- **狀態模式 (State Pattern)**：設計模式之一，用於對象的狀態管理，根據狀態的不同改變對象的行為。
- **視圖模型 (ViewModel)**：在 MVVM 架構中，用於管理 UI 狀態和與 UI 交互的邏輯。
- **Compose**：Android 的現代 UI 開發工具，使用聲明式語法構建 UI 元件。
- **資源類型封裝 (Resource Wrapping)**：將結果封裝在資源類型中，便於處理成功、錯誤及加載狀態。
- **單例模式 (Singleton Pattern)**：設計模式，確保一個類型只有一個實例，並提供全局訪問點。
- **Kotlin 協程 (Kotlin Coroutines)**：Kotlin 的非同步處理機制，簡化並行操作的實現。
- **註解 (Annotation)**：在程式碼中附加的元數據，用來給編譯器、工具或程式碼解析器提供額外的信息。
- **依賴提供 (Dependency Provision)**：通過 DI 框架將實例提供給需要的組件。
- **異常處理 (Exception Handling)**：處理程式運行中發生的異常情況，防止程式崩潰並提供有效的錯誤訊息。
- **地理位置服務 (Location Services)**：手機或設備提供的 API，用於獲取地理位置。
- **濃縮型任務 (Cancelable Task)**：可以在執行過程中取消的任務，適用於耗時操作。
- **網絡請求 (Network Request)**：應用程序向服務器發送請求，以獲取或發送數據。
- **成功回應 (Success Callback)**：回呼函數的一種，用於處理成功的結果。
- **失敗回應 (Failure Callback)**：回呼函數的一種，用於處理錯誤或異常情況。
- **Fused Location Provider**：Android 中一個高效的地理位置服務，結合多種位置來源（如 GPS 和 Wi-Fi）提供準確的位置信息。
- **天氣狀態 (Weather State)**：應用中顯示的天氣數據狀態，通常包括溫度、濕度等資訊。
- **可變狀態 (Mutable State)**：可以在應用中修改的狀態，通常由 ViewModel 管理。
- **應用程序上下文 (Application Context)**：整個應用程序的上下文，用於訪問全局資源或服務。
- **數據轉換器 (Converter Factory)**：用於將服務端返回的數據格式（如 JSON）轉換為應用程序中的數據類型。
- **應用程序級單例 (Application Singleton)**：應用程序範圍內的單一實例，通常用來存儲全局共享的資源。
- **任務 (Task)**：表示一個異步操作，通常用於處理背景工作的結果。
- **API 請求 (API Request)**：從客戶端向伺服器發送的請求，請求資料或進行操作。
- **錯誤處理 (Error Handling)**：處理可能發生的錯誤或異常，保證應用程式不會因為錯誤而崩潰。
- **應用級依賴注入 (Application-Level Dependency Injection)**：依賴注入的範疇涵蓋整個應用程序，適用於單例級別的依賴。
- **應用程序類 (Application Class)**：Android 中的一個類，代表應用程序的實例，通常用於初始化全局狀態。
- **網絡服務 (Network Service)**：提供網絡通信功能的服務，負責管理網絡請求和數據交換。
- **視圖狀態 (View State)**：用來表示界面顯示的狀態，包含界面的顯示數據及其狀態。
- **協程取消 (Coroutine Cancellation)**：允許取消正在執行的協程，釋放相關資源。
- **UI 元件 (UI Components)**：用來構建使用者界面的元素，如按鈕、文本框等。
- **數據傳輸對象 (DTO - Data Transfer Object)**：用於表示交換數據的對象，通常用於 API 請求和響應。
- **Kotlin DSL (Domain Specific Language)**：Kotlin 提供的領域特定語言，讓開發者能夠更簡潔地表達特定領域的邏輯。
- **觀察者模式 (Observer Pattern)**：設計模式，用於允許一個對象通知其他依賴於它的對象，通常用於 UI 更新。
- **異步響應 (Asynchronous Response)**：一種響應模式，在完成操作後回傳結果，不會阻塞程式。
- **應用架構 (Application Architecture)**：指構建應用程序時，如何組織代碼結構、模塊和依賴的策略。
- **簡單工廠模式 (Simple Factory Pattern)**：設計模式，用於創建一個類型的實例，而不暴露具體的創建邏輯。
- **網絡錯誤處理 (Network Error Handling)**：處理網絡請求過程中可能發生的錯誤情況，保證應用程序穩定運行。
- **位置管理器 (Location Manager)**：Android 系統服務，負責獲取和管理位置相關的數據。
- **依賴綁定 (Dependency Binding)**：將具體實現綁定到接口或抽象類型，讓系統可以靈活注入依賴。
- **單例組件 (Singleton Component)**：一種組件，系統中只有一個實例，並且可以在整個應用中共享。
- **Composable**：在 Jetpack Compose 中，一種 UI 元件，用來構建可重用的 UI 部件。
- **Modifier**：用來改變 UI 元件外觀或行為的 Compose 元素，例如調整大小、顏色、對齊等。
- **Column**：在 Jetpack Compose 中，一個垂直排列子元件的容器。
- **Row**：在 Jetpack Compose 中，一個水平排列子元件的容器。
- **Card**：一個圓角矩形的視覺容器，常用來顯示信息或內容。
- **Image Vector**：代表向量圖像的類型，通常用於顯示圖標。
- **Painter Resource**：用來加載圖像資源的對象，通常用於載入 drawable 中的圖片。
- **Text**：顯示文本內容的元件。
- **Spacer**：用來創建空間的元件，通常用於調整 UI 的布局間距。
- **Horizontal Arrangement**：在 Jetpack Compose 中，指定 Row 中子元件的水平排列方式。
- **Vertical Alignment**：在 Jetpack Compose 中，指定 Column 中子元件的垂直對齊方式。
- **Font Size**：字體的大小。
- **Font Weight**：字體的粗細，控制文本的視覺重量。
- **Text Style**：控制文本樣式的屬性集合，如字體大小、字體顏色等。
- **Lazy Row**：懶加載的水平排列容器，通常用於顯示大量的元素。
- **Items**：LazyRow 或 LazyColumn 中顯示的單個元素，每個元素會被懶加載。
- **State**：用來儲存應用程序的數據狀態。
- **ViewModel**：在 Jetpack 中，處理 UI 相關邏輯和數據管理的類。
- **Activity Result API**：Android 用來處理活動結果的 API，用於請求權限等操作。
- **Permission Launcher**：用來發起請求權限的工具。
- **Circular Progress Indicator**：圓形進度指示器，用來顯示加載狀態。
- **Error Handling**：錯誤處理，用於捕獲和處理錯誤情況。
- **Box**：Jetpack Compose 中的一個容器，用來包裹其他 UI 元件。
- **Lazy Column**：懶加載的垂直排列容器，通常用於顯示大量的元素。
- **Weather Data Display**：顯示天氣數據的 UI 元件。
- **Weather Card**：顯示天氣卡片的 UI 元件。
- **Weather Forecast**：顯示天氣預測的 UI 元件。
- **Weather Info**：儲存當前天氣數據的物件。
- **Weather Type**：表示天氣狀態（如晴天、陰天等）的類別。
- **Time Format**：顯示時間的格式。
- **Alignment**：UI 元件在父容器中的對齊方式。
- **HPA**：百帕（hectopascal），壓力單位。
- **Unit**：單位，用來表示數值的度量標準。
- **Modifier Padding**：用來設置內邊距的修飾器。
- **Weather Description**：天氣描述，如多雲、晴天等。
- **Icon Tint**：圖標顏色，用來改變圖標的顏色。
- **Weather Data Object**：包含天氣數據的物件，通常包括溫度、濕度、風速等信息。
- **Time String**：時間的字符串格式表示。
- **Weather State**：表示天氣的狀態，包括當前天氣、預測數據等。
- **Image Size**：設定圖片顯示大小的屬性。
- **Font Style**：設定字體樣式的屬性，如粗體、斜體等。
- **Activity**：Android 中的一個主要元件，用來處理 UI 和用戶交互。
- **Hilt**：Android 的依賴注入庫，用於簡化依賴注入過程。
- **Manifest Permission**：應用程式的 AndroidManifest 文件中聲明的權限。
- **Weather App**：顯示天氣數據的應用程序。
- **Time Format Pattern**：用來定義時間格式的字符串。
- **ViewModel State**：用來儲存和管理 ViewModel 的數據狀態。
- **Horizontal Padding**：設置元素的水平內邊距。
- **Vertical Padding**：設置元素的垂直內邊距。
- **Item Height**：設置每個項目的高度。
- **Lazy Loading**：延遲加載技術，僅當需要時加載數據或 UI 元件。
