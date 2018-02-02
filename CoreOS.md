# CoreOS
Tectonic 文檔
## 安裝在Red Hat Enterprise Linux上

Tectonic安裝部署Kubernetes集群時創建兩種類型的節點: worker nodes（執行容器）和master nodes（處理集群管理任務，並包含API服務器）。

在大規模集群中，worker nodes可能持有運行幾種不同類型應用程序的容器。
Red Hat Enterprise Linux增加驅動程序的靈活性和便捷性使得Kubernetes工作組件可以部署至特定的workloads 和hardware。

如果您還沒有創建群集, 使用Tectonic 安裝部署在裸機（或AWS）上。 安裝程序將使用Container Linux作為master nodes, 你可以配置額外的Container Linux worker nodes。

一旦部署了集群, 請按照配置和加入運行紅帽企業版Linux的其他worker nodes。

### 架構

#### 部署

####　執行
