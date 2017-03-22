# CHAPTER 4

# Conditions and Listeners

# 4.1 Introduction

DDS規範定義了用於通知應用程序DCPS通信狀態改變的兩個單獨的機制。大多數狀態類型定義包含與狀態改變相關的信息的結構，並且可以由應用使用條件或偵聽器來檢測。不同的狀態類型在4.2中描述。

每個實體類型（域參與者，主題，發布者，訂閱者，數據讀取器和數據寫入者）定義其自己對應的偵聽器接口。應用程序可以實現此接口，然後將其監聽器實現附加到實體。每個偵聽器接口包含可為該實體報告的每個狀態的操作。每當發生合格狀態更改時，將使用適當的操作異步回調監聽器。 4.3中討論了不同監聽器類型的細節。

條件與等待集合一起使用，以使應用程序同步等待事件。條件的基本使用模式包括創建條件對象，將它們附加到等待集，然後等待等待集，直到觸發其中一個條件。 wait的結果告訴應用程序哪些條件被觸發，允許應用程序採取適當的動作來獲得相應的狀態信息。

條件在4.4中有更詳細的描述。

# 4.2通信狀態類型

每種狀態類型都與特定實體類型相關聯。此部分由實體類型組織，具有在相關實體類型下的子部分中描述的相應狀態。

下面的大多數狀態是簡單的通信狀態。例外情況是DATA\_ON\_READERS和DATA\_AVAILABLE，它們是讀取狀態。普通通信狀態定義了IDL數據結構。下面的相應部分描述了此結構及其字段。讀取狀態是對應用程序的簡單通知，然後根據需要讀取或獲取樣本。

狀態數據結構中的增量值報告自上次訪問狀態以來的更改。當為該狀態調用偵聽器或從其實體讀取狀態時，會認為狀態被訪問。

狀態數據結構中具有InstanceHandle\_t類型的字段通過在Built-InTopics中用於該實體的實例句柄標識實體（主題，數據讀取器，數據寫入器等）。

## 4.2.1主題狀態類型

### 4.2.1.1主題狀態不一致

`INCONSISTENT_TOPIC`狀態表示嘗試註冊的主題已存在且具有不同的特性。 通常，現有主題可以具有與其相關聯的不同類型。 與不一致主題狀態關聯的IDL如下所示：

```cpp
struct InconsistentTopicStatus {
 long total_count;
 long total_count_change;
};
```

`total_count`值是已報告為不一致的主題的累積計數。 `total_count_change`值是自上次訪問此狀態以來不一致主題的增量計數。

##  4.2.2訂戶狀態類型

###  4.2.2.1讀取器狀態數據

`DATA_ON_READERS`狀態指示新數據在與訂戶相關聯的一些數據讀取器上可用。 此狀態被視為讀取狀態，並且不定義IDL結構。 接收此狀態的應用程序可以在訂戶上調用`get_datareaders()`以獲取具有可用數據的數據讀取器集合。

## 4.2.3數據讀取器狀態類型

### 4.2.3.1樣本被拒絕狀態

`SAMPLE_REJECTED`狀態指示由數據讀取器接收的樣本已被拒絕。 與拒絕樣本狀態相關聯的IDL如下所示：

```cpp
enum SampleRejectedStatusKind {
 NOT_REJECTED,
 REJECTED_BY_INSTANCES_LIMIT,
 REJECTED_BY_SAMPLES_LIMIT,
 REJECTED_BY_SAMPLES_PER_INSTANCE_LIMIT
};
struct SampleRejectedStatus {
 long total_count;
 long total_count_change;
 SampleRejectedStatusKind last_reason;
 InstanceHandle_t last_instance_handle;
};
```

`total_count`值是已報告為已拒絕的樣本的累積計數。 `total_count_change`值是自上次訪問此狀態以來的被拒絕樣本的增量計數。 `last_reason`值是最近拒絕的樣本被拒絕的原因。 `last_instance_handle`值指示最後被拒絕的樣本的實例。

