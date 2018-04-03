# CHAPTER 7

### Run-time Configuration

## 7.1 動手設定

OpenDDS使用以檔案為基礎的設定框架，其中包含全預選項，與特定發布者、訂閱者相關的選項\(如discovery設定以及傳送設定\)。  
OpenDDS也允許使用者透過執行文字指令，部份的選項可以透過API設定。  
這個章節將會描述OpenDDS支援的設定選項。

#### 1\) 一般設定選項：

設定在全域等級下DCPS實體的行為。  
這將可以讓在相同電腦環境下部署的不同進程共享同一個一般設定。  
\(例如所有的讀取者及寫入者都使用RTPS Discovery\)

#### 2\) Discovery設定選項：

設定Discovery機制下的行為。  
OpenDDS支援多種Discovery途徑及協調寫入者與讀取者。  
更多細節參照段落7.3。

#### 3\) 傳送設定選項：

設定可擴展傳送框架\(ETF\)，作為OpenDDS的DCPS層中的抽象傳輸層。  
每個可插拔式的傳送都可以被分別設定。

OpenDDS的設定檔案為易讀的ini格式文字檔。 表7-1為可允許的設定段落類型，他們都對應一種OpenDDS設定選項。

#### 表7-1 設定檔案段落

| 設定選項類型 | 檔案段落標題 |
| --- | --- |
| 全域設定 | \[common\] |
| Discovery | \[domain\] \[repository\] \[rtps\_discovery\] |
| 靜態 Discovery | \[endpoint\] \[topic\] \[datawriterqos\] \[datareaderqos\] \[publisherqos\] \[subscriberqos\] |
| 傳送 | \[config\] \[transport\] |

除了\[common\]之外，每個段落得標頭都可以用\[段落標題/實例\]的格式作設定，舉個例子，\[repository\]標頭可以這樣描述：  
\[repository/repo\_1\]，repository為段落標題，repo\_1為實例名稱，這個標頭下的設定將套用在repo\_1實例下的repository設定。  
如何使用實例來設定Discovery以及傳送選項將會在段落7.3以及7.4介紹。

`-DCPSConfigFile` 指令參數可以用來傳遞欲套用之OpenDDS設定檔案的位置。如下：

Windows：

```shell
publisher -DCPSConfigFile pub.ini
```

Unix：

```shell
./publisher -DCPSConfigFile pub.ini
```

指令參數將在domain參與因數\(domain participant factory\)初始化時傳遞至個別參與的實例。  
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

OpenDDS設定檔案中的\[common\]段落包含的選項像是除錯訊息輸出等級、DCPSInfoRepo進程位置以及預分配記憶體設定。下面為\[common\]設定範例：

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

\[common\]中的設定值若名稱以"DCPS"作開頭的皆可被指令列參數覆蓋。  
指令列參數名稱和設定檔中名稱相同，只是前面多了個"-"符號，例如：

```shell
subscriber -DCPSInfoRepo localhost:12345
```

下表描述\[common\]設定選項：

#### 表7-2 一般設定選項

