Last update: 2022/8/29  
# Veeam-Kasten-WorkGuide

### Veeam簡易介紹     
Kasten簡稱k10，主要功能為提供營運團隊一個方便使用、可擴展性及安全的平台

用來針對k8s平台的備份、復原、以及應用程序移動  


目前授權版本分為三種  
Free  
Enterprise Trial  
Enter prise  

[參考網站](https://www.kasten.io/product/#k10-editions "link")  


### 安裝步驟   
  
## Linux Clinet 準備  

## Kubernetes 操作及環境準備    

確認Kuberentes服務  
已下指令是Tanzu環境登入的指令  
如果是其它Kubernetes的平台，需要確認能夠正常的執行Kubernetes的相關操作  
可以跳轉到下一章節  

登入到Taznu環境  
```
export KUBECTL_VSPHERE_PASSWORD=P@ssw0rd
kubectl vsphere login --server=172.18.17.22 --insecure-skip-tls-verify  --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-name  [輸入姓名]-tkc1
kubectl vsphere login --server=172.18.17.22 --insecure-skip-tls-verify  --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-name  [輸入姓名]-tkc2
kubectl config use-context [輸入姓名]
```

下載gcallowroot yaml(TKC需要)  
```
cd 
git clone https://github.com/ReSin-Yan/NTUSTCourse
cd NTUSTCourse/Kubernetes
kubectl apply -f gcallowroot.yaml  
```

安裝NFS sub-dir(如有Kubernetes本身已有StorageClass也建議設定)  

```
cd 
git clone https://github.com/ReSin-Yan/NTUSTCourse
cd NTUSTCourse/Kubernetes
kubectl apply -f gcallowroot.yaml  
```


