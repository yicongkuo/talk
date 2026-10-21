如果是**既有 ArcGIS Enterprise 系統要移機**，我會建議不要只問「目前 Server 是幾 Core、幾 GB RAM」，因為那只能知道**現在配多少**，不代表知道「搬過去需要多少」。

比較合理的做法是拿 **「現況架構＋實際負載＋資料量＋成長量＋移機限制」** 五大類數據，再反推新環境規格。

## 1. 最優先要拿的數據

如果時間有限，我會先要求客戶提供下面這 **10 項**：

| 優先  | 數據                                   | 為什麼重要                            |
| --- | ------------------------------------ | -------------------------------- |
| ⭐⭐⭐ | **目前 VM/Server CPU、RAM、Disk 規格**     | 了解現有配置基準                         |
| ⭐⭐⭐ | **CPU 使用率 Peak / Average**           | 判斷 CPU 是否已經不足或過剩                 |
| ⭐⭐⭐ | **RAM 使用率 Peak / Average**           | 決定 32 / 64 / 128 GB              |
| ⭐⭐⭐ | **Disk 使用量與成長速度**                    | 決定新環境儲存容量                        |
| ⭐⭐⭐ | **Disk IOPS / Latency / Throughput** | GIS 系統很容易受 Storage I/O 影響        |
| ⭐⭐⭐ | **同時使用者 / Request 數**                | 判斷實際服務負載                         |
| ⭐⭐⭐ | **ArcGIS Service 數量與類型**             | Feature / Map / Image / GP 負載差很多 |
| ⭐⭐  | **ArcGIS Data Store 資料量**            | 決定 Data Store 儲存需求               |
| ⭐⭐  | **每日 / 每月資料成長量**                     | 避免新 Server 一年後又滿                 |
| ⭐⭐  | **Backup 大小與保留天數**                   | 決定備份空間與方案                        |

---

# 2. 建議收集「現況架構表」

首先要把目前 ArcGIS Enterprise 到底怎麼部署搞清楚。

可以要求對方填：

|項目|需要取得|
|---|---|
|ArcGIS Enterprise 版本|例如 11.3 / 11.5 / 12.0|
|OS|Windows / Linux、版本|
|Deployment|Single Machine / Distributed|
|Portal for ArcGIS|Server / VM|
|ArcGIS Server|Server / VM|
|ArcGIS Data Store|Server / VM|
|Web Adaptor|有／無|
|Reverse Proxy / Load Balancer|有／無|
|Enterprise Geodatabase|SQL Server / Oracle / PostgreSQL 等|
|File Share|有／無、容量|
|Raster / Image Server|有／無|
|Notebook Server|有／無|
|GeoAnalytics Server|有／無|
|Knowledge Server|有／無|
|外部 DB|有／無|
|備份 Server|有／無|

尤其要問清楚：

> **ArcGIS Data Store 是跟 ArcGIS Server 同機，還是另外一台？**

以及：

> **Enterprise Geodatabase 在哪裡？**

因為如果 GIS Database 是外部 SQL Server，那新 ArcGIS Server 的硬體估算方式會跟「所有東西都在單機」完全不同。

---

# 3. CPU / RAM：不要只拿「規格」，要拿「使用率」

這是移機估算最重要的一部分。

不要只拿：

> 目前 Server = 8 Core / 32 GB

最好拿至少 **7～30 天**的監控資料。

### CPU

至少需要：

|指標|建議|
|---|---|
|Average CPU|%|
|Peak CPU|%|
|Peak 發生時間|時間|
|CPU 使用率 >80% 時間|分鐘／小時|
|CPU Core 數|Core|
|CPU 型號|型號／時脈|

例如：

```
CPU：
8 vCPU
Average：35%
平日 Peak：68%
尖峰 Peak：91%
CPU >80%：每天約 15 分鐘
```

這比「8 Core」有價值得多。

---

# 4. RAM 要拿 Peak，而不是只看目前配置

例如：

```
RAM：32 GB

Average：18 GB
Working Set：22 GB
Peak：29 GB
Available Memory 最低：3 GB
```

如果長期：

> 32 GB RAM / 實際使用 29～31 GB

那搬機時就不應該繼續給 32 GB。

可能會建議：

> **64 GB**

反過來，如果：

> 64 GB RAM / 長期只使用 20 GB

也不代表一定要照搬 64 GB。

所以這部分最好拿：

**至少 7～30 天 RAM 使用率趨勢。**

---

# 5. Storage 是 GIS 移機最容易低估的部分

這個我會要求拿得比較詳細。

