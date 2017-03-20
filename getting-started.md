# CHAPTER 2

### Getting Started

## 2.1使用DCPS

本章重點介紹使用DCPS將數據從單個發布者進程分發到單個訂戶進程的示例應用程序。 它基於一個簡單的信使應用程序，其中單個發布者發布消息，並且單個訂閱者訂閱它們。 我們使用默認的QoS屬性和默認的TCP / IP傳輸。 此示例的完整源代碼可以在

$ DDS\_ROOT / DevGuideExamples / DCPS / Messenger /目錄。 其他DDS和DCPS功能將在後面的章節中討論。

## 2.1.1定義數據類型

DDS使用的每種數據類型都是使用IDL定義的。 OpenDDS使用\#pragma指令來標識DDS傳輸和處理的數據類型。 這些數據類型由TAO IDL編譯器和OpenDDS IDL編譯器處理，以生成必要的代碼，以使用OpenDDS傳輸這些類型的數據。 這裡是定義我們的消息數據類型的IDL文件：

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

DCPS\_DATA\_TYPE pragma標記用於OpenDDS的數據類型。必須在此pragma偽指令中使用完全作用域類型名稱。 OpenDDS要求數據類型是一個結構。

該結構可以包含標量類型（短，長，浮動等），枚舉，字符串，序列，數組，結構和聯合。此示例定義了Messenger模塊中用於此OpenDDS示例的結構Message。

DCPS\_DATA\_KEY pragma標識用作此類型的鍵的DCPS數據類型的字段。數據類型可以有零個或多個鍵。這些鍵用於標識主題內的不同實例。每個鍵應該是數字或枚舉類型，字符串或這些類型之一的typedef.1 pragma傳遞完全範圍類型和標識該類型的鍵的成員名。使用單獨的DCPS\_DATA\_KEY編譯指示指定多個鍵。在上面的示例中，我們將Messenger :: Message的subject\_id成員標識為鍵。以唯一subject\_id值發布的每個示例將被定義為屬於同一主題中的不同實例。由於我們使用默認QoS策略，具有相同subject\_id值的後續樣本將被視為該實例的替換值。

## 2.1.2處理IDL

OpenDDS IDL首先由TAO IDL編譯器處理。

`tao_idl Messenger.idl`

此外，我們需要使用OpenDDS IDL編譯器處理IDL文件，以生成OpenDDS需要編譯和解密消息的序列化和密鑰支持代碼，以及數據讀取器和寫入器的類型支持代碼。 此IDL編譯器位於$ DDS\_ROOT / bin /中，並為每個處理的IDL文件生成三個文件。 這三個文件都以原始IDL文件名開頭，顯示如下：

•&lt;filename&gt; TypeSupport.idl

1.其他類型（例如結構，序列和數組）不能直接用作鍵，但是當這些成員/元素是數字，枚舉或字符串類型時，結構體或數組元素的單個成員可以用作鍵。

• &lt;filename&gt;TypeSupportImpl.h

• &lt;filename&gt;TypeSupportImpl.cpp

例如，運行opendds\_idl如下

`opendds_idl Messenger.idl`

生成MessengerTypeSupport.idl，MessengerTypeSupportImpl.h和MessengerTypeSupportImpl.cpp。 IDL文件包含MessageTypeSupport，MessageDataWriter和MessageDataReader接口定義。這些是類型特定的DDS接口，我們稍後使用它們向域註冊我們的數據類型，發布該數據類型的樣本，並接收已發布的樣本。實現文件包含這些接口的實現。生成的IDL文件本身應該使用TAO IDL編譯器來編譯，以生成存根和骨架。這些和實現文件應該與使用消息類型的OpenDDS應用程序鏈接。 OpenDDS IDL編譯器有許多選項專門生成生成的代碼。這些選項在第8章中描述。

通常，您不直接調用上面的TAO或OpenDDS IDL編譯器，但讓您的構建環境為您做。通過從dcpsexe\_with\_tcp項目繼承，使用MPC時，簡化了整個過程。這是發布者和訂閱者共同的MPC文件部分

```cpp
project(*idl): dcps {
    // This project ensures the common components get built first.
    TypeSupport_Files {
        Messenger.idl
    }
    custom_only = 1
}
```

dcps父項目添加了類型支持自定義構建規則。 上面的TypeSupport\_Files部分告訴MPC使用OpenDDS IDL編譯器從Messenger.idl生成消息類型支持文件。 這是發布商部分：

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

dcpsexe\_with\_tcp項目鏈接在DCPS庫中。

為了完整性，這裡是MPC文件的訂戶部分：

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

## 2.1.3簡單消息發佈器

在本節中，我們將介紹設置一個簡單的OpenDDS發布過程所涉及的步驟。 代碼被分成邏輯部分，並解釋為我們呈現每個部分。

我們省略了代碼中一些不感興趣的部分（例如\#include指令，錯誤處理和跨進程同步）。 此示例發布者的完整源代碼位於$ DDS\_ROOT / DevGuideExamples / DCPS / Messenger /中的_Publisher.cpp和Writer.cpp文件_中。

### 2.1.3.1初始化參與者

`main（）`的第一部分將當前進程初始化為OpenDDS參與者。

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

`TheParticipantFactoryWithArgs` macro在`Service_Participant.h`中定義，並使用命令行參數初始化域參與者工廠。 這些命令行參數用於初始化OpenDDS服務使用的ORB以及服務本身。 這允許我們在命令行上傳遞`ORB_init（）`選項，以及-DCPS \*格式的OpenDDS配置選項。 可用的OpenDDS選項在第7章中有詳細描述。

