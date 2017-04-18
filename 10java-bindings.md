# CHAPTER 10

# Java Bindings

# 10.1 Introduction

OpenDDS提供Java JNI綁定。 Java應用程序可以像C ++應用程序一樣使用完整的OpenDDS中間件。

有關入門的信息，包括先決條件和依賴關係，請參閱$ DDS\_ROOT / java / INSTALL文件。

有關使用Java綁定開發應用程序時遇到的常見問題的信息，請參閱$ DDS\_ROOT / java / FAQ文件。

# 10.2 IDL和代碼生成

OpenDDS Java綁定不僅僅是一個居住在一個或兩個.jar文件中的庫。 DDS規範定義了DDS應用程序與DDS中間件之間的交互。 特別地，DDS應用程序發送和接收強類型的消息，這些類型由IDL中的應用程序開發人員定義。

為了使應用程序根據這些用戶定義的類型與中間件交互，必須在編譯時根據此IDL生成代碼。 生成C ++，Java，甚至一些額外的IDL代碼。 在大多數情況下，應用程序開發人員不需要關心所有生成的文件的詳細信息。 OpenDDS附帶的腳本自動執行此過程，以便最終的結果是一個本地庫（.so或.dll）和一個Java庫（.jar或只是一個classes目錄），它們一起包含所有生成的代碼。

以下是生成的文件和哪些工俱生成它們的描述。 在這個例子中，Foo.idl包含一個包含在模塊Baz（IDL模塊類似於C ++命名空間和Java包）的單個結構。 在每個文件名的右側是生成它的工具的名稱，後面是其目的的一些註釋。

#### 表10-1為生成的文件說明

| File | Generation Tool |
| :---: | :---: |
| Foo.idl | Developer-written description of the DDS sample type |
| Foo{C,S}.{h,inl,cpp} | tao\_idl: C++ representation of the IDL |
| FooTypeSupport.idl | opendds\_idl: DDS type-specific interfaces |
| FooTypeSupport{C,S}.{h,inl,cpp} | tao\_idl |
| Baz/BarSeq{Helper,Holder}.java | idl2jni |
| Baz/BarData{Reader,Writer}\*.java | idl2jni |
| Baz/BarTypeSupport\*.java | idl2jni \(except TypeSupportImpl, see below\) |
| FooTypeSupportJC.{h,cpp} | idl2jni: JNI native method implementations |
| FooTypeSupportImpl.{h,cpp} | opendds\_idl: DDS type-specific C++ impl. |
| Baz/BarTypeSupportImpl.java | opendds\_idl: DDS type-specific Java impl. |
| Baz/Bar\*.java | idl2jni: Java representation of IDL struct |
| FooJC.{h,cpp} | idl2jni: JNI native method implementations |

```cpp
Foo.idl:
module Baz {
#pragma DCPS_DATA_TYPE "Baz::Bar"
 struct Bar {
 long x;
 };
};
```

# 10.3設置OpenDDS Java項目

這些說明假設您已經完成了$ DDS\_ROOT / java / INSTALL文檔中的安裝步驟，包括定義了各種環境變量。

1\) 從一個空目錄開始，將用於您的IDL和從其生成的代碼。 $ DDS\_ROOT / java / tests / messenger / messenger\_idl /這樣設置。

2\) 創建一個描述使用OpenDDS的數據結構的IDL文件。 請參閱Messenger.idl作為示例。 該文件將至少包含一行以“\#pragma DCPS\_DATA\_TYPE”開頭。 為了這些說明，我們將調用文件Foo.idl。

3\) C ++生成的類將被打包在共享庫中，以便在JVM的運行時加載。 這需要導出打包類以獲得外部可見性。 ACE提供了一個用於生成正確的導出宏的實用程序腳本。 腳本用法如下所示：

Unix:

`$ACE_ROOT/bin/generate_export_file.pl Foo > Foo_Export.h`

Windows:

`%ACE_ROOT%\bin\generate_export_file.pl Foo > Foo_Export.h`

4\) 從此模組創建一個MPC文件Foo.mpc：

```cpp
project: dcps_java {
idlflags += -Wb,stub_export_include=Foo_Export.h \
-Wb,stub_export_macro=Foo_Export
dcps_ts_flags += -Wb,export_macro=Foo_Export
idl2jniflags += -Wb,stub_export_include=Foo_Export.h \
-Wb,stub_export_macro=Foo_Export
dynamicflags += FOO_BUILD_DLL
specific {
    jarname = DDS_Foo_types
}
TypeSupport_Files {
    Foo.idl
    }
}
```

