# TKGm Kasten 安裝  

### 環境簡介  
TKGm 版本:1.4.1  
TKC  版本:1.21.1  


### 安裝步驟  

#### Pre-Flight Checks  

helm安裝  
kubectl的context指向要裝Kasten的叢集  
需要有預設的storageClass  

如果叢集內存在CSI的供應商(如Tanzu,Redhat OKD)  
建議使用已下工具進行驗證是否符合kasten需求  
