# CHAPTER 2

### Getting Started

## 2.1使用DCPS

本章重點介紹使用 DCPS 將數據從單個 publisher 進程分發到單個 subscriber 進程的示例應用程序。它基於一個簡單的信訊息應用程序，其中單個 publisher 發布消息，並且單個 subscriber 訂閱它們。我們使用默認的 QoS 屬性和默認的 TCP/IP 傳輸。此示例的完整源代碼可以在 $DDSR/OOT/DevGuideExamples/DCPS/Messenger/ 目錄。其他 DDS 和 DCPS 功能將在後面的章節中討論。

## 2.1.1定義數據類型

DDS 使用的每種數據類型都是使用 IDL 定義的。OpenDDS使用 #pragma 指令來標識 DDS 傳輸和處理的數據類型。這些數據類型由 TAO IDL 編譯器和 OpenDDS IDL 編譯器處理，以生成必要的代碼，以使用 OpenDDS 傳輸這些類型的數據。這裡是定義我們的消息數據類型的IDL文件：

```cpp
module Messenger {
#pragma DCPS_DATA_TYPE "Messenger::Message"
#pragma DCPS_DATA_KEY "Messenger::Message subject_id"
struct Message {
    string from;
    string subject;
    long subject_id;
    string text;
    long count;
    };
};
```

DCPS_DATA_TYPE pragma 標記用於 OpenDDS 的數據類型。全範圍型必須配合 pragma 使用。OpenDDS 要求數據類型是一個結構。該結構可以包含純量類型（short, long, float,等）enumerations, strings,sequences, arrays, structures,和 unions。此 Opendds 範例定義了 structure Messenger 在  Messenger module 之中。

DCPS_DATA_KEY pragma 為 DCPS 的一個辨識片斷像是這個型態的 key 。數據類型可以有零個或多個 key 。這些 key 用來分辨不同的實例在 topic 中 。
每個 key 應該是數字或 enumerated 的型態，字串, 或這些型態的其中一種(註1)。
pragma 通過完全範圍類型和會員名稱型態的識別 key 。複數的 key 是被單獨指定 DCPS_DATA_KEY pragmas。在範例中，我們將 Messenger::Message 當作 subject_id 的辨識。每個範例推送都有獨特的 subject_id 值在同一 topic 下來辨認屬於哪個實例。
在使用默認的 QOS 下，具有相同 subject_id 的值會被後續相同的 subject_id 的值替換掉。
註1.其他類型（例如structures, sequences, 和 arrays）不能直接當做 key，但是當這些members/elements是numeric, enumerated, string 類型時，structs or elements of arrays 的單個成員可以用作 key。
## 2.1.2處理IDL

OpenDDS IDL 首先由 TAO IDL編 譯器處理。

`tao_idl Messenger.idl`

此外，我們需要使用OpenDDS IDL編譯器處理IDL文件，以生成OpenDDS需要編譯和解密消息的序列化和密鑰支持代碼，以及 data reader 和 data writer 支援的類型代碼。此 IDL 編譯器位於 $DDS/ROOT/bin/ 中，並為每個處理的 IDL 文件生成三個文件。這三個文件都以原始IDL文件名開頭，顯示如下：

•<ilename>TypeSupport.idl



• <filename>TypeSupportImpl.h

• <filename>TypeSupportImpl.cpp

例如，運行opendds_idl如下

opendds_idl Messenger.idl

生成 MessengerTypeSupport.idl，MessengerTypeSupportImpl.h和MessengerTypeSupportImpl.cpp。IDL文件包含 MessageTypeSupport，MessageDataWriter 和 MessageDataReader 接口定義。這些是類型特定的DDS接口，
稍候我們會使用這些特別的型態註冊 domain ,推播範例的資料類型,接收推送範例。實現文件包含這些接口的實現。生成的 IDL 文件本身應該使用 TAO IDL 編譯器來編譯，以生成 stubs 和 skeletons. 。這些和實現文件應該與使用消息類型的 OpenDDS 應用程序鏈接。OpenDDS IDL編譯器有許多選項專門生成生成的代碼。這些選項在第8章中描述。

