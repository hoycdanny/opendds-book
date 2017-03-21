# CHAPTER 5 

# Content-Subscription Profile

# 5.1介紹

DDS的內容訂閱簡檔由三個特徵組成，這三個特徵使得數據讀取器的行為能夠受其接收的數據樣本的內容的影響。這三個特點是：

•內容過濾主題

•查詢條件

•多主題

內容過濾的主題和多主題接口從`TopicDescription`接口繼承（而不是從Topic接口，如名稱可能建議的）。

內容過濾的主題和查詢條件允許使用類似SQL的參數化查詢字符串過濾（選擇）數據樣本。此外，查詢條件允許對從數據讀取器的`read()`或`take()`操作返回的結果集進行排序。多主題還具有此選擇功能，以及將來自不同數據寫入器的數據聚合到單個數據類型和數據讀取器的能力。

如果您不打算在應用程序中使用`Content-Subscription Profile`功能，您可以配置OpenDDS以在構建時刪除對它的支持。有關禁用此支持的信息，請參見第12頁。

# 5.2內容過濾主題

域參與者接口包含用於創建和刪除內容過濾主題的操作。創建內容過濾主題需要以下參數：

• 名稱

為此內容過濾主題分配名稱，以後可以與`lookup_topicdescription()`操作一起使用。

•相關主題

指定此內容篩選主題所基於的主題。這是匹配數據寫入者將用於發布數據樣本的相同主題。

•過濾表達式

類似於SQL的表達式（參見第5.2.1節），它定義了在內容過濾主題的數據讀取器應該接收的相關主題上發布的示例子集。

•表達式參數

過濾器表達式可以包含參數佔位符。此參數提供這些參數的初始值。在創建內容過濾主題後，可以更改表達式參數（過濾器表達式無法更改）。

一旦已經創建了內容過濾的主題，它被訂戶的`create_datareader()`操作用來獲得內容過濾數據讀取器。該數據讀取器在功能上等同於正常數據讀取器，除了丟棄不滿足過濾器表達式準則的傳入數據樣本。

首先在發布者處評估過濾器表達式，以便可以在甚至到達傳輸之前刪除用戶可以忽略的數據樣本。可以使用-`DCPSPublisherContentFilter 0`或配置文件的`[common]`部分中的等效設置關閉此功能。非默認`DEADLINE`或`LIVELINESS `QoS策略的行為可能受此策略的影響。必須特別注意“丟失”樣本如何影響QoS行為，請參閱`docs / design / CONTENT_SUBSCRIPTION`中的文檔。

**注意:**RTPS傳輸不總是做寫入器端過濾。 它當前不實現傳輸級過濾，但可以能夠在傳輸層上方進行過濾。

## 5.2.1過濾器表達式

過濾器表達式的形式語法在DDS規範的附錄A中定義。

本節提供了該語法的非正式摘要。 查詢表達式（5.3.1）和主題表達式（5.4.1）也在附件A中定義。

過濾器表達式是一個或多個謂詞的組合。每個謂詞是採用以下兩種形式之一的邏輯表達式：

•&lt;arg1&gt; &lt;RelOp&gt; &lt;arg2&gt;

- arg1和arg2是可以是文字值（整數，字符，浮點數，字符串或枚舉）的參數，形式為％n的參數佔位符（其中n是基於從零開始的索引到參數序列中） ，或字段參考。

- 至少一個參數必須是字段引用，它是IDL結構字段的名稱，可選地後跟任意數量的“。”和另一個表示嵌套結構的字段名稱。

- RelOp是來自列表的關係運算符：=，&gt;，&gt; =，&lt;，&lt;=，&lt;&gt;和“like”。 'like'是一個通配符匹配，使用％匹配任意數量的字符和\_匹配單個字符。

- 這種形式的謂詞的示例包括：a ='z'，b &lt;&gt;'str'，c &lt;d，e ='enumerator'，f&gt; = 3.14e3,27g，h &lt; 0

•&lt;arg1&gt; \[NOT\] BETWEEN &lt;arg2&gt; AND &lt;arg3&gt;

- 在此形式中，參數1必須是字段引用，參數2和3必須都是文字值或參數佔位符。

任何數量的謂詞可以通過使用括號和布爾運算符AND，OR和NOT組合形成一個過濾器表達式。

## 5.2.2內容過濾主題示例

以下代碼段為消息類型創建了內容過濾主題。首先，這裡是Message的IDL：

```cpp
 module Messenger {
 #pragma DCPS_DATA_TYPE "Messenger::Message"
  struct Message {
   long id;
  };
 };
```