| 選項名稱 | 敘述 | 預設值 |
| --- | --- | --- |
| DCPSBit=\[1\|0\] | 切換內建主題支援。 | 1 |
| DCPSBitLookupDurationMsec=msec | 框架等待當解讀實例控制所給的BIT data時遣在內建主題資訊的最長持續時間\(單位：毫秒\)。在框架取得且處理相關BIT資訊前，參與的程式碼會獲得一個給遠端實體的實例控制。框架將會在時間內等待，之後才提示操作失敗。 | 2000 |
| DCPSBitTransportIPAddress=addr | 內建主題之IP位置，用來識別tcp傳送將用到的本地端界面。 注意：這個屬性只對DCPSInfoRepo設定有效。 | INADDR\_ANY |
| DCPSBitTransportPort=port | 內建主題在TCP傳送中使用的端口。如果預設值'0'已經被使用，作業系統會自己挑別的端口來使用。 注意：這個屬性只對DCPSInfoRepo設定有效。 | 0 |
| DCPSChunks=n | 當`RESOURCE_LIMITS`Qos值為無限時，資料讀寫快取分配之可設定DCPSChunks數量會得到預先分配之快取。當所有DCPSChunks都被分配出去時，OpenDDS將會由heap再分配。 | 20 |
| DCPSChunkAssociationMultiplier=n | DCPSChunks的多工器或是藉由resource\_limits.max\_samples數值判斷預分配之DCPSChunks淺複製總數。將這個值設的比連接數量還大以防止預分配DCPSChunks過少。一個取樣寫入到多個資料讀取者不會執行多次完整複製而是只複製其開頭位址。這個動作佔用空間並不大，所以不需要把這個值設的太接近連結數量。 | 10 |
| DCPSDebugLevel=n | 一個0到10的整數，用以控制除錯訊息印出的數量。 | 0 |
| DCPSDefaultAddress | 預設值為傳送實例包含的`local_address`，只有在設定值不為空且沒有`local_address`在傳送中被指定時生效。 |  |
| DCPSDefaultDiscovery=\[,DEFAULT\_REPO\|,DEFAULT\_RTPS\|,DEFAULT\_STATIC\|,user-defined configuration instance name\] | 在無明確設定之domain中指定Discovery設定。`Default_REPO`將使用DCPSInfoRepo，`Default_RTPS`指定於Discovery部份之RTPS使用。`Default_STATIC`設定靜態Discovery之使用。細節請參見段落7.3。 | DEFAULT\_REPO |
| DCPSGlobalTransportConfig=name | 指定全域設定中的傳送設定名稱。這個設定將被所有沒有另外指定傳送設定之實體使用。$file中特殊的數值使用包含設定檔中所有被定義的實體的傳送設定。 | The default configuration is used as described in 7.4.1 |
| DCPSInfoRepo=objref | 物件指向DCPS資訊來源之位置。這可以是一個完整的CORBA IOR或是簡單的`host:port`字串 | file://repo.ior |
| DCPSLivelinessFactor=n | 當訊息送出時，最少的有效時間百分比。數值80等於20%最後一個有效訊息之潛在緩衝。 | 80 |
| DCPSPendingTimeout=sec | 一個資料寫入動作會封鎖一個刪除動作中未送出之樣品流失，其封鎖之最長持續時間。此數值之預設值為永久封鎖。 | 0 |
| DCPSPersistentDataDir=path | 持久資料將會儲存之檔案系統路徑。如果目錄不存在的話目錄將會被自動創建。 | OpenDDS-durable-data-dir |
| DCPSPublisherContentFilter=\[1\|0\] | 控制過濾內容Topic之過濾表達式的判斷策略。當允許\(1\)時，發布者可能會丟失幾個樣品，當這些樣品被訂閱者忽略時，他們不會在傳送中被處理。 | 1 |
| DCPSRTISerialization=\[0\|1\] | 當整合了使用RTPS傳送之RTI實體，控制是否使用非標準序列填充。 | 0 |
| DCPSTransportDebugLevel=n | 為一0到5之整數決定傳送日誌之訊息量。 | 0 |
| pool\_size=n\_bytes | 安全配置文件記憶體池大小，使用bytes。 | 41943040 \(40 MB\) |
| pool\_granularity=n\_bytes | 安全配置文件記憶體數量，使用bytes且必須為8的倍數。 | 8 |
| Scheduler=\[,SCHED\_RR\|,SCHED\_FIFO\|,SCHED\_OTHER\] | 選擇要使用的執行序之排程。在大多數系統中設定排成為其他值需要特權。可以設置為`SCHED_RR`、`SCHED_FIFO`、`SCHED_OTHER`。`SCHED_OTHER`為大部份系統之預設值。`SCHED_RR`為知更鳥排程演算法，`SCHED_FIFO`允許各個執行序執行直到其遭到封鎖或是在被換到其他執行序前完成。 | SCHED\_OTHER |
| scheduler\_slice=usec | 某些作業系統如SunOS需要設定時間片段值當選擇非預設之排程。這個選項單位為微秒。 | none |
| DCPSBidirGIOP=\[0\|1\] | 與DCPSInfoRepo互動時使用TAO’s BiDirectional GIOP功能。在允許BiDir時，因為相同的接口可以供客戶端以及伺服端同時，所以很少接口會被需要。 | 1 |

DCPSInfoRepo設定值將被傳送至`CORBA::ORB::string_to_object()`且為可被TAO理解的任何類型的物件位址\(檔案、corbaloc、corbaname\)。  
一個可被接受的簡單的終端描述格式為`(host):(port)`。這和`corbaloc::(host):(port)/DCPSInfoRepo`是一樣的。

當`RESOURCE_LIMITS`被設為無限時，`DCPSChunks`選項允許開發者調配記憶體被預配置的數量。  
一旦系統的記憶體配置短缺時，heap中額外的DCPSChunks可能被收回/重新配置。  
這個功能提供了相當的彈性但在記憶體短缺時會影響效能。

## 7.3 探索設定

