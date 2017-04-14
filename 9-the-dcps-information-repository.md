# CHAPTER 9

# The DCPS Information Repository

# 9.1 DCPS Information Repository Options

下表顯示了DCPSInfoRepo服務器的命令行選項：

#### 表9-1 DCPS信息庫選項

| Option | Description | Default |
| :---: | :---: | :---: |
| -o file | Write the IOR of the DCPSInfo object to the specified file | repo.ior |
| -NOBITS | Disable the publication of built-in topics | Built-in topics are published |
| -a address | Listening address for built-in topics \(when built-in topics are published\). | Random port |
| -z | Turn on verbose transport logging | Minimal transport logging. |
| -r | Resurrect from persistent file | 1\(true\) |
| -FederationId &lt;id&gt; | Unique identifier for this repository within any federation.This is supplied as a 32 bit decimal numeric value. | N/A |
| -FederateWith &lt;ref&gt; | Repository federation reference at which to join a federation.                           This is supplied as a valid CORBA object reference in string form: stringified IOR, file: or corbaloc: reference string. | N/A |
| -? | Display the command line usage and exit | N/A |

OpenDDS客戶端通常使用DCPSInfoRepo輸出的IOR文件來定位服務。 -o選項允許您將IOR文件放入特定於應用程序的目錄或文件名中。

該文件可以隨後由具有以下文件的客戶機使用：// IOR prefix.Applications不使用內置主題可能需要使用-NOBITS禁用它，以減少服務器上的負載。如果要發佈內置主題，那麼-a選項可以選擇用於這些主題的tcp傳輸的監聽地址。使用-z選項可以調用許多傳輸級調試消息。此選項僅在定義了DCPS\_TRANS\_VERBOSE\_DEBUG環境變量構建DCPS庫時有效。

-FederationId和-FederateWith選項用於將多個DCPSInfoRepo服務器的聯合控製到單個邏輯存儲庫中。有關聯合功能的說明以及如何使用這些選項，請參閱1.2。

文件持久性實現為ACE服務對象，並通過服務配置指令進行控制。目前可用的配置選項有：

#### 表9-2 為InfoRepo持久性指令

| Options | Description | Defaults |
| :---: | :---: | :---: |
| -file | Name of the persistent file | InforepoPersist |
| -reset | Wipe out old persistent data. | 0 \(false\) |

以下指令：

`static PersistenceUpdater_Static_Service "-file info.pr -reset 1"`

將持續DCPSInfoRepo更新到本地文件info.pr. 如果該名稱的文件已經存在，它的內容將被刪除。 用於與命令行選項-r，所述DCPSInfoRepo可以轉世到先前的狀態。 當使用持久性，使用具有下面的命令行選項一個TCP固定端口號開始DCPSInfoRepo過程。 這使得現有客戶端重新連接到重新啟動的InfoRepo。

`-ORBListenEndpoints iiop://:<port>`

# 9.2資料庫聯盟

**注意:**_資料庫聯盟應該被認為是一個實驗性的功能。_

庫聯合會允許多個華府公立學校信息資源庫服務器能夠彼此成一個單一的聯合服務合作。 這使得應用程序從一個資料庫獲取服務元數據和事件從另一個獲得他們，如果原來的庫不再可用。

而創造這個功能的動機是為了提供容錯到DDS服務元數據的措施的能力，其他的用例可以從這個特性中受益。 這包括最初獨立的系統，成為聯盟並獲得原本未到達的應用程序之間傳遞數據的能力的能力。 這樣的一個例子將包括已經獨立地建立的應用程序之間傳送數據的內部DDS服務兩個平台; 在操作期間的某些點上系統變得可到達，彼此建立聯盟儲存庫允許數據在不同的平台上的應用程序之間通過。

在OpenDDS目前的聯合功能只提供在應用程序和存儲庫啟動靜態指定庫的聯盟的能力。 有一種機制來動態地發現和加入聯盟計劃在未來OpenDDS釋放。

OpenDDS通過使用服務策略的活力質量自動檢測庫損失的內置主題。 當使用聯邦，活潑QoS策略被修改到一個非無窮大的值。 當熱鬧是失去了一個內置主題的應用程序將啟動故障轉移序列，使其具有不同的存儲庫服務器相關聯。

由於聯盟目前實行使用內置的主題`ParticipantDataDataReaderListener`實體，應用程序不應安裝自己的聽眾對這個話題。 這樣做會影響到聯合實施的能力來檢測庫故障。

