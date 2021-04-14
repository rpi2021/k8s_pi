# tf卡32G u20s安装MicroK8s 
ubuntu-20.04.2-preinstalled-server-arm64+raspi  
不要在低速卡上安装 k8s

Linux us22 5.4.0-1033-raspi #36-Ubuntu SMP PREEMPT Thu Mar 25 19:00:51 UTC 2021 aarch64 aarch64 aarch64 GNU/Linux  

sudo rm -rf /var/lib/dpkg/lock-frontend  
sudo rm -rf /var/lib/dpkg/lock  
sudo rm -rf /var/cache/apt/archives/lock  

速度还行，未改源  
sudo apt update && sudo apt upgrade -y    #49 120m  
  
  
sudo nano /etc/hostname  
us22  
  
sudo nano /etc/hosts  
127.0.0.1 localhost  

##The following lines are desirable for IPv6 capable hosts  
::1 ip6-localhost ip6-loopback  
fe00::0 ip6-localnet  
ff00::0 ip6-mcastprefix  
ff02::1 ip6-allnodes  
ff02::2 ip6-allrouters  
ff02::3 ip6-allhosts  

127.0.1.1 us22  

network-config  
   
sudo nano /etc/netplan/50-cloud-init.yaml   
启用配置  
sudo netplan try    
sudo netplan apply    
systemctl daemon-reload    
  
## pi设置
sudo nano /boot/firmware/cmdline.txt  
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory    
   
grep mem /proc/cgroups | awk '{ print $4 }'   
   
sudo apt install curl wget vim  unzip unrar git screen tmux nmon 
sudo apt install wireless-tools avahi-daemon samba   #100m 
##Adding group `sambashare' (GID 121) ...  
  
  
snap list  
Name    Version   Rev    Tracking       Publisher   Notes  
core18  20210309  2002   latest/stable  canonical✓  base  
lxd     4.0.4     19040  4.0/stable/…   canonical✓  -  
snapd   2.49.2    11584  latest/stable  canonical✓  snapd  
  
  
## 下载snap包，手动安装  1.20.5-arm64
microk8s_2097.snap                #180m  
microk8s_core_10956.snap          #90m  

手动安装这种方式缺少一个.assert文件， 需要额外加一个参数--dangerous，另外microk8s的这个镜像需要添加--classic(忘记时，会有提示) 
    
#sudo snap install microk8s --classic --channel=1.20/stable  
sudo snap install microk8s_core_10956.snap --classic --dangerous    
sudo snap install microk8s_2097.snap --classic --dangerous   


sudo snap install juju_15846.snap --classic --dangerous
juju 2.8.10 installed
  
#sudo apt install juju  

https://juju.is/docs/microk8s-cloud
Juju 是 Canonical 的服务建模和部署工具。 它支持非常广泛的云服务提供商，使您能够轻松地在任何云上部署任何您想要的服务。
此外，Juju 2.0 还支持 LXD，既适用于本地部署，也适合开发，并且可以在云实例或物理机上共同协作。此文将关注本地使用，通过一个没有任何Juju经验的LXD用户来体验。

juju clouds
Since Juju 2 is being run for the first time, downloaded the latest public cloud information.
Only clouds with registered credentials are shown.
There are more clouds, use --all to see them.
You can bootstrap a new controller using one of these clouds...

Clouds available on the client:
Cloud      Regions  Default    Type  Credentials  Source    Description
localhost  1        localhost  lxd   0            built-in  LXD Container Hypervisor

  
  
https://github.com/ubuntu/microk8s/issues/943
sudo microk8s.juju status --debug
Juju is not available for arm64


ERROR No controllers registered.

Please either create a new controller using "juju bootstrap" or connect to
another controller that you have been given access to using "juju register".

sudo microk8s.juju list-clouds


  
#我的集群使用Pi 3s和4s的混合。  
#我发现1.19将安装和工作在4s上，但不是3s。  
#对于3s我不得不使用1.18。  
https://forum.snapcraft.io/t/unable-to-install-microk8s-on-raspberrypi-due-to-configuration-hook-timeout/19877/5  
https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04  
sudo swapon --show  
free -h  
df -h  
  
sudo fallocate -l 1G /swapfile  
ls -lh /swapfile  
sudo chmod 600 /swapfile  
sudo mkswap /swapfile  
sudo swapon /swapfile  
sudo swapon --show  
  
sudo cp /etc/fstab /etc/fstab.bak  
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab  
  
sudo nano /etc/fstab  
LABEL=writable  /        ext4   defaults        0 0  
LABEL=system-boot       /boot/firmware  vfat    defaults        0       1  
/swapfile none swap sw 0 0  
  
  
snap list  
Name      Version    Rev    Tracking       Publisher   Notes  
core      16-2.49.2  x1     -              -           core  
core18    20210309   2002   latest/stable  canonical✓  base  
lxd       4.0.4      19040  4.0/stable/…   canonical✓  -  
microk8s  v1.20.5    x1     -              -           classic  
snapd     2.49.2     11584  latest/stable  canonical✓  snapd    
  
sudo snap info core18  
  
  
 df -h  
