# **CHAPTER 1**

### **Introduction**


OpenDDS是建立於 OMG Data Distribution Service (DDS) 的 Real-Time Systems V1.4(OMG 文件格式/2015-04-10) 和 Real-time Publish-Subscribe Wire Protocol DDS Interoperability Wire Protocol Specification(DDSI-RTPS) v2.2 (OMG Document formal/2014-09-01)。OpenDDS 是由 Object Computing, Inc. (OCI) 贊助。 Opendds 網址為[http://www.opendds.org/](http://opendds.org/)。開發人員指南基於 3.10 釋出版本的 Opendds。

DDS 的定義為在分散式的應用程式有效的傳輸分散式的應用程式資料。這個服務不特定為 CORBA 。提供 Platform Independent Model (PIM) 以及將 Platform Specific Model (PSM) 映射成 CORBA IDL。

有關DDS的其他詳細信息，開發人員應參考DDS規範（OMG文件格式/ 2015-04-10），因為它包含對所有服務的功能的深入涵蓋。

OpenDDS 是 C++ 所寫的開原軟體，OCI 提供 OMG's DDS 規範開發和商業支持。能夠從[http://www.opendds.org/downloads.html](http://www.opendds.org/downloads.html)下載和 OCI TAO version 2.0a and 2.2a 級別的補丁以及最新的文件。

注意:OpenDDS目前實現OMG DDS 1.4版規範。 有關詳細訊息，請參閱[http://www.opendds.org/](http://www.opendds.org/)中的規格訊息。

# 1.1. DCPS概述

在本節中，我們將介紹DCPS層的主要概念和實體，並討論它們如何相互影響和一起工作。

## 1.1.1基本概念

圖1-1顯示了DDS DCPS層的概述。 子節點的概念定義再下面那張圖中所示。

### ![](/1.1.jpg)

### 1.1.1.1 Domain

Domain 是 DCPS 中的基本分區單元。每個實體屬於一個domain，並且只能與該 domain 中的其他實體相互影響。

程式碼可以自由的與多個 domain 作用，但必需藉由分離不同 domain 的實體來做到。

### 1.1.1.2 DomainParticipant

domain participant是應用程序與特定的 domain 互相作用的入口點。domain participant 是個工廠給許多物件參與讀取寫入的地方。

### 1.1.1.3 Topic

在應用程序中 Topic 是publishing 和 subscribing 溝通的基本手段。每個 topiic 在 domain 中不能重複，以及發布它的特定數據類型。每個主題數據類型可以指定組零個或多個字段以及他的key(make up its key)。在發送資料時,發送程序總是會指定一個 topic 。Subscribers 通過 topic 請求數據。在DCPS術語中，你在 topic 推送個人不同的範例資料。每個實例都與唯一的 key 有關聯。推送過程中多個相同實例範本資料使用相同的key。

### 1.1.1.4 DataWriter

推送應用程序碼使用 DataWriter 將值傳遞到DDS。每個 DataWriter 都綁定到一個特定的 Topic。應用程式使用特別的 DataWriter 介面來發送樣本到 topic。DataWriter 負責封裝資料並傳送。

### 1.1.1.5 Publisher

Publisher 負責接收發送資料並傳給在同一 domain 底下有關的 Subscribe。精確的資訊實作是由服務來決定。

### 1.1.1.6 Subscribe

Subscribe 接收從 Publisher 的資料並給相關的 DataReader。

### 1.1.1.7 DataReader

DataReader 從 Subscribe 接收資料並轉換成對應 topic 的資料型態。每個 DataReader 綁定一個特定的 Topic。應用程式使用 DataReader 的特別的介面來接收樣本。

## 1.1.2 Built-In Topics

DDS中定義了數個 Topic。訂閱內建的 Topic 可以讓開發者存取 domain 的狀態包含，註冊的 Topic ,那些 DataReader DataWriter 是連接的還有不同接口的 QOS 設定。在訂閱時,應用程式會接收在 domain 改變的實體範例。

下表顯示了在DDS規範中定義的內置主題：

| 主題名稱 | 描述 |
| :--- | :--- |
| DCPSParticipant | 每個實例的 domain participant |
| DCPSTopic Each | 每個實例表的一般（非內置）主題。 |
| DCPSPublication | 每個實例的 DataWriter|
| DCPSSubscription | 每個實例的 DataReader  |

## 1.1.3 服務質量方針(QOS)

DDS規範定義了許多服務質量（QoS）方針，應用程式使用指定的 QOS 需要的服務。 參與者指定他們從服務中需要什麼行為，服務決定如何實現這些行為。 這些這些
這些 qos 可以給各種 DCPS 實例((topic, data writer, data reader, publisher, subscriber, domain participant) 然而不是所有的 qos 是對所有的實體是有效的。

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
 
•DDS :: SampleInfo包含一個以“opendds_reserved”開頭的附加字段，

•特定於類型的DataReaders（包括內置 topic 的DataReaders）具有其他操作read_instance_w_condition（）和take_instance_w_condition（）。額外的擴展行為由OpenDDS模塊/命名空間/包中的各種類和接口提供。 這些功能包括像是 Recorder和Replayer（見第12章）以及以下功能：

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

ETF使應用程序開發人員能夠實現自己的定制傳輸。實現自定義傳輸需要在多個特定傳輸框架之下。 udp 傳輸提供了開發人員在創建自己的實體時可以使用的良好基礎。 有關詳細信息，請參閱$ DDS_ROOT/dds/DCPS/transport/udp/ 目錄。

### 1.2.3.3 DDS發現

DDS應用程序必須通過某個中央代理或通過某種分佈式方案彼此發現。 OpenDDS的一個重要特性是，DDS應用程序可以配置為使用DCPSInfoRepo或RTPS發現執行發現，但利用不同的傳輸類型在數據寫入器和數據讀取器之間進行數據傳輸。 OMG DDS規範（正式/ 2015-04-10）將發現的細節留給實現。在DDS實現之間的互操作性的情況下，OMG DDSI-RTPS（正式/ 2014-09-01）規範提供了對等發現的對等方式的要求。

OpenDDS提供兩個發現選項。

1）信息庫：一種集中的存儲庫風格，作為一個單獨的過程運行，允許發布者和訂閱者集中發現另一個

2）RTPS發現：使用RTPS協議來通告可用性和位置信息的對等風格的發現。

與其他DDS實現的互操作性必須利用對等方法，但在僅支持OpenDDS的部署中很有用。

### 使用DCPSInfoRepo進行集中式發現

OpenDDS實現一個稱為DCPS信息庫的獨立服務

（DCPSInfoRepo）實現集中式發現方法。 它被實現為CORBA服務器。 當客戶端請求主題的預訂時，DCPS信息庫定位主題並且通知任何現有的發布者新訂戶的位置。

每當在非RTPS配置中使用OpenDDS時，DCPSInfoRepo進程需要運行。 RTPS配置不使用DCPSInfoRepo。 DCPSInfoRepo不參與數據傳播，其作用範圍受到OpenDDS應用程序發現彼此的限制。

![](/1.3.jpg)

應用程序開發人員可以自由運行多個信息庫，每個管理其自己的非重疊的DCPS域集合。

還可以操作具有多於單個存儲庫的域，從而形成分佈式虛擬存儲庫。這稱為存儲庫聯合。為了使各個存儲庫參與聯合，每個庫必須在啟動時指定其自己的聯合標識符值（一個32位數字值）。有關存儲庫聯合的更多信息，請參見9.2。

### 使用RTPS的對等發現

需要對等發現模式的DDS應用程序可以由OpenDDS功能提供。這種發現方式僅通過使用當前版本的RTPS協議來實現。這種簡單的發現形式是通過在應用程序進程中運行的DDS應用程序數據讀取器和數據寫入程序的簡單配置來實現的，如錯誤：未找到參考源。由於每個參與的進程為其數據讀取器和寫入器激活OpenDDS中的DDSI-RTPS發現機制，因此使用默認或配置的網絡端口來創建網絡端點，使得DDS參與者可以開始廣告其數據讀取器和數據寫入器的可用性。在一段時間之後，基於標準尋找彼此的那些將彼此找到並且基於如在可擴展傳輸框架（ETF）中討論的配置的可插入傳輸建立連接。這種靈活配置方法的更詳細描述在第7.4.1.1節和第7.4.5.5節中討論。

以下是開發人員在開發和部署使用RTPS發現的應用程序時需要考慮的其他實現限制：1）由於UDP端口分配給域ID的方式，域ID應在0到231（含）之間。在每個OpenDDS流程中，每個域中最多支持120個域參與者。

2）主題名稱和類型標識符限制為256個字符。

3）由於分配GUID的方式（如果嘗試，將發出警告），OpenDDS的本地多播傳輸不與RTPS發現一起工作。

關於RTPS發現如何發生的更多細節，可以在實時發布 - 訂閱有線協議DDS互操作性有線協議規範（DDSI-RTPS）v2.2（OMG文件格式/ 2014 -09-01）。

### 1.2.3.4 Threading

OpenDDS創建自己的ORB（當需要一個ORB時）以及運行該ORB的單獨的線程。 它還使用自己的線程來處理傳入和傳出傳輸I / O。 創建單獨的線程以在意外連接關閉時清除資源。 您的應用程序可能會通過DCPS的偵聽器機制從這些線程中調用。

當通過DDS發布樣本時，OpenDDS通常嘗試使用調用線程將樣本發送到任何已連接的訂閱者。 如果發送調用塊，則樣本可以排隊等待在單獨的服務線程上發送。 此行為取決於第3章中描述的QoS策略。

訂戶中的所有傳入數據由服務線程讀取並排隊等待應用程序讀取。 從服務線程調用DataReader偵聽器。

### 1.2.3.5配置

OpenDDS包括一個基於文件的配置框架，用於配置全局項，如調試級別，內存分配和發現，以及發布商和訂閱者的傳輸實現詳細信息。配置也可以直接在代碼中實現，但是，建議將配置外部化，以便於維護和減少運行時錯誤。第7章介紹了完整的配置選項集。

## 1.3安裝

有關如何構建OpenDDS的步驟可以在$ DDS\_ROOT / INSTALL中找到。為了避免編譯您不會使用的OpenDDS代碼，有一些功能比可以從構建中排除。下面討論這些特徵。需要小尺寸配置或與安全性平台兼容性的用戶應考慮使用本指南第13章中所述的OpenDDS安全配置文件。

### 1.3.1啟用或禁用功能的構建

大多數功能都由configure腳本支持。 configure腳本創建具有正確內容的配置文件，然後運行MPC。如果使用configure腳本，請使用“--help”命令行選項運行它，並查找要啟用/禁用的功能。

如果不使用configure腳本，請繼續閱讀以下有關直接運行MPC的說明。

對於下述功能，MPC用於啟用（默認）功能或禁用功能。對於名為feature的功能部件，使用以下步驟從構建中禁用功能部件：

1）使用MPC的命令行“features”參數：

mwc.pl -type &lt;type&gt; -features feature = 0 DDS.mwc或者，將line feature = 0添加到文件$ ACE\_ROOT / bin / MakeProjectCreator / config / default.features中，並使用MPC重新生成項目文件。

2）如果您使用gnuace MPC項目類型（如果您將使用GNU make作為您的構建系統，則是這種情況），請將文件$ ACE\_ROOT / include / makeinclude / platform\_macros.GNU中添加“feature = 0”。

要顯式啟用該功能，請使用上面的feature = 1。

**注意:**您還可以使用$ DDS\_ROOT / configure腳本啟用或禁用功能。 要禁用該功能，請將--no-feature傳遞給腳本，以啟用pass -feature。 在這種情況下，在要素名稱中使用“ - ”而不是“\_”。 例如，要禁用下面討論的功能content\_subscription，請將-no-content-subscription傳遞給configure腳本。

### 1.3.2禁用內置主題的構建

支持功能名稱：built\_in\_topics通過禁用內置主題支持，可以將核心DDS庫的佔用空間減少高達30％。請參見第6章以確定是否構建沒有BIT支持。

### 1.3.3禁用合規性配置文件特性的建立

DDS規範定義了遵從性概況以提供用於指示DDS實現可能支持或可能不支持的某些特徵集合的通用術語。以下給出這些配置文件以及用於禁用對該配置文件或該配置文件的組件的支持的MPC功能的名稱。

許多配置文件選項涉及QoS設置。如果嘗試使用與禁用的配置文件不兼容的QoS值，則會發生運行時錯誤。如果配置文件涉及類，則如果嘗試使用該類並禁用配置文件，則會發生編譯時錯誤。

#### 1.3.3.1內容訂閱配置文件

功能名稱：content\_subscription

此配置文件添加第5章中討論的類ContentFilteredTopic，QueryCondition和MultiTopic。此外，可以通過使用下表中給出的功能來排除個別類。

**Table 1-2: Content-Subscription Class Features**

| Class | Feature |
| :--- | :--- |
| ContentFilteredTopic | content\_filtered\_topic |
| QueryCondition | query\_condition |
| MultiTopic | multi\_topic |

#### 1.3.3.2持續性簡介

功能名稱：persistence\_profile

此配置文件添加QoS策略DURABILITY\_SERVICE和DURABILITY QoS策略類型的設置“TRANSIENT”和“PERSISTENT”。

#### 1.3.3.3所有權簡介

功能名稱：ownership\_profile

此配置文件添加：

•所有權類型的設置“EXCLUSIVE”

•支持OWNERSHIP\_STRENGTH策略

•為HISTORY QoS策略設置深度&gt; 1。

**注意:**目前，即使已禁用ownership\_profile，仍支持HISTORY 深度&gt; 1的OpenDDS代碼。

#### 1.3.3.4對像模型概要

功能名稱：object\_model\_profile

此配置文件包括對“GROUP”的presentENTATION access\_scope設置的支持。

**注意:**目前，當object\_model\_profile被禁用時，“TOPIC”的PRESENTATION access\_scope也被排除。





