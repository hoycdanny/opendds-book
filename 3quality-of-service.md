* [ ] # CHAPTER 3

### Quality of Service

# 3.1 介紹

前面的示例為各種實體使用默認QoS策略。 本章將討論在OpenDDS中實現的QoS策略及其使用的細節。 有關本章中討論的策略的更多信息，請參閱DDS規範。

# 3.2 QoS策略

每個策略定義一個結構以指定其數據。 每個實體支持策略的子集並且定義由支持的策略結構組成的QoS結構。

給定實體的允許策略集由嵌套在其QoS結構中的策略結構限制。 例如，發布者的QoS結構在規範的IDL中定義如下：

```cpp
module DDS {
 struct PublisherQos {
  PresentationQosPolicy presentation;
  PartitionQosPolicy partition;
  GroupDataQosPolicy group_data;
  EntityFactoryQosPolicy entity_factory;
 };
};
```

設置策略與獲取已設置的默認值的結構一樣簡單，根據需要修改單個策略結構，然後將QoS結構應用於實體（通常在創建時）。 我們展示瞭如何獲取第3.2.1節中各種實體類型的默認QoS策略的示例。

應用程序可以通過對實體調用set\_qos（）操作來更改任何實體的QoS。 如果QoS是可改變的，則如果現有關聯不再兼容，則去除它們，並且如果它們變得兼容，則添加新關聯。 DCPSInfoRepo根據QoS規範重新評估QoS兼容性和關聯。 如果兼容性檢查失敗，則調用set\_qos（）將返回錯誤。 協會重新評估可能導致刪除現有的關聯或添加新的關聯。

如果用戶嘗試更改不可變（不可更改）的QoS策略，則 `set_qos()` returns  `DDS::RETCODE_IMMUTABLE_POLICY`.

QoS策略的子集是可改變的。 一些可改變的QoS策略**（例如USER\_DATA，TOPIC\_DATA，GROUP\_DATA，LIFESPAN，OWNERSHIP\_STRENGTH，TIME\_BASED\_FILTER，ENTITY\_FACTORY，WRITER\_DATA\_LIFECYCLE和READER\_DATA\_LIFECYCLE）**不需要兼容性和關聯重新評估。 **DEADLINE**和**LATENCY\_BUDGET** QoS策略要求兼容性重新評估，但不要求關聯。 **PARTITION QoS**策略不需要兼容性重新評估，但需要關聯重新評估。

DDS規範將**TRANSPORT\_PRIORITY**列為可更改，但是OpenDDS實現不支持動態修改此策略。

## 3.2.1默認QoS策略值

應用程序通過為實體實例化適當類型的QoS結構並通過引用適當的工廠實體上的適當get\_default\_entity\_qos（）操作來獲得實體的默認QoS策略。 （例如，您將使用域參與者獲取發布者或訂閱者的默認QoS。）以下示例說明如何獲取發布者，訂閱者，主題，域參與者，數據寫入者和數據讀取者的默認策略。

```cpp
// Get default Publisher QoS from a DomainParticipant:
DDS::PublisherQos pub_qos;
DDS::ReturnCode_t ret;
ret = domain_participant->get_default_publisher_qos(pub_qos);
if (DDS::RETCODE_OK != ret) {
 std::cerr << "Could not get default publisher QoS" << std::endl;
}
// Get default Subscriber QoS from a DomainParticipant:
DDS::SubscriberQos sub_qos;
ret = domain_participant->get_default_subscriber_qos(sub_qos);
if (DDS::RETCODE_OK != ret) {
 std::cerr << "Could not get default subscriber QoS" << std::endl;
// Get default Topic QoS from a DomainParticipant:
DDS::TopicQos topic_qos;
ret = domain_participant->get_default_topic_qos(topic_qos);
if (DDS::RETCODE_OK != ret) {
 std::cerr << "Could not get default topic QoS" << std::endl;
}
// Get default DomainParticipant QoS from a DomainParticipantFactory:
DDS::DomainParticipantQos dp_qos;
ret = domain_participant_factory->get_default_participant_qos(dp_qos);
if (DDS::RETCODE_OK != ret) {
 std::cerr << "Could not get default participant QoS" << std::endl;
}
// Get default DataWriter QoS from a Publisher:
DDS::DataWriterQos dw_qos;
ret = pub->get_default_datawriter_qos(dw_qos);
if (DDS::RETCODE_OK != ret) {
 std::cerr << "Could not get default data writer QoS" << std::endl;
}
// Get default DataReader QoS from a Subscriber:
DDS::DataReaderQos dr_qos;
ret = sub->get_default_datareader_qos(dr_qos);
if (DDS::RETCODE_OK != ret) {
 std::cerr << "Could not get default data reader QoS" << std::endl;
}
```