### 4.2.3.2活力改變狀態

`LIVELINESS_CHANGED`狀態表示已為此數據讀取器的發布實例的一個或多個數據寫入器進行了活動更改。 與活動狀態更改狀態相關聯的IDL如下所示：

```cpp
struct LivelinessChangedStatus {
 long alive_count;
 long not_alive_count;
 long alive_count_change;
 long not_alive_count_change;
 InstanceHandle_t last_publication_handle;
};
```

`alive_count`值是此數據讀取器正在讀取的主題上當前處於活動狀態的數據寫入程序的總數。 `not_alive_count`值是寫入數據讀取器主題的數據寫入程序的總數，不再斷言其活動性。 `alive_count_change`值是自上次訪問狀態以來活動計數的變化。

`not_alive_count_change`值是自上次訪問狀態以來的不活動計數的更改。 `last_publication_handle`是活動已經改變的最後一個數據寫入者的句柄。

### 4.2.3.3請求的截止期限錯過狀態

`REQUESTED_DEADLINE_MISSED`狀態表示通過截止期限QoS策略請求的截止期限不適用於特定實例。 與請求的截止期限錯過狀態相關聯的IDL如下所示：

```cpp
struct RequestedDeadlineMissedStatus {
 long total_count;
 long total_count_change;
 InstanceHandle_t last_instance_handle;
};
```

`total_count`值是已報告的錯過的請求截止日期的累積計數。 `total_count_change`值是自上次訪問此狀態以來錯過的請求截止日期的增量計數。 `last_instance_handle`值指示最後錯過的截止時間的實例。

### 4.2.3.4請求的QoS狀態不兼容

`REQUESTED_INCOMPATIBLE_QOS`狀態指示所請求的一個或多個QoS策略值與所提供的QoS策略值不兼容。 與請求的不兼容QoS狀態相關聯的IDL如下所示：

```cpp
struct QosPolicyCount {
 QosPolicyId_t policy_id;
 long count;
};
typedef sequence<QosPolicyCount> QosPolicyCountSeq;
struct RequestedIncompatibleQosStatus {
 long total_count;
 long total_count_change;
 QosPolicyId_t last_policy_id;
 QosPolicyCountSeq policies;
};
```

`total_count`值是已報告具有不兼容QoS的數據寫入器的累積計數。 `total_count_change`值是自上次訪問此狀態以來不兼容數據寫入程序的增量計數。 `last_policy_id`值標識在檢測到的最後一個不兼容性中不兼容的QoS策略之一。 策略值是指示為每個QoS策略檢測到的不兼容的總數的值序列。

### 4.2.3.5數據可用狀態

`DATA_AVAILABLE`狀態指示樣本在數據寫入器上可用。 此狀態被視為讀取狀態，並且不定義IDL結構。 接收此狀態的應用程序可以使用數據讀取器上的各種讀取和讀取操作來檢索數據。

### 4.2.3.6樣本丟失狀態

`SAMPLE_LOST`狀態指示樣本已丟失且從未被數據讀取器接收。 與丟失樣本狀態相關聯的IDL如下所示：

```cpp
struct SampleLostStatus {
 long total_count;
 long total_count_change;
};
```

`total_count`值是報告為丟失的樣本的累積計數。 `total_count_change`值是自上次訪問此狀態以來丟失樣本的增量計數。

### 4.2.3.7訂閱匹配狀態

`SUBSCRIPTION_MATCHED`狀態表示兼容數據寫入器已匹配或先前匹配的數據寫入器已停止匹配。 與訂閱匹配狀態相關聯的IDL如下所示：

```cpp
struct SubscriptionMatchedStatus {
 long total_count;
 long total_count_change;
 long current_count;
 long current_count_change;
 InstanceHandle_t last_publication_handle;
};
```

