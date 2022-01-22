# TKGm Kasten 安裝  

### 環境簡介  
TKGm 版本:1.4.1  
TKC  版本:1.21.1  


### 安裝步驟  

#### Pre-Flight Checks  

helm安裝  
```
sudo apt-get install -y curl
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
kubectl的context指向要裝Kasten的叢集  
需要有預設的storageClass  

如果叢集內存在CSI的供應商(如Tanzu,Redhat OKD)  
建議使用已下工具進行驗證是否符合kasten需求  
```
helm repo add kasten https://charts.kasten.io/
curl https://docs.kasten.io/tools/k10_primer.sh | bash
curl -s https://docs.kasten.io/tools/k10_primer.sh  | bash /dev/stdin -s ${STORAGE_CLASS}  
```  