下表總結了可應用策略的OpenDDS中每個實體類型的默認QoS策略。

#### 表3-1為預設的DomainParticipant QoS策略

| Policy | Member | Default Value |
| :---: | :---: | :---: |
| USER\_DATA | value | \(empty sequence\) |
| ENTITY\_FACTORY | autoenable\_created\_entities | true |

#### 表3-2為預設主題QoS策略

| Policy | Member | Default Value |
| :---: | :---: | :---: |
| TOPIC\_DATA | value | \(empty sequence\) |
| DURABILITY | kind                                                          service\_cleanup\_delay.sec                   service\_cleanup\_delay.nanosec | VOLATILE\_DURABILITY\_QOS                DURATION\_ZERO\_SEC                          DURATION\_ZERO\_NSEC |
| DURABILITY\_SERVICE | service\_cleanup\_delay.sec                    service\_cleanup\_delay.nanosec          history\_kind                                            history\_depth                                        max\_samples                                       max\_instances                                     max\_samples\_per\_instance | DURATION\_ZERO\_SEC                           DURATION\_ZERO\_NSEC                       KEEP\_LAST\_HISTORY\_QOS                 1                                                              LENGTH\_UNLIMITED                            LENGTH\_UNLIMITED                            LENGTH\_UNLIMITED |
| DEADLINE | period.sec                                               period.nanosec | DURATION\_INFINITY\_SEC                     DURATION\_INFINITY\_NSEC |
| LATENCY\_BUDGET | duration.sec                                            duration.nanosec | DURATION\_ZERO\_SEC                            DURATION\_ZERO\_NSEC |
| LIVELINESS | kind                                                           lease\_duration.sec                                 lease\_duration.nanosec | AUTOMATIC\_LIVELINESS\_QOS            DURATION\_INFINITY\_SEC                    DURATION\_INFINITY\_NSEC |

##### Quality of Service\(跟上面同一個表格\)

| Policy | Member | Default Value |
| :---: | :---: | :---: |
| RELIABILITY | kind                                                         max\_blocking\_time.sec                       max\_blocking\_time.nanosec | BEST\_EFFORT\_RELIABILITY\_QOS        DURATION\_INFINITY\_SEC                    DURATION\_INFINITY\_NSEC |
| DESTINATION\_ORDER | kind | BY\_RECEPTION\_TIMESTAMP\_             DESTINATIONORDER\_QOS |
| HISTORY | kind                                                          depth | KEEP\_LAST\_HISTORY\_QOS                  1 |
| RESOURCE\_LIMITS | max\_samples                                        max\_instances                                     max\_samples\_per\_instance | LENGTH\_UNLIMITED                              LENGTH\_UNLIMITED                            LENGTH\_UNLIMITED |
| TRANSPORT\_PRIORITY | value | 0 |
| LIFESPAN | duration.sec                                            duration.nanosec | DURATION\_INFINITY\_SEC                         DURATION\_INFINITY\_NSEC |
| OWNERSHIP | kind | SHARED\_OWNERSHIP\_QOS |

#### 表3-3為預設發布服務器QoS策略

| Policy | Member | Default Value |
| :---: | :---: | :---: |
| PRESENTATION | access\_scope                                         coherent\_access                                   ordered\_access | INSTANCE\_PRESENTATION\_QOS         0                                                              0 |
| PARTITION | name | \(empty sequence\) |
| GROUP\_DATA | value | \(empty sequence\) |
| ENTITY\_FACTORY | autoenable\_created\_entities | true |

#### 表3-4為預設用戶QoS策略

| Policy | Member | Default Value |
| :---: | :---: | :---: |
| PRESENTATION | access\_scope                                         coherent\_access                                   ordered\_access | INSTANCE\_PRESENTATION\_QOS         0                                                               0 |
| PARTITION | name | \(empty sequence\) |
| GROUP\_DATA | value | \(empty sequence\) |
| ENTITY\_FACTORY | autoenable\_created\_entities | true |

#### 表3-5為預設DataWriter QoS策略

