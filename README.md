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
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=172.18.19.226 \
    --set nfs.path=/home/ubuntu/nfs
```


## 安裝說明  
請參考以下安裝參數設定 
會標明必須或是需要討論  
版本為5.x.x  
安裝方式為對外(可以改成air-gapped)  
其餘參數會主要針對雲端環境(Azure,vsphere,aws)，或是Kasten的其餘服務(Promethues,Grafana,Kanister)  
這邊只擷取可能會用到的部分  

設定namespace的scc(OCP需要)  
```
--set scc.create=true
```

### 可以討論的參數  

如果可以配合建立命名空間 kasten-io就輸入以下，不行則填入客戶提供的namespace  
```
--namespace=kasten-io
```

以下兩個參數為volumesnapshot不支援的情形下的替代方案  
第二行為根據namespace來進行設定，POC可以不用更改，但是實務上建議改成第三行by Object啟用  

```
--set injectKanisterSidecar.enabled=true 
--set-string injectKanisterSidecar.namespaceSelector.matchLabels.k10/injectKanisterSidecar=true
--set-string injectKanisterSidecar.objectSelector.matchLabels=true
```

ingress網路 or LoadBalance  
網路服務此段會根據使用者有沒有L4 or L7的服務來進行設定  
考量到的點是，POC情境下，使用者不接受tunnel的方式or服務正式上線  

L4 LoadBalance  
```
--set externalGateway.create=true  
```

L7 ingress  
```
--set ingress.create=true
--set ingress.class=contour
--set ingress.host=kastendemo.com
```

驗證服務相關  
大致上可以分為  
basicAuth  
Open ID  
openshift  
ldap  

POC階段建議使用basicAuth，可以自行創建帳號密碼(需要使用htpasswd創建)進行登入    
產生htpasswd(需要預先安裝)  
```
sudo apt install apache2-utils
cd 
mkdir htpasswd 
cd htpasswd/
htpasswd -c $PWD/.htpasswd kasten  
#輸入密碼之後
cat .htpasswd  
```


```
--set auth.basicAuth.enabled=true 
--set auth.basicAuth.htpasswd='example:$apr1$qrAVXu.v$Q8YVc50vtiS8KPmiyrkld0'

```

以下參數主要為設定AirGapped的情境下，repo的位置，如果非安裝包的路徑，只需要修改[kastenrepo.veeam.com/kasten]，其他都是必須  
```
--set global.airgapped.repository=kastenrepo.veeam.com/kasten 
--set global.upstreamCertifiedImages=true 
```

### 非必要但是建議放入的參數  
Kasten備份出來的Config會放到由CSI產生的volume內(預設是20GB)，POC應該不影響，但是實務上要調整成大一點的空間  
如果CSI本身有支援extend，那就可以在之後更改，如果沒有，就必須要在一開始就設定好大小  
```
--set global.persistence.catalog.size=200Gi 
--set global.persistence.jobs.size=200Gi
--set global.persistence.logging.size=200Gi
```  


### 參考安裝指令參考  
環境有L4  

```
helm install k10 kasten/k10 --namespace=kasten-io \
--set injectKanisterSidecar.enabled=true \
--set-string injectKanisterSidecar.namespaceSelector.matchLabels.k10/injectKanisterSidecar=true \
--set global.persistence.catalog.size=50Gi \
--set global.persistence.job.size=50Gi \
--set global.persistence.logging.size=50Gi
```
之後使用tunnel的方式導向連線端  