在DDS實現中，參與者會在應用程式進程中成為實例且找到同樣參與其中的對方進行溝通。  
一個DDS的實現中，兩個在相同domain的DDS參與者所交換的資料會由DDS的功能來給資料的內文。  
在一個DDS應用程式中，參與者會被指派到一個domain且需要確定他們設定允許其他在相同domain的參與者能找到自己。

OpenDDS提供一個集中的探索機制、一個點對點的探索機制以及靜態的探索機制。  
集中的機制使用在DCPSInfoRepo進程中的分離服務。  
RTPS點對點機制使用DDSI-RTPS探索協定標準實現非集中式之探索。  
靜態探索機制使用設定檔決定哪些讀者與寫者應該連結起來，以及使用底層傳送決定哪個讀者或是寫者存在。  
DDS應用程式的部署會需要數個設定選項存在。  
除了靜態探索之外，每個機制會在沒有提供指令或是檔案設定時使用預設值。  
接下來的段落會介紹如何進階配置探索設定。  
舉個例子，有些部署也許會需要多個DCPSInfoRepo服務或是DDSI\_RTPS探索來滿足互通需求。

### Domain設定

OpenDDS設定檔使用`[domain]`段落設定一個或多個discovery domain，且每個domain指向相同檔案中的discovery設定或是預設設定。  
OpenDDS應用程式可以使用集中式的Discovery機制或是點對點的Discovery機制。前者為DCPSInfoRepo服務提供，後者為RTPS Discovery協定標準或是在相同部署中兩者的結合提供。

DCPSInfoRepo方法的設定在\[repository\]段落中，而RTPS Discovery設定在\[rtps\_discovery\]段落中。  
靜態Discovery機制並沒有特別的段落做設定用，不過使用者可以參考`DEFAULT_STATIC`實例。每個domain只能參考一種Discovery設定段落。

參考段落7.3.2關於設定\[repository\]段落，7.3.3關於設定\[rtps\_discovery\]以及7.3.4關於設定靜態Discovery&gt;

最後有兩種方式可以指派一整數給domain，一種是直接設實例的數值為該整數，如下：

```
[domain/1]
DiscoveryConfig=DiscoveryConfig1
(more properties...)
```

這個範例設定一個由domain關鍵字識別之domain且加上了實例數值/1。在斜線後的實例數值將指派給domain。也可以用字串代替整數，如下

```
[domain/books]
DomainId=1
DiscoveryConfig=DoscoveryConfig1
```

domain給指定了個比較可識別的名稱，`DomainId`屬性則指派了DDS應用程式讀取設定所需的整數。多個domain實體可以被單一個此格式的設定檔識別。  
一旦一個或多個實例被建立，Discovery屬性必須被該domain識別。`DiscoveryConfig`屬性必須指向另一個持有Discovery設定之段落或是指定其中一個內部預設的Discovery數值\(如`DEFAULT_REPO`、`DEFAULT_RTPS`、`DEFAULT_STATIC`\)。範例中的實體名稱為`Discovery_Config1`。  
這個實例的名稱必須和\[repository\]或是\[rtps\_discovery\]中的敘述連結，如下：

```
[domain/1]
DiscoveryConfig=DiscoveryConfig1
[repository/DiscoveryConfig1]
RepositoryIor=host1.mydomain.com:12345
```

這裡我們的domain指向了使用DCPSInfoRepo服務的\[repository\]段落，詳情參照7.3.2。  
當特定的domain不被設定檔識別，例如，如果OpenDDS應用程式指派了domain ID 3給了他的參與者且上面例子沒有提供domain設定，可以像下面這樣使用：

```
[common]
DCPSInfoRepo=host3.mydomain.com:12345
DCPSDefaultDiscovery=DEFAULT_REPO
[domain/1]
DiscoveryConfig=DiscoveryConfig1
[repository/DiscoveryConfig1]
RepositoryIor=host1.mydomain.com:12345
```

`DCPSDefaultDiscovery`屬性告訴應用程式指派任何沒能從設定黨獲取domain id的參與者使用Default\_Repo，且DCPSInfoRepo服務可以在host3.mydomain.com:12345被找到。在表7-2中`DCPSDefaultDiscovery`屬性有另外兩個值可以被使用，分別是DEFAULT\_RTPS和DEFAULT\_STATIC。  
最後一個在DCPSDefaultDiscovery屬性中的選項告訴應用程式使用其中一種已定義的Discovery設定作為所有不是從檔案中獲取設定值的參與者的預設設定，如下：

