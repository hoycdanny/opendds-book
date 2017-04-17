# CHAPTER 11

# Modeling SDK

OpenDDS Modeling SDK是可以由應用程序使用的建模工具

開發人員將所需的中間件組件和數據結構定義為UML模型，然後生成使用OpenDDS實現模型的代碼。 然後生成的代碼可以被編譯並與應用程序鏈接，以便為應用程序提供無縫的中間件支持。

# 11.1概述

## 11.1.1模組抓取

使用Eclipse插件中包含的圖形模型捕獲編輯器捕獲定義DCPS元素和策略以及數據定義的UML模型。 UML模型的元素遵循DDS規範（OMG：formal / 2015-04-10）中定義的DDS UML平台獨立模型（PIM）的結構。

在插件中打開一個新的OpenDDS模型從頂層主圖開始。

該圖包括要包含在模型中的任何包結構以及模型的本地QoS策略定義，數據定義和DCPS元素。 可以包括零個或多個策略或數據定義元素。 零或一個DCPS元素定義可以包含在任何給定的模型中。

![](/assets/11.1.jpg)

僅為QoS策略創建單獨的模型，僅支持數據定義或僅支持DCPS元素。 對其他模型的引用允許將外部定義的模型包含在模型中。 這允許在不同的DCPS模型之間共享數據定義和QoS策略，以及將外部定義的數據包含在一組新的數據定義中。

![](/assets/11.2.jpg)

## 11.1.2代碼生成

一旦模型被捕獲，源代碼可以從它們生成。 然後可以將該源代碼編譯成鏈接庫，該鏈接庫將模型中定義的中間件元素提供給鏈接庫的應用程序。 代碼生成使用單獨的基於表單的編輯器完成。

代碼生成的具體內容對於各個生成表單是唯一的，並且與執行生成代碼的模型保持分離。 代碼生成一次在單個模型上執行，並且包括定制生成的代碼以及指定用於在構建時定位資源的搜索路徑的能力。

可以生成可在相同應用程序或不同應用程序中創建的模型變體（相同模型的不同定制）。 還可以指定在構建時搜索頭文件和鏈接庫的位置。

詳見11.3.2節“生成代碼”。

## 11.1.3編程

為了使用由模型定義的中間件，應用程序需要鏈接生成的代碼。 這是通過頭文件和鏈接庫完成的。 生成的模型中包含了使用MPC便攜式構建工具構建應用程序的支持。

有關詳細信息，請參見第11.3節“開發應用程序”。

# 11.2安裝和入門

與由開發人員的源代碼編譯的OpenDDS中間件不同，編譯的Modeling SDK可通過Eclipse更新站點下載。

## 11.2.1先決條件

•Java運行時環境（JRE）

* 版本6更新24（1.6.0\_24）是本文撰寫的最新版本

* 從[http://www.java.com](http://www.java.com)下載

•Eclipse IDE

* 4.4版“Luna”，4.4.1是本文的最新版本

* 下載  
  網址[https://eclipse.org/downloads/packages/release/Luna/SR1A](https://eclipse.org/downloads/packages/release/Luna/SR1A)

## 11.2.2安裝

1）從Eclipse中，打開幫助菜單，選擇安裝新軟件。

![](/assets/11.3.jpg)

9\) 單擊可用軟件站點的超鏈接。

10\) 應啟用標準的eclipse.org網站\(Eclipse Project Updates和Galileo\)。 如果它們被禁用，請立即啟用它們。

11\) 添加一個名為OpenDDS的新站點條目，URL為[http://www.opendds.org/modeling/eclipse\_44      
](http://www.opendds.org/modeling/eclipse_44)

12\) 單擊“確定”關閉“首選項”對話框並返回到“安裝”對話框。

13\) 在“使用”組合框中，選擇OpenDDS的新條目。

14\) 選擇“OpenDDS建模SDK”，然後單擊“下一步”。

15\) 查看“安裝詳細信息”列表，然後單擊下一步。 查看許可證，選擇接受（如果您接受），然後單擊完成。

16\) Eclipse將從他們依賴的eclipse.org下載OpenDDS插件和各種插件。 將出現安全警告，因為OpenDDS插件未簽名。 也可能會提示接受eclipse.org的證書。

17\) Eclipse將提示用戶重新啟動以便使用新安裝的軟件。

## 11.2.3入門指南