如果不需要創建一個jar文件，你可以省略特定的{...}塊。 在這種情況下，您可以直接使用將在當前目錄的classes子目錄下生成的Java .class文件。

5\) 運行MPC生成特定於平台的構建文件。

Unix:

`$ACE_ROOT/bin/mwc.pl -type gnuace`

Windows:

`%ACE_ROOT%\bin\mwc.pl -type [CompilerType]`

CompilerType可以是vc71，vc8，vc9，vc10和nmake

確保這是運行ActiveState Perl。

6\) 編譯生成的C ++和Java代碼

Unix：

make（GNU make，所以這可能是Solaris系統上的“gmake”）

Windows：

使用您首選的方法構建生成的.sln（解決方案）文件。 這可以是Visual Studio IDE或其中一個命令行工具。 如果使用IDE，請使用devenv或vcexpress（Express Edition）從命令提示符啟動它，以便它繼承環境變量。 用於構建的命令行工具包括vcbuild並使用適當的參數調用IDE（devenv或vcexpress）。

成功完成後，您將擁有本機庫和Java .jar文件。 本地圖書館名稱如下：

Unix:

`libFoo.so`

Windows:

`Foo.dll (Release) or Food.dll (Debug)`

您可以通過在Foo.mpc文件中添加如下所示的行來更改這些庫（包括.jar文件）的位置：

`libout = $(PROJECT_ROOT)/lib`

其中PROJECT\_ROOT可以是構建時定義的任何環境變量。

7\) 您現在擁有編譯和運行Java OpenDDS應用程序所需的所有Java和C ++代碼。 生成的.jar文件需要添加到您的類路徑中。 生成的C ++庫需要在運行時加載：

Unix：

將包含libFoo.so的目錄添加到LD\_LIBRARY\_PATH。

Windows：

將包含Foo.dll（或Food.dll）的目錄添加到PATH。 如果您正在使用調試版本（Food.dll），則需要通知OpenDDS中間件它不應該查找Foo.dll。 為此，請將-Dopendds.native.debug = 1添加到Java VM參數。

有關使用OpenDDS Java綁定發布和訂閱應用程序的示例，請參閱$ DDS\_ROOT / java / tests / messenger中的發布者和訂閱者目錄。

8\) 如果對Foo.idl進行後續更改，請先重新運行MPC（上述步驟5）。

這是必需的，因為對Foo.idl的某些更改會影響生成哪些文件並需要編譯。

# 10.4簡單消息發布者

本節介紹一個簡單的OpenDDS Java發布過程。 完整的代碼可以在`$ DDS_ROOT / java / tests / messenger / publisher / TestPublisher.java`中找到。

這裡省略了諸如導入和錯誤處理等不感興趣的細分。 代碼已經分解，並以邏輯子部分進行了說明。

## 10.4.1初始化參與者

通過獲取對參與者工廠的初始引用來啟動DDS應用程序。 調用靜態方法`TheParticipantFactory.WithArgs()`返回一個Factory引用。 這也透明地初始化了C ++參與者工廠。 然後，我們可以為特定域創建參與者。

```cpp
public static void main(String[] args) {
    DomainParticipantFactory dpf =
        TheParticipantFactory.WithArgs(new StringSeqHolder(args));
    if (dpf == null) {
        System.err.println ("Domain Participant Factory not found");
        return;
    }
    final int DOMAIN_ID = 42;
    DomainParticipant dp = dpf.create_participant(DOMAIN_ID,
        PARTICIPANT_QOS_DEFAULT.get(), null, DEFAULT_STATUS_MASK.value);
    if (dp == null) {
        System.err.println ("Domain Participant creation failed");
        return;
    }
```

對象創建失敗由null返回指示。`create_participant()`的第三個參數帶有參與者事件偵聽器。 如果一個不可用，可以通過我們的示例中的代碼來傳遞一個null。

## 10.4.2註冊數據類型並創建主題

接下來，我們使用`register_type()`操作向DomainParticipant註冊我們的數據類型。 我們可以指定一個類型名稱或傳遞一個空字符串。 傳遞一個空字符串表示中間件應該簡單地使用IDL編譯器為該類型生成的標識符。

```cpp
MessageTypeSupportImpl servant = new MessageTypeSupportImpl();
if (servant.register_type(dp, "") != RETCODE_OK.value) {
    System.err.println ("register_type failed");
    return;
}
```