`total_count`值是與此數據讀取器兼容匹配的數據寫入程序的累積計數。 `total_count_change`值是自上次訪問此狀態以來的總計數的增量更改。 `current_count`值是與此數據讀取器匹配的數據寫入程序的當前數量。 `current_count_change`值是自上次訪問此狀態以來的當前計數的更改。 `last_publication_handle`值是最後一個數據寫入器匹配的句柄。

## 4.2.4數據寫入器狀態類型

### 4.2.4.1活力丟失狀態

`LIVELINESS_LOST`狀態表示數據寫入器通過其活動性QoS執行的活動未得到遵守。 這意味著任何連接的數據讀取器將認為此數據寫入器不再活動。與活動狀態丟失狀態相關聯的IDL如下所示：

```cpp
struct LivelinessLostStatus {
 long total_count;
 long total_count_change;
};
```

`total_count`值是活動數據寫入程序變為不活動的累積次數。 `total_count_change`值是自上次訪問此狀態以來的總計數的增量更改。

### 4.2.4.2提交的截止期限錯過狀態

`OFFERED_DEADLINE_MISSED`狀態表示由一個或多個實例錯過了數據寫入程序提供的最後期限。 與提交的截止期限錯過狀態相關聯的IDL如下所示：

```cpp
struct OfferedDeadlineMissedStatus {
 long total_count;
 long total_count_change;
 InstanceHandle_t last_instance_handle;
};
```

`total_count`值是針對實例錯過的截止時間的累計次數。 `total_count_change`值是自上次訪問此狀態以來的總計數的增量更改。 `last_instance_handle`值表示錯過最後期限的最後一個實例。

### 4.2.4.3提供的QoS狀態不兼容

`OFFERED_INCOMPATIBLE_QOS`狀態指示所提供的QoS與數據讀取器的所請求的QoS不兼容。 與提供的不兼容QoS狀態相關聯的IDL如下所示：

```cpp
struct QosPolicyCount {
 QosPolicyId_t policy_id;
 long count;
};
typedef sequence<QosPolicyCount> QosPolicyCountSeq;
struct OfferedIncompatibleQosStatus {
 long total_count;
 long total_count_change;
 QosPolicyId_t last_policy_id;
 QosPolicyCountSeq policies;
};
```

`total_count`值是已找到具有不兼容QoS的數據讀取器的累計次數。 `total_count_change`值是自上次訪問此狀態以來的總計數的增量更改。 `last_policy_id`值標識在檢測到的最後一個不兼容性中不兼容的QoS策略之一。 策略值是指示為每個QoS策略檢測到的不兼容的總數的值序列。

### 4.2.4.4發布匹配狀態

`PUBLICATION_MATCHED`狀態指示兼容數據讀取器已被匹配或先前匹配的數據讀取器已停止匹配。 與發布匹配狀態關聯的IDL如下所示：

```cpp
struct PublicationMatchedStatus {
 long total_count;
 long total_count_change;
 long current_count;
 long current_count_change;
 InstanceHandle_t last_subscription_handle;
};
```

`total_count`值是與此數據寫入器兼容匹配的數據讀取器的累積計數。 `total_count_change`值是自上次訪問此狀態以來的總計數的增量更改。 `current_count`值是與此數據寫入器匹配的數據讀取器的當前數量。 `current_count_change`值是自上次訪問此狀態以來的當前計數的更改。 `last_subscription_handle`值是最後一個數據讀取器匹配的句柄。

# 4.3 Listeners

每個實體根據它可以報告的狀態定義自己的監聽器接口。 任何實體的偵聽器接口也從其擁有的實體的偵聽器繼承，允許它處理所有實體的狀態。 例如，訂閱者偵聽器直接定義了處理數據讀取器狀態的操作，並且還從數據讀取器偵聽器繼承。

每個狀態操作採用`on_ <status_name>`（`<entity>`，`<status_struct>`）的一般形式，其中`<status_name>`是要報告的狀態的名稱，`<entity>`是對報告狀態的實體的引用， 而`<status_struct>`是具有狀態詳細信息的結構。 讀取狀態省略第二個參數。 例如，以下是“丟失樣本”狀態的操作：

