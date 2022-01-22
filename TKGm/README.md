# TKGm Kasten 安裝  

### 環境簡介  

 | 套件名稱 | 版本  |
|-------|-------|
| TKGm | 1.4.1 |  
| TKC | 1.21.1 |  
| kasten | 4.5.7 |  
| L4工具 | [metalLB](https://github.com/ReSin-Yan/kuberenetes-base/blob/main/k8stools/service/loadbalancetools/installMetallb.sh "link")   |  
| L7工具 | [contour](https://github.com/ReSin-Yan/kuberenetes-base/blob/main/k8stools/service/ingress/install.sh "link")  |  

### 安裝步驟  

#### 環境準備  

環境更新及安裝基本套件  
```
sudo apt-get update && sudo apt-get -y upgrade
sudo apt-get -y install vim build-essential curl ssh
sudo apt-get install net-tools default-jdk git
sudo apt install apache2-utils  
```
安裝kubernetes套件  
```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo  apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" >> kubernetes.list
sudo cp kubernetes.list /etc/apt/sources.list.d/
rm kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00
```
helm安裝  
```
sudo apt-get install -y curl
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

#### Pre-Flight Checks  


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
kubectl create ns demo
```  

安裝metalLB(L4工具)跟contour(L7 ingress工具)  
[metalLB](https://github.com/ReSin-Yan/kuberenetes-base/blob/main/k8stools/service/loadbalancetools/installMetallb.sh "link")  
[contour](https://github.com/ReSin-Yan/kuberenetes-base/blob/main/k8stools/service/ingress/install.sh "link")

產生htpasswd(需要預先安裝)  
```
sudo apt install apache2-utils
mkdir htpasswd 
cd htpasswd/
htpasswd -c /home/user/htpasswd/.htpasswd kasten  
#輸入密碼之後
cat .htpasswd  
```


接著, 透過以下指令執行 Kasten K10 的安裝動作, 這些參數會針對 Kasten 與指定的 namespace 啟動 Kanister Sidecar 功能  
並且使用L7+htpasswd的功能  
 | 參數名稱 | 參數  |
|-------|-------|
| --namespace | kasten-io |  
| ingress.create | true |  
| ingress.class | contour |  
| ingress.host | kastendemo5.com(設為你的需要) |  
| auth.basicAuth.enabled | true |  
| auth.basicAuth.htpasswd | htpasswd產生的值 |  

``` 
helm install k10 kasten/k10 --namespace=kasten-io --set ingress.create=true --set ingress.class=contour --set ingress.host=kastendemo5.com --set auth.basicAuth.enabled=true --set auth.basicAuth.htpasswd='example:$apr1$qrAVXu.v$Q8YVc50vtiS8KPmiyrkld0' --set injectKanisterSidecar.enabled=true --set-string injectKanisterSidecar.namespaceSelector.matchLabels.k10/injectKanisterSidecar=true 
``` 
