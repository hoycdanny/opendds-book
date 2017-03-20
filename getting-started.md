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
  participant->create_topic("Movie Discussion List",type_name,TOPIC_QOS_DEFAULT, 0, // No listener required OpenDDS::DCPS::DEFAULT_STATUS_MASK);
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

```
 // Create the Datareader
 DDS::DataReader_var dr =
  sub->create_datareader(topic,DATAREADER_QOS_DEFAULT,listener, OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 if (!dr) {
  std::cerr << "create_datareader failed." << std::endl;
  return 1;
 }
```





































































































































