```cpp
 void on_sample_lost(in DataReader the_reader, in SampleLostStatus status);
```

偵聽器可以傳遞到用於創建實體的工廠函數，也可以在創建實體後通過調用`set_listener()`來顯式設置。 這兩個函數還將狀態掩碼用作參數。 掩碼指示在該偵聽器中啟用了哪些狀態。 每個狀態的掩碼位值在`DdsDcpsInfrastructure.idl`中定義：

```cpp
module DDS {
 typedef unsigned long StatusKind;
 typedef unsigned long StatusMask; // bit-mask StatusKind
 const StatusKind INCONSISTENT_TOPIC_STATUS = 0x0001 << 0;
 const StatusKind OFFERED_DEADLINE_MISSED_STATUS = 0x0001 << 1;
 const StatusKind REQUESTED_DEADLINE_MISSED_STATUS = 0x0001 << 2;
 const StatusKind OFFERED_INCOMPATIBLE_QOS_STATUS = 0x0001 << 5;
 const StatusKind REQUESTED_INCOMPATIBLE_QOS_STATUS= 0x0001 << 6;
 const StatusKind SAMPLE_LOST_STATUS = 0x0001 << 7;
 const StatusKind SAMPLE_REJECTED_STATUS = 0x0001 << 8;
 const StatusKind DATA_ON_READERS_STATUS = 0x0001 << 9;
 const StatusKind DATA_AVAILABLE_STATUS = 0x0001 << 10;
 const StatusKind LIVELINESS_LOST_STATUS = 0x0001 << 11;
 const StatusKind LIVELINESS_CHANGED_STATUS = 0x0001 << 12;
 const StatusKind PUBLICATION_MATCHED_STATUS = 0x0001 << 13;
 const StatusKind SUBSCRIPTION_MATCHED_STATUS = 0x0001 << 14;
};
```

只需做一個位“或”的所需狀態位為你的監聽器構造一個掩碼。

以下是將偵聽器附加到數據讀取器的示例（僅適用於數據可用狀態）：

```cpp
 DDS::DataReaderListener_var listener (new DataReaderListenerImpl);
 // Create the Datareader
 DDS::DataReader_var dr = sub->create_datareader(
  topic,
  DATAREADER_QOS_DEFAULT,
  listener,
  DDS::DATA_AVAILABLE_STATUS);
```

下面是一個示例，顯示如何使用`set_listener()`更改偵聽器：

```cpp
dr->set_listener(listener,DDS::DATA_AVAILABLE_STATUS | DDS::LIVELINESS_CHANGED_STATUS);
```

當普通通信狀態改變時，OpenDDS調用最具體的相關偵聽器操作。 這意味著，例如，數據讀取器的監聽器將優先於用戶的監聽器以用於與數據讀取器相關的狀態。

以下部分定義不同的偵聽器接口。 有關各個狀態的更多詳細信息，請參閱4.2。

## 4.3.1主題偵聽器

```cpp
interface TopicListener : Listener {
 void on_inconsistent_topic(in Topic the_topic,
 in InconsistentTopicStatus status);
};
```

## 4.3.2數據寫入器監聽器

```cpp
interface DataWriterListener : Listener {
 void on_offered_deadline_missed(in DataWriter writer, in OfferedDeadlineMissedStatus status);
 void on_offered_incompatible_qos(in DataWriter writer,in OfferedIncompatibleQosStatus status);
 void on_liveliness_lost(in DataWriter writer, in LivelinessLostStatus status);
 void on_publication_matched(in DataWriter writer, in PublicationMatchedStatus status);
};
```

## 4.3.3發布商偵聽器

```cpp
interface PublisherListener : DataWriterListener {
};
```

## 4.3.4數據讀取器偵聽器