OpenDDS Modeling SDK包含一個Eclipse Perspective。 通過轉到窗口菜單並選擇Open Perspective - &gt; Other - &gt; OpenDDS Modeling打開它。

要開始使用OpenDDS Modeling SDK，請參閱Eclipse中安裝的幫助內容。

首先轉到幫助菜單並選擇幫助內容。 “OpenDDS建模SDK指南”有一個頂級項目，其中包含描述建模和代碼生成活動的所有OpenDDS特定內容。

# 11.3開發應用程序

為了使用OpenDDS建模SDK構建應用程序，必須了解一些關鍵概念。 概念關注：

1）支持的資料庫庫

2）生成的模組代碼

3）申請代碼

## 11.3.1建立模組支援資料庫

OpenDDS Modeling SDK包括一個支持庫，位於`$ DDS_ROOT / tools / modeling / codegen / model`中。 該支持庫與Modeling SDK生成的代碼相結合，大大減少了構建OpenDDS應用程序所需的代碼量。

支持庫是由OpenDDS Modeling SDK應用程序使用的C ++庫。 大多數開發人員需要的支持庫中有兩個類是Application和Service類。

### 11.3.1.1應用類

OpenDDS :: Model :: Application類負責OpenDDS庫的初始化和最終化。 任何使用OpenDDS的應用程序都需要實例化Application類的單個實例，並且還可以在使用OpenDDS進行通信時不會銷毀Application對象。

Application類初始化用於創建OpenDDS參與者的工廠。 該工廠需要用戶提供的命令行參數。 為了提供它們，Application對象必須提供相同的命令行參數。

### 11.3.1.2服務類

OpenDDS :: Model :: Service類負責在OpenDDS Modeling SDK模型中描述的OpenDDS實體的創建。 由於模型可以是通用的，描述比單個應用程序使用的更廣泛的域，Service類使用惰性實例來創建OpenDDS實體。

為了正確實例化這些實體，它必須知道：

•實體之間的關係

•實體使用的傳輸配置

## 11.3.2生成代碼

OpenDDS Modeling SDK生成特定於模型的代碼供OpenDDS Modeling SDK應用程序使用。用.codegen文件開始（這指的是.opendds模型文件），在表11-1中描述的文件。 生成代碼的過程記錄在Eclipse幫助中。

#### 表11-1為生成的文件

| File Name | Description |
| :---: | :---: |
| &lt;ModelName&gt;.idl | Data types from the model’s DataLib |
| &lt;ModelName&gt;\_T.h | C++ class from the model’s DcpsLib |
| &lt;ModelName&gt;\_T.cpp | C++ implementation of the model’s DcpsLib |
| &lt;ModelName&gt;.mpc | MPC project file for the generated C++ library |
| &lt;ModelName&gt;.mpb | MPC base project for use by the application |
| &lt;ModelName&gt;\_paths.mpb | MPC base project with paths, see section 11.3.3.7 |
| &lt;ModelName&gt;Traits.h | Transport configuration from the .codegen file |
| &lt;ModelName&gt;Traits.cpp | Transport configuration from the .codegen file |

### 11.3.2.1 DCPS模組類

DCPS庫模擬DDS實體之間的關係，包括主題，DomainParticipants，Publishers，Subscriber，DataWriters和DataReaders以及它們對應的域。

對於模型中的每個DCPS庫，OpenDDS Modeling SDK生成一個以DCPS庫命名的類。該DCPS模型類以DCPS庫命名，並在代碼生成目標目錄中的&lt;ModelName&gt; \_T.h文件中找到。

模型類包含一個內部類，名為Elements，定義了庫中建模的每個DCPS實體的枚舉標識符，以及由庫的主題引用的每個類型。此Elements類包含以下各項的枚舉定義：

•DomainParticipants

•類型

•主題

•內容過濾主題

•多主題

•發行商

•訂閱者

•數據作者

•數據讀取器

此外，DCPS模型類捕獲這些實體之間的關係。這些

實例化DCPS實體時，Service類使用關係。

### 11.3.2.2特徵類

DCPS模型中的實體按名稱引用其傳輸配置。 Codegen文件編輯器的“模型自定義”選項卡用於定義每個名稱的傳輸配置。

可以為特定代碼生成文件定義多個配置集。