| Policy | Member | Default Value |
| :---: | :---: | :---: |
| DURABILITY | kind                                                          service\_cleanup\_delay.sec                   service\_cleanup\_delay.nanosec | VOLATILE\_DURABILITY\_QOS                 DURATION\_ZERO\_SEC                          DURATION\_ZERO\_NSEC |
| DURABILITY\_SERVICE | service\_cleanup\_delay.sec                    service\_cleanup\_delay.nanosec          history\_kind                                            history\_depth                                        max\_samples、max\_instances、max\_samples\_per\_instance | DURATION\_ZERO\_SEC                           DURATION\_ZERO\_NSEC                       KEEP\_LAST\_HISTORY\_QOS                 1                                                              LENGTH\_UNLIMITED                              LENGTH\_UNLIMITED                              LENGTH\_UNLIMITED |
| DEADLINE | period.sec                                               period.nanosec | DURATION\_INFINITY\_SEC                     DURATION\_INFINITY\_NSEC |
| LATENCY\_BUDGET | duration.sec                                            duration.nanosec | DURATION\_ZERO\_SEC                           DURATION\_ZERO\_NSEC |
| LIVELINESS | kind                                                           lease\_duration.sec                                 lease\_duration.nanosec | AUTOMATIC\_LIVELINESS\_QOS            DURATION\_INFINITY\_SEC                    DURATION\_INFINITY\_NSEC |
| RELIABILITY | kind                                                         max\_blocking\_time.sec                       max\_blocking\_time.nanosec | RELIABLE\_RELIABILITY\_QOS2              0                                                              100000000 \(100 ms\) |
| DESTINATION\_ORDER | kind | BY\_RECEPTION\_TIMESTAMP\_               DESTINATIONORDER\_QOS |
| HISTORY | kind                                                          depth | KEEP\_LAST\_HISTORY\_QOS                  1 |
| RESOURCE\_LIMITS | max\_samples                                        max\_instances                                     max\_samples\_per\_instance | LENGTH\_UNLIMITED                             LENGTH\_UNLIMITED                            LENGTH\_UNLIMITED |
| TRANSPORT\_PRIORITY | value | 0 |
| LIFESPAN | duration.sec                                            duration.nanosec | DURATION\_INFINITY\_SEC                     DURATION\_INFINITY\_NSEC |
| USER\_DATA | value | \(empty sequence\) |
| OWNERSHIP | kind | SHARED\_OWNERSHIP\_QOS |
| OWNERSHIP\_STRENGTH | value | 0 |
| WRITER\_DATA\_LIFECYCLE | autodispose\_unregistered\_instances | 1 |

#### 表3-6為預設DataReader QoS策略

| Policy | Member | Default Value |
| :---: | :---: | :---: |
| DURABILITY | kind                                                          service\_cleanup\_delay.sec                   service\_cleanup\_delay.nanosec | VOLATILE\_DURABILITY\_QOS                DURATION\_ZERO\_SEC                          DURATION\_ZERO\_NSEC |
| DEADLINE | period.sec                                               period.nanosec | DURATION\_INFINITY\_SEC                     DURATION\_INFINITY\_NSEC |
| LATENCY\_BUDGET | duration.sec                                            duration.nanosec | DURATION\_ZERO\_SEC                           DURATION\_ZERO\_NSEC |
| LIVELINESS | kind                                                           lease\_duration.sec                                 lease\_duration.nanosec | AUTOMATIC\_LIVELINESS\_QOS            DURATION\_INFINITY\_SEC                    DURATION\_INFINITY\_NSEC |
| RELIABILITY | kind                                                         max\_blocking\_time.sec                       max\_blocking\_time.nanosec | BEST\_EFFORT\_RELIABILITY\_QOS        DURATION\_INFINITY\_SEC                    DURATION\_INFINITY\_NSEC |
| DESTINATION\_ORDER  | kind | BY\_RECEPTION\_TIMESTAMP\_             DESTINATIONORDER\_QOS |
| HISTORY  | kind                                                          depth | KEEP\_LAST\_HISTORY\_QOS                  1 |
| RESOURCE\_LIMITS | max\_samples                                        max\_instances                                     max\_samples\_per\_instance |  LENGTH\_UNLIMITED                             LENGTH\_UNLIMITED                            LENGTH\_UNLIMITED  |
| USER\_DATA  | value  | \(empty sequence\) |
| OWNERSHIP  | kind | SHARED\_OWNERSHIP\_QOS |
| TIME\_BASED\_FILTER  | minimum\_separation.sec                     minimum\_separation.nanosec | DURATION\_ZERO\_SEC                           DURATION\_ZERO\_NSEC |
| READER\_DATA\_LIFECYCLE | autopurge\_nowriter\_samples\_delay.sec                                                               autopurge\_nowriter\_samples\_delay.nanosec                                                       autopurge\_disposed\_samples\_delay.sec                                                             autopurge\_disposed\_samples\_delay.nanosec | DURATION\_INFINITY\_SEC                     DURATION\_INFINITY\_NSEC                 DURATION\_INFINITY\_SEC                    DURATION\_INFINITY\_NSEC |