通常，您不直接調用 TAO 或 OpenDDS IDL 編譯器，但讓您的構建環境為您做。
使用 MPC 簡化整個程序藉由繼承 dcpsexe_with_tcp 專案。
這是 publisher 和 subscriber 共同的 MPC 文件部分

```cpp
project(*idl): dcps {
    // This project ensures the common components get built first.
    TypeSupport_Files {
        Messenger.idl
    }
    custom_only = 1
}
```

dcps 父項目添加了類型支援自定義構建規則。上面的 TypeSupport_Files 部分告訴MPC使用 OpenDDS IDL 編譯器從 Messenger.idl 生成訊息類型支援文件。 這是 publisher  部分：

```cpp
project(*Publisher): dcpsexe_with_tcp {
    exename = publisher
    after += *idl
    TypeSupport_Files {
        Messenger.idl
    }
    Source_Files {
        Publisher.cpp
    }
}
```

dcpsexe_with_tcp 項目鏈接在DCPS庫中。

為了完整性，這裡是MPC文件的 subscriber  部分：

```cpp
project(*Subscriber): dcpsexe_with_tcp {
    exename = subscriber
    after += *idl
    TypeSupport_Files {
        Messenger.idl
    }
Source_Files {
    Subscriber.cpp
        DataReaderListenerImpl.cpp
    }
}
```

## 2.1.3簡單消息 Publisher

在本節中，我們將介紹設置一個簡單的 OpenDDS 推撥程序所涉及的步驟。代碼被分成邏輯部分，並解釋為我們呈現每個部分。

我們省略了代碼中一些不感興趣的部分（例如 #include指令，錯誤處理和跨進程同步）。 此示例發布者的完整源代碼位於 $DDS/ROOT/DevGuideExamples/DCPS/Messenger/ 中的 Publisher.cpp 和 Writer.cpp 文件中。

### 2.1.3.1初始化參與者

`main()`的第一部分將當前進程初始化為 OpenDDS 參與者。
```
int main (int argc, char *argv[]) {
 try {
  DDS::DomainParticipantFactory_var dpf =
   TheParticipantFactoryWithArgs(argc, argv);
  DDS::DomainParticipant_var participant =
   dpf->create_participant(42, // domain ID
     PARTICIPANT_QOS_DEFAULT,
     0, // No listener required
     OpenDDS::DCPS::DEFAULT_STATUS_MASK);
  if (!participant) {
   std::cerr << "create_participant failed." << std::endl;
   return 1;
  }
```

`TheParticipantFactoryWithArgs` 巨集 `Service_Participant.h` 中定義，並以命令列參數來初始化  Domain Participant Factory。這些命令參數用於初始化 ORB 也就是 OpenDDS 服務本身。這也允許我們略過 ORB_init()選項以及 OpenDDS 的 DCPS* 選項設定。可用的OpenDDS選項在第7章中有詳細描述。

`create_participant()` 使用domain 參與工廠註冊 ID 42 的 domain。參與者使用默認的 QOS 而且沒有監聽者。使用 OpenSSD 默認狀態遮罩確保所有相關的溝通狀態改變(像是 資料可用性、liveliness lost)中介層可以和應用層溝通(像是 通過監聽者回傳)。

可以定義 doamin ID 範圍(0x00000000 ~ 0x7FFFFFFF)。其他保留值為內部使用。
Domain Participant 物件引用我們註冊的訊息資料型態。

### 2.1.3.2註冊數據類型和創建 Topic

首先，我們創建一個 MessageTypeSupportImpl 的物件並用註冊 register_type() 。在此範例中註冊一個空字串的型態，這會使 MessageTypeSupportImpl 為接口辨識名稱。也可以使用"Message" 為該特定辨識名稱。

```cpp
 Messenger::MessageTypeSupport_var mts =
  new Messenger::MessageTypeSupportImpl();
 if (DDS::RETCODE_OK != mts->register_type(participant, "")) {
  std::cerr << "register_type failed." << std::endl;
  return 1;
 }
```

接下來，我們註冊該類型名稱並且創建 topic 藉由使用 create_topic()。

