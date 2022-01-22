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
TKGm 1.4.1版本目前尚未支援CSI  


#### Kasten Install  

建立命名空間，Kasten的備份會以命名空間為主要備份方式  
但是目前時間點Tanzu CSI尚未支援Kasten的原因，需要將命名空間打上TAG  
利用Kasten的功能Kanister的方式來進行備份  
```
kubectl label namespace demo k10/injectKanisterSidecar=true
kubectl label namespace default k10/injectKanisterSidecar=true
```  
新增一個demo的命名空間，並且將default的命名空間跟demo一起打上TAG  

進入kasten的安裝，安裝依照以下步驟進行(都複製貼上就好)  
把檔案加入至helm，並且把helm更新  
```  
helm repo add kasten https://charts.kasten.io --force-update && helm repo update
```  
創建命名空間kasten-io  
```  
kubectl create ns kasten-io
```  

安裝metalLB(L4工具)跟contour(L7 ingress工具)  

產生

接著, 透過以下指令執行 Kasten K10 的安裝動作, 這些參數會針對 Kasten 與指定的 namespace 啟動 Kanister Sidecar 功能  
``` 
helm install k10 kasten/k10 --namespace=kasten-io --set inget ingress.class=contour --set ingress.host=kastendemo5.com --set auth.basicAuth.enabled=true --set auth.basicAuth.htpasswd='example:$apr1$qrAVXu.v$Q8YVc50vtiS8KPmiyrkld0'
``` 