2.對於OpenDDS版本（最多2.0），數據寫入程序的默認可靠性類型是盡力而為。 對於版本2.0.1和更高版本，此更改為可靠（符合DDS規範）。

## 3.2.2生命力

LIVELINESS策略通過它們各自的QoS結構的活動成員適用於主題，數據讀取器和數據寫入器實體。 在主題上設置此策略意味著對於該主題的所有數據讀取器和數據寫入器都有效。 下面是與活動性QoS策略相關的IDL：

```cpp
enum LivelinessQosPolicyKind {
 AUTOMATIC_LIVELINESS_QOS,
 MANUAL_BY_PARTICIPANT_LIVELINESS_QOS,
 MANUAL_BY_TOPIC_LIVELINESS_QOS
};
struct LivelinessQosPolicy {
 LivelinessQosPolicyKind kind;
 Duration_t lease_duration;
};
```

LIVELINESS策略控制服務何時以及如何確定參與者是否活著，這意味著他們仍可訪問和活動。 種類成員設置指示活動是由服務自動聲明還是由指定實體手動聲明。 `AUTOMATIC_LIVELINESS_QOS`的設置意味著如果參與者尚未為`lease_duration`發送任何網絡流量，則服務將發送活動狀態指示。 `MANUAL_BY_PARTICIPANT_LIVELINESS_QOS`或`MANUAL_BY_TOPIC_LIVELINESS_QOS`設置意味著指定的實體（“按主題”設置或域參與者的“按參與者”設置的數據寫入者）必須寫入樣本或在指定的心跳間隔內手動斷言其活動。 所需的心跳間隔由`lease_duration`成員指定。 默認租用持續時間是預定義的無限值，禁用任何活動性測試。

要手動聲明活動而不發布示例，應用程序必須在指定的心跳間隔內對數據寫入程序（對於“按主題”設置）或域參與者（對於“按參與者”設置）調用`assert_liveliness()`操作。

數據作者指定（提供）他們自己的活性標準，並且數據讀取者指定（請求）他們作者的期望的活潑性。在租賃期內（通過寫樣本或通過斷言活動）未聽到的寫入者導致`LIVELINESS_CHANGED_STATUS`通信狀態和通知的改變（例如，通過調用數據讀取器偵聽器的`on_liveliness_changed()`回調操作或通過發信號通知任何相關的等待集合）。在數據寫入器和數據讀取器之間建立關聯期間考慮該策略。關聯的雙方的值必須是兼容的，以便建立關聯。兼容性是通過將數據讀取器的請求活力與數據寫入器提供的活力進行比較來確定的。在確定兼容性時考慮活動性（自動，按主題手動，參與者手動）和租賃期的價值。作家提供的活潑的生活必須大於或等於讀者所要求的活潑。活力類型值按如下順序排列：

```cpp
MANUAL_BY_TOPIC_LIVELINESS_QOS >
MANUAL_BY_PARTICIPANT_LIVELINESS_QOS >
AUTOMATIC_LIVELINESS_QOS
```

此外，作者的提供租賃期必須小於或等於讀者的請求租期。 這兩個條件必須滿足提供和請求的活力策略設置被認為兼容和建立關聯。

## 3.2.3可靠性

RELIABILITY策略通過其各自的QoS結構的可靠性成員應用於主題，數據讀取器和數據寫入器實體。 下面是與可靠性QoS策略相關的IDL：

```cpp
enum ReliabilityQosPolicyKind {
 BEST_EFFORT_RELIABILITY_QOS,
 RELIABLE_RELIABILITY_QOS
};
struct ReliabilityQosPolicy {
 ReliabilityQosPolicyKind kind;
 Duration_t max_blocking_time;
};
```

此策略控制數據讀取器和寫入器如何處理它們處理的數據樣本。 “best effort”值（`BEST_EFFORT_RELIABILITY_QOS`）對樣本的可靠性沒有作出承諾，並且在某些情況下可能會丟棄樣本。 “可靠”值（`RELIABLE_RELIABILITY_QOS`）表示服務最終應將所有值提供給合格的數據讀取器。

