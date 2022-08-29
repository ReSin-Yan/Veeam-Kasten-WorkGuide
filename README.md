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
  

## Kubernetes 操作  

登入到Taznu環境  
```
export KUBECTL_VSPHERE_PASSWORD=1qaz@WSX
kubectl vsphere login --insecure-skip-tls-verify --server 172.18.17.22 --vsphere-username ntust@vsphere.local --tanzu-kubernetes-cluster-name ntust-tkc[輸入編號]
kubectl config use-context ntust-tkc[輸入編號]
```

將此專案透過git下載  
```
cd 
git clone https://github.com/ReSin-Yan/NTUSTCourse
cd NTUSTCourse/Kubernetes
kubectl apply -f gcallowroot.yaml  