```
[common]
DCPSDefaultDiscovery=DiscoveryConfig2
[domain/1]
DiscoveryConfig=DiscoveryConfig1
[repository/DiscoveryConfig1]
RepositoryIor=host1.mydomain.com:12345
[domain/2]
DiscoveryConfig=DiscoveryConfig2
[repository/DiscoveryConfig2]
RepositoryIor=host2.mydomain.com:12345
```

加入`DCPSDefaultDiscovery`屬性到\[common\]段落，沒有特別指派設定的參與者將會使用DiscoveryConfig2。更多細節參考段落7.3.3。  
以下為\[domain\]段落中可用的屬性。

#### 表7-3 Domain段落設定屬性

| 選項 | 敘述 |
| --- | --- |
| DomainId=n | 一個代表正與repository關聯之domain的整數 |
| DomainRepoKey=k | 映射repository之鍵值。 \(已經無效了，只提供過去版本相容\) |
| DiscoveryConfig=config instance name | 使用者定義之字串，提交到同一設定檔中的\[repository\]或是\[rtps\_discovery\]中的實例名稱或是預設值\(DEFAULT\_REPO、DEFAULT\_RTPS和DEFAULT\_STATIC\)。 |
| DefaultTransportConfig=config | 使用者定義字串，提交到\[config\]中的實例名稱。 |

## 7.4傳輸設定

OpenDDS 3.0 開始一個新的傳輸設定設計。  
基本目設計為：

* 允許佈署忽略傳輸設定，使用預設設定\(在 publisher 或 subscriber 不需要有關傳輸的程式碼\)
* 能夠只使用傳輸設定文件或是輸入指令
* 允許 data writers 和 writers 個別佈署最小傳輸。Publishers 與 subscribers 討論傳送方法，基於傳輸設定的細節、 QoS 設定 、網路的可行性。
* 在複雜網路中支援更寬廣的應用
* 支援傳輸優化佈署\(像是搭配和分享記憶體傳輸\)-注意這些不是完全的實作
* 支援 RELIABILITY QoS 整合底層傳輸
* 避免依賴 ACE 服務設定和它的設定檔。不幸的這些新的功能與先前的 OpenDDS transport configuration 程式碼不相容。請看 $DDS\_ROOT/docs/OpenDDS\_3.0\_Transition.txt 獲得如何改造你已經有的應用程式來使用新的傳輸設定設計。

## 7.4.1概述

## 7.4.1.1傳輸概念

這個章節會概述有關傳輸設定和他們的作用。每個傳輸的特別實作（例如tcp,udp,multicast,shmem,或rtps\_udp）還有設定自定義參數。藉由TransportRegistry管理傳輸設定、傳輸實例還有透過設定文件或是API來創建。

傳輸設定可以指定DomainParticipants,Publishers,Subscribers,DataWriters,DataReaders。當DataReader或DataWriter使啟用時，他可以使用他找到的設定或者直接使用父節點的設定。例如DataWrite指定了一個傳輸設定那他就會是那個設定。如果沒有指定設定它會試著去尋找Publisher或是同一doamin的。如果這些還是沒有指定設定檔，那會去TransportRegistry得到全域傳輸設定。如果沒有被使用者指定，那就會使用預設參數包含啟用所有傳輸實例。如果你沒有指定載入或是連結到任何傳輸實例，OpenDDS會使用tcp來進行所有的通訊。

## 7.4.1.2OpenDDS如何選擇傳輸

顯然在OpenDDSDataReader是被動的等待DataWriters的連接。DataReader"聆聽"每一個在傳輸設定的每一個實例。DataWriters使用傳輸實例去"連接"這些DataReader。因為被討論的邏輯連結並不是真的物理連結，OpenDDS把他們叫作資料連結\(DataLinks\)。

當DataWriter嘗試去連結DataReader，它會先嘗試去看有沒有任何的連結在和DataReader通訊。DataWriters重複的在傳輸實例中尋找reader定義的傳輸實例。如果存在資料連結那DataWriter和DataReader就會用來通訊。

如果找不到存在的資料連結，DataWriter嘗試去連線其他傳輸設定中不同設定的的傳輸實例。略過不是"吻合"的另一個傳輸實例。舉例來說writer去指定udp和tcp的傳輸實例，但reader只有指定tcp那udp就會被忽略。配對演算法也會被QoS設定參數和其他傳輸實例的細節。成功連線後的第1部份是傳輸所有的推送資料樣本。

## 7.4.2範例設定檔案

將會透過檔案解釋基本傳輸設定功能和描述一般使用情境。以下參考文件將會完全的解說。

## 7.4.2.1單個傳輸設定

這是簡單的OpenDDS設定檔，提供給你的應用程式去使用的傳輸設定。這個簡單的設定檔可以被兩個在不同網路電腦上的應用溝通時使用。