當歷史QoS策略設置為“保留所有”並且寫入程序由於資源限制（由於傳輸背壓（詳細信息請參閱1.1.5））而無法返回時，將使用此策略的`max_blocking_time`成員。當這種情況發生並且寫入器阻塞超過指定時間時，寫入失敗，並返回超時返回碼。此策略對數據閱讀器和主題的默認值為“盡力而為”，而數據寫入程序的默認值為“可靠”。

在創建數據寫入器和數據讀取器之間的關聯時考慮此策略。為了創建關聯，關聯的兩側的值必須兼容。數據寫入器的可靠性必須大於或等於數據讀取器的值。

## 3.2.4歷史

HISTORY策略確定樣本如何保存在特定實例的數據寫入器和數據讀取器中。 對於數據寫入程序，這些值將保持，直到發布者檢索它們並成功地將它們發送到所有連接的訂閱者。 對於數據讀取器，這些值保持到由應用程序“採取”。 該策略通過它們各自的QoS結構的歷史成員應用於主題，數據讀取器和數據寫入器實體。 以下是與歷史QoS策略相關的IDL：

```cpp
enum HistoryQosPolicyKind {
 KEEP_LAST_HISTORY_QOS,
 KEEP_ALL_HISTORY_QOS
};
struct HistoryQosPolicy {
 HistoryQosPolicyKind kind;
 long depth;
};
```

“保留所有”值（`KEEP_ALL_HISTORY_QOS`）指定應保留該實例的所有可能的樣本。 當指定“保留所有”並且未讀樣本的數量等於`max_samples_per_instance`的“資源限制”字段時，則拒絕任何傳入樣本。

“保留最後”值（`KEEP_LAST_HISTORY_QOS`）指定僅應保留最後的深度值。 當數據寫入器包含給定實例的深度樣本時，該實例的新樣本的寫入被排隊等待傳送，並且最舊的未發送樣本被丟棄。 當數據讀取器包含給定實例的深度樣本時，保存該實例的任何輸入樣本，並且丟棄最舊的樣本。

此策略默認為深度為1的“保留最後一個”。

## 3.2.5耐久性

DURABILITY策略控制數據寫入者是否應在已發送給已知訂閱者之後維護樣本。 此策略通過其各自的QoS結構的耐用性成員適用於主題，數據讀取器和數據寫入器實體。 下面是與持久性QoS策略相關的IDL：

```cpp
enum DurabilityQosPolicyKind {
 VOLATILE_DURABILITY_QOS, // Least Durability
 TRANSIENT_LOCAL_DURABILITY_QOS,
 TRANSIENT_DURABILITY_QOS,
 PERSISTENT_DURABILITY_QOS // Greatest Durability
};
struct DurabilityQosPolicy {
 DurabilityQosPolicyKind kind;
};
```

默認情況下，類型為`VOLATILE_DURABILITY_QOS`。

持久性類型為`VOLATILE_DURABILITY_QOS`意味著樣品在發送給所有已知訂閱者後將被丟棄。作為副作用，訂戶在連接之前無法恢復發送的樣本。

持久性類型為`TRANSIENT_LOCAL_DURABILITY_QOS`意味著與數據寫入器關聯/連接的數據讀取器將發送數據寫入器歷史記錄中的所有樣本。

持久性類型為`TRANSIENT_DURABILITY_QOS`意味著樣本比數據寫入者更長，並且只要該過程存活就持續。樣本保存在內存中，但不會持久保存到永久存儲器中。將在同一域內訂閱相同主題和分區的數據讀取器將發送屬於同一主題/分區的所有緩存樣本。

持久性類型`PERSISTENT_DURABILITY_QOS`提供與暫時持久性基本上相同的功能，除非緩存的樣本被持久化，並且能夠經受過程破壞。

當指定了暫態或持久性持久性時，`DURABILITY_SERVICE QoS`策略會為持久性高速緩存指定其他調整參數。

在創建數據寫入器和數據讀取器之間的關聯時考慮持久性策略。為了創建關聯，關聯的兩側的值必須兼容。數據寫入器的耐久性類型值必須大於或等於數據讀取器的對應值。耐久性類型值按如下順序排列：

```cpp
PERSISTENT_DURABILITY_QOS >
TRANSIENT_DURABILITY_QOS >
TRANSIENT_LOCAL_DURABILITY_QOS >
VOLATILE_DURABILITY_QOS
```

## 3.2.6 DURABILITY\_SERVICE

