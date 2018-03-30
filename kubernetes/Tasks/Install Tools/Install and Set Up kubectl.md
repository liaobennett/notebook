# Install and Set Up kubectl

## Before you begin

使用與您的服務器版本相同或更高版本的kubectl版本。對較新的服務器使用較舊的kubectl可能會產生驗證錯誤。

### Install kubectl binary via curl



## Install kubectl

這裡有一些安裝kubectl的方法。選擇最適合您的環境的人。
使用Kubernetes命令行工具kubectl在Kubernetes上部署和管理應用程序。使用kubectl，您可以檢查群集資源;創建，刪除和更新組件;並查看您的新群集並調出示例應用程序。

### Linux

Download the latest release with the command:

1. curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

2. Make the kubectl binary executable.

chmod +x ./kubectl

3. Move the binary in to your PATH.

 sudo mv ./kubectl /usr/local/bin/kubectl
