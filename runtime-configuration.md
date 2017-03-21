# CHAPTER 7

### Run-time Configuration

## 7.1 動手設定

OpenDDS使用以檔案為基礎的設定框架，其中包含全預選項，與特定發布者、訂閱者相關的選項(如探索設定以及傳送設定)。
OpenDDS也允許使用者透過執行文字指令，部份的選項可以透過API設定。
這個章節將會描述OpenDDS支援的設定選項。
#### 1) 一般設定選項：
設定在全域等級下DCPS實體的行為。
這將可以讓在相同電腦環境下部署的不同進程共享同一個一般設定。
(例如所有的讀取者及寫入者都使用RTPS探索)
#### 2) 探索設定選項：
設定探索機制下的行為。
OpenDDS支援多種途徑探索及協調寫入者與讀取者。
更多細節參照段落7.3。
#### 3) 傳送設定選項：
設定可擴展傳送框架(ETF)，作為OpenDDS的DCPS層中的抽象傳輸層。
每個可插拔式的傳送都可以被分別設定。

OpenDDS的設定檔案為易讀的ini格式文字檔。 表7-1為可允許的設定段落類型，他們都對應一種OpenDDS設定選項。

#### 表7-1 設定檔案段落

| 設定選項類型 | 檔案段落標題                                                                      |
|--------------|-----------------------------------------------------------------------------------|
| 全域設定     | [common]                                                                          |
| 探索         | [domain] [repository] [rtps_discovery]                                            |
| 靜態探索     | [endpoint] [topic] [datawriterqos] [datareaderqos] [publisherqos] [subscriberqos] |
| 傳送         | [config] [transport]                                                              |



除了[common]之外，每個段落得標頭都可以用[段落標題/實例]的格式作設定，舉個例子，[repository]標頭可以這樣描述：
[repository/repo_1]，repository為段落標題，repo_1為實例名稱，這個標頭下的設定將套用在repo_1實例下的repository設定。
如何使用實例來設定探索以及傳送選項將會在段落7.3以及7.4介紹。

`-DCPSConfigFile` 指令參數可以用來傳遞欲套用之OpenDDS設定檔案的位置。如下：

Windows：
```shell
publisher -DCPSConfigFile pub.ini
```
Unix：
```shell
./publisher -DCPSConfigFile pub.ini
```
指令參數將在區域參與因數(domain participant factory)初始化時傳遞至個別參與的實例。
這將由巨集`TheParticipanFactoryWithArgs`：
```cpp
#include <dds/DCPS/Service_Participant.h>
int main (int argc, char* argv[])
{
  DDS::DomainParticipantFactory_var dpf =
  TheParticipantFactoryWithArgs(argc, argv)
```
`Service_Participant`集合也提供允許應用程式設定DDS服務的方法，詳情參照標頭檔`$DDS_ROOT/dds/DCPS/Service_Participant.h`。
以下段落將詳細描述各個不同的設定檔案段落以及相關的可用選項。


## 7.2 一般設定選項
OpenDDS設定檔案中的[common]段落包含的選項像是除錯訊息輸出等級、DCPSInfoRepo進程位置以及預分配記憶體設定。下面為[common]設定範例：
```
[common]
DCPSDebugLevel=0
DCPSInfoRepo=localhost:12345
DCPSLivelinessFactor=80
DCPSChunks=20
DCPSChunksAssociationMultiplier=10
DCPSBitLookupDurationMsec=2000
DCPSPendingTimeout=30
```

並不是必須指定每個選項的設定值。

[common]中的設定值若名稱以"DCPS"作開頭的皆可被指令列參數覆蓋。
指令列參數名稱和設定檔中名稱相同，只是前面多了個"-"符號，例如：
```shell
subscriber -DCPSInfoRepo localhost:12345
```
下表描述[common]設定選項：

#### 表7-2 一般設定選項