```cpp
CORBA::String_var type_name = mts->get_type_name ();
 DDS::Topic_var topic =
  participant->create_topic ("Movie Discussion List",
       type_name,
       TOPIC_QOS_DEFAULT,
       0, // No listener required
       OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 if (!topic) {
  std::cerr << "create_topic failed." << std::endl;
  return 1;
 }
```

我們已經創建好一個為 "Movie Discussion List" 的 topic 名稱並使用預設的 QOS。

### 2.1.3.3 創造 Publisher

現在我們使用默認的 QOS 來創造 Publisher。

```cpp
DDS::Publisher_var pub =
 participant->create_publisher(PUBLISHER_QOS_DEFAULT,
     0, // No listener required
     OpenDDS::DCPS::DEFAULT_STATUS_MASK);
if (!pub) {
 std::cerr << "create_publisher failed." << std::endl;
 return 1;
 }
```

### 2.1.3.4創造 DataWriter 並等待 Subscriber

隨著 publisher 到位，我們創造 DataWriter。

```cpp
// Create the datawriter
 DDS::DataWriter_var writer =
  pub->create_datawriter(topic,
    DATAWRITER_QOS_DEFAULT,
    0, // No listener required
    OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 if (!writer) {
  std::cerr << "create_datawriter failed." << std::endl;
  return 1;
 }
```

當我們創造 DataWriter 時，我們傳遞 topci對象引用，默認QoS策略和空偵聽器引用。 我們現在將數據寫入器引用縮小到**MessageDataWriter**對象引用，以便我們可以使用特定於類型的發布操作。
當我們創造 DataWriter ，引用 topic 物件、預設的 QOS 和空的監聽‧。我線現在將 DataWriter 指向 MessageDataWriter 參考，之後可以使用特定類型推撥選項。

```cpp
Messenger::MessageDataWriter_var message_writer =
 Messenger::MessageDataWriter::_narrow(writer);
```

在範例代碼中使用條件和等待集，以便 publisher 等待 subscriber 連線並完成初始化。這個例子中會等待 subscriber 失敗，因為 publisher 發送了範例資料在 subscriber 連線之前。

等待用戶所涉及的基本步驟是：

1）從我們創建的 data writer 獲取狀態條件

2）在條件中啟用 Publication Matched 狀態

3）創建等待集

4）將狀態條件附加到等待集合

5）獲取發布匹配狀態

6）如果匹配的當前計數為一個或多個，則從等待集中分離條件並繼續發布

7）等待等待集（可以被指定的時間段限制）

8）循環回到步驟5

這裡是相應的代碼：

```cpp
// Block until Subscriber is available
 DDS::StatusCondition_var condition =
  writer->get_statuscondition();
  condition>set_enabled_statuses(
  DDS::PUBLICATION_MATCHED_STATUS);
 DDS::WaitSet_var ws = new DDS::WaitSet;
 ws->attach_condition(condition);
 while (true) {
  DDS::PublicationMatchedStatus matches;
  if (writer->get_publication_matched_status(matches)
    != DDS::RETCODE_OK) {
   std::cerr << "get_publication_matched_status failed!"
             << std::endl;
   return 1;
  }
 if (matches.current_count >= 1) {
   break;
 }
 DDS::ConditionSeq conditions;
 DDS::Duration_t timeout = { 60, 0 };
 if (ws->wait(conditions, timeout) != DDS::RETCODE_OK) {
   std::cerr << "wait failed!" << std::endl;
   return 1;
  }
 }
 ws->detach_condition(condition);
```

有關狀態，條件和等待集的更多詳細信息，請參閱第4章。

### 2.1.3.5樣品公佈

消息發布相當簡單：