`create_participant（）`操作使用域參與者工廠將此進程註冊為由ID 42指定的域中的參與者。參與者使用默認QoS策略和沒有偵聽器。 OpenDDS默認狀態掩碼的使用確保中間件中的所有相關通信狀態改變（例如，可用的數據，活力丟失）被傳送到應用程序（例如，通過收聽者上的回調）。

用戶可以使用範圍（0x0〜0x7FFFFFFF）中的ID定義任意數量的域。 所有其他值保留供實施內部使用。

返回的域參與者對象引用然後用於註冊我們的消息數據類型。

### 2.1.3.2註冊數據類型和創建主題

首先，我們創建一個MessageTypeSupportImpl對象，然後註冊一個類型的類型

名稱使用`register_type（）`操作。 在此示例中，我們使用nil字符串類型名稱註冊該類型，這將使MessageTypeSupport接口存儲庫標識符用作類型名稱。 也可以使用諸如“消息”的特定類型名稱。

```cpp
 Messenger::MessageTypeSupport_var mts =
  new Messenger::MessageTypeSupportImpl();
 if (DDS::RETCODE_OK != mts->register_type(participant, "")) {
  std::cerr << "register_type failed." << std::endl;
  return 1;
 }
```

接下來，我們從類型支持對象獲取註冊的類型名稱，並通過將類型名稱傳遞給`create_topic（）`操作中的參與者來創建主題。

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

我們已經創建了一個名為“電影討論列表”的主題，註冊類型和默認QoS策略。

### 2.1.3.3創建發布服務器

現在，我們已準備好使用默認發布商QoS創建發布商。

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

### 2.1.3.4創建DataWriter並等待訂閱服務器

隨著發布商到位，我們創建數據寫入器。

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

當我們創建數據寫入器時，我們傳遞主題對象引用，默認QoS策略和空偵聽器引用。 我們現在將數據寫入器引用縮小到**MessageDataWriter**對象引用，以便我們可以使用特定於類型的發布操作。

```cpp
Messenger::MessageDataWriter_var message_writer =
 Messenger::MessageDataWriter::_narrow(writer);
```

示例代碼使用條件和等待集，以便發布者等待訂閱者連接並完全初始化。 在這樣的簡單示例中，無法等待訂戶可能導致發布者在訂閱者連接之前發布其樣本。

等待用戶所涉及的基本步驟是：

1）從我們創建的數據寫入器獲取狀態條件

2）在條件中啟用“發布匹配”狀態

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

對於每個循環迭代，調用`write（）`會將消息分發給為我們的主題註冊的所有連接的訂閱者。 由於subject\_id是Message的關鍵字，因此每當subject\_id增加並且`write（）`被調用時，將創建一個新的實例（參見1.1.1.3）。`write（）`的第二個參數指定了我們發布樣例的實例。 它應該傳遞由`register_instance（）`或`DDS :: HANDLE_NIL`返回的句柄。 傳遞`DDS :: HANDLE_NIL`值表示數據寫入程序應通過檢查樣本的鍵來確定實例。 有關在發布期間使用實例句柄的詳細信息，請參見第2.2.1節。

## 2.1.4設置訂閱服務器

訂閱者的許多代碼與我們剛剛完成瀏覽的發布商相同或類似。 我們將快速完成類似的部分，並參考上面的討論細節。 此樣本訂閱程序的完整源代碼位於$ DDS\_ROOT / DevGuideExamples / DCPS / Messenger /中的Subscriber.cpp和DataReaderListener.cpp文件中。

### 2.1.4.1初始化參與者

訂閱者的開頭與發布商相同，因為我們初始化服務並加入我們的域：

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

接下來，我們初始化消息類型和主題。 請注意，如果主題已在此域中使用相同的數據類型和兼容的QoS初始化，create\_topic（）調用將返回與現有主題相對應的引用。 如果在我們的create\_topic（）調用中指定的類型或QoS與現有主題的類型或QoS不匹配，那麼調用將失敗。 還有一個find\_topic（）操作，我們的訂閱者可以使用它來簡單地檢索現有的主題。

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

### 2.1.4.3創建用戶

接下來，我們創建具有默認QoS的訂戶。

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

我們需要將一個監聽器對象與我們創建的數據讀取器相關聯，因此我們可以使用它來檢測數據何時可用。 下面的代碼構造監聽器對象。 DataReaderListenerImpl類在下一小節中顯示。

```cpp
DDS::DataReaderListener_var listener(new DataReaderListenerImpl);
```

偵聽器在堆上分配並分配給DataReaderListener\_var對象。 此類型提供引用計數行為，因此當刪除最後一個引用時，偵聽器將自動清除。 此用法通常用於OpenDDS應用程序代碼中的堆分配，並使應用程序開發人員無需主動管理已分配對象的生命週期。

現在我們可以創建數據讀取器，並將其與我們的主題，默認的QoS屬性和我們剛剛創建的監聽器對象相關聯。

```cpp
 // Create the Datareader
 DDS::DataReader_var dr =
  sub->create_datareader(topic,DATAREADER_QOS_DEFAULT,listener, OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 if (!dr) {
  std::cerr << "create_datareader failed." << std::endl;
  return 1;
 }
```

這個線程現在可以自由地執行其他應用程序工作。 當樣本可用時，我們的偵聽器對象將在OpenDDS線程上調用。

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

在填充消息結構之後，我們調用`register_instance()`函數註冊實例。 實例由subject\_id值99標識（因為我們之前將該字段指定為鍵）。我們稍後可以在發布樣本時使用返回的實例句柄：

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