Filesystem      Size  Used Avail Use% Mounted on  
udev            406M     0  406M   0% /dev  
tmpfs            91M  5.6M   86M   7% /run  
/dev/mmcblk0p2   30G  3.6G   25G  13% /  
tmpfs           455M     0  455M   0% /dev/shm  
tmpfs           5.0M     0  5.0M   0% /run/lock  
tmpfs           455M     0  455M   0% /sys/fs/cgroup  
/dev/loop0       49M   49M     0 100% /snap/core18/2002  
/dev/loop2       62M   62M     0 100% /snap/lxd/19040  
/dev/loop1       49M   49M     0 100% /snap/core18/1949  
/dev/loop3       27M   27M     0 100% /snap/snapd/10709  
/dev/loop4       29M   29M     0 100% /snap/snapd/11584  
/dev/mmcblk0p1  253M  119M  134M  47% /boot/firmware  
tmpfs            91M     0   91M   0% /run/user/1000  
/dev/loop5       88M   88M     0 100% /snap/core/x1  
/dev/loop6      178M  178M     0 100% /snap/microk8s/x1  
  
  
sudo usermod -aG microk8s ubuntu  
  
https://microk8s.io/docs  
  
sudo chown -f -R $USER ~/.kube  
sudo chown -f -R $USER ~/snap  
  
  
#导入两容器，arm64标签错，需先修改
microk8s ctr image import pause-arm64.tar   
microk8s ctr image import metrics-server-arm64.tar  
  
  
microk8s ctr images list
REF                                                                     TYPE                                                 DIGEST                                                                  SIZE      PLATFORMS   LABELS                          
k8s.gcr.io/metrics-server-arm64:v0.3.6                                  application/vnd.docker.distribution.manifest.v2+json sha256:0546946c43b5f659b99cb85ee77f16217e2b8ddf827dd1c0bdf4666b2b62aa6e 37.7 MiB  linux/arm64 io.cri-containerd.image=managed 
k8s.gcr.io/pause:3.1                                                    application/vnd.docker.distribution.manifest.v2+json sha256:6bd8184e563c993ef03907e08365043bef15224ec9cc05fb7deb78941702db77 516.5 KiB linux/arm64 io.cri-containerd.image=managed 
sha256:7d100074337d3fda1b3d6b93c789a9398100be75df8f57057d49a2d060f0c94b application/vnd.docker.distribution.manifest.v2+json sha256:0546946c43b5f659b99cb85ee77f16217e2b8ddf827dd1c0bdf4666b2b62aa6e 37.7 MiB  linux/arm64 io.cri-containerd.image=managed 
sha256:c3f1477624fef65f3485375a233cd8a3d4cd93c812a818c069c9f194fda9e06c application/vnd.docker.distribution.manifest.v2+json sha256:6bd8184e563c993ef03907e08365043bef15224ec9cc05fb7deb78941702db77 516.5 KiB linux/arm64 io.cri-containerd.image=managed 
  

  
  

  
#sudo snap remove microk8s --purge

## microk8s
sudo microk8s enable dns dashboard storage  

##启用插件
microk8s.enable dns dashboard ingress
##查看进度
microk8s.kubectl get pods --all-namespaces
##查看详情
microk8s.kubectl describe pod --all-namespaces


Enabling DNS
Applying manifest
serviceaccount/coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
clusterrole.rbac.authorization.k8s.io/coredns created
clusterrolebinding.rbac.authorization.k8s.io/coredns created
Restarting kubelet
DNS is enabled
Enabling Kubernetes Dashboard
Enabling Metrics-Server
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
Warning: apiregistration.k8s.io/v1beta1 APIService is deprecated in v1.19+, unavailable in v1.22+; use apiregistration.k8s.io/v1 APIService
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/microk8s-admin created
Metrics-Server is enabled
Applying manifest
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created

If RBAC is not enabled access the dashboard using the default token retrieved with:

token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
microk8s kubectl -n kube-system describe secret $token

In an RBAC enabled setup (microk8s enable RBAC) you need to create a user with restricted
permissions as shown in:
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

Enabling default storage class
deployment.apps/hostpath-provisioner created
serviceaccount/microk8s-hostpath created
clusterrole.rbac.authorization.k8s.io/microk8s-hostpath created
clusterrolebinding.rbac.authorization.k8s.io/microk8s-hostpath created
Error from server: error when creating "/root/snap/microk8s/x1/tmp/temp.storage.yaml": context deadline exceeded



sudo microk8s.start  
sudo microk8s.stop  


sudo microk8s.status  

sudo microk8s.reset  
microk8s status  
  


sudo snap alias microk8s.kubectl kubectl  
snap unalias  

microk8s.kubectl get nodes  
NAME   STATUS   ROLES    AGE   VERSION  
us22   NotReady   <none>   4h10m   v1.20.5-34+057bda1e79d450
   
  
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
  
microk8s.kubectl config view --raw > $HOME/.kube/config  
  
microk8s kubectl get pods  
No resources found in default namespace.  
  
microk8s kubectl create deployment nginx --image=nginx  
  

microk8s.kubectl config view --raw > $HOME/.kube/config  
microk8s.kubectl cluster-info dump

关于 CrashLoopBackOff 问题

#sudo iptables -P FORWARD ACCEPT  
##1.17版本是 cni0; 之前版本是 cnr0, 参照官网 TroubleShooting  
#sudo ufw allow in on cni0 && sudo ufw allow out on cni0  
#sudo ufw default allow routed  

sudo ufw allow in on cbr0 && sudo ufw allow out on cbr0
sudo ufw default allow routed
sudo iptables -P FORWARD ACCEPT


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