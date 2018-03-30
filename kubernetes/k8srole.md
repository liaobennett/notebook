安裝kubernetes二進位檔案安裝的方式，安裝過程中也能加深對各個元件和配置參數的理解。

用到及涉及到的元件說明
K8s：kubernetes的簡稱，k（8個字母）s。

Kubernetes 集群中主要存在兩種類型的節點，分別是 master 節點，以及 minion 節點。

Minion 節點是實際運行 Docker 容器的節點，負責和節點上運行的 Docker 進行交互，並且提供了代理功能。

Master 節點負責對外提供一系列管理集群的 API 介面，並且通過和 Minion 節點交互來實現對集群的操作管理。

apiserver：用戶和 kubernetes 集群交互的入口，封裝了核心物件的增刪改查操作，提供了 RESTFul 風格的 API 介面，通過 etcd 來實現持久化並維護物件的一致性。

scheduler：負責集群資源的調度和管理，例如當有 pod 異常退出需要重新分配機器時，scheduler 通過一定的調度演算法從而找到最合適的節點。

controller-manager：主要是用於保證 replicationController 定義的複製數量和實際運行的 pod 數量一致，另外還保證了從 service 到 pod 的映射關係總是最新的。

kubelet：運行在 minion 節點，負責和節點上的 Docker 交互，例如啟停容器，監控運行狀態等。

proxy：運行在 minion 節點，負責為 pod 提供代理功能，會定期從 etcd 獲取 service 資訊，並根據 service 資訊通過修改 iptables 來實現流量轉發（最初的版本是直接通過程式提供轉發功能，效率較低。），將流量轉發到要訪問的 pod 所在的節點上去。

etcd：key-value鍵值存儲資料庫，用來存儲kubernetes的資訊的。

flannel：Flannel 是 CoreOS 團隊針對 Kubernetes 設計的一個覆蓋網路（Overlay Network）工具，需要另外下載部署。我們知道當我們啟動 Docker 後會有一個用於和容器進行交互的 IP 位址，如果不去管理的話可能這個 IP 位址在各個機器上是一樣的，並且僅限於在本機上進行通信，無法訪問到其他機器上的 Docker 容器。Flannel 的目的就是為集群中的所有節點重新規劃 IP 位址的使用規則，從而使得不同節點上的容器能夠獲得同屬一個內網且不重複的 IP 位址，並讓屬於不同節點上的容器能夠直接通過內網 IP 通信。

kube-dns：用來為kubernetes service分配子功能變數名稱，在集群中可以通過名稱訪問service。
通常kube-dns會為service賦予一個名為“service名稱.namespace.svc.cluster.local”的A記錄，用來解析service的cluster ip。在實際應用中，
如果訪問default namespace下的服務，則可以通過“service名稱”直接訪問。如果訪問其他namespace下的服務，則可以通過“service名稱.namespace”訪問。

kube-router： 取代 kube-proxy,本教程用kube-router元件取代kube-proxy,用lvs做svc負載均衡，更快穩定。

kube-dashboard：kubernetes官方的web ui管理介面

core-dns： 取代 kube-dns，更穩定。