接下來我們有創建數據讀取器的代碼：

```
 CORBA::String_var type_name = message_type_support->get_type_name();
 DDS::Topic_var topic = dp->create_topic("MyTopic", type_name,TOPIC_QOS_DEFAULT,NULL,OpenDDS::DCPS::DEFAULT_STATUS_MASK);
 DDS::ContentFilteredTopic_var cft =
 participant->create_contentfilteredtopic("MyTopic-Filtered",topic,"id > 1", StringSeq());
 DDS::DataReader_var dr =
 subscriber->create_datareader(cft, dr_qos, NULL, OpenDDS::DCPS::DEFAULT_STATUS_MASK);
```

數據讀取器'dr'將僅接收具有大於1的'id'的值的樣本。

# 5.3查詢條件

查詢條件接口從讀取條件接口繼承，因此查詢條件具有讀取條件的所有功能以及本節中描述的附加功能。其中一個繼承的功能是查詢條件可以像任何其他條件一樣使用等待集（見第4.4節）。

DataReader接口包含用於創建（`create_querycondition`）和刪除（`delete_readcondition`）查詢條件的操作。創建查詢條件需要以下參數：

•樣本，視圖和實例狀態掩碼

這些都是傳遞給`create_readcondition()`的狀態掩碼，

`read()`或`take()`。

•查詢表達式

描述導致條件被觸發的樣本子集的類SQL表達式（見5.3.1）。此相同的表達式用於過濾從`read_w_condition()`或`take_w_condition()`操作返回的數據集。它還可以對該數據集施加排序順序（ORDER BY）。

•查詢參數

查詢表達式可以包含參數佔位符。此參數提供這些參數的初始值。可以在創建查詢條件後更改查詢參數（查詢表達式不能更改）。

特定查詢條件可以與等待集（`attach_condition`）一起使用，其具有數據讀取器（`read_w_condition`，`take_w_condition`，`read_next_instance_w_condition`，`take_next_instance_w_condition`）或兩者。當與等待集一起使用時，`ORDER BY`子句對觸發等待集沒有影響。當與數據讀取器的讀\*\(\)或\*\(\)操作一起使用時，生成的數據集將僅包含與查詢表達式匹配的樣本，如果存在`ORDER BY`子句，它們將由`ORDER BY`字段排序。

以逗號分隔的字段引用列表。 如果存在`ORDER BY`子句，則過濾器表達式可能為空。 下面的字符串是查詢表達式的示例：

•m&gt; 100 ORDER BY n

•ORDER BY p.q，r，s.t.u

•NOT v LIKE'z％'

## 5.3.2查詢條件示例

以下代碼片段為使用帶有'`key`'字段（整數類型）的`struct'Message`'的類型創建並使用查詢條件。

```cpp
 DDS::QueryCondition_var dr_qc =
  dr->create_querycondition(DDS::ANY_SAMPLE_STATE, DDS::ANY_VIEW_STATE, DDS::ALIVE_INSTANCE_STATE, "key > 1",
 DDS::StringSeq());
 DDS::WaitSet_var ws = new DDS::WaitSet;
 ws->attach_condition(dr_qc);
 DDS::ConditionSeq active;
 DDS::Duration_t three_sec = {3, 0};
 DDS::ReturnCode_t ret = ws->wait(active, three_sec);
  // error handling not shown
 ws->detach_condition(dr_qc);
 MessageDataReader_var mdr = MessageDataReader::_narrow(dr);
 MessageSeq data;
 DDS::SampleInfoSeq infoseq;
 ret = mdr->take_w_condition(data, infoseq, DDS::LENGTH_UNLIMITED, dr_qc);
  // error handling not shown
 dr->delete_readcondition(dr_qc);
```

使用鍵&lt;= 1接收的任何樣本都不會觸發條件（滿足等待），也不會在來自take\_w\_condition（）的'data'序列中返回。

## 5.4多主題

多主題是比其他兩個`Content-Subscription`功能更複雜的功能，因此描述它需要一些新的術語。

`MultiTopic`接口繼承了`TopicDescription`接口，就像

`ContentFilteredTopic`。 為多主題創建的數據讀取器被稱為“多主題數據讀取器”。多主題數據讀取器接收屬於任何數量的常規主題的樣本。 這些主題被稱為其“組成主題”。多主題具有被稱為“結果類型”的DCPS數據類型。多主題數據讀取器實現針對所得類型的類型特定數據讀取器接口。 例如，如果生成的類型是`Message`，則可以將多主題數據讀取器縮小到`MessageDataReader`接口。