```
[common]

DCPSGlobalTransportConfig=myconfig

[config/myconfig]
transports=mytcp

[transport/mytcp]
transport_type=tcp
local_address=myhost
```

這個設定檔說明\(從下往上\):  
1. 定義一個傳輸實例使用 tcp 和定義一個 myhost 的網址。代表我們要用的網路介面。  
2. 定義傳輸設定叫 mytcp 的唯一傳輸實例。  
3. 使傳輸設定為全域的設定並命名為 myconfig 給所有實體行程。

這個行程使用這個設定檔直到我們對所有的DataWriter和DataReader都做各自的設定\(除非我們在程式中綁定其他的設定在7.4.2.3中會描述\)

## 7.4.2.2使用混合傳輸

這個範例讓應用程式主要去使用多播\(multicast\)和當不能使用多播時"fallback"到tcp。設定檔如下：

```
[common]
DCPSGlobalTransportConfig=myconfig

[config/myconfig]
transports=mymulticast,mytcp

[transport/mymulticast]
transport_type=multicast

[transport/mytcp]
transport_type=tcp
```

這個叫 myconfig 的傳輸設定檔包含兩個傳輸實例 mymulticast 和 mytcp 。也沒有任何參數在指定再 transport\_type 旁邊，所以他們使用預設的傳輸設定實例。使用者可以自由的使用列表上的傳輸設定。  
假設所有的傳輸都用這個設定檔，應用程式會在 data writer 和 data reader 之中使用多播去初始化。如果多播初始化失敗不管甚麼原因\(也許路由器干涉多播\)會去初始化 tcp 連線。

## 7.4.2.3使用混合設定

很多的應用程式的通訊只靠一個設定是不夠的。這些應用程式需要多個設定來滿足不同的實體程序。

例如有一個電腦上有兩個網路介面，上面的應用程式通訊以些是經過其中一個介面，剩下的經過另外一個。這是我們的設定檔。

```
[common]
DCPSGlobalTransportConfig=config_a

[config/config_a]
transports=tcp_a

[config/config_b]
transports=tcp_b

[transport/tcp_a]
transport_type=tcp
local_address=hosta

[transport/tcp_b]
transport_type=tcp
local_address=hostb
```

假設hosta和hostb是指派給兩個網路介面的名子，現在可以分離各自的tcp設定。把"A"設定設為預設，"B"設定為手動設定。

OpenDDS提供兩種機制去選擇實體設定:

* 透過程式來選擇實體的設定\(reader, writer, publisher, subscriber 或 domain participant\)
* 藉由 doamin 的關聯設定

這裡有些綁定機制的程式碼

```
DDS
:
:
DomainParticipant_var dp 
=

dpf
-
>
create_participant
(
MY_DOMAIN
,

PARTICIPANT_QOS_DEFAULT
,

DDS
:
:
DomainParticipantListener
:
:
_nil
(
)
,

OpenDDS
:
:
DCPS
:
:
DEFAULT_STATUS_MASK
)
;

OpenDDS
:
:
DCPS
:
:
TransportRegistry
:
:
instance
(
)
-
>
bind_config
(
"config_b"
,
 dp
)
;
```

任何在這個DoaminParticipant下的DataWriter，DataReader會使用"B"那邊的設定。

---

注意

當直接綁定DataWriter或DataReader的設定時，要再啟動reader或writer之前呼叫bind\_config。這不是問題，當去綁定DomainParticipants，Publishers，Subscribers的設定。在3.2.16有更多關於如何創造沒有啟用的實體。

---

## 7.4.3傳輸註冊表範例

OpenDDS允許開發者去定義傳輸設定或是透過C++API實例。OpenDDS::DCPS::TransportRegistry用於建構OpenDDS::DCPS::TransportConfig和OpenDDS::DCPS::TransportInst物件。TransportConfig和TransportInst類別是包含推送資料成員相應的操作定義。這個章節包含相當於簡單傳輸設定描述的程式。首先要載入需要的標頭檔。

```
#include 
<
dds
/
DCPS
/
transport
/
framework
/
TransportRegistry
.
h
>

#include 
<
dds
/
DCPS
/
transport
/
framework
/
TransportConfig
.
h
>

#include 
<
dds
/
DCPS
/
transport
/
framework
/
TransportInst
.
h
>

#include 
<
dds
/
DCPS
/
transport
/
tcp
/
TcpInst
.
h
>

using namespace OpenDDS
:
:
DCPS
;
pp
```

接下來我們創造傳輸設定、創造傳輸實例、設定傳輸實例還有增加傳輸設定集合。

