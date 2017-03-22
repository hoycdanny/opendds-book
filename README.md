# **CHAPTER 1**

### **Introduction**


OpenDDS是建立於 OMG Data Distribution Service (DDS) 的 Real-Time Systems V1.4(OMG 文件格式/2015-04-10) 和 Real-time Publish-Subscribe Wire Protocol DDS Interoperability Wire Protocol Specification(DDSI-RTPS) v2.2 (OMG Document formal/2014-09-01)。OpenDDS 是由 Object Computing, Inc. (OCI) 贊助。 OpenDDS 網址為[http://www.OpenDDS.org/](http://OpenDDS.org/)。開發人員指南基於 3.10 釋出版本的 OpenDDS。

DDS 的定義為在分散式的應用程式有效的傳輸分散式的應用程式資料。這個服務不特定為 CORBA 。提供 Platform Independent Model (PIM) 以及將 Platform Specific Model (PSM) 映射成 CORBA IDL。

有關DDS的其他詳細信息，開發人員應參考DDS規範（OMG文件格式/ 2015-04-10），因為它包含對所有服務的功能的深入涵蓋。

OpenDDS 是 C++ 所寫的開原軟體，OCI 提供 OMG's DDS 規範開發和商業支持。能夠從[http://www.OpenDDS.org/downloads.html](http://www.OpenDDS.org/downloads.html)下載和 OCI TAO version 2.0a and 2.2a 級別的補丁以及最新的文件。

注意:OpenDDS目前實現OMG DDS 1.4版規範。 有關詳細訊息，請參閱[http://www.OpenDDS.org/](http://www.OpenDDS.org/)中的規格訊息。

# 1.1. DCPS概述

在本節中，我們將介紹DCPS層的主要概念和實體，並討論它們如何相互影響和一起工作。

## 1.1.1基本概念

圖1-1顯示了DDS DCPS層的概述。 子節點的概念定義再下面那張圖中所示。

### ![](/1.1.jpg)

### 1.1.1.1 Domain

Domain 是 DCPS 中的基本分區單元。每個實體屬於一個Domain，並且只能與該 Domain 中的其他實體相互影響。

程式碼可以自由的與多個 Domain 作用，但必需藉由分離不同 Domain 的實體來做到。

### 1.1.1.2 DomainParticipant

Domain participant是應用程序與特定的 Domain 互相作用的入口點。Domain participant 是個工廠給許多物件參與讀取寫入的地方。

### 1.1.1.3 Topic

在應用程序中 Topic 是Publishing 和 Subscribing 溝通的基本手段。每個 Topic 在 Domain 中不能重複，以及發布它的特定數據類型。每個主題數據類型可以指定組零個或多個字段以及他的key(make up its key)。在發送資料時,發送程序總是會指定一個 Topic 。Subscribers 通過 Topic 請求數據。在DCPS術語中，你在 Topic 推送個人不同的範例資料。每個實例都與唯一的 key 有關聯。推送過程中多個相同實例範本資料使用相同的key。

### 1.1.1.4 DataWriter

推送應用程序碼使用 DataWriter 將值傳遞到DDS。每個 DataWriter 都綁定到一個特定的 Topic。應用程式使用特別的 DataWriter 介面來發送樣本到 Topic。DataWriter 負責封裝資料並傳送。

### 1.1.1.5 Publisher

Publisher 負責接收發送資料並傳給在同一 Domain 底下有關的 Subscribe。精確的資訊實作是由服務來決定。

### 1.1.1.6 Subscribe

Subscribe 接收從 Publisher 的資料並給相關的 DataReader。

### 1.1.1.7 DataReader

DataReader 從 Subscribe 接收資料並轉換成對應 Topic 的資料型態。每個 DataReader 綁定一個特定的 Topic。應用程式使用 DataReader 的特別的介面來接收樣本。

## 1.1.2 Built-In Topics

DDS中定義了數個 Topic。訂閱內建的 Topic 可以讓開發者存取 Domain 的狀態包含，註冊的 Topic ,那些 DataReader DataWriter 是連接的還有不同接口的 QOS 設定。在訂閱時,應用程式會接收在 Domain 改變的實體範例。

下表顯示了在DDS規範中定義的內置主題：

| 主題名稱 | 描述 |
| :--- | :--- |
| DCPSParticipant | 每個實例的 Domain participant |
| DCPSTopic Each | 每個實例表的一般（非內置）主題。 |
| DCPSPublication | 每個實例的 DataWriter|
| DCPSSubscription | 每個實例的 DataReader  |

## 1.1.3 服務質量方針(QOS)

DDS規範定義了許多服務質量（QoS）方針，應用程式使用指定的 QOS 需要的服務。 參與者指定他們從服務中需要什麼行為，服務決定如何實現這些行為。 這些這些
這些 qos 可以給各種 DCPS 實例((Topic, data writer, data reader, publisher, subscriber, Domain participant) 然而不是所有的 qos 是對所有的實體是有效的。

Subscribers and publishers 使用 equest-versus-offered (RxO) model 來配對。
訂閱者請求一組最低限度需要的qos。 發布者向潛在訂戶提供一組QoS。 DDS實現嘗試將所有的需求和提供的 qos 配對; 如果這些 qos 是兼容的，則形成關聯。

OpenDDS當前實現的 QoS 將在第3章中詳細討論。

## 1.1.4 Listeners

DCPS層為每個實體定義回調接口，其允許應用進程“監聽”關於該實體的某些狀態改變或事件。 例如，當有可用於讀取的數據值時，通知 Data Reader Listener。

### 1.1.5 Conditions

允許 Conditions 和 Wait Sets 代替 Listeners 監聽有關的 DDS。 一般模式是應用程序創建一個特定類型的Condition，例如StatusCondition，並將其附加到WaitSet。

  •應用程序等待WaitSet，直到一個或多個 Conditions 成為真。

  •應用程序調用對相應實體對象以提取必要的信息。

  •DataReader接口還具有讀取ReadCondition參數的方法。

  •QueryCondition 物件可以被作為實現 Content-Subscription 。 QueryCondition 可用來擴展 ReadCondition。

## 1.2 OpenDDS實現

### 1.2.1合規性

OpenDDS符合OMG DDS和OMG DDSI-RTPS規範。 遵守的細節在這裡。

### 1.2.1.1 DDS合規性

DDS規範的第2節定義了DDS實現的五個合規點：

1）最低配置文件

2）內容訂閱

3）持續性

4）所有權簡介

5）物件模型配置文件

OpenDDS符合整個DDS規範（包括所有可選配置文件）。 這包括實施所有服務品質，並註意以下事項：

•僅當使用 TCP or IP Multicas 傳輸（包含所有有效選項）或使用 RTPS_UDP 傳輸時，才支持RELIABILITY.kind = RELIABLE。

•TRANSPORT_PRIORITY 不能更改。

### 1.2.1.2 DDSI-RTPS合規性

OpenDDS實現符合OMG DDSI-RTPS規範的要求。

# OpenDDS RTPS實現要點
實施 OMG DDSI-RTPS規範（formal/ 2014-09-01），但不是合規要求。使用 OpenDDS RTPS 功能來傳輸或發現要注意。每個項目提供了DDSI-RTPS規範的部分編號，以供進一步參考。

#### 未在OpenDDS中實現的項目：

1）寫入器端內容過濾（8.7.3）OpenDDS可能仍然丟棄任何相關 readers 不需要的樣本（由於內容過濾） - 這是在傳輸層之上完成的

2）PRESENTATION QoS的相干集（8.7.5）

3）定向寫入（8.7.6）

4）屬性列表（8.7.7）

5）原始資料的有效性（8.7.8） - 這將僅用於臨時和持久持久性，這是RTPS規範不支持的（8.7.2.2.1）

6）Key Hashes（8.7.9），為選用

7）nackSuppressionDuration（表8.47）和heartbeatSuppressionDuration

（表8.62）。

注意:上面的項目3和4在 DDSI-RTPS 規範中描述。 然而，它們在DDS規範中沒有相應的概念。

## 1.2.2擴展到DDS規範

•在 DDSIDL 模組e (C++ namespace, Java package)中資料型態，介面，常數直接對應 DDS 規範，有少數例外:

•DDS :: SampleInfo包含一個以“OpenDDS_reserved”開頭的附加字段，

•特定於類型的DataReaders（包括內置 Topic 的DataReaders）具有其他操作read_instance_w_condition（）和take_instance_w_condition（）。額外的擴展行為由OpenDDS模塊/命名空間/包中的各種類和接口提供。 這些功能包括像是 Recorder和Replayer（見第12章）以及以下功能：

•OpenDDS::DCPS::TypeSupport 在DDS規範中未找到的unregister_type()。

•OpenDDS::DCPS::ALL_STATUS_MASK，NO_STATUS_MASK 和 DEFAULT_STATUS_MASK 是 DDS::Entity，DDS::StatusCondition 和各種create\_\*（）操作使用的 DDS::StatusMask 常用常數。

## 1.2.3 OpenDDS架構

本節簡要概述了OpenDDS實現，其特性及其一些組件。$DDS_ROOT 環境變量應指向 OpenDDS 的基本目錄。 OpenDDS的源代碼可以在 $DDS_ROOT/dds/ 目錄下找到。 DDS測試可以在 $DDS_ROOT/tests/下找到。

### 1.2.3.1 設計哲學

OpenDDS 實現和 API 基於對 OMG IDL PSM 的相當嚴格的解釋。幾乎在所有情況下，OMG 的 CORBA IDL 的 C++ 語言映射用於定義 DDS 規範中的 IDL 如何映射到 OpenDDS 向客戶端提供的 C++ API。

與 OMG IDL PSM 的主要偏離在於本地接口用於實體和各種其他接口。這些在 DDS 規範中定義為非約束（非本地）接口。將它們定義為本地接口可以提高性能，減少內存使用，簡化客戶端與這些接口的交互，並使客戶端更容易構建自己的實現。

### 1.2.3.2 可擴展傳輸框架（ETF）

OpenDDS使用 DDS 規範定義的 IDL 接口來初始化和控制服務使用。數據傳輸通過 OpenDDS 特定的傳輸框架來實現，該傳輸框架允許服務與各種傳輸協議一起使用。這被稱為可插拔傳輸，使 OpenDDS 的可擴展性成為其架構的重要組成部分。 OpenDDS目前支持 TCP/IP，UDP/IP ，IP multicast，共享存儲和 RTPS_UDP 傳輸協議，如圖1-2所示。傳輸通常通過配置文件指定，並附加到發布者和訂閱者進程中的各種實體。有關配置ETF組件的詳細信息，請參見第7.4.4節。

![](/1.2.jpg)

ETF使應用程序開發人員能夠實現自己的定制傳輸。實現自定義傳輸需要在多個特定傳輸框架之下。 udp 傳輸提供了開發人員在創建自己的實體時可以使用的良好基礎。 有關詳細信息，請參閱$DDS_ROOT/dds/DCPS/transport/udp/ 目錄。

### 1.2.3.3 DDS發現

DDS 應用程序必須通過某個中央代理或通過某種分佈式方案彼此發現。一個 OpenDDS 重要的特點就是 DDS 應用可以設定去執行 DCPSInfoRepo 或 RTPS discovery 來做發現，但利用不同的傳輸形式和資料型態在資料讀取與資料寫入。OMG DDS 格式 (formal/2015-04-10) 留下了發現的實作細節。在 DDS 實現之間的互操作性的情況下，OMG DDSI-RTPS（formal/ 2014-09-01）規範提供了對等發現的對等方式的要求。

OpenDDS 提供兩個發現選項。

1）信息庫：一種集中的存儲庫風格，作為一個單獨的過程運行，允許發布者和訂閱者集中發現另一個

2）RTPS發現：在點對點樣式使用，利用 RTPS 協議來廣播可用性還自己的資訊。

與其他DDS實現的互操作性必須利用點對點方法，但在只有 OpenDDS 的部署中很有用。

### 使用DCPSInfoRepo進行集中式發現

OpenDDS實現一個稱為DCPS的獨立訊息服務（DCPSInfoRepo）實現集中式發現方法。它被實現為 CORBA 服務器。當客戶端請求預訂 Topic 時，DCPS 信息庫確認訂閱者的位置並通知給所有存在的通送者新的接收者的位置。在不是使用 RTPS 設定時 DCPSInfoRepo 將會啟用。RTPS 啟動時不會使用 DCPSInfoRepo 。DCPSInfoRepo不參與數據傳播，其作用只用於 OpenDDS 應用程式發現彼此。

![](/1.3.jpg)

應用程序開發人員可以自由運行多個信息庫，每個管理其自己的非重疊的DCPS域集合。

還可以操作具有多於單個存儲庫的 Domain ，從而形成分佈式虛擬存儲庫。這稱為存儲庫聯合。為了使各個存儲庫參與聯合，每個庫必須在啟動時指定其自己的聯合標識符值（一個32 bit的數）。有關存儲庫聯合的更多信息，請參見9.2。

### RTPS 的點對點發現

DDS 的點對點模式可相容於 OpenDDS 的模式。此方法可由當前版本的 RTPS 協議來實現。這種簡單的發現形式是通過簡單的 DDS 應用設定完成讀取資料和寫入資料會顯示錯誤：找不到參考源。
每個參與形成都會啟動 DDSI—RTPS 資料讀寫的發現機制，預設或是設定的網路阜號這樣 DDS 參與者可以開始廣播它的資料讀取接收是可以使用的。一段時間後，基於標準找到彼此和討論設定基本連接設定插件在 Extensible Transport Framework (ETF)之中。這種靈活配置方法的更詳細描述在第7.4.1.1節和第7.4.5.5節中討論。

以下是開發人員開發和佈署 RTPS discovery 應用時要考慮的限制：
1）使用 UDP 阜來指定的 Domain ID 只能從 0到231（包含）。每個 OpenDDS 程序只能包含 120 個 doamin 。

2）Topic 和型態辨識只能小於 256 的字元

3）OpenDDS 本機嘗試 RTPS Discovery 不執行因為 GUID 已分配（如果嘗試，將發出警告）。


關於更多 RTPS discover 問題，可以再章節8.5的 Real-time Publish-Subscribe Wire Protocol DDS Interoperability Wire Protocol Specification (DDSI-RTPS) v2.2 (OMG Document formal/2014-09-01)找到很好的參考。

### 1.2.3.4 Threading

OpenDDS 創建自己的  ORB（當他需要一個）以及從正再執行的 ORB 上分離。它還使用自己的線程來處理傳入和傳出傳輸 I/O。創建單獨的線程以在意外連接關閉時清除資源。可以從 DCPS 來監聽線程。
當透過 DDS 發佈範例時，OpenDDS 使用呼叫行程來執行一般嘗試發送範例到所有以連線的訂閱者。如果使用呼叫區塊，範例可以被排到分配發送服務線程。此行為取決於第3章中描述的QoS。
所有訂閱者接收的資料都由服務程序並由應用程式排程。從服務線程調用 DataReader 偵聽器。

### 1.2.3.5配置

OpenDDS包括一個基於文件的配置框架，用於配置全域，如調試級別，內存分配和發現，以及發布商和訂閱者的傳輸實現詳細信息。配置也可以直接在代碼中實現，但是，建議將配置外部化，以便於維護和減少運行時錯誤。第7章介紹了完整的配置選項集。

## 1.3安裝

有關如何構建OpenDDS的步驟可以在 $DDS\ROOT\INSTALL 中找到。為了避免編譯您不會使用的OpenDDS代碼，有一些功能比可以從構建中排除。下面討論這些。需要小尺寸配置或與安全性平台兼容性的用戶應考慮使用本指南第13章中所述的OpenDDS安全配置文件。

### 1.3.1啟用或禁用功能的構建

大多數功能都由 configure script 支持。 configure script 創建具有正確內容的配置文件，然後運行 MPC。如果使用configure script，請使用“--help”命令行選項運行它，並查找要啟用/禁用的功能。

如果不使用configure腳本，請繼續閱讀以下有關直接運行MPC的說明。

對於下述功能，MPC用於啟用（默認）功能或禁用功能。對於名為feature的功能，使用以下步驟從構建中禁用功能：

1）使用MPC的命令行“features”參數：

mwc.pl -type <type> -features feature = 0 DDS.mwc 或者，將 line feature = 0 添加到文件 $ACE/ROOT/bin/MakeProjectCreator/config/default.features 中，並使用MPC重新生成項目文件。

2）如果您使用gnuace MPC項目類型（如果您將使用GNU make作為您的構建系統，則是這種情況），請將文件 $ACE/ROOT/include/makeinclude/platform/macros.GNU 中添加“feature = 0”。

要顯式啟用該功能，請使用上面的feature = 1。

**注意:**您還可以使用 $DDS/ROOT/configure 腳本啟用或禁用功能。要禁用該功能，請將 --no-feature 傳遞給腳本，以啟用 pass --feature。在這種情況下，在名稱中使用“ - ”而不是“_”。 例如，要禁用下面討論的功能content_subscription，請將-no-content-subscription傳遞給configure腳本。

### 1.3.2禁用內置主題的構建
特徵名稱：built_in_Topics
你可以藉由禁用內置的 Topic 減少 DDS 核心函數庫減少高達30% 。看第 6 章確定如果你不需要 BIT 支援。

### 1.3.3禁用合規性配置文件特性的建立

DDS規範定義了遵從性概況以提供用於指示DDS實現可能支持或可能不支持的某些特徵集合的通用術語。
DDS規範定義了 compliance profiles 提供一般術語指示哪些特定的設定 DDS 可能不會支援。
這些配置文件以及用於禁用支持的 MPC 功能的名稱為該配置文件或該配置文件的組件

許多配置文件選項涉及QoS設置。如果嘗試使用不啟用的文件 QOS 的值將會發生運行錯誤。如果配置文件涉及不啟用的 class 則會發生編譯錯誤。

#### 1.3.3.1內容訂閱配置文件

特徵名稱：content_subscription

此配置文件添加第5章中ContentFilteredTopic, QueryCondition, 和 MultiTopic
discussed。此外，可以執行使用該功能參照下表。

**Table 1-2: Content-Subscription Class Features**

| Class | Feature |
| :--- | :--- |
| ContentFilteredTopic | content_filtered_Topic |
| QueryCondition | query_condition |
| MultiTopic | multi_Topic |

#### 1.3.3.2持續性簡介

特徵名稱：persistence_profile

此配置文件添加 QoS 方針 DURABILITY_SERVICE 和 QoS類型設置 “TRANSIENT” 和 “PERSISTENT” 。

#### 1.3.3.3所有權簡介

特徵名稱：ownership_profile

此配置文件添加：

•所有權類型的設置 “EXCLUSIVE”

•支援 OWNERSHIP_STRENGTH

•為HISTORY QoS 設置 depth>1。

**注意:**目前，即使已禁用ownership_profile，仍支持HISTORY depth>1的 OpenDDS 代碼。

#### 1.3.3.4對像模型概要

特徵名稱：object_model_profile

此配置文件包括對“GROUP”的 PRESENTATION access_scope設置支援。

**注意:**目前，當object_model_profile被禁用時， PRESENTATION access_scope 的 "Topic" 也被排除。