多主題的主題表達式（參見第5.4.1節）描述了傳入數據（組成主題）的不同字段如何映射到結果類型的字段。

域參與者界麵包含創建和刪除多主題的操作。

創建多主題需要以下參數：

• 名稱

為此多主題分配名稱，稍後可以與`lookup_topicdescription()`操作一起使用。

•類型名稱

指定多主題的結果類型。此類型必須在創建多主題之前註冊其類型支持。

•主題表達式（也稱為預訂表達式）

定義組成主題字段到結果類型字段的映射的類SQL表達式（見5.4.1節）。它還可以指定一個過濾器（WHERE子句）。

•表達式參數

主題表達式可以包含參數佔位符。此參數提供這些參數的初始值。可以在創建多主題後更改表達式參數（主題表達式無法更改）。

一旦創建了多主題，它被訂閱者的`create_datareader()`操作使用以獲得多主題數據讀取器。該數據讀取器被應用用來接收所產生類型的構造樣本。構建這些樣本的方式在下面的5.4.2.2節中描述。

## 5.4.1主題表達式

主題表達式使用與完整SQL查詢非常相似的語法：

`SELECT <aggregation> FROM <selection> [WHERE <condition>]`

•聚合可以是“\*”或逗號分隔的字段說明符列表。 每個字段說明符具有以下語法：

- `<constituent_field> [[AS] <result_field>]]`

- `constituent_field`是一個字段的字段引用（見第1.1.1節）

組成主題（該主題未指定）。

- 可選的結果字段是對結果類型中字段的字段引用。 如果存在，結果字段是構造的樣本中的`constituent_field`的目的地。 如果不存在，則在所得到的類型中將constituent\_field數據分配給具有相同名稱的字段。 可選的“AS”沒有效果。

- 如果使用“\*”作為聚合，則結果類型中的每個字段都將從構成主題類型之一的同名字段中分配值。

•選擇列出一個或多個組成主題名稱。 主題名稱由“join”關鍵字分隔（所有3個連接關鍵字都是等效的）：

- `<topic> [{NATURAL INNER | 自然| INNER NATURAL} JOIN <topic>]` ...

- 主題名稱只能包含字母，數字和短劃線（但不能以數字開頭）。

- 自然連接操作是可交換和關聯的，因此主題的順序沒有影響。

- 自然聯接的語義是，具有相同名稱的任何字段被視為“連接鍵”，用於組合來自這些鍵出現的主題的數據。 在本章的後續章節中將更詳細地描述聯接操作。

•條件具有與過濾器表達式完全相同的語法和語義（請參見第5.2.1節）。 條件中的字段引用必須匹配結果類型中的字段名稱，而不匹配組成主題類型中的字段名稱。

## 5.4.2使用說明

### 5.4.2.1連接鍵和DCPS數據鍵

DCPS數據密鑰（`#pragma DCPS_DATA_KEY`）的概念已在第2.1.1節中討論過。多主題的加入鍵是一個獨特但相關的概念。

連接鍵是出現在多個組成主題的`struct`中的任何字段名。

連接鍵的存在對這些主題的數據樣本如何組合成一個構造的樣本（見第5.4.2.2節）進行約束。具體來說，對於來自組成主題的那些數據樣本，該鍵的值必須相等，以組合成所得到的類型的樣本。如果多個連接鍵對於相同的兩個或多個主題是共同的，則所有鍵的值必須相等，以便組合數據。

DDS規範要求連接鍵字段具有相同的類型。此外，OpenDDS對IDL必須定義DCPS數據密鑰以處理多主題有兩個要求：

1）每個連接鍵字段還必須是其組成主題的類型的DCPS數據鍵。

2）結果類型必須包含每個連接鍵，這些字段必須是生成類型的DCPS數據鍵。

第5.4.3.1節中的示例滿足這兩個要求。請注意，沒有必要在聚合中列出連接鍵（SELECT子句）。

### 5.4.2.2如何構建結果樣本

雖然多主題中的許多概念都是從關係領域借用的

數據庫，諸如DDS之類的實時中間件不是數據庫。代替一次處理一批數據，每個樣本從組成主題之一到達數據讀取器觸發多主題專用處理，其導致結果類型和插入的零個，一個或多個樣本的構造的那些構建的樣本到多主題數據讀取器中。

具體地，樣本到達具有類型“TA”的構成主題“A”導致多主題數據讀取器中的以下步驟（這是實際

算法）：