不要只問：

> C 槽剩多少？

而是要求：

|Disk|Total|Used|Free|IOPS|Latency|Throughput|
|---|---|---|---|---|---|---|
|OS|500 GB|150 GB|350 GB|-|-|-|
|GIS Data|2 TB|1.2 TB|800 GB|1,500|5 ms|150 MB/s|
|Backup|4 TB|2.5 TB|1.5 TB|-|-|-|

### 特別要拿

**資料量成長速度。**

例如：

```
2024/01：500 GB
2025/01：750 GB
2026/01：1.2 TB
```

就可以估算：

> 約 350 GB/year

如果預計新系統使用 5 年：

> 1.2 TB + 350 GB × 5  
> ≈ 2.95 TB

再加上：

- 預留空間
- Cache
- Log
- 暫存
- Backup
- 未來成長

最後可能需要規劃 **4～6 TB usable capacity**。

---

# 6. ArcGIS Service 清單非常重要

這個甚至比「使用者人數」更重要。

請對方匯出：

|Service|Type|Instance|Request|Data|
|---|---|---|---|---|
|MapService A|Map Server|2|高|Enterprise DB|
|FeatureService B|Feature Server|2|高|Data Store|
|MapService C|Map Server|1|低|File GDB|
|ImageService D|Image Server|2|中|Raster|

至少要知道：

### Service 數量

```
Map Service：XX
Feature Service：XX
Image Service：XX
Geoprocessing Service：XX
Scene Service：XX
Vector Tile：XX
Hosted Feature Layer：XX
Hosted Tile Layer：XX
```

因為：

> **10 個 Map Service ≠ 10 個 Image Service**

它們的 CPU / RAM / I/O 特性可能完全不同。

---

# 7. Instance / Service Pool 資訊

如果是 ArcGIS Server，最好再拿：

- Min instances
- Max instances
- SOC 數量
- Service instance 使用量
- Service 啟動時間
- Service crash / restart 紀錄

例如：

```
MapService A
Min instances = 2
Max instances = 8

MapService B
Min instances = 1
Max instances = 20
```

如果有一個 Feature Service：

> Max Instance = 50

那就會直接影響 Server 的 RAM / CPU 估算。

---

# 8. 使用者與流量

不要只問：

> 有多少帳號？

最好分成：

|指標|數值|
|---|---|
|Portal 使用者總數|500|
|每日活躍使用者|150|
|同時在線使用者|50|
|尖峰同時使用者|80|
|每日 Request|500,000|
|Peak Request/sec|100|
|平均 Request/sec|20|

尤其：

**Concurrent Users（同時使用者）**

比「總帳號數」更有參考價值。

---

# 9. 一定要拿 Peak 時段

例如系統平常：

> CPU 30%

但每天 09:00～10:00：

> CPU 85%

那真正的 sizing 應該針對：

> **09:00～10:00**

而不是平均值。

建議取得：

- CPU
- RAM
- Disk I/O
- Network
- Request/sec

的 **日／週／月 Peak**。

最好至少涵蓋：

> **正常日＋尖峰日＋特殊事件日**

例如政府系統可能有：

- 報表發布
- 申報期間
- 災害事件
- 大量資料上傳
- 年度更新

這些才是真正的壓力測試情境。

---

# 10. Network 流量

如果新環境跟 Database / File Server 分開，這項很重要。

需要：

|指標|數據|
|---|---|
|NIC|1GbE / 10GbE|
|Average In|Mbps|
|Peak In|Mbps|
|Average Out|Mbps|
|Peak Out|Mbps|
|GIS DB Traffic|Mbps|
|Client Traffic|Mbps|

例如：

```
平均：150 Mbps
Peak：850 Mbps
```

那就不應該只是看到「目前是 1GbE」就照搬。

---

# 11. Database 也要一起盤點

這個很容易漏掉。

如果有：

**Enterprise Geodatabase**

要拿：

- DB Server CPU
- DB Server RAM
- DB Server Storage
- Database Size
- Table 數量
- Feature Class 數量
- Index Size
- Database IOPS
- DB Connection 數量
- Peak DB CPU
- Backup Size

例如：

```
SQL Server
Database：1.8 TB
RAM：128 GB
CPU：16 Core
Peak CPU：75%
IOPS：4,000
```

這時候即使 ArcGIS Server 本身只需要：

> 8 Core / 32 GB

也不能把整個系統理解成「一台 8 Core / 32 GB Server」就夠。

---

# 12. 備份資料一定要問

移機規劃很容易只算 Production Storage。