該聯合會執行使用保留DDS域聯盟內分發庫數據。 用於聯合默認域由恆定聯合器定義:: `DEFAULT_FEDERATIONDOMAIN`。

目前，只有聯盟拓撲的靜態指定是可用的。 這意味著每個DCPS信息儲存庫，以及使用聯合DDS服務的每個應用程序，需要包括聯盟配置作為其配置數據的一部分。 這是通過指定在聯合到每個參與的過程中的每個可用的存儲庫，並如第7.3.2.1中所述分配每個存儲庫，以在配置文件中不同的密鑰值來完成。

每個應用程序和存儲庫必須包括同一組中其配置信息庫。 故障轉移測序將試圖到達下一個資料庫我的數字序列庫鍵值（從最後包裝到第一）。 該序列是唯一的配置的每個應用程序，並且應該是不同的，以避免過載的任何個人信息庫。

一旦拓撲信息已被指定，那麼庫將需要有兩個額外的命令行參數來啟動。 這些示於表9-1。 之一，-FederationId &lt;值&gt;，指定了聯邦內的儲存庫的唯一標識符。 這是一個32位的數值，需要對所有可能的聯合拓撲結構獨特。

所需要的第二個命令行參數是`-FederateWith <REF>`。 這將導致儲存庫在初始化後的&lt;REF&gt;對象引用，並從應用程序接受連接之前加入聯邦.

只有那些開始與聯邦標識號倉庫可參加的聯盟。 第一個庫開始不應該給`-FederateWith`命令行指令。 所有的人都需要有這樣的指令，以建立初始聯合會。 有提供的命令行工具（聯合會），可用於建立社團聯會，如果這不是在啟動時完成。 見說明第9.2.1節。 這是可能的，與當前的靜態中的實現，一個倉庫的聯合拓撲之前未能完全建立可能會導致部分不可用的服務。 由於此電流限制，強烈建議始終建立資料庫的聯合拓撲啟動應用程序之前。

## 9.2.1聯合管理

新的命令行工具已經提供了允許庫聯盟的一些最低限度的運行時管理。 這個工具允許倉庫開始沒有`-FederateWith`選項被命令參加的聯盟。 由於聯合存儲庫和故障切換排序的操作依賴於連接的拓撲結構的存在，建議，該工具開始將使用聯合組庫的應用之前使用。

該命令被命名為repoctl和位於`$ DDS_ROOT / bin /`目錄。 它的命令格式語法：

`repoctl <cmd> <arguments>`

其中每個單獨的命令都有自己的格式如表9-3所示。 某些選項包含端點信息。 該信息由一個可選的主機規範，用冒號從所需端口規範隔開。 此端點信息被用來創建使用`corbaloc CORBA`對象引用：在以找到存儲服務器的“聯合器”對象語法。

#### 表9-3為repoctl庫管理命令

| Command | Syntax | Description |
| :---: | :---: | :---: |
| join | repoctl join &lt;target&gt; &lt;peer&gt;                   \[ &lt;federation domain&gt; \] | Calls the &lt;peer&gt; to join &lt;target&gt; to the federation.                                              &lt;federation domain&gt; is passed if present, or the default Federation Domain value is passed. |
| leave | repoctl leave &lt;target&gt; | Causes the &lt;target&gt; to gracefully leave the federation, removing all managed associations between applications using &lt;target&gt; as a repository with applications that are not using &lt;target&gt; as a repository. |
| shutdown | repoctl shutdown &lt;target&gt; | Causes the &lt;target&gt; to shutdown without removing any managed associations. This is the same effect as a repository which has crashed during operation. |
| kill | repoctl kill &lt;target&gt; | Kills the &lt;target&gt; repository regardless of its federation status. |
| help | repoctl help  | Prints a usage message and quits. |

`repoctl join 2112 otherhost:1812`

這產生`corbaloc :: OTHERHOST`的`CORBA`對象引用`：1812 /`了聯邦者連接到和調用連接操作聯合器。 該加入操作調用傳遞默認聯邦域值（因為我們沒有指定一個），其通過解析對象引用`corbaloc ::localhost`獲得的接合庫的位置：2112 /聯合器。

命令參數的完整說明示於表9-4。

#### 表9-4為聯合會管理命令參數

| Option  | Description |
| :---: | :---: |
| &lt;target&gt; | This is endpoint information that can be used to locate the Federator::Manager CORBA interface of a repository which is used to manage federation behavior. This is used to command leave and shutdown federation operations and to identify the joining repository for the join command. |
| &lt;peer&gt; | This is endpoint information that can be used to locate the Federator::Manager CORBA interface of a repository which is used to manage federation behavior. This is used to command join federation operations. |
| &lt;federation domain&gt; | This is the domain specification used by federation participants to distribute service metadata amongst the federated repositories. This only needs to be specified if more than one federation exists among the same set of repositories, which is currently not supported. The default domain is sufficient for single federations. |