```
TransportConfig_rch cfg 
=
 TheTransportRegistry
-
>
create_config
(
"myconfig"
)
;

TransportInst_rch inst 
=
 TheTransportRegistry
-
>
create_inst
(
"mytcp"
,
// name 
"tcp"
)
;
// type
// Must cast to TcpInst to get access to transport-specific options

TcpInst_rch tcp_inst 
=
 dynamic_rchandle_cast
<
TcpInst
>
(
inst
)
;

tcp_inst
-
>
local_address_str_ 
=
"myhost"
;
// Add the inst to the config

cfg
-
>
instances_
.
push_back
(
inst
)
;
```

最後創造我們新的全域傳輸設定。

```
TheTransportRegistry
-
>
global_config
(
"myconfig"
)
;
```

這些程式碼因該概啟用DataReader和DataWriter之前。

這些標頭檔可以列出所有數據成員和可用函數。觀看下面章節的選項描述來完整的了解設定方法。

7.4.4傳輸設定選項

OpenDDS的傳輸設定檔案指定的傳輸設定透過\[config/\]格式，name是那個進程中的為一值。

表7-12傳輸設定選項

| 選項 | 描述 | 預設 |
| :--- | :--- | :--- |
| Transports=inst1\[,inst2\]\[,...\] | 將會使用的傳輸實例名稱列表。每個傳輸設定都需要 | none |
| swap\_bytes=\[0\|1\] | 0為在本機對資料做序例化1為不再本機做序列化。接收方要調整字母順序所以兩台機器不用相同。這個選項的目的是為了讓開發者決定那邊去做排序。如果必要的話。 | 0 |
| passive\_connect\_duration=msec | 被動建立連線逾時\(毫秒\)。預設會等待10秒。如果設0則會無線等待\(不建議\) | 10000 |

passive\_connect\_duration選項通常會設定不是零的正整數。沒有適合的連線預時，subscriber的狀態會變成鎖死在等待遠端初始化連線。因為會有多重傳輸實例在publisher和subscriber端，這個值要設的夠高讓publisher去重複嘗試到成功。

除此之外使用者定義設定，OpenDDS可以隱含設定兩個通訊設定。第1個是預設設定和所有程序的連線通訊實例。如果沒有那就只會用TCP。所有傳輸都會用預設的傳輸設定。在使用者沒有定義設定時會使用全域設定。

第2隱含傳輸設定，每當OpenDDS設定檔被使用。他被命名與正在讀取的檔名相同並且包含所有再檔案中定義的實例，以字母順序排序。讓使用者更簡單的使用設定藉由指定同依檔案中的DCPSGlobalTransportConfiguration=$file。$file的值總是綁店實體檔案的檔名。

## 7.4.5傳送實例選項

傳輸實例在OpenDDS設定檔中的\[transport/\]部份，為進程中的唯一值。每個傳輸實例必須指定transport\_type為有效的傳輸實例型態。下列出其他可以用的選項，開始使用這些選項給所有的傳輸型態並跟隨每一個指定型態。

當使用動態函式庫，OpenDDS傳輸函數庫是動態載入再定義每個設定檔案中的實例。在使用制定傳輸實例或是靜態連結，應用程式開發者要負責確保每個傳輸實例程式是可執行的。

## 7.4.5.1所有傳輸常用選項

下面的表格總結一下常用的選項

表7-13常用的傳輸設定選項

| 選項 | 敘述 | 預設 |
| :--- | :--- | :--- |
| transport\_type=transport | 傳輸型態;列出可被程式擴展的傳輸架構,tcp,udp,multicast,shem,andrtps\_udp是被包含在OpenDDS | none |
| queue\_messages\_per\_pool=n | 當被檢測到backpressure要送出的訊息是在排隊。訊息駐列就要增加，增加的量就是這個值 | 10 |
| queue\_initial\_pools=n | 初始化backpressure的駐列數。預設50的訊息\(5個池，每個10個訊息\) | 5 |
| max\_packet\_size=n | 最大傳輸封包，包含傳輸標頭，樣本標頭，樣本資料 | 2147481599 |
| max\_samples\_per\_packet=n | 最大傳輸樣本數 | 10 |
| optimum\_packet\_size=n | 當累積封包數大於這個大小將會送出去，這個會受到你的網路和應用性質影響 | 4096 |
| thread\_per\_connection=\[0\|1\] | 開啟或關閉一個線程一個傳輸策略。預設為關閉。 | 0 |
| datalink\_release\_delay=sec | 資料連結\(datalink\)在沒有關聯後的幾秒釋放掉。增加這個值可以減少reader/writer重新建立連線的次數和頻率 | 10 |