| 選項名稱                                                                                                        | 敘述                                                                                                                                                                                                                                                                  | 預設值                                                  |
|-----------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| DCPSBit=[1\|0]                                                                                                   | 切換內建主題支援。                                                                                                                                                                                                                                                    | 1                                                       |
| DCPSBitLookupDurationMsec=msec                                                                                  | 框架等待當解讀實例控制所給的BIT data時遣在內建主題資訊的最長持續時間(單位：毫秒)。在框架取得且處理相關BIT資訊前，參與的程式碼會獲得一個給遠端實體的實例控制。框架將會在時間內等待，之後才提示操作失敗。                                                               | 2000                                                    |
| DCPSBitTransportIPAddress=addr                                                                                  | 內建主題之IP位置，用來識別tcp傳送將用到的本地端界面。 注意：這個屬性只對DCPSInfoRepo設定有效。                                                                                                                                                                        | INADDR_ANY                                              |
| DCPSBitTransportPort=port                                                                                       | 內建主題在TCP傳送中使用的端口。如果預設值'0'已經被使用，作業系統會自己挑別的端口來使用。 注意：這個屬性只對DCPSInfoRepo設定有效。                                                                                                                                     | 0                                                       |
| DCPSChunks=n                                                                                                    | 當`RESOURCE_LIMITS`Qos值為無限時，資料讀寫快取分配之可設定DCPSChunks數量會得到預先分配之快取。當所有DCPSChunks都被分配出去時，OpenDDS將會由heap再分配。                                                                                                                           | 20                                                      |
| DCPSChunkAssociationMultiplier=n                                                                                | DCPSChunks的多工器或是藉由resource_limits.max_samples數值判斷預分配之DCPSChunks淺複製總數。將這個值設的比連接數量還大以防止預分配DCPSChunks過少。一個取樣寫入到多個資料讀取者不會執行多次完整複製而是只複製其開頭位址。這個動作佔用空間並不大，所以不需要把這個值設的太接近連結數量。 | 10                                                      |
| DCPSDebugLevel=n                                                                                                | 一個0到10的整數，用以控制除錯訊息印出的數量。                                                                                                                                                                                                                         | 0                                                       |
| DCPSDefaultAddress                                                                                              | 預設值為傳送實例包含的`local_address`，只有在設定值不為空且沒有`local_address`在傳送中被指定時生效。                                                                                                                                                                  |                                                         |
| DCPSDefaultDiscovery=[,DEFAULT_REPO\|,DEFAULT_RTPS\|,DEFAULT_STATIC\|,user-defined configuration instance name] | 在無明確設定之域名中指定探索設定。`Default_REPO`將使用DCPSInfoRepo，`Default_RTPS`指定於探索部份之RTPS使用。`Default_STATIC`設定靜態探索之使用。細節請參見段落7.3。                                                                                                   | DEFAULT_REPO                                            |
| DCPSGlobalTransportConfig=name                                                                                  | 指定全域設定中的傳送設定名稱。這個設定將被所有沒有另外指定傳送設定之實體使用。$file中特殊的數值使用包含設定檔中所有被定義的實體的傳送設定。                                                                                                                           | The default configuration is used as described in 7.4.1 |
| DCPSInfoRepo=objref                                                                                             | 物件指向DCPS資訊來源之位置。這可以是一個完整的CORBA IOR或是簡單的`host:port`字串                                                                                                                                                                                      | file://repo.ior                                         |
| DCPSLivelinessFactor=n                                                                                          | 當訊息送出時，最少的有效時間百分比。數值80等於20%最後一個有效訊息之潛在緩衝。                                                                                                                                                                                         | 80                                                      |
| DCPSPendingTimeout=sec                                                                                          | 一個資料寫入動作會封鎖一個刪除動作中未送出之樣品流失，其封鎖之最長持續時間。此數值之預設值為永久封鎖。                                                                                                                                                                | 0                                                       |
| DCPSPersistentDataDir=path                                                                                      | 持久資料將會儲存之檔案系統路徑。如果目錄不存在的話目錄將會被自動創建。                                                                                                                                                                                                | OpenDDS-durable-data-dir                                |
| DCPSPublisherContentFilter=[1\|0]                                                                               | 控制過濾內容主題之過濾表達式的判斷策略。當允許(1)時，發布者可能會丟失幾個樣品，當這些樣品被訂閱者忽略時，他們不會在傳送中被處理。                                                                                                                                     | 1                                                       |
| DCPSRTISerialization=[0\|1]                                                                                     | 當整合了使用RTPS傳送之RTI實體，控制是否使用非標準序列填充。                                                                                                                                                                                                           | 0                                                       |
| DCPSTransportDebugLevel=n                                                                                       | 為一0到5之整數決定傳送日誌之訊息量。                                                                                                                                                                                                                                  | 0                                                       |
| pool_size=n_bytes                                                                                               | 安全配置文件記憶體池大小，使用bytes。                                                                                                                                                                                                                                 | 41943040 (40 MB)                                        |
| pool_granularity=n_bytes                                                                                        | 安全配置文件記憶體數量，使用bytes且必須為8的倍數。                                                                                                                                                                                                                    | 8                                                       |
| Scheduler=[,SCHED_RR\|,SCHED_FIFO\|,SCHED_OTHER]                                                                | 選擇要使用的執行序之排程。在大多數系統中設定排成為其他值需要特權。可以設置為`SCHED_RR`、`SCHED_FIFO`、`SCHED_OTHER`。`SCHED_OTHER`為大部份系統之預設值。`SCHED_RR`為知更鳥排程演算法，`SCHED_FIFO`允許各個執行序執行直到其遭到封鎖或是在被換到其他執行序前完成。      | SCHED_OTHER                                             |
| scheduler_slice=usec                                                                                            | 某些作業系統如SunOS需要設定時間片段值當選擇非預設之排程。這個選項單位為微秒。                                                                                                                                                                                         | none                                                    |
| DCPSBidirGIOP=[0\|1]                                                                                            | 與DCPSInfoRepo互動時使用TAO’s BiDirectional GIOP功能。在允許BiDir時，因為相同的接口可以供客戶端以及伺服端同時，所以很少接口會被需要。                                                                                                                                 | 1                                                       |