1）構造所得類型的樣本，並且將來自存在於所得到的類型中並且在聚合（或者是連接密鑰）中的TA的字段從傳入樣本複製到構造的樣本。

2）具有至少一個與A相同的連接鍵的每個主題“B”被考慮用於連接操作。連接讀取主題B上的`READ_SAMPLE_STATE`樣本，其鍵值與構建的樣本中的值相匹配。連接的結果可以是零個，一個或多個樣本。如步驟1所述，將TB中的字段複製到生成的樣本中。

3）按照步驟2中所述處理主題“B”的連接鍵（連接到其他主題），這將繼續到通過連接鍵連接的所有其他主題。

4）在步驟2或3中未訪問的任何組成主題被處理為“交叉連接”（也稱為跨產品連接）。這些是沒有關鍵約束的連接。

5）如果任何構造的樣本結果，它們被插入到多主題數據讀取器的內部數據結構中，就好像它們已經通過正常機製到達。通知應用程序監聽器和條件。

### 5.4.2.3用於訂閱者偵聽器

如果應用程序註冊了一個訂閱者監聽器，用於讀取條件狀態更改（`DATA_ON_READERS_STATUS`），同一訂戶也包含多主題，則應用程序必須在訂閱者監聽器的`on_data_on_readers()`回調方法的實現中調用`notify_datareaders()`。 此要求是必需的，因為多主題內部使用數據讀取器偵聽器，當訂閱者偵聽器註冊時它們被搶占。

## 5.4.3多主題示例

本示例基於DDS規範的附錄A第A.3節中使用的示例主題表達式。 它說明瞭如何使用多主題連接操作的屬性來關聯來自單獨主題（以及可能不同的發布者）的數據。

### 5.4.3.1 IDL和主題表達式

通常，我們將使用與主題名稱和主題類型相同的字符串。 在這個例子中，我們將使用不同的字符串作為類型名稱和主題名稱，以便說明何時使用。

下面是組成主題數據類型的IDL：

```cpp
#pragma DCPS_DATA_TYPE "LocationInfo"
#pragma DCPS_DATA_KEY "LocationInfo flight_id"
struct LocationInfo {
 unsigned long flight_id;
 long x;
 long y;
 long z;
};
#pragma DCPS_DATA_TYPE "PlanInfo"
#pragma DCPS_DATA_KEY "PlanInfo flight_id"
struct PlanInfo {
 unsigned long flight_id;
 string flight_name;
 string tailno;
};
```

請注意，鍵字段的名稱和類型匹配，因此它們被設計為用作連接鍵。 生成的類型（下面）也有該關鍵字段。

接下來，我們得到結果數據類型的IDL：

```cpp
#pragma DCPS_DATA_TYPE "Resulting"
#pragma DCPS_DATA_KEY "Resulting flight_id"
struct Resulting {
 unsigned long flight_id;
 string flight_name;
 long x;
 long y;
 long height;
};
```

基於此IDL，可以使用以下主題表達式組合來自使用類型`LocationInfo`的主題位置和使用類型`PlanInfo`的主題FlightPlan的數據：

`SELECT flight_name，x，y，z` AS `height FROM Location NATURAL JOIN FlightPlan WHERE height <1000 AND x <23`總之，IDL和主題表達式描述了這個多主題將如何工作。

多主題數據讀取器將構造屬於由flight\_id鍵入的實例的樣本。 只有當`Location`和`FlightPlan`主題都有相應的實例時，結果類型的實例才會存在。 域中的一些其他域參與者或參與者將發布關於這些主題的數據，並且他們甚至不需要互相意識。 由於它們每個使用相同的`flight_id`來引用航班，所以多主題可以關聯來自不同來源的傳入數據。

### 5.4.3.2創建多主題數據讀取器

為多主題創建數據讀取器包括幾個步驟。 首先註冊對結果類型的類型支持，然後創建多主題本身，其次是數據讀取器：

```cpp
ResultingTypeSupport_var ts_res = new ResultingTypeSupportImpl;
 ts_res->register_type(dp, "");
 CORBA::String_var type_name = ts_res->get_type_name();
 DDS::MultiTopic_var mt =
 dp->create_multitopic("MyMultiTopic",type_name,"SELECT flight_name, x, y, z AS height " "FROM Location NATURAL JOIN FlightPlan " "WHERE height < 1000 AND x<23",DDS::StringSeq());
 DDS::DataReader_var dr = sub->create_datareader(mt,DATAREADER_QOS_DEFAULT, NULL, OpenDDS::DCPS::DEFAULT_STATUS_MASK);
```