```cpp
// Write samples
 Messenger::Message message;
 message.subject_id = 99;
 message.from = "Comic Book Guy";
 message.subject = "Review";
 message.text = "Worst. Movie. Ever.";
 message.count = 0;
 for (int i = 0; i < 10; ++i) {
  DDS::ReturnCode_t error = message_writer->write(message,DDS::HANDLE_NIL);
  ++message.count;
  ++message.subject_id;
 if (error != DDS::RETCODE_OK) {
 // Log or otherwise handle the error condition
 return 1;
 }
}
```
對於每個循環迭代，調用 `write（）` 會將訊息分歲給所有以連線並註冊我們 topic 的 subscriber 。當 subject_id 為訊息的 key 時，在呼叫 'write()' 會去遞增 subject_id，會去新增一個新的實例(參見1.1.1.3）。'write()' 第二個參數指定正在發送的實例樣本。他將略過或是處理 'register_instance()' 或 'DDS::HANDLE_NIL' 的回傳。傳遞 DDS::HANDLE_NIL 的值藉由檢查樣本的 key 來確定實例。有關發布的實例的處理細節參考(2.2.1節)。

## 2.1.4 設置訂閱 Subscriber

Subscriber 的許多代碼與剛解釋的 publisher 類似或相似。將會快速略過相似的部分和參考討論相關的細節。
此範例 subscriber 的完整源代碼位於 $DDS/ROOT/DevGuideExamples/DCPS/Messenger/ 中的'Subscriber.cpp'和'DataReaderListener.cpp'文件中。

### 2.1.4.1初始化參與者

subscriber 的開頭與 publisher 相同，我們初始化服務並加入我們的 doamin：

```cpp
int main (int argc, char *argv[])
{
 try {
  DDS::DomainParticipantFactory_var dpf =
   TheParticipantFactoryWithArgs(argc, argv);
  DDS::DomainParticipant_var participant =
  dpf->create_participant(42, // Domain ID PARTICIPANT_QOS_DEFAULT, 0, // No listener required OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 if (!participant) {
  std::cerr << "create_participant failed." << std::endl;
  return 1 ;
 }
```

### 2.1.4.2註冊數據類型和創建主題

接下來，我們初始化訊息類型和 topic。
注意，如果 topic 已經被初始化相同的資料型態和相容的 QoS 在這一個 domain ，'create_topic()' 會調用已經存在的 topic。如果與現有的 topic 不到符合的型態或是 QoS 'create_topic'  會調用失敗。還有一個 'find_topic()' 用來檢索存在的 topic。

```cpp
Messenger::MessageTypeSupport_var mts =
 new Messenger::MessageTypeSupportImpl();
 if (DDS::RETCODE_OK != mts->register_type(participant, "")) {
  std::cerr << "Failed to register the MessageTypeSupport." << std::endl;
  return 1;
 }
 CORBA::String_var type_name = mts->get_type_name ();
 DDS::Topic_var topic =
  participant->create_topic("Movie Discussion List",
type_name,TOPIC_QOS_DEFAULT, 0, // No listener required OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 if (!topic) {
  std::cerr << "Failed to create_topic." << std::endl;
  return 1;
 }
```

### 2.1.4.3創建 subscriber

接下來，我們創建具有默認QoS的 subscriber 。

```cpp
// Create the subscriber
 DDS::Subscriber_var sub =
  participant->create_subscriber(SUBSCRIBER_QOS_DEFAULT,0, // No listener required OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 if (!sub) {
  std::cerr << "Failed to create_subscriber." << std::endl;
  return 1;
 }
```

### 2.1.4.4創建DataReader和Listener
我們需要一個跟我們創造的 data reader 有關的監聽物件，讓我們可以在有效資料到達時使用。下面是有關監聽物件的程式。'DataReaderListenerImpl class' 在下一小節中顯示。

```cpp
DDS::DataReaderListener_var listener(new DataReaderListenerImpl);
```
聆聽者在堆疊上分配並指派給 'DataReaderListener_var' 物件。此類型提供引用計數行為，因此當刪除最後一個引用時，聆聽者將自動清除。此用法通常用於 OpenDDS 應用程序代碼中的堆分配，並使應用程序開發人員無需主動管理已分配對象的生命週期。
現在可以創造一個跟我們的 topic、預設 QoS 、聆聽物件有關的 Data reader 。

```cpp
 // Create the Datareader
 DDS::DataReader_var dr =
  sub->create_datareader(topic,DATAREADER_QOS_DEFAULT,listener, OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 if (!dr) {
  std::cerr << "create_datareader failed." << std::endl;
  return 1;
 }
```

這個線程現在可以自由地執行其他應用程序工作。當樣本可使用時，聆聽物件將呼叫 OpenDDS 執行序。

## 2.1.5數據讀取器偵聽器實現

我們的監聽器類實現由DDS規範定義的`DDS :: DataReaderListener`接口。 DataReaderListener包裝在一個`DCPS :: LocalObject`中，它解決了諸如`_narrow`和`_ptr_type`之類的模糊繼承成員。 接口定義了我們必須實現的多個操作，每個操作被調用以通知我們不同的事件。`OpenDDS :: DCPS :: DataReaderListener`定義了OpenDDS的特殊需求的操作，例如斷開連接和重新連接的事件更新。 這裡是接口定義：

```cpp
module DDS {
 local interface DataReaderListener : Listener {
  void on_requested_deadline_missed(in DataReader reader,in RequestedDeadlineMissedStatus status);
  void on_requested_incompatible_qos(in DataReader reader,in RequestedIncompatibleQosStatus status);
  void on_sample_rejected(in DataReader reader,in SampleRejectedStatus status);
  void on_liveliness_changed(in DataReader reader,in LivelinessChangedStatus status);
  void on_data_available(in DataReader reader);
  void on_subscription_matched(in DataReader reader,in SubscriptionMatchedStatus status);
  void on_sample_lost(in DataReader reader, in SampleLostStatus status);
 };
};
```

我們的示例偵聽器類使用簡單的打印語句來存儲大多數這些偵聽器操作。 這個例子真正需要的唯一操作是`on_data_available（）`，它是我們需要探索的這個類的唯一成員函數。

```cpp
void DataReaderListenerImpl::on_data_available(DDS::DataReader_ptr reader)
{
 ++num_reads_;
 try {
  Messenger::MessageDataReader_var reader_i =
  Messenger::MessageDataReader::_narrow(reader);
 if (!reader_i) {
  std::cerr << "read: _narrow failed." << std::endl;
  return;
 }
```

上面的代碼將傳遞到偵聽器的通用數據讀取器縮小到特定於類型的MessageDataReader接口。 以下代碼從消息讀取器獲取下一個樣本。 如果獲取成功並返回有效數據，我們打印出每個消息的字段。

```cpp
Messenger::Message message;
 DDS::SampleInfo si ;
 DDS::ReturnCode_t status = reader_i->take_next_sample(message, si) ;
 if (status == DDS::RETCODE_OK) {
  if (si.valid_data == 1) {
   std::cout << "Message: subject = " << message.subject.in() << std::endl
    << " subject_id = " << message.subject_id << std::endl
    << " from = " << message.from.in() << std::endl
    << " count = " << message.count << std::endl
    << " text = " << message.text.in() << std::endl;
  }
  else if (si.instance_state == DDS::NOT_ALIVE_DISPOSED_INSTANCE_STATE)
 {
   std::cout << "instance is disposed" << std::endl;
 }
  else if (si.instance_state == DDS::NOT_ALIVE_NO_WRITERS_INSTANCE_STATE)
 {
   std::cout << "instance is unregistered" << std::endl;
 }
  else
 {
   std::cerr << "ERROR: received unknown instance state "
    << si.instance_state << std::endl;
 }
 } else if (status == DDS::RETCODE_NO_DATA) {
    cerr << "ERROR: reader received DDS::RETCODE_NO_DATA!" << std::endl;
 } else {
    cerr << "ERROR: read Message: Error: " << status << std::endl;
 }
```

請注意，樣本讀取可能包含無效數據。 valid\_data標誌指示樣本是否具有有效數據。有兩個示例，其中無效數據傳遞到偵聽器回調以用於通知。一個是dispose通知，當DataWriter顯式調用`dispose（）`時接收。另一個是未註冊的通知，它在DataWriter顯式調用`unregister（）`時接收。處理通知的實例狀態設置為`NOT_ALIVE_DISPOSED_INSTANCE_STATE`，而註銷通知的實例狀態設置為`NOT_ALIVE_NO_WRITERS_INSTANCE_STATE`。如果有其他樣本可用，服務將再次調用此函數。然而，一次讀取單個樣本的值不是處理傳入數據的最有效的方式。數據讀取器接口提供了許多不同的選項，以更有效的方式處理數據。我們在第2.2節討論這些操作中的一些。

## 2.1.6在OpenDDS客戶端中清除

在發布者和訂閱者完成後，我們可以使用以下代碼清理OpenDDS相關對象：

```cpp
participant->delete_contained_entities();
dpf->delete_participant(participant);
TheServiceParticipant->shutdown ();
```

域參與者的`delete_contained_entities（）`操作將刪除使用該參與者創建的所有主題，訂閱者和發布者。一旦完成，我們可以使用域參與者工廠刪除我們的域參與者。

由於DDS中的數據的發布和訂閱是分離的，如果在已經由訂閱接收到的所有數據之前發布被取消關聯`(shutdown)`，則不能保證傳送數據。如果應用程序要求接收所有發布的數據，則`wait_for_acknowledgements（）`操作可用於允許發布等待直到接收到所有寫入的數據。數據讀取器必須具有RELIABILITY QoS（這是默認值）的RELIABLE設置才能使`wait_for_acknowledgements（）`正常工作。此操作在單個DataWriter上調用，並包括超時值以綁定等待時間。以下代碼說明使用`wait_for_acknowledgements（）`阻止最多15秒鐘等待訂閱確認收到所有寫入數據：

```cpp
DDS::Duration_t shutdown_delay = {15, 0};
 DDS::ReturnCode_t result;
 result = writer->wait_for_acknowledgments(shutdown_delay);
 if( result != DDS::RETCODE_OK) {
 std::cerr << "Failed while waiting for acknowledgment of "
           << "data being received by subscriptions, some data "
           << "may not have been delivered." << std::endl;
 }
```

## 2.1.7運行示例

我們現在準備好運行我們的簡單例子。在自己的窗口中運行這些命令應該使您最容易理解輸出。

首先，我們將啟動一個DCPSInfoRepo服務，以便我們的發布商和訂閱者可以找到另一個。

**注意:**如果通過將環境配置為使用RTPS發現來使用對等發現，則不需要執行此步驟。

DCPSInfoRepo可執行文件位於$ DDS\_ROOT / bin / DCPSInfoRepo中。當我們啟動DCPSInfoRepo時，我們需要確保發布者和訂閱者應用程序進程也可以找到啟動的DCPSInfoRepo。此信息可以通過以下三種方式之一提供：

a。）命令行上的參數。

b。）生成並放置在共享文件中供應用程序使用。

c。）放置在配置文件中的參數，供其他進程使用。

對於我們的簡單示例，我們將使用選項'b'通過將DCPSInfoRepo的位置屬性生成為一個文件，以便我們的簡單發布者和訂閱者可以讀取它並連接到它。

從您當前的目錄類型：

Windows:

```cpp
%DDS_ROOT%\bin\DCPSInfoRepo -o simple.ior
```

Unix:

```cpp
$DDS_ROOT/bin/DCPSInfoRepo -o simple.ior
```

-o參數指示DCPSInfoRepo生成其到文件simple.ior的連接信息，供發布者和訂閱者使用。 在單獨的窗口中導航到包含simple.ior文件的相同目錄，並在我們的示例中通過鍵入以下命令啟動訂閱者應用程序：

Windows:

```cpp
subscriber -DCPSInfoRepo file://simple.ior
```

Unix:

```cpp
./subscriber -DCPSInfoRepo file://simple.ior
```

命令行參數指示應用程序使用指定的文件來定位DCPSInfoRepo。 我們的訂閱者現在正在等待發送郵件，因此我們現在將在具有相同參數的單獨窗口中啟動發布商：

Windows:

```cpp
publisher -DCPSInfoRepo file://simple.ior
```

Unix:

```cpp
./publisher -DCPSInfoRepo file://simple.ior
```

發布者連接到DCPSInfoRepo以查找任何訂戶的位置，並開始發布消息以及將它們寫入控制台。 在訂閱者窗口中，您現在還應該看到來自訂閱者的控制台輸出，該訂閱者正在閱讀主題的消息，演示一個簡單的發布和訂閱應用程序。

您可以閱讀第7.3.3節和第7.4.5.5節中有關為RTPS和其他更高級的配置選項配置應用程序的更多信息。 要了解有關配置和使用DCPSInfoRepo的更多信息，請參閱第7.3節和第9章。有關設置和使用修改應用程序行為的QoS功能的更多信息，請參閱第3章。

## 2.1.8使用RTPS運行我們的示例

之前的OpenDDS示例演示瞭如何使用基本的OpenDDS配置和使用DCPSInfoRepo服務的集中發現構建和執行OpenDDS應用程序。以下詳細描述了使用RTPS運行同一示例進行發現和可互操作傳輸所需要的。這在您的OpenDDS應用程序需要與DDS規範的非OpenDDS實現互操作的情況下，或者如果您不想在您的OpenDDS部署中使用集中式發現，這是非常重要的。

上面的Messenger示例的編碼和構建沒有更改為使用RTPS，因此您不需要修改或重建您的發布商和訂閱服務。這是OpenDDS架構的一個優點，就是為了啟用RTPS功能，它是一個配置練習。第7章將介紹有關所有可用傳輸（包括RTPS）的配置的更多詳細信息，但是，對於本練習，我們將使用發布者和訂戶將共享的配置文件為Messenger示例啟用RTPS。

導航到您的發布商和訂閱者已建立的目錄。創建一個名為rtps.ini的新文本文件，並使用以下內容填充它：

> \[common\]
>
> DCPSGlobalTransportConfig=$file
>
> DCPSDefaultDiscovery=DEFAULT\_RTPS

> \[transport/the\_rtps\_transport\]
>
> transport\_type=rtps\_udp

在接下來的章節中指定了配置文件的更多細節，但是需要調用兩條感興趣的行來將發現方法和數據傳輸協議設置為RTPS。

現在讓我們通過先啟動訂閱者進程然後再通過發布者開始發送數據來重新運行我們的RTPS示例。 最好在單獨的窗口中啟動它們，以分別查看兩個工作。

使用-DCPSConfigFile命令行參數啟動訂戶以指向新創建的配置文件...

Windows: 

```
subscriber -DCPSConfigFile rtps.ini 
```

Unix:

```
./subscriber -DCPSConfigFile rtps.ini
```

**現在使用相同的參數啟動發布商...**

Windows:

```
publisher -DCPSConfigFile rtps.ini 
```

Unix: 

```
./publisher -DCPSConfigFile rtps.ini
```

由於在RTPS規範中沒有集中式發現，因此有允許等待時間允許發現發生的規定。 規格將默認值設置為30秒。 當開始兩個上述過程時，可能有多達30秒的延遲，這取決於它們彼此相隔多遠開始。 此時間可以在稍後討論的第7.3.3節中討論的OpenDDS配置文件中進行調整。

由於OpenDDS的架構允許可插拔發現和可插入傳輸，上面rtps.ini文件中調用的兩個配置條目可以使用RTPS單獨更改，另一個不使用RTPS（例如使用DCPSInfoRepo的集中式發現）。 在我們的示例中將它們都設置為RTPS使得此應用程序與其他非OpenDDS實現完全可互操作。

# 2.2數據處理優化

## 2.2.1在發布服務器中註冊和使用實例

前面的示例隱式地指定它通過樣本的數據字段發布的實例。 當`write()`被調用時，數據寫入器查詢樣本的關鍵字段以確定實例。 發布者還可以通過在數據寫入程序上調用`register_instance()`來顯式註冊實例：

```cpp
Messenger::Message message;
 message.subject_id = 99;
 DDS::InstanceHandle_t handle =
  message_writer->register_instance(message);
```

在填充消息結構之後，我們調用`register_instance()`函數註冊實例。 實例由subject\_id值99標識（因為我們之前將該字段指定為鍵）。
我們稍後可以在發布樣本時使用返回的實例句柄：

```cpp
DDS::ReturnCode_t ret = data_writer->write(message, handle);
```

使用實例句柄發布樣本可能比強制編寫器查詢實例稍微更有效，並且在實例上發布第一個樣本時效率更高。 沒有顯式註冊，第一次寫入會導致OpenDDS為該實例分配資源。

由於資源限制可能導致實例註冊失敗，因此許多應用程序會將註冊視為設置發布者的一部分，並在初始化數據寫入器時始終執行此操作。

## 2.2.2讀取多個樣本

DDS規範提供了用於讀取和寫入數據樣本的多個操作。 在上面的示例中，我們使用了`take_next_sample()`操作來讀取下一個示例，並從讀取器“獲取”它的所有權。 消息數據閱讀器還具有以下操作。

•take\(\) - 從讀取器獲取最多max\_samples個值的序列

•take\_instance\(\) - 為指定實例獲取一系列值

•take\_next\_instance\(\) - 獲取屬於同一實例的一系列樣本，而不指定實例。

還有對應於獲得相同值的這些“獲取”操作中的每一個的“讀取”操作，但將樣本留在讀取器中，並且簡單地在SampleInfo中將它們標記為讀取。

由於這些其他操作讀取一系列值，所以當樣本快速到達時，它們更有效。 這裡是一個示例調用`take()`，一次最多讀取5個樣本。

```cpp
MessageSeq messages(5);
 DDS::SampleInfoSeq sampleInfos(5);
 DDS::ReturnCode_t status =
  message_dr->take(messages, sampleInfos, 5, DDS::ANY_SAMPLE_STATE, DDS::ANY_VIEW_STATE, DDS::ANY_INSTANCE_STATE);
```

三個狀態參數潛在地專門化從讀取器返回哪些樣本。有關其用法的詳細信息，請參閱DDS規範。

 2.2.3零複製讀

返回樣本序列的讀取和獲取操作向用戶提供獲得樣本的副本（單拷貝讀取）或對樣本的引用（零拷貝讀取）的選項。與大型樣品類型的單拷貝讀取相比，零拷貝讀取可以顯著提高性能。測試顯示，使用零拷貝讀取，8KB或更小的樣本不會獲得太多，但在小樣本上使用零拷貝幾乎沒有性能損失。

應用程序開發人員可以通過使用max\_len為零構造的樣本序列調用`take()`或`read()`來指定零拷貝讀取優化的使用。消息序列和样本信息序列構造函數都使用max\_len作為它們的第一個參數，並指定默認值0。以下示例代碼取自DevGuideExamples / DCPS / Messenger\_ZeroCopy / DataReaderListenerImpl.cpp：

```cpp
Messenger::MessageSeq messages;
 DDS::SampleInfoSeq info;
 // get references to the samples (zero-copy read of the samples)
 DDS::ReturnCode_t status = dr->take(messages,info, DDS::LENGTH_UNLIMITED, DDS::ANY_SAMPLE_STATE, DDS::ANY_VIEW_STATE, DDS::ANY_INSTANCE_STATE);
```

在零拷貝讀取/讀取和單拷貝讀取/讀取之後，樣本和信息序列的長度被設置為讀取的樣本數。 對於零拷貝讀操作，**max\_len設置為值&gt; = length。**

由於應用程序代碼要求數據的零拷貝貸款，它必須在完成數據後返回該貸款：

```
dr->return_loan(messages, info);
```

調用`return_loan()`會導致序列的`max_len`設置為0，並且其自己的成員設置為`false`，允許相同的序列用於另一個零拷貝讀取。

如果數據樣本序列構造函數和info序列構造函數的第一個參數更改為大於零的值，則返回的樣本值將是副本。當複制值時，應用程序開發人員可以調用`return_loan()`，但不是必須這樣做。

如果未指定序列構造函數的`max_len(first)`參數，那麼缺省值為0;因此使用零拷貝讀取。因為這個默認值，當它被銷毀時，序列將自動調用`return_loan()`。為了符合DDS規範並且可移植到DDS的其他實現，應用程序不應該依賴於此自動`return_loan()`功能。

樣本和信息序列的第二個參數是序列中可用的最大時隙。如果`read()`或`take()`操作的`max_samples`參數大於此值，則`read()`或`take()`返回的最大樣本將受序列構造函數的此參數限制。

雖然應用程序可以通過調用`length(len)`操作來更改零拷貝序列的長度，但建議不要這樣做，因為此調用會導致複製數據並創建單拷貝樣本序列。