這些配置組合分為實例，每個實例由一個名稱標識。 可以定義多個實例，代表使用應用程序的模型的不同部署場景。

對於每個這些實例，都會生成一個Traits類。 traits類提供了Codegen編輯器中針對特定傳輸配置名稱建模的傳輸配置。

### 11.3.2.3服務類型

該服務是一個模板，需要兩個參數：（1）實體模型，在DCPS模型中的Elements類，（2）傳輸配置，在一個Traits類中。 OpenDDS建模SDK為DCPS庫和傳輸配置模型實例的每個組合生成一個typedef。 typedef被命名

&lt;InstanceName&gt; &lt;DCPSLibraryName&gt;類型。

### 11.3.2.4數據庫生成代碼

從數據庫中生成IDL，由IDL編譯器處理。 IDL編譯器生成類型支持代碼，用於對數據類型進行序列化和反序列化。

### 11.3.2.5 QoS策略庫生成的代碼

沒有從QoS策略庫生成的特定編譯單元。 相反，DCPS庫存儲其模型實體的QoS策略。 該QoS策略隨後被Service類查詢，該類在實體創建時設置QoS策略。

## 11.3.3申請代碼要求

### 11.3.3.1所需標題

除了Tcp.h頭（用於靜態鏈接）外，應用程序還需要包含Traits頭。 這些將包括構建發布應用程序所需的一切。 以下是一個示例發布應用程序的最後的\#include部分，MinimalPublisher.cpp。

```cpp
#ifdef ACE_AS_STATIC_LIBS
#include <dds/DCPS/transport/tcp/Tcp.h>
#endif
#include "model/MinimalTraits.h"
```

### 11.3.3.2異常處理

建議使用Modeling SDK應用程序捕獲CORBA :: Exception對象和std :: exception對象。

```cpp
int ACE_TMAIN(int argc, ACE_TCHAR* argv[])
{
    try {
    // Create and use OpenDDS Modeling SDK (see sections below)
    } catch (const CORBA::Exception& e) {
    // Handle exception and return non-zero
    } catch (const OpenDDS::DCPS::Transport::Exception& te) {
    // Handle exception and return non-zero
    } catch (const std::exception& ex) {
    // Handle exception and return non-zero
    }
    return 0;
}
```

### 11.3.3.3實例化

如上所述，OpenDDS Modeling SDK應用程序必須在其生命週期內創建一個OpenDDS :: Model :: Application對象。 這個Application對象又被傳遞給由traits頭中的一個typedef聲明指定的Service對象的構造函數。

然後，該服務用於創建OpenDDS實體。 要使用Elements類中指定的枚舉標識符指定要創建的特定實體。 該服務為實體創建提供了此接口：

```cpp
DDS::DomainParticipant_var participant(Elements::Participants::Values part);
DDS::TopicDescription_var topic(Elements::Participants::Values part,
Elements::Topics::Values topic);
DDS::Publisher_var publisher(Elements::Publishers::Values publisher);
DDS::Subscriber_var subscriber(Elements::Subscribers::Values subscriber);
DDS::DataWriter_var writer(Elements::DataWriters::Values writer);
DDS::DataReader_var reader(Elements::DataReaders::Values reader);
```

需要注意的是，必要時，服務還會創建任何所需的中間實體，例如DomainParticipants，Publishers，Subscribers和Topics。

### 11.3.3.4發行人代碼

使用上面所示的writer\(\)方法，MinimalPublisher.cpp繼續：

```cpp
int ACE_TMAIN(int argc, ACE_TCHAR* argv[])
{
    try {
    OpenDDS::Model::Application application(argc, argv);
    MinimalLib::DefaultMinimalType model(application, argc, argv);
    using OpenDDS::Model::MinimalLib::Elements;
    DDS::DataWriter_var writer = model.writer(Elements::DataWriters::writer);
```

剩下的是將DataWriter縮小到類型特定的數據寫入器，並發送樣本。

```cpp
data1::MessageDataWriter_var msg_writer =
    data1::MessageDataWriter::_narrow(writer);
data1::Message message;
// Populate message and send
message.text = "Worst. Movie. Ever.";
DDS::ReturnCode_t error = msg_writer->write(message, DDS::HANDLE_NIL);
if (error != DDS::RETCODE_OK) {
    // Handle error
}
```

整體來說，我們的發布應用程序MinimalPublisher.cpp看起來像這樣：