開啟thread\_per\_connection選項將會增加寫資料給多個不同程序的Reader的效能，只要切換程序的動作可以平行於寫入動作。切換的網路消耗最好藉由實驗來測試。如果有多張網卡的電腦，增加每張網卡的傳輸設定是一個增加效能的方法。

## 7.4.5.2TCP/IP設定選項

這裡有多個tcp傳輸設定選項。一個好的傳輸設定會提供增加潛在要求的彈性。幾乎所有的選項都允許客制化合理預設連線和重新連線的策略，可是最終這些值的設定都要注意網路品質和想要的DDS應用的QoS上與目標環境。

local\_address用於建立同行溝通。預設TCP傳輸會選擇暫時的阜號來解決建立NIC的FQDN\(fullyqualifieddomainname\)。因此如果你有多個NIC或是希望去指定阜號會想明確的地址\(address\)。當你設定主機間的通訊，local\_address就不能是127.0.0.1要設成實體IP或是你可以不指定IP那將會使用FQDN和一個短暫的阜號。

FQDN依賴系統的設定。在缺少FQDN\(例如：example.ociweb.com\)，OpenDDS將會去探索一個短的名子\(例如:example\)如果失敗將會使用地址來當作名子\(例如：localhost\)。

---

注意

## OpenDDS在建制ACE/TAO時開啟IPv6之後就可以支援IPv6。local\_address也要是十進位的IPv6或有阜好的FQDN而且要能解決IPv6。

TCP傳輸要在一個被使用的獨立的函式庫中。當使用動態函式庫時，OpenDDS會自動的載入傳輸設定檔中的傳輸函式庫或是預設傳輸。

在建立靜態TCP函式庫時，你的應用程式要能夠直接的指定函式庫。為你的應用程式要先載入合適的標頭檔來初始化。

你也可以以程序方式來描述設定publisher和subscriber。設定Publisher和Subscriber要相同，除非不同的阜號/地址要個別的設定傳輸實例。

根據下表總結一下傳輸設定選項是不同於tcp傳輸。

| 選項 | 描述 | 預設 |
| :--- | :--- | :--- |
| conn\_retry\_attempts=n | 重試連線次數和使用on\_publication\_lost\(\)跟on\_subscription\_lost\(\)回傳 | 3 |
| conn\_retry\_initial\_delay=msec | 重新連線時間\(毫秒\)。在失去連線重新連線時觸發。 | 500 |
| conn\_retry\_backoff\_multiplier=n | 當重新連線失敗時，將延遲的時間乘上設定的數值為下次重新連線的時間。例如：conn\_retry\_initial\_delay=500conn\_retry\_backoff\_multiplier=1.5，第一次重先連線時間為500ms，如果失敗下次連線時間為750ms | 2.0 |
| enable\_nagle\_algorithm=\[0\|1\] | 啟用或停用Nagle演算法。啟用將會增加延遲時間和吞吐量。預設為禁用 | 0 |
| local\_address=host:port | 主機位址和阜號。預設為FQDN和系統設定阜號。如果只指定主機但":"不能省略。 | fqhn:0 |
| max\_output\_pause\_period=msec | 最大訊息等待時間。如果如果樣本排隊超過時間則會呼叫on\_\*\_lost\(\)。預設為不使用。 | 0 |
| passive\_reconnect\_duration=msec | 被動等待連接超過時間\(ms\)未被重先連接則會呼叫on\_\*\_lost\(\) | 2000 |
| pub\_address=host:port | 覆蓋同級發送的網址。用於防火牆和進階網路設定 |  |

## TCP/IP重新連線選項

當 TCP/IP 段開連線時 OpenDDS 會嘗試重新連線。重新連線的過程如果連線成功則結束。  
1.斷開連線時立即重新連線  
2.如果連驗失敗等待 conn\_retry\_initial\_delay 毫秒後重新連線  
3.當嘗試失敗次數大於 conn\_retry\_attempts ，等待 conn\_retry\_initial\_delay \* conn\_retry\_backoff\_multiplier 毫秒後重新連線。

7.4.5.3UDP/IP傳輸設定選項

UDP傳輸只有努力地發送，跟TCP、local\_address一樣支援IPv4、IPv6。

UDP依賴在其他的函數庫、設定上，因此要要連結上就像其他傳輸設定。OpenDDS會自動的使用動態函數庫當被設定在設定檔中。如果使用靜態函數庫你的應用程式要直接的指向函數庫。除此之外你的應用程式也要載入正確的函數庫來初始化服務。

