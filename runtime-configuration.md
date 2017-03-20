# CHAPTER 7

### Run-time Configuration

## 7.1 動手設定

OpenDDS使用以檔案為基礎的設定框架，其中包含全預選項，與特定發布者、訂閱者相關的選項(如探索設定以及傳送設定)。
OpenDDS也允許使用者透過執行文字指令，部份的選項可以透過API設定。
這個章節將會描述OpenDDS支援的設定選項。
#### 1) 普通設定選項：
設定在全域等級下DCPS實體的行為。
這將可以讓在相同電腦環境下部署的不同進程共享同一個普通設定。
(例如所有的讀取者及寫入者都使用RTPS探索)
#### 2) 探索設定選項：
設定探索機制下的行為。
OpenDDS支援多種途徑探索及協調寫入者與讀取者。
更多細節參照段落7.3。
#### 3) 傳送設定選項：
設定可擴展傳送框架(ETF)，作為OpenDDS的DCPS層中的抽象傳輸層。
每個可插拔式的傳送都可以被分別設定。

OpenDDS的設定檔案為易讀的ini格式文字檔。 表7-1為可允許的設定段落類型，他們都對應一種OpenDDS設定選項。

#### 表7-1 設定檔案段落

| 設定選項類型 | 檔案段落標題                                                                      |
|--------------|-----------------------------------------------------------------------------------|
| 全域設定     | [common]                                                                          |
| 探索         | [domain] [repository] [rtps_discovery]                                            |
| 靜態探索     | [endpoint] [topic] [datawriterqos] [datareaderqos] [publisherqos] [subscriberqos] |
| 傳送         | [config] [transport]                                                              |



除了[common]之外，每個段落得標頭都可以用[段落標題/實例]的格式作設定，舉個例子，[repository]標頭可以這樣描述：
[repository/repo_1]，repository為段落標題，repo_1為實例名稱，這個標頭下的設定將套用在repo_1實例下的repository設定。
如何使用實例來設定探索以及傳送選項將會在段落7.3以及7.4介紹。

`-DCPSConfigFile` 指令參數可以用來傳遞欲套用之OpenDDS設定檔案的位置。如下：

Windows：
```
publisher -DCPSConfigFile pub.ini
```
Unix：
```
./publisher -DCPSConfigFile pub.ini
```
指令參數將在區域參與因數(domain participant factory)初始化時傳遞至個別參與的實例。
這將由巨集`TheParticipanFactoryWithArgs`：
```cpp
#include <dds/DCPS/Service_Participant.h>
int main (int argc, char* argv[])
{
  DDS::DomainParticipantFactory_var dpf =
  TheParticipantFactoryWithArgs(argc, argv)
```
`Service_Participant`集合也提供允許應用程式設定DDS服務的方法，詳情參照標頭檔`$DDS_ROOT/dds/DCPS/Service_Participant.h`。
以下段落將詳細描述各個不同的設定檔案段落以及相關的可用選項。