```cpp
#ifdef ACE_AS_STATIC_LIBS
#include <dds/DCPS/transport/tcp/Tcp.h>
#endif
#include "model/MinimalTraits.h"
int ACE_TMAIN(int argc, ACE_TCHAR* argv[])
{
    try {
        OpenDDS::Model::Application application(argc, argv);
        MinimalLib::DefaultMinimalType model(application, argc, argv);
        using OpenDDS::Model::MinimalLib::Elements;
        DDS::DataWriter_var writer = model.writer(Elements::DataWriters::writer);
        data1::MessageDataWriter_var msg_writer =
            data1::MessageDataWriter::_narrow(writer);
        data1::Message message;
        // Populate message and send
        message.text = "Worst. Movie. Ever.";
        DDS::ReturnCode_t error = msg_writer->write(message, DDS::HANDLE_NIL);
        if (error != DDS::RETCODE_OK) {
            // Handle error
        }
        } catch (const CORBA::Exception& e) {
        // Handle exception and return non-zero
        } catch (const std::exception& ex) {
        // Handle exception and return non-zero
}
return 0;
}
```

請注意，此最小範例忽略日誌記錄和同步，這些問題不是OpenDDS Modeling SDK特有的。

### 11.3.3.5用戶代碼

訂閱者代碼很像發布商。 為了簡單起見，OpenDDS Modeling SDK訂閱者可能希望利用名為OpenDDS :: Modeling :: NullReaderListener的Reader Listener的基類。 NullReaderListener實現整個DataReaderListener接口並記錄每個回調。

訂閱者可以通過從NullReaderListener派生一個類並重寫感興趣的接口，例如on\_data\_available來創建一個監聽器。

```cpp
#ifdef ACE_AS_STATIC_LIBS
#include <dds/DCPS/transport/tcp/Tcp.h>
#endif
#include "model/MinimalTraits.h"
#include <model/NullReaderListener.h>
class ReaderListener : public OpenDDS::Model::NullReaderListener {
public:
    virtual void on_data_available(DDS::DataReader_ptr reader)
        ACE_THROW_SPEC((CORBA::SystemException)) {
    data1::MessageDataReader_var reader_i =
            data1::MessageDataReader::_narrow(reader);
    if (!reader_i) {
        // Handle error
        ACE_OS::exit(-1);
    }
    data1::Message msg;
    DDS::SampleInfo info;
    // Read until no more messages
    while (true) {
    DDS::ReturnCode_t error = reader_i->take_next_sample(msg, info);
    if (error == DDS::RETCODE_OK) {
        if (info.valid_data) {
            std::cout << "Message: " << msg.text.in() << std::endl;
    }
        } else {
            if (error != DDS::RETCODE_NO_DATA) {
            // Handle error
            }
            break;
            }
        }
    }
};
```

在主要功能中，從服務對象創建數據讀取器：

```cpp
DDS::DataReader_var reader = model.reader(Elements::DataReaders::reader);
```

當然，DataReaderListener必須與數據讀取器相關聯才能獲得它的回傳。

```cpp
DDS::DataReaderListener_var listener(new ReaderListener);
reader->set_listener(listener, OpenDDS::DCPS::DEFAULT_STATUS_MASK);
```

剩餘的用戶代碼與任何OpenDDS建模SDK應用程序的要求相同，因為它必須通過OpenDDS :: Modeling :: Application對像初始化OpenDDS庫，並使用適當的DCPS模型Elements類和traits類創建一個Service對象。

下面是一個例子訂閱應用程序MinimalSubscriber.cpp。

