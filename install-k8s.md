![](https://i.imgur.com/rIuapmL.png)


---

![](https://i.imgur.com/l9ejOAw.png)

## k8s啟動時master會在namespace kube-system 產生以下pod
- 
    - API Server: 接收命令，接收 kubctl 命令 (Authentication)
    - Scheduler: 管理 Pod (安排 Pod 去哪 Run)
    - Controller: 照顧 Pod (萬一 Pod 掛點，會找其他 Container 讓他浴火重生)
    - etcd: database
    - calico
        - 可做到企業級網路，可跨區域機房，讓網路透通
        - 原本是 flannel，頂多做到 locla site
    - cordDNS
        - 名稱解析
    - kube proxy


#  kubernetes特點

![](https://i.imgur.com/lpoMVGs.png)
- CRI(ContaunerRuntimeInterface)
裡面有cri-o container-D 

- CNI(ContainerNetworkInterface)

- CSI(ContainerStorageInterface)


## pod特性
![](https://i.imgur.com/lChbLgZ.png)
![](https://i.imgur.com/Ep5JV5z.png)
![](https://i.imgur.com/07eK98t.png)
![](https://i.imgur.com/HX26LYN.png)
![](https://i.imgur.com/S15m6j1.png)

---
## 安裝k8s

### 準備所需套件docker kubelet kubeadm kubectl
#### 需先確認安裝版本
- 找到可用的版本
>$ apt-cache madison kubeadm
- 指定版本
K_VER="1.14.9-00"
- ubuntu 16.04 之後
>$ apt install -y kubelet=${K_VER} kubectl=${K_VER} kubeadm=${K_VER}
- 或者使用 apt-get
>$ apt-get install -y kubelet=${K_VER} kubectl=${K_VER} kubeadm=${K_VER}
sudo apt install -y kubelet kubeadm kubectl
#鎖定版本，避免 apt upgrade 被更新了
#如果是自己安裝的 Base Image，記得一定要做這件事情，不然可能有些 Node 會自己升級 K8s 版本。
>$ apt-mark hold kubelet kubeadm kubectl

## 指定mater主機並安裝
- --service-cidr代表service的網段
- --pod-network-cidr代表pod的網段
>$ sudo kubeadm init --service-cidr 10.98.0.0/24 --pod-network-cidr 10.233.0.0/16 --apiserver-advertise-address #master_IP

## 將 bigred 設成 K8S 管理者
```
$ mkdir -p $HOME/.kube; sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config; sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
>$ sudo reboot

>$ kubectl get nodes
```
NAME   STATUS     ROLES    AGE     VERSION
cg60   NotReady   master   7m36s   v1.19.4
```

## 設定 K8S Master 可以執行 Pod
>$ kubectl taint node #node_name node-role.kubernetes.io/master:NoSchedule-
```
node/#node_name untainted
```
## 安裝 Calico 網路套件
Calico是讓不同電腦之間的pod可以網路互通
>$ kubectl apply -f calico.yaml

yaml檔設定過於複雜待補充

---

## 加入 K8S Worker

- 在 master 終端機操作
>$ echo " sudo `kubeadm token create --print-join-command 2>/dev/null`"
```
sudo kubeadm join 192.168.122.60:6443 --token 4zxa7e.ypdrn7c1ufgjokou     --discovery-token-ca-cert-hash sha256:026977e72c44bf6e4e6e39b73635de8f7a117000d10b9c764f222a51d6d50508 
```
-在 worker終端機執行剛剛拿到的token就可以加入k8s worker

## 設定 K8S Worker 標籤

- 在 master 終端機操作
>$ kubectl get nodes
```
NAME   STATUS     ROLES    AGE   VERSION
cg60    Ready       master    15m   v1.18.5
cg61   NotReady  <none>   95s   v1.18.5
```
-  設定標籤
>$ kubectl label node cg61 node-role.kubernetes.io/worker=worker

>$ kubectl get nodes
```
NAME   STATUS   ROLES    AGE    VERSION
cg60    Ready     master    39m    v1.18.5
cg61    Ready     worker    6m3s   v1.18.5
```
