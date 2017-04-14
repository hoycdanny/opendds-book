# CHAPTER 6

# Built-In Topics

# 6.1 Introduction

在OpenDDS中，默認情況下創建和發佈內置主題，以交換有關在部署中操作的DDS參與者的信息。 當使用DCPSInfoRepo服務在集中式發現方法中使用OpenDDS時，此服務會發佈內置主題。 對於DDSI-RTPS部署，過程中實例化的內部OpenDDS實現創建並使用內置主題，以便與其他DDS參與者完成相似的發現信息交換。 有關RTPS發現配置的說明，請參見第7.3.3節。

# 6.2 DCPSInfoRepo配置的內置主題

使用DCPSInfoRepo時，可以使用-NOBITS的命令行選項來抑制內置主題的發布。

為每個域定義了四個獨立的主題。 每個專用於特定實體（域參與者，主題，數據寫入器，數據讀取器），並且發布描述域中每個實體的狀態的實例。

為每個域參與者自動創建對內置主題的訂閱。 參與者對內置主題的支持可以通過DCPSBit配置選項切換（請參見第7.2節中的表）（注意：此選項不能用於RTPS發現）。 要查看內置的主題數據，只需獲取內置的訂閱服務器，然後使用它來訪問數據讀取器以獲取感興趣的內置主題。 數據讀取器可以像任何其他數據讀取器一樣使用。第6.3至6.6節提供了為四個內置主題中的每一個發布的數據的詳細信息。 顯示如何從內置主題中讀取的示例遵循這些部分。

如果您不打算在應用程序中使用內置主題，則可以在構建時配置OpenDDS以刪除內置主題支持。 這樣做可以將核心DDS庫的佔地面積減少高達30％。 有關禁用BuiltIn-Topic支持的信息，請參見第1.3.2節。

# 6.3 DCPS參與者主題

DCPSParticipant主題發布有關域的域參與者的信息。 以下是定義為此主題發布的結構的IDL：

```cpp
struct ParticipantBuiltinTopicData {
 BuiltinTopicKey_t key; // struct containing an array of 3 longs
 UserDataQosPolicy user_data;
};
```

每個域參與者都由一個唯一的密鑰定義，並且是本主題中自己的實例。

# 6.4 DCPSTopic主題

**注意:**配置為RTPS發現時，OpenDDS不支持此內置主題。

DCPSTopic主題發布有關域中主題的信息。 以下是定義為此主題發布的結構的IDL：

```cpp
struct TopicBuiltinTopicData {
 BuiltinTopicKey_t key;
 string name;
 string type_name;
 DurabilityQosPolicy durability;
 QosPolicy deadline;
 LatencyBudgetQosPolicy latency_budget;
 LivelinessQosPolicy liveliness;
 ReliabilityQosPolicy reliability;
 TransportPriorityQosPolicy transport_priority;
 LifespanQosPolicy lifespan;
 DestinationOrderQosPolicy destination_order;
 HistoryQosPolicy history;
 ResourceLimitsQosPolicy resource_limits;
 OwnershipQosPolicy ownership;
 TopicDataQosPolicy topic_data;
};
```

每個主題都由一個唯一的密鑰標識，並且是這個內置主題中的自己的實例。 上述成員標識主題名稱，主題名稱以及該主題的QoS策略集。

# 6.5 DCPS發布主題

DCPSPublication主題發布有關域中數據作者的信息。

以下是定義為此主題發布的結構的IDL：

```cpp
struct PublicationBuiltinTopicData {
 BuiltinTopicKey_t key;
 BuiltinTopicKey_t participant_key;
 string topic_name;
 string type_name;
 DurabilityQosPolicy durability;
 DeadlineQosPolicy deadline;
 LatencyBudgetQosPolicy latency_budget;
 LivelinessQosPolicy liveliness;
 ReliabilityQosPolicy reliability;
 LifespanQosPolicy lifespan;
 UserDataQosPolicy user_data;
 OwnershipStrengthQosPolicy ownership_strength;
 PresentationQosPolicy presentation;
 PartitionQosPolicy partition;
 TopicDataQosPolicy topic_data;
 GroupDataQosPolicy group_data;
};
```

每個數據寫入器在創建時都會被分配一個唯一的密鑰，並在此主題中定義自己的實例。 上述字段標識數據寫入器所屬的域參與者（通過其密鑰），主題名稱和類型以及應用於數據寫入器的各種QoS策略。

# 6.6 DCPS訂閱主題

DCPSSubscription主題發布域中數據讀取器的信息。

以下是定義為此主題發布的結構的IDL：

```cpp
struct SubscriptionBuiltinTopicData {
 BuiltinTopicKey_t key;
 BuiltinTopicKey_t participant_key;
 string topic_name;
 string type_name;
 DurabilityQosPolicy durability;
 DeadlineQosPolicy deadline;
 LatencyBudgetQosPolicy latency_budget;
 LivelinessQosPolicy liveliness;
 ReliabilityQosPolicy reliability;
 DestinationOrderQosPolicy destination_order;
 UserDataQosPolicy user_data;
 TimeBasedFilterQosPolicy time_based_filter;
 PresentationQosPolicy presentation;
 PartitionQosPolicy partition;
 TopicDataQosPolicy topic_data;
 GroupDataQosPolicy group_data;
};
```

每個數據讀取器在創建時都被分配一個唯一的密鑰，並在此主題中定義了自己的實例。 上述字段標識數據讀取器所屬的域參與者（通過其密鑰），主題名稱和類型以及應用於數據讀取器的各種QoS策略。

# 6.7內置主題訂閱示例

以下代碼使用域參與者獲取內置訂閱者。 然後使用訂閱者獲取DCPSP參與者主題的數據讀取器，然後讀取該讀取器的樣本。

```
 Subscriber_var bit_subscriber = participant->get_builtin_subscriber() ;
 DDS::DataReader_var dr =
  bit_subscriber->lookup_datareader(BUILT_IN_PARTICIPANT_TOPIC);
 DDS::ParticipantBuiltinTopicDataDataReader_var part_dr =
  DDS::ParticipantBuiltinTopicDataDataReader::_narrow(dr);
 DDS::ParticipantBuiltinTopicDataSeq part_data;
 DDS::SampleInfoSeq infos;
 DDS::ReturnCode_t ret = part_dr->read(part_data, infos, 20,DDS::ANY_SAMPLE_STATE,DDS::ANY_VIEW_STATE,DDS::ANY_INSTANCE_STATE) ;
 // Check return status and read the participant data
```

其他內置主題的代碼是相似的。



