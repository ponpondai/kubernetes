# 使用kubernetes

- 確認k8s是否啟動
>$ kubectl get pod -n kube-system
```
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6b4c6ff56f-rl72w   1/1     Running   0          2m38s
calico-node-4pm6m                          1/1     Running   0          2m38s
coredns-66bff467f8-49v6z                   1/1     Running   0          7m35s
coredns-66bff467f8-sl522                   1/1     Running   0          7m35s
etcd-                                 1/1     Running   1          7m43s
kube-apiserver-                       1/1     Running   1          7m43s
kube-controller-manager-            1/1     Running   1          7m43s
kube-proxy-22sf7                           1/1     Running   1          7m35s
kube-scheduler-                       1/1     Running   1          7m43s
```
- 確認叢集主機狀況
>$ kubectl get nodes
```
NAME   STATUS     ROLES    AGE   VERSION
mas01    Ready    master   15m   v1.18.5
wka01   NotReady  worker   95s   v1.18.5

```
## 常用命令
### 命令式起pod
- 起一個pod image是hazelcast/hazelcast 並給pod一個port號5701
>$ kubectl run hazelcast --image=hazelcast/hazelcast --port=5701
- -it(i=stdin=標準輸入,t=TTY=終端設備)為給pod終端機(image CMD為bash時一定要給終端機) 設定如果pod停止或是損毀時，不會自動重啟
>$ kubectl run -i -t busybox --image=busybox --restart=Never
- '- -'執行default命令(image檔Entrypoint)後面接參數(arg)
>$ kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>
- '- -command'為至換掉image檔的Entrypoint
>命令 kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>

```
描述	        Docker 字段名稱	         Kubernetes 字段名稱
容器執行的命令	 Entrypoint	         command
傳給命令的參數	 Cmd	                 args
```
### 宣告式
- 用yaml檔建立pod
>$ kubectl create -f XXX.yaml
- 將檔案cat出來導入create創造pod 
>$ cat pod.json | kubectl create -f -
- 將檔案以 -o後指定格式進行vi編輯，然後以編輯內容產生pod
>$ kubectl create -f docker-registry.yaml --edit -o json

### yaml template
```yaml=
#api版本
apiVersion: v1
#執行物件
kind: Pod
#物件資訊
metadata:
  name: nna
  labels:
    dt: hdp
    #pod資訊
spec:
  hostname: nnaˋ之
  subdomain: hdp 
  #貨櫃資訊
  containers:
  - name: nna
  #貨櫃使用image
    image: dt210.img
    #image是否使用最新版本Always/Never
    imagePullPolicy: Never
    #使用終端機
    stdin: true
    tty: true
    #用yaml給container起用hostport功能
    ports:
      - containerPort: 22
        hostPort: 22101
    command: [/bin/bash]
    #pod裡面資料夾位置
    volumeMounts:
    - mountPath: "/home/bigred/nn"
      name: nn
    - mountPath: "/home/bigred/sn"
      name: sn
    - name: hdpetc
      mountPath: /opt/hadoop-2.10.1/etc/hadoop/
  #本機要儲存的地點
  volumes:
  - name: nn
    hostPath:
      path: /opt/nna/nn
  - name: sn
    hostPath:
      path: /opt/nna/sn
#把configmap這個物件，變成一種volume，存放叢集設定檔
  - name: hdpetc
    configMap:
      name: hdpetc
      #查詢dns時會新增以下domain name查詢
  dnsConfig:
    searches:
    - hdp.default.svc.dt.io
    #選擇pod產生在哪一台叢集電腦
  nodeSelector:
    kubernetes.io/hostname : lcs81
```