接下來，我們使用支持服務器類型的註冊名稱創建一個主題。

```cpp
Topic top = dp.create_topic("Movie Discussion List",servant.get_type_name(),TOPIC_QOS_DEFAULT.get(), null,DEFAULT_STATUS_MASK.value);
```

現在我們有一個名為“電影討論列表”的主題，註冊的數據類型和預設的QoS策略。

## 10.4.3創建發布者

接下來，我們創建一個發布商：

```cpp
Publisher pub = dp.create_publisher(
PUBLISHER_QOS_DEFAULT.get(),null,DEFAULT_STATUS_MASK.value);
```

## 10.4.4創建一個DataWriter並註冊一個實例

隨著發布商，我們現在可以創建一個DataWriter：

```cpp
DataWriter dw = pub.create_datawriter(
top, DATAWRITER_QOS_DEFAULT.get(), null, DEFAULT_STATUS_MASK.value);
```

DataWriter是針對特定主題的。 對於我們的示例，我們使用默認的DataWriter QoS策略和一個空的DataWriterListener。

接下來，我們將通用DataWriter縮小到特定類型的DataWriter，並註冊我們希望發布的實例。 在我們的數據定義IDL中，我們指定了subject\_id字段作為關鍵字，因此需要使用實例ID填充（我們的示例中為99）:

```cpp
MessageDataWriter mdw = MessageDataWriterHelper.narrow(dw);
Message msg = new Message();
msg.subject_id = 99;
int handle = mdw.register(msg);
```

我們的例子等待任何對等體被初始化和連接。 然後，它會發布一些消息，這些消息在同一個域中分發給此主題的任何訂閱者。

```
msg.from = "OpenDDS-Java";
msg.subject = "Review";
msg.text = "Worst. Movie. Ever.";
msg.count = 0;
int ret = mdw.write(msg, handle);
```

# 10.5設置用戶

用戶的大部分初始化代碼與發布者相同。 用戶需要在同一個域中創建一個參與者，註冊相同的數據類型，並創建相同的命名主題。

```cpp
public static void main(String[] args) {

 DomainParticipantFactory dpf =
  TheParticipantFactory.WithArgs(new StringSeqHolder(args));
 if (dpf == null) {
  System.err.println ("Domain Participant Factory not found");
  return;
 }
 DomainParticipant dp = dpf.create_participant(42,
  PARTICIPANT_QOS_DEFAULT.get(), null, DEFAULT_STATUS_MASK.value);
 if (dp == null) {
  System.err.println("Domain Participant creation failed");
  return;
 }
 MessageTypeSupportImpl servant = new MessageTypeSupportImpl();
 if (servant.register_type(dp, "") != RETCODE_OK.value) {
  System.err.println ("register_type failed");
  return;
 }
 Topic top = dp.create_topic("Movie Discussion List",servant.get_type_name(),TOPIC_QOS_DEFAULT.get(), null,DEFAULT_STATUS_MASK.value);
```

## 10.5.1創建用戶

與發布商一樣，我們創建一個訂閱者：

```
Subscriber sub = dp.create_subscriber(
    SUBSCRIBER_QOS_DEFAULT.get(), null, DEFAULT_STATUS_MASK.value);
```

## 10.5.2創建一個DataReader和監聽器

向中間件提供DataReaderListener是通知數據接收和訪問數據的最簡單的方法。 因此，我們創建一個DataReaderListenerImpl的實例，並將其作為DataReader創建參數傳遞：

```
DataReaderListenerImpl listener = new DataReaderListenerImpl();
    DataReader dr = sub.create_datareader(
        top, DATAREADER_QOS_DEFAULT.get(), listener,
        DEFAULT_STATUS_MASK.value);
```

偵聽器將在中間件的線程中接收任何傳入的消息。 此時應用程序線程可以自由執行其他任務。

# 10.6 DataReader監聽器實現

定義的應用程序DataReaderListenerImpl需要實現規範的DDS.DataReaderListener接口。 OpenDDS提供了一個抽像類DDS.\_DataReaderListenerLocalBase。 應用程序的監聽器類擴展了這個抽像類，並實現了抽象方法來添加應用程序特定的功能。

我們的示例DataReaderListener存儲大部分偵聽器方法。 實現的唯一方法是可從中間件獲取消息回調：