`DURABILITY_SERVICE`策略控制在`TRANSIENT`或`PERSISTENT`持久性緩存中刪除樣本。 此策略通過其各自QoS結構的`durability_service`成員適用於主題和數據寫入程序實體，並提供了為樣本緩存指定`HISTORY`和`RESOURCE_LIMITS`的方法。 下面是與持久性服務QoS策略相關的IDL：

```cpp
struct DurabilityServiceQosPolicy {
 Duration_t service_cleanup_delay;
 HistoryQosPolicyKind history_kind;
 long history_depth;
 long max_samples;
 long max_instances;
 long max_samples_per_instance;
};
```

歷史和資源限製成員類似於，但獨立於`HISTORY`和`RESOURCE_LIMITS`策略中找到的成員。 `service_cleanup_delay`可以設置為所需的值。 默認情況下，它設置為零，這意味著從不清除緩存的樣本。

## 3.2.7 RESOURCE\_LIMITS

`RESOURCE_LIMITS`策略確定服務可以消耗以滿足請求的QoS的資源量。 此策略通過其各自的QoS結構的`resource_limits`成員適用於主題，數據讀取器和數據寫入器實體。 以下是與資源限制QoS策略相關的IDL。

```cpp
struct ResourceLimitsQosPolicy {
 long max_samples;
 long max_instances;
 long max_samples_per_instance;
};
```

`max_samples`成員指定單個數據寫入器或數據讀取器可以在其所有實例中管理的最大樣本數。 `max_instances`成員指定數據寫入器或數據讀取器可以管理的最大實例數。 `max_samples_per_instance`成員指定可以在單個數據寫入器或數據讀取器中為單個實例管理的最大樣本數。 所有這些成員的值默認為無限制（`DDS :: LENGTH_UNLIMITED`）。

數據寫入器使用資源來對寫入數據寫入器但未發送到所有數據讀取器的樣本進行排隊，因為來自傳輸的背壓。 數據讀取器使用資源來對已經接收但尚未從數據讀取器讀取/取得的樣本進行排隊。

## 3.2.8分區

`PARTITION`QoS策略允許在域中創建邏輯分區。 它只允許數據讀取器和數據寫入器關聯，如果他們有匹配的分區字符串。

該策略通過它們各自的QoS結構的分區成員適用於發布者和訂戶實體。 下面是與分區QoS策略相關的IDL。

```cpp
struct PartitionQosPolicy {
 StringSeq name;
};
```

名稱成員默認為空的字符串序列。 默認分區名稱為空字符串，並使實體參與默認分區。 分區名稱可能包含由POSIX fnmatch函數（POSIX 1003.2-1992第B.6節）定義的通配符。

數據讀取器和數據寫入器關聯的建立取決於發布和訂閱端上匹配的分區字符串。 無法匹配分區不會被視為失敗，不會觸發任何回調或設置任何狀態值。

此政策的值可能隨時更改。 對此策略的更改可能會導致關聯被刪除或添加。

## 3.2.9最後期限

`DEADLINE`QoS策略允許應用程序檢測何時在指定的時間內未寫入或讀取數據。 此策略通過其各自的QoS結構的截止成員適用於主題，數據寫入器和數據讀取器實體。 以下是與截止期限QoS策略相關的IDL。

```cpp
struct DeadlineQosPolicy {
 Duration_t period;
};
```

週期成員的默認值是無限的，這不需要任何行為。當此策略設置為有限值時，數據寫入器監視應用程序對數據所做的更改，並通過設置相應的狀態條件並觸發`on_offered_deadline_missed()`監聽器回調來指示未能遵守策略。檢測到數據在期限到期之前未更改的數據讀取器將設置相應的狀態條件，並觸發`on_requested_deadline_missed()`偵聽器回調。

在創建數據寫入器和數據讀取器之間的關聯時考慮此策略。為了創建關聯，關聯的兩側的值必須兼容。數據讀取器的截止期限必須大於或等於數據寫入器的相應值。

在啟用關聯實體後，此策略的值可能會更改。在做出數據讀取器或數據寫入器的策略的情況下，只有當改變保持與讀取器或寫入器正參與的所有關聯的遠端一致時，才成功地應用改變。如果主題的策略更改，則它將僅影響在更改後創建的數據讀取器和寫入器。任何現有的讀者或作者以及它們之間的任何現有關聯都不會受到主題策略值更改的影響。

## 3.2.10生活用品

