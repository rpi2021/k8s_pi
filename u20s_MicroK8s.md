# u20s安装MicroK8s
ubuntu-20.04.2-preinstalled-server-arm64+raspi  
MicroK8s 1.20.5-arm64   适合离线开发、原型开发和测试，尤其是运行VM作为小、便宜、可靠的k8s用于CI/CD。支持arm架构，也适合开发 IoT 应用，通过 MicroK8s 部署应用到小型Linux设备上。  
  
## pi设置
sudo nano /boot/firmware/cmdline.txt  
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory    
   
grep mem /proc/cgroups | awk '{ print $4 }'   

## 先安装docker, 用于下载容器；再导入  
curl -sSL https://get.docker.com | sh   
sudo apt install docker-compose  
sudo usermod -aG docker ubuntu  

docker pull mirrorgooglecontainers/pause-arm64:3.1  
docker tag mirrorgooglecontainers/pause-arm64:3.1 k8s.gcr.io/pause:3.1  
#删除 mirrorgooglecontainers 的相关 tag  
docker rmi mirrorgooglecontainers/pause-arm64:3.1  
  
mkdir pause_arm  
cd pause_arm  
docker save k8s.gcr.io.pause > pause.tar    
  
tar xvf pause.tar    #容器标签错  
rm pause.tar  
  
修改amd64 为arm64；再打包  
tar -cf ~/pause-arm64.tar *  
cd ~/  

microk8s ctr image import pause-arm64.tar   
   

https://hub.docker.com/layers/mirrorgooglecontainers/metrics-server-arm64

k8s.gcr.io/metrics-server-arm64      v0.3.6   
docker save k8s.gcr.io/metrics-server:3.1 > metrics-server.tar  
同样解压改标  
microk8s ctr image import metrics-server-arm64.tar  
  
  
microk8s ctr images list
  
## 下载snap包，手动安装  1.20.5-arm64
microk8s_2097.snap                #180m  
microk8s_core_10862.snap           #90m  

手动安装这种方式缺少一个.assert文件， 需要额外加一个参数--dangerous，另外microk8s的这个镜像需要添加--classic(忘记时，会有提示) 
    
#snap install microk8s --classic --channel=1.20/stable  
sudo snap install microk8s_core_10862.snap --classic --dangerous    
sudo snap install microk8s_2097.snap --classic --dangerous   


snap list  
Name      Version   Rev    Tracking       Publisher   Notes    
core      16-2.49   x1     -              -           core    
core18    20210309  2002   latest/stable  canonical✓  base    
juju      2.8.10    x1     -              -           classic   
lxd       4.0.4     19040  4.0/stable/…   canonical✓  -   
microk8s  v1.20.5   x1     -              -           classic   
snapd     2.49.2    11584  latest/stable  canonical✓  snapd   
  
sudo snap info core18    
  
sudo usermod -aG microk8s ubuntu 
  
https://microk8s.io/docs  
  
sudo chown -f -R $USER ~/.kube  
sudo chown -f -R $USER ~/snap  
  
#sudo snap remove microk8s --purge

## microk8s
sudo microk8s enable dns dashboard storage  


sudo microk8s.start  
sudo microk8s.stop  


sudo microk8s.status  

sudo microk8s.reset  
microk8s status  
  

microk8s.kubectl get nodes  
NAME   STATUS   ROLES    AGE   VERSION  
u20s   Ready    <none>   21h   v1.20.5-34+057bda1e79d450   
  
microk8s kubectl get services  
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE  
kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   4h26m  
  
microk8s kubectl get ns  
NAME                 STATUS   AGE  
kube-system          Active   21h  
kube-public          Active   21h  
kube-node-lease      Active   21h  
default              Active   21h  
container-registry   Active   167m  
  
microk8s kubectl get pods  
No resources found in default namespace.  
  
microk8s kubectl create deployment nginx --image=nginx  
  

microk8s.kubectl config view --raw > $HOME/.kube/config  
   
  

microk8s inspect

ssh -L 8001:localhost:8001 ubuntu@u20s.local  
microk8s.kubectl proxy --accept-hosts=.\* --address=0.0.0.0  


$ token=$(microk8s.kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)  
$ microk8s.kubectl -n kube-system describe secret $token  


检查pod状态
kubectl describe pod  [podname]   -n kube-system 


## 问题
kubectl get pods -n kube-system
NAME                                      READY   STATUS     RESTARTS   AGE
calico-kube-controllers-847c8c99d-8shsv   0/1     Pending    0          20m
calico-node-fmmrr                         0/1     Init:0/3   0          20m

mkubectl describe pod calico-node-fmmrr -n kube-system
......
Events:
  Type     Reason                  Age                   From               Message
  ----     ------                  ----                  ----               -------
  Normal   Scheduled               20m                   default-scheduler  Successfully assigned kube-system/calico-node-fmmrr to irving-workstation
  Warning  FailedCreatePodSandBox  10m (x15 over 20m)    kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to get sandbox image "k8s.gcr.io/pause:3.1": failed to pull image "k8s.gcr.io/pause:3.1": failed to pull and unpack image "k8s.gcr.io/pause:3.1": failed to resolve reference "k8s.gcr.io/pause:3.1": failed to do request: Head "https://k8s.gcr.io/v2/pause/manifests/3.1": dial tcp 108.177.125.82:443: i/o timeout
  Warning  FailedCreatePodSandBox  66s (x13 over 9m38s)  kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to get sandbox image "k8s.gcr.io/pause:3.1": failed to pull image "k8s.gcr.io/pause:3.1": failed to pull and unpack image "k8s.gcr.io/pause:3.1": failed to resolve reference "k8s.gcr.io/pause:3.1": failed to do request: Head "https://k8s.gcr.io/v2/pause/manifests/3.1": dial tcp 108.177.97.82:443: i/o timeout


我们只需要从别的地方下载到所需要的镜像 (这里是 k8s.gcr.io/pause:3.1)，保证运行在我们机器上的 Pod 可以正常获取镜像就可以了。具体怎么解决可以参考其他文章，我个人使用的是阿里云镜像加速器。

当你本地 Docker 镜像仓库中有了 k8s.gcr.io/pause:3.1 这个镜像之后，Pod 却仍然无法获取镜像，这是怎么回事？

原来 Microk8s 使用的 CRE 是 containerd，我们需要再将 Docker 镜像仓库里的镜像放到 Microk8s 使用的镜像仓库里去：

docker save k8s.gcr.io/pause:3.1 > pause.tar
microk8s ctr image import pause.tar
这下再看 pod 状态已经都正常了：

NAME                                      READY   STATUS    RESTARTS   AGE
calico-node-fmmrr                         1/1     Running   1          60m
calico-kube-controllers-847c8c99d-8shsv   1/1     Running   0          60m