DCPSInfoRepo設定值將被傳送至`CORBA::ORB::string_to_object()`且為可被TAO理解的任何類型的物件位址(檔案、corbaloc、corbaname)。
一個可被接受的簡單的終端描述格式為`(host):(port)`。這和`corbaloc::(host):(port)/DCPSInfoRepo`是一樣的。

當`RESOURCE_LIMITS`被設為無限時，`DCPSChunks`選項允許開發者調配記憶體被預配置的數量。
一旦系統的記憶體配置短缺時，heap中額外的DCPSChunks可能被收回/重新配置。
這個功能提供了相當的彈性但在記憶體短缺時會影響效能。

## 7.3 探索設定

在DDS實現中，參與者會在應用程式進程中成為實例且找到同樣參與其中的對方進行溝通。
一個DDS的實現中，兩個在相同網域的DDS參與者所交換的資料會由DDS的功能來給資料的內文。
在一個DDS應用程式中，參與者會被指派到一個網域且需要確定他們設定允許其他在相同網域的參與者能找到自己。

OpenDDS提供一個集中的探索機制、一個點對點的探索機制以及靜態的探索機制。
集中的機制使用在DCPSInfoRepo進程中的分離服務。
RTPS點對點機制使用DDSI-RTPS探索協定標準實現非集中式之探索。
靜態探索機制使用設定檔決定哪些讀者與寫者應該連結起來，以及使用底層傳送決定哪個讀者或是寫者存在。
DDS應用程式的部署會需要數個設定選項存在。
除了靜態探索之外，每個機制會在沒有提供指令或是檔案設定時使用預設值。
接下來的段落會介紹如何進階配置探索設定。
舉個例子，有些部署也許會需要多個DCPSInfoRepo服務或是DDSI_RTPS探索來滿足互通需求。

### 區域設定