LIFESPAN QoS策略允許應用程序指定樣本到期時間。 過期樣品將不會發送給訂閱者。 此策略通過其各自的QoS結構的壽命成員應用於主題和數據寫入器實體。 下面是與壽命QoS策略相關的IDL。

```cpp
struct LifespanQosPolicy {
 Duration_t duration;
}
```

持續時間成員的默認值是無限的，這意味著樣本永不過期。

當使用除`VOLATILE`之外的`DURABILITY`類型時，OpenDDS目前支持發布商端過期的樣本檢測。 當它們被放置在高速緩存中到期後，目前OpenDDS實現可能無法從數據寫入和數據讀取緩存中刪除樣本。

此方法的值可能隨時更改。 對此方法的更改僅影響更改後寫入的數據。

## 3.2.11 USER\_DATA

`USER_DATA`策略通過它們各自的QoS結構的`user_data`成員應用於域參與者，數據讀取器和數據寫入器實體。 以下是與用戶數據QoS策略相關的IDL：

```cpp
struct UserDataQosPolicy {
 sequence<octet> value;
};
```

默認情況下，不設置值成員。 它可以設置為任何可用於將信息附加到創建的實體的八位字節序列。 `USER_DATA`策略的值在各自的內置主題數據中可用。 遠程應用程序可以通過內置主題獲取信息，並將其用於自己的目的。 例如，應用程序可以通過`USER_DATA`策略附加安全憑證，遠程應用程序可以使用該策略來驗證源。

## 3.2.12 TOPIC\_DATA

`TOPIC_DATA`策略通過TopicQoS結構的`topic_data`成員適用於主題實體。 以下是與主題數據QoS策略相關的IDL：

```cpp
struct TopicDataQosPolicy {
 sequence<octet> value;
};
```

默認情況下，未設置該值。 可以將其設置為將附加信息附加到創建的主題。 TOPIC\_DATA策略的值在數據寫入程序，數據讀取器和主題內置主題數據中可用。 遠程應用程序可以通過內置主題獲取信息，並以應用程序定義的方式使用它。

## 3.2.13 GROUP\_DATA

`GROUP_DATA`策略通過它們各自的QoS結構的`group_data`成員應用於發布者和訂閱者實體。 以下是與組數據QoS策略相關的IDL：

```cpp
struct GroupDataQosPolicy {
 sequence<octet> value;
};
```

默認情況下，不設置值成員。 它可以設置為將附加信息附加到創建的實體。 `GROUP_DATA`策略的值通過內置主題傳播。 數據寫入器內置主題數據包含來自發布者的`GROUP_DATA`，並且數據讀取器內置主題數據包含來自訂閱者的`GROUP_DATA`。 `GROUP_DATA`策略可以用於實現類似於1.1.6中描述的`PARTITION`策略的匹配機制，除非可以基於應用定義的策略做出決定。

## 3.2.14TRANSPORT\_PRIORITY

`TRANSPORT_PRIORITY`策略通過它們各自的QoS策略結構的`transport_priority`成員應用於主題和數據寫入器實體。 以下是與`TransportPriority `QoS策略相關的IDL：

```cpp
struct TransportPriorityQosPolicy {
 long value;
};
```

`transport_priority`的默認值成員為零。該策略被認為是傳輸層指示以什麼優先級發送消息的提示。較高的值表示較高的優先級。 OpenDDS將優先級值直接映射到線程和`DiffServ`代碼點值。默認優先級為零不會修改消息中的線程或代碼點。

OpenDDS將嘗試設置發送傳輸的線程優先級以及任何關聯的接收傳輸。傳輸優先級值從零（default）通過線性優先級線性映射，不進行縮放。如果最低線程優先級不為零，則將其映射到傳輸優先級值零。當系統上的優先級值反轉（較高的數值是較低優先級）時，OpenDDS將這些值映射到從零開始的遞增優先級值。低於系統上最小（最低）線程優先級的優先級值將映射到最低優先級。

大於系統上最大（最高）線程優先級的優先級值將映射到該最高優先級。在大多數係統上，線程優先級只能在進程調度程序已設置為允許這些操作時設置。設置進程調度程序通常是特權操作，並且需要係統特權才能執行。在基於POSIX的系統上，`sched_get_priority_min()`和`sched_get_priority_max()`的系統調用用於確定線程優先級的系統範圍。

