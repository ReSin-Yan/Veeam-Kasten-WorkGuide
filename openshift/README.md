# 安裝說明  
請參考以下安裝參數設定 
會標明必須或是需要討論  
安裝環境為openshift  
版本為5.0.0  
安裝方式為AirGapped  
License 五個節點之內為free，從介面輸入License  
其餘參數會主要針對雲端環境(Azure,vsphere,aws)，或是Kasten的其餘服務(Promethues,Grafana,Kanister)  
這邊只擷取可能會用到的部分  

## 必要的參數  
### 安裝指令  
```
helm install k10 k10-5.0.0.tgz 
```
設定namespace的scc(OCP的特別技術)  
```
--set scc.create=true
```

### 可以討論的參數  

如果客戶可以配合建立project kasten-io就輸入已下，不行則填入客戶提供的project name  
```
--namespace=kasten-io
```

已下兩個參數為volumesnapshot不支援的情形下的替代方案  
第二行為根據namespace來進行設定，POC可以不用更改，但是實務上建議改成第三行by Object啟用  
```
--set injectKanisterSidecar.enabled=true 
--set-string injectKanisterSidecar.namespaceSelector.matchLabels.k10/injectKanisterSidecar=true
--set-string injectKanisterSidecar.objectSelector.matchLabels=true
```

已下參數主要為設定AirGapped的情境下，repo的位置，如果非安裝包的路徑，只需要修改[kastenrepo.veeam.com/kasten]，其他都是必須  
```
--set global.airgapped.repository=kastenrepo.veeam.com/kasten 
--set global.upstreamCertifiedImages=true 
```
ingress網路 or LoadBalance  
網路服務此段會根據使用者有沒有L4 or L7的服務來進行設定  
考量到的點是，POC情境下，使用者不接受tunnel的方式or服務正式上線  
L4 LoadBalance  
```
--set externalGateway.create
--set externalGateway.annotations
--set externalGateway.fqdn.name
--set externalGateway.fqdn.type
--set externalGateway.awsSSLCertARN
```
L7 ingress  
```
--set ingress.create
--set ingress.class
--set ingress.host
--set ingress.urlPath
--set ingress.annotations
--set ingress.tls.enabled
--set ingress.tls.secretName
--set ingress.pathType
```

### 非必要但是建議放入的參數  
Kasten備份出來的Config會放到由CSI產生的volume內(預設是20GB)，POC應該不影響，但是實務上要調整成大一點的空間  
如果CSI本身有支援extend，那就可以在之後更改，如果沒有，就必須要在一開始就設定好大小  
```
--set global.persistence.catalog.size=200Gi 
--set global.persistence.job.size=200Gi
--set global.persistence.logging.size=200Gi
```

## 零壹OCP安裝指令參考  
```
helm install k10 k10-5.0.0.tgz --namespace=kasten-io \
--set injectKanisterSidecar.enabled=true \
--set-string injectKanisterSidecar.namespaceSelector.matchLabels.k10/injectKanisterSidecar=true \
--set scc.create=true \
--set global.airgapped.repository=kastenrepo.veeam.com/kasten \
--set global.upstreamCertifiedImages=true \
--set global.persistence.catalog.size=50Gi \
--set global.persistence.job.size=50Gi \
--set global.persistence.logging.size=50Gi
```
之後使用tunnel的方式導向連線端  
