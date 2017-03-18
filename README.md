# **CHAPTER 1**

### **Introduction**

OpenDDS是用於即時系統規範v1.4（OMG文件格式/ 2015-04-10）的OMG數據分發服務（DDS）和即時發布 - 訂閱有線協定DDS互操作性有線協定規範（DDSI）的開放原始碼實現 -RTPS）v2.2（OMG文件格式/ 2014-09-01）。 OpenDDS由Object Computing，Inc。（OCI）贊助，可在[http://www.opendds.org/](http://opendds.org/)獲得。 本開發人員指南基於OpenDDS的3.10版本。

DDS定義用於在分佈式應用中的參與者之間有效地分發應用數據的服務。 此服務不是特定於CORBA的。 規範提供了平台獨立模型（PIM）以及將PIM映射到CORBA IDL實現的平台特定模型（PSM）。

有關DDS的其他詳細信息，開發人員應參考DDS規範（OMG文檔格式/ 2015-04-10），因為它包含對所有服務的功能的深入涵蓋。

OpenDDS是OCI開發和商業支持的OMG的DDS規範的開源C ++實現。 它可從[http://www.opendds.org/downloads.html](http://www.opendds.org/downloads.html)下載，並與OCI TAO 2.0a和2.2a的最新補丁級別以及最新的DOC Group版本相容。

注意:OpenDDS目前實現OMG DDS 1.4版規範。 有關詳細訊息，請參閱[http://www.opendds.org/](http://www.opendds.org/)中的規格訊息。



# 1.1. DCPS概述

在本節中，我們將介紹DCPS層的主要概念和實體，並討論它們如何相互影響和一起工作。

## 1.1.1基本概念

圖1-1顯示了DDS DCPS層的概述。 以下小節定義了此圖中所示的概念。

## ![](/1.1.jpg)

## 1.1.1.1 Domain

domain是DCPS中的基本分區單元。 每個其他實體屬於一個domain，並且只能與該domain中的其他實體相互影響。

應用程序代碼可以與多個domain自由交互，但必須通過屬於不同domain的單獨實體進行相互影響。

## 1.1.1.2 DomainParticipant

domain參與者是應用程序在特定domain內交互的入口點。 domain參與者是牽涉到寫入或讀取數據的許多對象的工廠。

1.1.1.3主題



## 





































