## 9.2.2實施例聯合會

為了說明安裝和使用聯邦的，本節走過一種建立聯盟和一個使用它的工作服務一個簡單的例子。

這個例子是基於兩庫聯合，與被配置為使用聯合存儲庫的簡單的消息發布者和訂閱從2.1。

### 9.2.2.1配置聯合例

有兩個配置文件來創建本實施例中每一個用於該消息發布者和訂閱之一。

這個示例的信息發布服務器配置pub.ini如下：

```cpp
 [common]
 DCPSDebugLevel=0
 [domain/information]
 DomainId=42
 DomainRepoKey=1
 [repository/primary]
 RepositoryKey=1
 RepositoryIor=corbaloc::localhost:2112/InfoRepo
 [repository/secondary]
 RepositoryKey=2
 RepositoryIor=file://repo.ior
```

注意，DCPSInfo屬性/值對已經從\[共同\]部分刪去。

如7.5所描述這已被替換為\[域/用戶\]部分。 用戶域是42，從而使結構域被配置為使用對服務元數據和事件的主存儲庫。

的\[存儲庫/初級\]和\[庫/次級\]節定義的一級和二級存儲庫來（兩個儲存的）的聯盟內使用這種應用。 所述RepositoryKey屬性是用於唯一地標識存儲庫（並允許域與它相關聯，如在前面的\[域/信息\]部分）內部密鑰值。 該RepositoryIor屬性包含解析對象引用的字符串值達到指定的存儲庫。的主存儲庫是在本地主機的端口2112引用並預計經由TAO IORTable可用與/ InfoRepo的對象名。 二級庫，預計將提供通過在本地目錄中名為repo.ior文件的IOR值。

訂閱者過程被配置為與文件sub.ini如下：

```cpp
 [common]
 DCPSDebugLevel=0
 [domain/information]
 DomainId=42
 DomainRepoKey=1
 [repository/primary]
 RepositoryKey=1
 RepositoryIor=file://repo.ior
 [repository/secondary]
 RepositoryKey=2
 RepositoryIor=corbaloc::localhost:2112/InfoRepo
```

注意，這是相同，除了該用戶的pub.ini文件已經指定位於所述本地主機的端口2112的存儲庫是二次和位於由repo.ior文件的庫是主。 這是分配的出版商的對面。 這意味著，當該用戶正在使用由位於該文件中包含的IOR庫開始發布商使用的倉庫處的元數據和事件端口2112開始。 在每一種情況下，如果庫被檢測為不可用的應用程序將嘗試使用其他資料庫，如果能夠達成。

該庫不需要為了參加聯盟任何特殊的配置規格，因此沒有文件需要他們在這個例子中。

### 9.2.2.2運行聯邦實施例

例子是通過首先啟動存儲庫和建立聯盟它們，然後啟動應用程序發布者和訂閱處理方式如在第2.1.7節的例子做同樣執行。

開始第一個資源庫為：

```cpp
$DDS/bin/DCPSInfoRepo -ORBSvcConf tcp.conf -o repo.ior -FederationId 1024
```

該-o repo.ior選項確保庫IOR將被放入該文件作為

通過配置文件的預期。 該-FederationId 1024選項值1024分配到這個倉庫作為聯盟中唯一的ID。 該-ORBSvcConf tcp.conf選項是一樣的前面的示例所示。

啟動第二個資料庫為：

```
$DDS/bin/DCPSInfoRepo -ORBSvcConf tcp.conf -ORBListenEndpoints iiop://localhost:2112 -FederationId 2048 -FederateWith file://repo.ior
```

請注意，這是所有打算成為一個單一的命令行上。 該-ORBSvcConf tcp.conf選項是一樣的前面的示例所示。 該-ORBListenEndpoints IIOP：//本地主機：2112選項確保存儲庫將是以前的配置文件都期待在端口上偵聽。 該-FederationId 2048選項指定為聯盟內的存儲庫唯一的ID值2048。 該FederateWith文件：//repo.ior選項發起聯盟與位於包含指定的文件中的IOR庫 - 寫由先前啟動庫。

一旦庫已啟動，聯盟已經建立（這將自動第二庫已初始化後進行），應用程序發布和訂閱過程可以開始，因為他們沒有在第2.1.7節前面的例子應該執行。

