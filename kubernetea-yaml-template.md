---
tags: K8S
---
# K8S yaml template

## 撰寫K8S 常見物件 yaml檔 的範例

## 1. POD
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: POD
spec:
  hostname: phn 
  containers:
  - name: cd
    image: busybox
```
## 2. Deployment Object
``` yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: s1.dep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: s1.pod
  template:
    metadata:
      labels:
        app: s1.pod
    spec:
      containers:
      - name: derbyapp
        image: busybox
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 8888
```
## 3. Volume
### 3-1 Hostpath
- 這是 PersistantVolume(pv), 它會自動產生 /opt/hostpath 目錄, pod-hp 這個 POD 如被刪除, /opt/hostpath 會被保留
```yaml=
kind: Pod
apiVersion: v1
metadata:
  name: pod-hp
spec:
  containers:
    - name: pod-hp
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: hp-volume
  volumes: 
    - name: hp-volume
      hostPath:                        
        path: /opt/hostpath
```
### 3-2 Local
- 只要是 Local Volume 一定要透過 Persistent Volume 定義檔來使用, 在定義檔中, 還一定要宣告 nodeAffinity，在主機, 必需自行先建立/opt/local 目錄
```yaml=
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  local:
    path: "/opt/local"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - cg60(主機名稱) ' > pv-local.yaml 
```
### 3-3 PersistantVolumeClaim(PVC)
- 建立 PVC 會立即搜尋可用的 PV, 然後建立連接
- storageClassName: "" K3S必須宣告這行
```yaml=
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-local
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ""  
  resources:
    requests:
      storage: 3Gi
```
### 3-3-1 PVC的使用
```yaml=
kind: Pod
apiVersion: v1
metadata:
  name: pod-pvc1
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
       claimName: pvc-local
  containers:
    - name: pod-pvc1
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-storage
```
## 4. Service
- 沒有宣告 Service Type, K8S Service 內定使用 ClusterIP, 這個 Cluster IP 只有 K8S 叢集中的 Node (實體主機)可以存取
### 4-1 ClusterIP
``` yaml
kind: Service
apiVersion: v1
metadata:
  name: s1-service
spec:
  selector:
    app: s1.pod 
  ports:
    - port: 9999
      targetPort: 8888
```
### 4-2 External IP
- 修改 IP 位址, 可以是 Kubernetes 叢集中任何一部實體主機對外的 IP 位址, 包括 Master 主機
```yaml=
kind: Service
apiVersion: v1
metadata:
  name: myextip
spec:
  externalIPs:
  - $IP
  selector:
    app: s1.pod
  ports:
  - port: 8080
    targetPort: 8888
```
### 4-3 NodePort

```yaml=
apiVersion: v1
kind: Service
metadata:  
  name: my-nodeport-service
spec:
  selector:
    app: s1.pod
  type: NodePort
  ports:
  - port: 9999
    targetPort: 8888
    nodePort: 30036
    protocol: TCP 

```
## 5.  Horizontal Pod Autoscaler
- 設定 50 代表 HPA 將會維持每個 Pod 的平均 CPU 使用率為 50 %
```yaml=
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-sp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-dep
  minReplicas: 2
  maxReplicas: 6
  targetCPUUtilizationPercentage: 50
```
- 要使用 HPA 就一定要宣告資源請求
```yaml=
apiVersion: apps/v1
kind: Deployment
	::
      containers:
	::
        resources:
          requests:
            cpu: 70m

```
## 6. Persistant Volume
### 6-1 HostPath PersistentVolume
- 它會自動產生 /opt/hostpath 目錄, pod-hp 這個 POD 如被刪除, /opt/hostpath 會被保留
- 不會產生 PV Object
- 
```yaml=
kind: Pod
apiVersion: v1
metadata:
  name: pod-hp
spec:
  containers:
    - name: pod-hp
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: hp-volume
  volumes:
    - name: hp-volume
      hostPath:
        path: /opt/hostpath
```
### 6-2 Local PersistentVolume
- Local與Host最大差別，Local一定要透過PVC宣告檔來使用
- 只要是 Local Volume 一定要透過 Persistent Volume 定義檔來使用, 在定義檔中, 還一定要宣告 nodeAffinity
- 必需自行先建立 /opt/local 目錄
```yaml=
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  local:
    path: "/opt/local"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - #主機名稱
```