# u20s安装MicroK8s
ubuntu-20.04.2-preinstalled-server-arm64+raspi  
MicroK8s适合离线开发、原型开发和测试，尤其是运行VM作为小、便宜、可靠的k8s用于CI/CD。支持arm架构，也适合开发 IoT 应用，通过 MicroK8s 部署应用到小型Linux设备上。  
1.20.5-arm  
## 环境
### pi
sudo nano /boot/firmware/cmdline.txt cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory  
  
grep mem /proc/cgroups | awk '{ print $4 }'  

###  先安装docker, 用于下载容器  

     
  
  
### 下载snap包，手动安装  1.20.5-arm
microk8s_2097.snap   #180m  
microk8s_core_10862.snap  #90M  

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

### microk8s
sudo microk8s enable dns dashboard storage  


sudo microk8s.start  
sudo microk8s.stop  


sudo microk8s.status  

sudo microk8s.reset  
microk8s status  

microk8s inspect

ssh -L 8001:localhost:8001 ubuntu@u20s.local  
microk8s.kubectl proxy --accept-hosts=.\* --address=0.0.0.0  


$ token=$(microk8s.kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)  
$ microk8s.kubectl -n kube-system describe secret $token  