但實際上應該算：

```
Production
   ↓
Backup
   ↓
Retention
   ↓
DR
```

需要知道：

- ArcGIS Backup 大小
- 每日 Backup 大小
- Full / Incremental
- 保留幾天
- 保留幾份
- Backup 是否放另一台
- 是否有異地備份
- RPO
- RTO

例如：

```
每日 Backup：150 GB
Retention：30 天

粗估：
150 GB × 30 = 4.5 TB
```

實際還要考慮備份方式與壓縮。

---

# 13. 建議直接跟客戶要「30 天監控資料」

如果你是在做**正式移機專案的前期需求調查**，我最推薦的方式不是人工問一大堆數字，而是要求對方提供：

### Server

```
CPU Usage
Memory Usage
Disk Usage
Disk IOPS
Disk Latency
Disk Throughput
Network Throughput
```

### ArcGIS

```
Service Count
Service Type
Service Instance
Request Count
Request/sec
Error Count
Service Restart
Log Volume
Portal User Count
```

### Database

```
DB Size
CPU
Memory
IOPS
Connection
Query Load
Backup Size
```

**最好提供 30 天的 hourly data，至少 7 天。**

---

# 14. 我會把移機估算做成這個公式

最後不是直接：

> 舊 Server 8 Core → 新 Server 8 Core

而是：

**新環境規格 = 現況實際負載 × 成長係數 × 安全裕度**

例如：

```
CPU

目前 Peak = 65%
預估 3 年成長 = 30%
安全裕度 = 30%

→ 新環境至少要能承受：
65% × 1.3 × 1.3
≈ 110% 的目前 CPU capacity
```

所以可能從：

> 8 Core

升到：

> **12 Core**

RAM 也可以用類似方式估算。

---

## 15. 最後我會要求客戶填這張表

如果你要實際執行專案，我建議直接做成 **「ArcGIS Enterprise 移機現況調查表」**：

|類別|調查項目|客戶提供資料|
|---|---|---|
|基本|ArcGIS Enterprise Version||
|基本|OS Version||
|基本|VM / Physical||
|CPU|Core / vCPU||
|CPU|CPU Model||
|CPU|Average Usage||
|CPU|Peak Usage||
|RAM|Installed RAM||
|RAM|Average Usage||
|RAM|Peak Usage||
|Storage|OS Disk||
|Storage|Data Disk||
|Storage|Used / Free||
|Storage|IOPS||
|Storage|Latency||
|Storage|Throughput||
|Network|NIC Speed||
|Network|Average Traffic||
|Network|Peak Traffic||
|ArcGIS|Portal Users||
|ArcGIS|Daily Active Users||
|ArcGIS|Concurrent Users||
|ArcGIS|Map Services||
|ArcGIS|Feature Services||
|ArcGIS|Image Services||
|ArcGIS|GP Services||
|ArcGIS|Scene Services||
|Data Store|Database Size||
|Data Store|Growth / Month||
|Database|DB Type||
|Database|DB Size||
|Database|DB CPU Peak||
|Database|DB RAM||
|Backup|Backup Size||
|Backup|Retention||
|Backup|RPO||
|Backup|RTO||
|未來|預估使用者成長||
|未來|預估資料成長||
|未來|預估服務成長||

### 最關鍵的觀念

如果是我要幫你做這個 **ArcGIS Enterprise 既有系統移機 sizing**，我會把資料分成三個等級：

**A. 一定要拿**

- CPU / RAM Peak
- Disk 使用量
- Disk IOPS / Latency
- Service 數量與類型
- Concurrent Users
- Request 數
- Data Store / DB Size
- 資料成長量

**B. 最好拿**

- 30 天監控趨勢
- Service Instance
- Network Peak
- DB I/O
- Backup 大小
- Log 大小

**C. 用來決定「升級還是照搬」**

- 未來 3～5 年使用者成長
- 資料成長
- Service 成長
- SLA
- RPO / RTO
- 是否需要 HA / DR
- 是否有未來新增 Notebook、Image Server、GeoAnalytics 等元件

**尤其不要用「目前硬體規格」直接當新環境規格。**  
最合理的 sizing 應該是以 **實際 Peak Load + 未來成長 + 安全裕度** 反推。

如果你是正在做一個**客戶既有 ArcGIS Enterprise 移機案的規劃/報價**，我也可以直接幫你整理成一份可交給客戶填寫的 **Excel 格式「ArcGIS Enterprise 移機需求與現況調查表」**，並且加上「填入數據後如何推算 CPU / RAM / Storage」的欄位。