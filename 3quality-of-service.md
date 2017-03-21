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

##### Quality of Service

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
| PRESENTATION |  |  |
|  |  |  |
|  |  |  |
|  |  |  |

#### 表3-4為預設用戶QoS策略

#### 表3-5為預設DataWriter QoS策略