```cpp
#ifdef ACE_AS_STATIC_LIBS
#include <dds/DCPS/transport/tcp/Tcp.h>
#endif
#include "model/MinimalTraits.h"
#include <model/NullReaderListener.h>
class ReaderListener : public OpenDDS::Model::NullReaderListener {
public:
    virtual void on_data_available(DDS::DataReader_ptr reader)
                ACE_THROW_SPEC((CORBA::SystemException)) {
        data1::MessageDataReader_var reader_i =
            data1::MessageDataReader::_narrow(reader);
        if (!reader_i) {
            // Handle error
            ACE_OS::exit(-1);
        }
        data1::Message msg;
        DDS::SampleInfo info;
        // Read until no more messages
        while (true) {
            DDS::ReturnCode_t error = reader_i->take_next_sample(msg, info);
            if (error == DDS::RETCODE_OK) {
                if (info.valid_data) {
                    std::cout << "Message: " << msg.text.in() << std::endl;
            }
                } else {
                if (error != DDS::RETCODE_NO_DATA) {
                // Handle error
                }
                break;
            }
        }
    }
};

int ACE_TMAIN(int argc, ACE_TCHAR* argv[])
{
try {
    OpenDDS::Model::Application application(argc, argv);
    MinimalLib::DefaultMinimalType model(application, argc, argv);
    using OpenDDS::Model::MinimalLib::Elements;
    DDS::DataReader_var reader = model.reader(Elements::DataReaders::reader);
    DDS::DataReaderListener_var listener(new ReaderListener);
    reader->set_listener(listener, OpenDDS::DCPS::DEFAULT_STATUS_MASK);
    // Call on_data_available in case there are samples which are waiting
    listener->on_data_available(reader);
    // At this point the application can wait for an exteral “stop” indication
    // such as blocking until the user terminates the program with Ctrl-C.
    } catch (const CORBA::Exception& e) {
    e._tao_print_exception("Exception caught in main():");
    return -1;
    } catch (const std::exception& ex) {
    // Handle error
        return -1;
    }
    return 0;
}
```

### 11.3.3.6 MPC項目

為了利用OpenDDS Modeling SDK支持庫，OpenDDS Modeling SDK MPC項目應該從dds\_model項目基礎繼承。 這是除了非建模SDK項目繼承的dcpsexe基礎之外的。

```cpp
project(*Publisher) : dcpsexe, dds_model {
// project configuration
}
```

生成的模型庫將在目標目錄中生成MPC項目文件和基礎項目文件，並負責構建模型共享庫。 OpenDDS建模應用程序必須（1）在其構建中包括生成的模型庫，（2）確保在生成的模型庫後構建其項目。

```cpp
project(*Publisher) : dcpsexe, dds_model {
    // project configuration
    libs += Minimal
    after += Minimal
}
```

這兩個都可以通過從模組資料庫的項目基礎繼承，以模組資料庫命名。

```cpp
project(*Publisher) : dcpsexe, dds_model, Minimal {
// project configuration
}
```

請注意，在創建項目文件期間，MPC現在可以找到Minimal.mpb文件。 這可以通過--include命令行選項來實現。

使用任一形式，MPC文件必須告訴構建系統在哪裡查找生成的模型庫。

```cpp
project(*Publisher) : dcpsexe, dds_model, Minimal {
    // project configuration
    libpaths += model
}
```

此設置基於Codegen文件編輯器中提供給目標文件夾設置的內容。

最後，像其他MPC項目一樣，它的源文件必須包括在內：

```cpp
Source_Files {
    MinimalPublisher.cpp
}
```

最終的MPC項目對於出版商來說是這樣的：

```cpp
project(*Publisher) : dcpsexe, dds_model, Minimal {
    exename = publisher
    libpaths += model
    Source_Files {
    MinimalPublisher.cpp
    }
}
```

對於訂閱者也是如此：

```cpp
project(*Subscriber) : dcpsexe, dds_model, Minimal {
exename = subscriber
libpaths += model
Source_Files {
MinimalSubscriber.cpp
}
}
```

### 11.3.3.7模組之間的依賴關係

一個最後的考慮 - 生成的模型庫本身可以依賴於其他生成的模型庫。 例如，可能會有一個外部數據類型庫被生成到不同的目錄。

這種可能性可能會導致大量的項目文件維護，隨著模型隨著時間的推移而改變其依賴性。 為了幫助克服這個負擔，生成的模型庫在一個名為&lt;ModelName&gt; \_paths.mpb的單獨的MPB文件中記錄了所有外部引用的模型庫的路徑。 從這個路徑繼承基礎項目將繼承所需的設置以包括依賴模型。

我們的完整MPC文件如下所示：

```cpp
project(*Publisher) : dcpsexe, dds_model, Minimal, Minimal_paths {
    exename = publisher
    libpaths += model
    Source_Files {
    MinimalPublisher.cpp
    }
}
    project(*Subscriber) : dcpsexe, dds_model, Minimal, Minimal_paths {
    exename = subscriber
    libpaths += model
    Source_Files {
    MinimalSubscriber.cpp
}
```