```cpp
interface DataReaderListener : Listener {
 void on_requested_deadline_missed(in DataReader the_reader, in RequestedDeadlineMissedStatus status);
 void on_requested_incompatible_qos(in DataReader the_reader, in RequestedIncompatibleQosStatus status);
 void on_sample_rejected(in DataReader the_reader,in SampleRejectedStatus status);
 void on_liveliness_changed(in DataReader the_reader,in LivelinessChangedStatus status);
 void on_data_available(in DataReader the_reader);
 void on_subscription_matched(in DataReader the_reader,in SubscriptionMatchedStatus status);
 void on_sample_lost(in DataReader the_reader, in SampleLostStatus status);
};
```

## 4.3.5訂閱者監聽器

```cpp
interface SubscriberListener : DataReaderListener {
 void on_data_on_readers(in Subscriber the_subscriber);
};
```

## 4.3.6域參與者偵聽器

```cpp
interface DomainParticipantListener : TopicListener, PublisherListener, SubscriberListener {
};
```

# 4.4條件

DDS規範定義了四種類型的條件：

•狀態條件

•讀條件

•查詢條件

•保護條件

## 4.4.1狀態條件

每個實體都有一個與其相關的狀態條件對象，以及允許應用程序訪問狀態條件的`get_statuscondition()`操作。 每個條件都有一組啟用狀態，可以觸發該條件。 將一個或多個條件附加到等待集允許應用程序開發人員等待條件的狀態集。 一旦啟用狀態被觸發，等待調用從等待集返回，開發者可以查詢實體上的相關狀態條件。 查詢狀態條件將重置狀態。

### 4.4.1.1狀態條件示例

此示例在數據寫入程序上啟用提供的不兼容QoS狀態，等待數據寫入，然後在觸發時進行查詢。 第一步是從數據寫入器獲取狀態條件，啟用所需的狀態，並將其附加到等待集：

```cpp
 DDS::StatusCondition_var cond = data_writer->get_statuscondition();
 cond->set_enabled_statuses(DDS::OFFERED_INCOMPATIBLE_QOS_STATUS);
 DDS::WaitSet_var ws = new DDS::WaitSet;
 ws->attach_condition(cond);
```

現在我們可以等待10秒的條件：

```cpp
 DDS::ConditionSeq active;
 DDS::Duration ten_seconds = {10, 0};
 int result = ws->wait(active, ten_seconds);
```

此操作的結果是活動序列中的超時或一組觸發條件：

```cpp
 if (result == DDS::RETCODE_TIMEOUT) {
     cout << "Wait timed out" << std::endl;
   } else if (result == DDS::RETCODE_OK) {
     DDS::OfferedIncompatibleQosStatus incompatibleStatus;
     data_writer->get_offered_incompatible_qos(incompatibleStatus);
     // Access status fields as desired...
 }
```

開發人員可以選擇將多個條件附加到單個等待集以及根據條件啟用多個狀態。

## 4.4.2附加條件類型

DDS規範還定義了三種其他類型的條件：讀取條件，查詢條件和保護條件。 這些條件不直接涉及狀態的處理，而是允許將其他活動集成到條件和等待設置機制中。 這些都是其他條件簡要說明。 有關更多信息，請參閱DDS規範或`$ DDS_ROOT / tests /`中的OpenDDS測試。

### 4.4.2.1讀條件

使用數據讀取器和傳遞到讀取和獲取操作的相同掩碼創建讀取條件。 當等待此條件時，每當樣本匹配指定的掩碼時觸發。 然後可以使用將讀取條件作為參數的`read_w_condition()`和`take_w_condition()`操作來檢索這些樣本。

### 4.4.2.2查詢條件

查詢條件是使用有限形式的類似SQL的查詢創建的讀取條件的特殊形式。 這允許應用程序過濾觸發條件的數據樣本，然後使用正常的讀取條件機制讀取。 有關查詢條件的更多信息，請參見第5.3節。

### 4.4.2.3保護條件

保護條件是一個簡單的接口，允許應用程序創建自己的條件對象，並在應用程序事件（OpenDDS外部）發生時觸發它。



