Last update: 2022/9  
# Veeam-Kasten-WorkGuide

### Veeam簡易介紹     
Kasten簡稱k10，主要功能為提供營運團隊一個方便使用、可擴展性及安全的平台

用來針對k8s平台的備份、復原、以及應用程序移動  


目前授權版本分為  
Enterprise Trial  
Enter prise  

[參考網站](https://www.kasten.io/product/#k10-editions "link")  


### 安裝步驟   
  
## Linux Client 準備  

環境更新及安裝基本套件  
```
sudo apt-get update && sudo apt-get -y upgrade
sudo apt-get -y install vim build-essential curl ssh
sudo apt-get install net-tools
```

安裝Docker engine    
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

確認安裝版本
```
sudo docker --version
```

安裝helm
```
cd
sudo apt-get install -y curl
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
``` 

安裝MINIO  
```
sudo docker run -d -p 9000:9000 -p 9090:9090 --name minio1   -e "MINIO_ROOT_USER=kasten"   -e "MINIO_ROOT_PASSWORD=P@ssw0rd"   -v /mnt/data:/data   --restart=always  minio/minio server /data --console-address ':9090'
```

安裝NFS  
```
sudo apt-get install nfs-kernel-server nfs-common
mkdir nfsshare
sudo chmod -R 777 /home/ubuntu/nfsshare/
```
編輯/etc/exports  
```
sudo vim /etc/exports  
#新增以下
/home/ubuntu/nfsshare/    *(rw,sync,no_root_squash,no_all_squash)
```

重啟服務  
```
/etc/init.d/nfs-kernel-server restart
``` 

安裝kubectl vsphere plugin    
```
unzip vsphereplugin.zip
sudo cp bin/* /usr/local/bin
```

## Kubernetes 操作及環境準備    

確認Kuberentes服務  
已下指令是Tanzu環境登入的指令  
如果是其它Kubernetes的平台，需要確認能夠正常的執行Kubernetes的相關操作  
可以跳轉到下一章節  

登入到Taznu環境  
```
export KUBECTL_VSPHERE_PASSWORD=P@ssw0rd
kubectl vsphere login --server=10.66.99.2 --insecure-skip-tls-verify  --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-name  [輸入姓名]-tkc1
kubectl vsphere login --server=10.66.99.2 --insecure-skip-tls-verify  --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-name  [輸入姓名]-tkc2
kubectl config use-context [輸入姓名]-tkc[x]
```

下載gcallowroot yaml(TKC需要)  
```
sudo apt-get install git
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
    --set nfs.server=[ip] \
    --set nfs.path=/home/ubuntu/nfsshare
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
環境有L4且對外  

```
helm repo add kasten https://charts.kasten.io/
helm repo update
kubectl create namespace kasten-io
```

```
helm install k10 kasten/k10 \
--namespace=kasten-io \
--set injectKanisterSidecar.enabled=true \
--set-string injectKanisterSidecar.namespaceSelector.matchLabels.k10/injectKanisterSidecar=true \
--set global.persistence.catalog.size=50Gi \
--set global.persistence.job.size=50Gi \
--set global.persistence.logging.size=50Gi \
--set externalGateway.create=true  \
--set global.persistence.storageClass=wcppolicy \
--set auth.basicAuth.enabled=true \
--set auth.basicAuth.htpasswd='kasten:$apr1$UtUFc7QC$15rWGptryX75BCJ32X8Hv0' \
--set features.vbrTkgsEnabled=true
```

## 環境設定    

### Location Profile  

### Infrastructure Profiles  

建立PV  
```
tee pv.yaml <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 172.18.19.238
    path: "/home/ubuntu/nfsshare"
  mountOptions:
    - nfsvers=4.2
EOF
```


建立PVC  
```
tee pvc.yaml <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 10Gi
  volumeName: nfs
EOF
```

分別執行  
```
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml -n kasten-io
```

## 測試環境建立    

建立兩個命名空間  
此兩個命名空間將會作來對比於是否具備Kanister的方式
```
kubectl create ns nfs-csi
kubectl create ns vsan-csi
kubectl label namespace nfs-csi k10/injectKanisterSidecar=true
```

根據兩個volume建立不同的部屬環境  

```
cd
git clone https://github.com/ReSin-Yan/Veeam-Kasten-WorkGuide.git
cd Veeam-Kasten-WorkGuide/nfscsi/
kubectl apply -f pre.yaml  -n nfs-csi
kubectl apply -f post.yaml  -n nfs-csi
kubectl get svc -n nfs-csi
```

```
cd 
cd Veeam-Kasten-WorkGuide/vsancsi/
kubectl apply -f pre.yaml  -n vsan-csi
kubectl apply -f post.yaml  -n vsan-csi
kubectl get svc -n vsan-csi
```


### Veeam簡易介紹     
Kasten簡稱k10，主要功能為提供營運團隊一個方便使用、可擴展性及安全的平台

用來針對k8s平台的備份、復原、以及應用程序移動  