下表總結UDP獨特的傳輸設定。

表7-13UDP/IP設定選項

| 選項 | 敘述 | 預設 |
| :--- | :--- | :--- |
| local\_address=host:port | socket的主機和阜號。預設主機是本機。阜號可省略但":"不能省。 | fqdh:0 |
| send\_buffer\_size=n | UDP的有效傳送緩衝大小。 | PlatformvalueofACE\_DEFAULT\_MAX\_SOCKET\_BUFSIZ |
| rcv\_buffer\_size=n | UDP接收緩衝的大小 | PlatformvalueofACE\_DEFAULT\_MAX\_SOCKET\_BUFSIZ |

7.4.5.4IP多播傳輸設定

多包設定統一支持最大努力和可靠的傳輸基於傳輸設定參數。

在點對點交換數據盡力以最少的消耗交換數據，但是任何的保證傳輸，資料有可能丟失因為沒有回應、無法到達另一點或是重複接收。

可靠的傳輸沒有複製關聯點的額外的行程和頻寬。可靠的傳輸經過兩的主機器:雙方交握和被動的資料遺失確認。這些機器是有限制的確保行為確定性和可以依照使用者環境去設定邊界。

多撥支援多個設定選項。

default\_to\_ipv6和port\_offset選項影響預設的多播群組地址選擇。default\_to\_ipv6設為1\(啟動\)，預設的IPv6為FF01::80。port\_offset選項預設為49152。

group\_address選項可以手動設定來定義多播群組加入交換資料。IPv4和IPv6都有支援。OpenDDSIPv6默認支援需求ACE/TAO組件，當建置支援IPv6啟動。

在本機多播網路介面可以指定加入多播群組在指定的介面。local\_address選項可以去設定網路位置來讓本機網卡接收多傳輸。

如果想一可靠的傳輸 reliable 選項使要被指定的\(預設為啟動\)。設定會影響其餘機器的多播傳輸。  
syn\_backoff， syn\_interval 和 syn\_timeout 設定選項影響機器交握。syn\_backoff 是一個指數基底用來計算避免延遲重試。syn\_interval 選項定義了最小重試交握毫秒數。syn\_timeout 定義了最大重試交握次數。

可以由syn\_backoff和sys\_interval來計算交握嘗試次數時間\(boundedbysyn\_timeout\)。

delay=syn\_interval\*syn\_backoff^number\_of\_retrues

例如預設假定嘗試交握延遲時間為0.25、1000、2000、4000和8000毫秒。

nak\_depth、nak\_interval和nak\_timeout設定選項影響NegativeAcknowledgmentmechanism。nak\_depth確定最大傳輸服務修理數據。nak\_interval設定選項定義最小修理請求延遲。亂數間格是為了避免傳輸碰撞。最大延遲時間為兩倍的最小延遲時間。

nak\_timeout設定選項最大延遲放棄等待時間。

nak\_delay\_intervals設定選項定義最初naks的間格數。

nak\_mak設定選項限制最大次數的naked範例遺失。適用這個選項nak將不會反覆傳送封包直到nak\_timeout。

正確來說有一對區需求超越數值藉由使用ETF傳輸。

* 每個 DDS 域的多播群組最多一個
* 給予每個參與者附上單個多播傳輸的群組;如果你想要發送接收範例在相同程序中的多播群組，參與者是必須的。

多播存在依賴於函數庫因此必須連結和設定像是其他的傳輸設定。OpenDDS會自動的載入設定檔中的動態函式庫。在使用靜態的函式庫時你的應用程式要直接連結到指定函式庫。除此之外你的應程式要載入正確的標頭檔來初始化服務。

下表總結了多播傳輸唯一的設定

表7-16IP多播傳輸設定選項

| 選項 | 敘述 | 預設 |
| :--- | :--- | :--- |
| default\_to\_ipv6=\[0\|1\] |  | 0 |
| group\_address=host:port |  | 224.0.0.128:,\[FF01::80\]: |
| local\_address=address |  |  |
| nak\_delay\_intervals=n |  |  |
| nak\_depth=n |  | 32 |
| nak\_interval |  | 500 |
| nak\_max=n |  | 3 |
| nak\_timeout=msec |  | 30000 |
| port\_offset=n |  | 49152 |
| rcv\_buffer\_size=n |  | 0 |
| reliable=\[0\|1\] |  | 1 |
| syn\_vackoff=n |  | 1 |
| syn\_interval=msec |  | 250 |
| syn\_timeout=msec |  | 30000 |
| ttl=n |  | 1 |
| async\_send=\[0\|1\] |  |  |