如果傳輸實現支持，OpenDDS將嘗試在用於為數據寫入程序發送數據的套接字上設置`DiffServ`代碼點。 如果網絡硬件尊重碼點值，則較高碼點值將導致較高優先級樣本的較好（較快）傳輸。 默認值0將被映射到零的默認代碼點。 然後將從1到63的優先級值映射到相應的碼點值，並且將較高優先級值映射到最高碼點值（63）。

OpenDDS目前不支持在創建數據寫入器之後修改`transport_priority`策略值。 這可以通過創建新的數據寫入器來解決，因為需要不同的優先級值。

## 3.2.15 LATENCY\_BUDGET

`LATENCY_BUDGET`策略通過其各自的QoS策略結構的`latency_budget`成員應用於主題，數據讀取器和數據寫入器實體。 以下是與`LatencyBudget `QoS策略相關的IDL：

```cpp
struct LatencyBudgetQosPolicy {
 Duration_t duration;
};
```

duration的默認值為0，表示應該將延遲最小化。 該策略被認為是傳輸層指示正被發送的樣本的緊急性的提示。 OpenDDS使用該值來綁定延遲間隔，用於報告將樣品從發布傳輸到預訂的不可接受的延遲。 此策略僅用於此時的監視目的。 使用`TRANSPORT_PRIORITY`策略來修改樣本的發送。 數據寫入器策略值僅用於兼容性比較，如果保留為默認值0，將導致來自數據讀取器的所有請求的持續時間值匹配。

已添加其他監聽器擴展，以允許報告延遲超出策略持續時間設置。 `OpenDDS :: DCPS :: DataReaderListener`接口具有用於通知接收到樣本的測量傳輸延遲大於`latency_budget`策略持續時間的附加操作。 此方法的IDL為：

```cpp
struct BudgetExceededStatus {
  long total_count;
long total_count_change;
 DDS::InstanceHandle_t last_instance_handle;
 };
void on_budget_exceeded(in DDS::DataReader reader,in BudgetExceededStatus status);
```

要使用擴展偵聽器回調，您需要從擴展接口派生監聽器實現，如以下代碼段所示：

```cpp
 class DataReaderListenerImpl
 : public virtual
 OpenDDS::DCPS::LocalObject<OpenDDS::DCPS::DataReaderListener>
```

然後，您必須為on\_budget\_exceeded（）操作提供非null實現。

請注意，您還需要為以下擴展操作提供空實現：

```cpp
on_subscription_disconnected()
 on_subscription_reconnected()
 on_subscription_lost()
 on_connection_deleted()
```

OpenDDS還通過數據讀取器的擴展接口使摘要延遲統計信息可用。 此擴展接口位於`OpenDDS :: DCPS`模塊中，IDL定義為：

```cpp
 struct LatencyStatistics {
  GUID_t publication;
  unsigned long n;
  double maximum;
  double minimum;
  double mean;
  double variance;
 };
 typedef sequence<LatencyStatistics> LatencyStatisticsSeq;
 local interface DataReaderEx : DDS::DataReader {
  /// Obtain a sequence of statistics summaries.
  void get_latency_stats( inout LatencyStatisticsSeq stats);
  /// Clear any intermediate statistical values.
  void reset_latency_stats();
  /// Statistics gathering enable state.
  attribute boolean statistics_enabled;
 };
```

要收集此統計摘要數據，您將需要使用擴展界面。 您可以通過動態強制轉換OpenDDS數據讀取器指針並直接調用操作來完成此操作。 在下面的示例中，我們假設reader通過調用`DDS :: Subscriber :: create_datareader()`正確初始化：

```cpp
 DDS::DataReader_var reader;
 // ...
 // To start collecting new data.
 dynamic_cast<OpenDDS::DCPS::DataReaderImpl*>(reader.in())->
  reset_latency_stats();
 dynamic_cast<OpenDDS::DCPS::DataReaderImpl*>(reader.in())->
  statistics_enabled(true);
 // ...
 // To collect data.
 OpenDDS::DCPS::LatencyStatisticsSeq stats;
 dynamic_cast<OpenDDS::DCPS::DataReaderImpl*>(reader.in())->
  get_latency_stats(stats);
 for (unsigned long i = 0; i < stats.length(); ++i)
 {
  std::cout << "stats[" << i << "]:" << std::endl;
  std::cout << " n = " << stats[i].n << std::endl;
  std::cout << " max = " << stats[i].maximum << std::endl;
  std::cout << " min = " << stats[i].minimum << std::endl;
  std::cout << " mean = " << stats[i].mean << std::endl;
  std::cout << " variance = " << stats[i].variance << std::endl;
 }
```































