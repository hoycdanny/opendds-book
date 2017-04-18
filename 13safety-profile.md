# CHAPTER 13 

# Safety Profile

# 13.1概述

Safety Profile配置允許OpenDDS在具有有限的操作系統和標準庫功能的環境中使用，並且只有在系統啟動時才需要動態內存分配。

OpenDDS安全簡介（以及ACE中的相應功能）是針對Open Group的FACE規範2.1版（[http://www.opengroup.org/face/techstandard-2.1](http://www.opengroup.org/face/techstandard-2.1)）開發的。

它可以與FACE Transport Services的支持一起使用來創建符合FACE標準的DDS應用程序，也可以被一般DDS應用程序用於未寫入FACE Transport Services API的應用程序。 後一種用例由開發人員指南的這一部分描述。 有關前一個用例的更多信息，請參閱源分發中的文件FACE / README.txt或通過[sales@ociweb.com](/sales@ociweb.com)（商業支持）或[opendds-main@lists.sourceforge.net](/opendds-main@lists.sourceforge.net)（社區支持）與我們聯繫。

# 13.2 OpenDDS的安全配置子集

當為安全配置文件配置時，OpenDDS的以下功能不可用：

* DCPSInfoRepo及其相關的庫和工具
* 傳輸類型：tcp，udp，組播，共享內存
  * rtps\_udp傳輸類型可用（使用UDP單播或多播）
* OpenDDS Monitor庫和監視GUI

在開發安全配置文件時，以下DDS合規性配置文件被禁用：

* content\_subscription
* ownership\_kind\_exclusive
* object\_model\_profile
* persistence\_profile

有關合規性配置文件的詳細信息，請參見第1.3.3節。 可能在安全性配置文件構建中啟用任何這些合規性配置文件將導致編譯時或運行時錯誤。

要構建OpenDDS安全配置文件，請將命令行參數“--safety-profile”傳遞給配置腳本以及平台或配置所需的任何其他參數。

在配置腳本中啟用安全配置文件時，上面列出的四個合規性配置文件默認為禁用。 有關配置腳本的更多信息，請參閱源分發中的第1.3節和INSTALL文件。

# 13.3 ACE的安全配置文件

OpenDDS使用ACE作為其平台抽像庫，在OpenDDS的安全配置文件配置中，必須在ACE中啟用以下安全配置文件配置之一：

* FACE安全基（始終使用內存池）
* 內存池擴展的FACE安全性
* 通過標準C ++動態分配擴展FACE Safety

OpenDDS的配置腳本將自動配置ACE。傳遞命令行

參數“--safety-profile = base”選擇安全基本配置文件。否則，“--safetyprofile”（無等號）配置將默認為Safety Extended with Memory Pool。

安全擴展與標準C ++動態分配配置不是由配置腳本自動生成的，但是可以通過configure（並在運行make之前）編輯文件“build / target / ACE\_wrappers / ace / config.h”。刪除ACE\_HAS\_ALLOC\_HOOKS的宏定義以禁用內存池。

ACE的安全配置文件配置已在Linux和LynxOS-178版本2.3.2 +修補程序上進行了測試。其他平台也可能工作，但可能需要額外的配置。

# 13.4運行時可配置選項

可以通過在配置文件的\[common\]部分中設置值來配置OpenDDS使用的內存池。 請參見第7.2節和表7-2的pool\_size和pool\_granularity行。

# 13.5運行ACE和OpenDDS測試

配置和構建OpenDDS安全配置文件後，請注意有兩個子目錄

頂級的每個包含一些二進製文物：

* build / host具有構建時代碼生成器tao\_idl和opendds\_idl
* build / target具有運行時庫，用於安全性配置文件ACE和OpenDDS以及OpenDDS測試

因此，測試需要相對於構建/目標子目錄。 源 - 在生成的文件build / target / setenv.sh中獲取所有需要的環境變量。 ACE測試不是默認構建的，但是一旦設置了這個環境，構建它們就是生成makefile並運行make：

1.  cd $ACE\_ROOT/tests 
2.  $ACE\_ROOT/bin/mwc.pl -type gnuace 
3.  make

通過更改為$ ACE\_ROOT / tests目錄並使用run\_test.pl運行ACE測試。 通過配置所需的“-Config XYZ”選項（使用run\_test.pl -h查看可用的配置選項）。

通過更改為$ DDS\_ROOT並使用bin / auto\_run\_tests.pl運行OpenDDS測試。 默認情況下，通過“-Config OPENDDS\_SAFETY\_PROFILE”，“-Config SAFETY\_BASE”（如果使用安全基礎），“-Config RTPS”和與每個禁用的合規性配置文件相對應的-Config選項：“-Config DDS\_NO\_OBJECT\_MODEL\_PROFILE -Config DDS\_NO\_OWNERSHIP\_KIND\_EXCLUSIVE -Config DDS\_NO\_PERSISTENCE\_PROFILE -Config DDS\_NO\_CONTENT\_SUBSCRIPTION“。

或者，可以使用該測試目錄中的run\_test.pl運行單個測試。 將相同的一組-Config選項傳遞給run\_test.pl。

# 13.6在應用程序中使用內存池

當構建時啟用內存池時，由OpenDDS或ACE中的代碼（由OpenDDS調用的方法）進行的所有動態分配都將通過該池進行。 由於池是一個通用動態分配器，因此應用程序代碼也可能需要使用池。 由於這些API是OpenDDS的內部API，因此它們可能會在將來的版本中更改。

OpenDDS :: DCPS :: MemoryPool（dds / DCPS / MemoryPool.h）類包含池實現。 然而，大多數客戶端代碼不應該直接與它進行交互。 SafetyProfilePool類（dds / DCPS / SafetyProfilePool.h）將池適配到ACE\_Allocator接口。 PoolAllocator &lt;Pool&gt;（PoolAllocator.h）將池適配為C ++ Allocator概念（C ++ 03）。 由於PoolAllocator是無狀態的，它取決於ACE\_Allocator的單例。

當OpenDDS配置了內存池時，ACE\_Allocator的單例實例將指向SafetyProfilePool類的對象。

使用C ++標準庫類的應用程序代碼可以直接使用PoolAllocator，也可以使用PoolAllocator.h中定義的宏（例如OPENDDS\_STRING）。

分配動態內存的原始（非類型）緩衝區的應用程序代碼可以直接或通過ACE\_Allocator :: instance（）單例使用SafetyProfilePool。 從堆分配對象的應用程序代碼可以使用PoolAllocator &lt;T&gt;模板。

由應用程序開發人員編寫的類可以從PoolAllocationBase（請參閱PoolAllocationBase.h）產生，以繼承類作用域操作符號new和delete，從而將這些類型的所有動態分配重定向到pool中。

