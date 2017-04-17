# CHAPTER 12 

# Recorder and Replayer

# 12.1概述

OpenDDS的記錄器功能允許應用程序記錄發佈在任意主題上的樣本，而無需事先知道該主題使用的數據類型。 類似地，Replayer功能允許將這些記錄的樣本重新發布回相同或其他主題。 這些功能與其他數據讀取器和寫入器的不同之處在於它們能夠處理任何數據類型，即使在應用程序構建時未知。 有效地，樣本被處理，就好像每個樣本都是不飽和的字節序列。

本章的目的是描述OpenDDS的公共API以啟用錄音/重放用例。

# 12.2 API結構

在OpenDDS :: DCPS命名空間中定義了兩個新的用戶可見類（其行為有點像其DDS實體對應方）以及關聯的偵聽器接口。

偵聽器可以由應用程序可選地實現。 Recorder類與DataReader類似，並且Replayer類與DataWriter類似。

Recorder和Replayer都分別使用底層的OpenDDS發現和傳輸庫，就像它們分別是DataReader和DataWriter一樣。 域中的常規OpenDDS應用程序將“查看”Recorder對象，就像它們是遠程DataReaders和Replayers一樣，就像它們是DataWriters一樣。

# 12.3使用模式

應用程序根據需要創建任意數量的記錄器和重放器。 這可以基於使用內置主題來動態地發現哪些主題在域中是活動的。 創建記錄器或Replayer需要應用程序提供主題名稱和類型名稱（如DomainParticipant :: create\_topic（））以及相關的QoS數據結構。 記錄器需要SubscriberQos和DataReaderQos，而Replayer需要PublisherQos和DataWriterQos。 這些值用於發現的讀/寫器匹配。 請參閱下面有關QoS處理的部分，了解Recorder和Replayer如何使用QoS。 以下是創建錄像機所需的代碼：

```cpp
OpenDDS::DCPS::Recorder_var recorder =
service_participant->create_recorder(domain_participant,topic.in(),sub_qos,dr_qos,recorder_listener);
```

數據樣本通過RecorderListener使用簡單的“每個樣本回調”模式提供給應用程序。 樣本作為OpenDDS :: DCPS :: RawDataSample對像傳遞。 該對象包括該數據樣本的時間戳以及封送的樣本值。 這是用戶定義的記錄器監聽器的類定義。

```cpp
class MessengerRecorderListener : public OpenDDS::DCPS::RecorderListener
{
public:
    MessengerRecorderListener();
    virtual void on_sample_data_received(OpenDDS::DCPS::Recorder*,const OpenDDS::DCPS::RawDataSample& sample);
    virtual void on_recorder_matched(OpenDDS::DCPS::Recorder*,const DDS::SubscriptionMatchedStatus& status );
};
```

應用程序可以將數據存儲在任何適合的位置（在內存，文件系統，數據庫等中）。 在任何時候，應用程序可以將相同的樣本提供給為同一主題配置的Replayer對象。 應用程序的責任是確保主題類型匹配。 這是一個示例調用，將示例重播到所有與重播者主題相連的讀卡器：

```cpp
replayer->write(sample);
```

因為存儲的數據取決於數據結構的定義，所以它不能在OpenDDS的不同版本或OpenDDS參與者使用的不同版本的IDL中使用。

# 12.4 QoS處理

缺乏對數據樣本的詳細了解使得在Replayer方面使用了許多正常的DDS QoS屬性。 屬性可以分為幾類：

• Supported

– Liveliness

– Time-Based Filter

– Lifespan

– Durability \(transient local level, see details below\)

– Presentation \(topic level only\)

– Transport Priority \(pass-thru to transport\)

• Unsupported

– Deadline \(still used for reader/writer match\)

– History

– Resource Limits

– Durability Service

– Ownership and Ownership Strength \(still used for reader/writer match\)

• Affects reader/writer matching and Built-In Topics but otherwise ignored

– Partition

– Reliability \(still used by transport negotiation\)

– Destination Order

– Latency Budget

– User/Group Data

## 12.4.1耐用性細節

在記錄儀方面，瞬態局部耐久性與任何正常的DataReader相同。 從匹配的DataWriters接收到耐用數據。 在Replayer方面有一些差異。 與正常的DDS DataWriter相反，Replayer不緩存/存儲任何數據樣本（它們只是發送到傳輸）。 由於實例不知道，根據通常的歷史和資源限制規則存儲數據樣本是不可能的。 相反，可以通過“拉”模式支持瞬態本地耐久性，當中間件在發現新的遠程DataReader時調用ReplayerListener上的方法。 然後，應用程序可以使用應發送到新加入的DataReader的任何數據樣本，在Replayer上調用方法。 確定哪些樣品留給應用程序。