```cpp
public class DataReaderListenerImpl extends DDS._DataReaderListenerLocalBase {

    private int num_reads_;
    public synchronized void on_data_available(DDS.DataReader reader) {
        ++num_reads_;
        MessageDataReader mdr = MessageDataReaderHelper.narrow(reader);
        if (mdr == null) {
            System.err.println ("read: narrow failed.");
            return;
        }
```

監聽器回調傳遞給一個通用DataReader的引用。 應用程序將其縮小到特定於類型的DataReader：

```cpp
MessageHolder mh = new MessageHolder(new Message());
SampleInfoHolder sih = new SampleInfoHolder(new SampleInfo(0, 0, 0,
new DDS.Time_t(), 0, 0, 0, 0, 0, 0, 0, false));
int status = mdr.take_next_sample(mh, sih);
```

然後，它為實際的消息和關聯的SampleInfo創建持有者對象，並從DataReader獲取下一個樣本。 一旦取出，該樣本將從DataReader的可用樣本池中刪除。

```cpp
if (status == RETCODE_OK.value) {
    System.out.println ("SampleInfo.sample_rank = "+ sih.value.sample_rank);
    System.out.println ("SampleInfo.instance_state = "+ sih.value.instance_state);
if (sih.value.valid_data) {
    System.out.println("Message: subject = " + mh.value.subject);
    System.out.println(" subject_id = " + mh.value.subject_id);
    System.out.println(" from = " + mh.value.from);
    System.out.println(" count = " + mh.value.count);
    System.out.println(" text = " + mh.value.text);
    System.out.println("SampleInfo.sample_rank = " + sih.value.sample_rank);
    }
else if (sih.value.instance_state == NOT_ALIVE_DISPOSED_INSTANCE_STATE.value) {
    System.out.println ("instance is disposed");
}
else if (sih.value.instance_state == NOT_ALIVE_NO_WRITERS_INSTANCE_STATE.value) {
    System.out.println ("instance is unregistered");
}
else {
    System.out.println ("DataReaderListenerImpl::on_data_available: "+"received unknown instance state "+ sih.value.instance_state);
}
    } else if (status == RETCODE_NO_DATA.value) {
        System.err.println ("ERROR: reader received DDS::RETCODE_NO_DATA!");
    } else {
        System.err.println ("ERROR: read Message: Error: "+ status);
        }
    }
}
```

SampleInfo包含有關消息的元信息，如消息有效性，實例狀態等。

# 10.7清理OpenDDS Java客戶機

應用程序應該通過以下步驟清理其OpenDDS環境：

```cpp
dp.delete_contained_entities();
```

清理與該參與者相關聯的所有主題，訂閱者和發布商。

```cpp
dpf.delete_participant(dp);
```

DomainParticipantFactory回收與DomainParticipant相關聯的任何資源。

```cpp
TheServiceParticipant.shutdown();
```

關閉ServiceParticipant。 這將清理所有與OpenDDS相關的資源。

清理這些資源是必要的，以防止DCPSInfoRepo在不再存在的端點之間形成關聯。

# 10.8配置舉例

OpenDDS提供基於文件的配置機制。 配置文件的語法與Windows INI文件類似。 屬性分為對應於通用和單獨傳輸配置的命名部分。

Messenger示例具有DCPSInfoRepo對象位置和全局傳輸配置的公共屬性：

```cpp
[common]
DCPSInfoRepo=file://repo.ior
DCPSGlobalTransportConfig=$file
```

以及具有傳輸類型屬性的傳輸實例部分：

```cpp
[transport/1]
transport_type=tcp
```

\[transport / 1\]部分包含名為“1”的傳輸實例的配置信息。 它被定義為tcp類型。 上面的全局傳輸配置設置導致此傳輸實例被該進程中的所有讀寫器使用。

有關所有OpenDDS配置參數的完整說明，請參見第7章。

# 10.9運行例子

要運行Messenger Java OpenDDS應用程序，請使用以下命令：

OpenDDS為JMS版本1.1提供部分支持

&lt;[http://docs.oracle.com/javaee/6/tutorial/doc/bncdq.html](http://docs.oracle.com/javaee/6/tutorial/doc/bncdq.html)&gt;。 企業Java應用程序可以像標準的Java和C ++應用程序一樣使用完整的OpenDDS中間件。

有關開始使用OpenDDS JMS支持的信息，請參閱`$ DDS_ROOT / java / jms /`目錄中的INSTALL文件，包括先決條件和依賴關係。

