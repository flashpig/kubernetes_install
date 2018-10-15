# kubernetes_install
通过kubeadm安装kubernetes,使用阿里云资源  
参考文章:https://www.kubernetes.org.cn/4619.html

1.使用阿里云的仓库  

    https://opsx.alibaba.com/mirror  
    安装docker  
    # step 1: 安装必要的一些系统工具
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
    # Step 2: 添加软件源信息
    sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    # Step 3: 更新并安装 Docker-CE
    sudo yum makecache fast
    sudo yum -y install docker-ce
    # Step 4: 开启Docker服务
    sudo service docker start

    # 注意：
    # 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
    # vim /etc/yum.repos.d/docker-ce.repo
    #   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
    #
    # 安装指定版本的Docker-CE:
    # Step 1: 查找Docker-CE的版本:
    # yum list docker-ce.x86_64 --showduplicates | sort -r
    #   Loading mirror speeds from cached hostfile
    #   Loaded plugins: branch, fastestmirror, langpacks
    #   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
    #   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
    #   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
    #   Available Packages
    # Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
    # sudo yum -y install docker-ce-[VERSION]  

    安装kubernetes组件: kubelet, kubeadm, kubectl  
    cat <<EOF > ~/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF

    sudo cp ./kubernetes.repo /etc/yum.repos.d/kubernetes.repo
    setenforce 0
    yum makecache fast
    yum install -y kubelet kubeadm kubectl
    systemctl enable kubelet && systemctl start kubelet

2.使用阿里云容器镜像服务,构建必要的镜像  

    # kubernetes
    k8s.gcr.io/kube-apiserver:v1.12.0
    k8s.gcr.io/kube-controller-manager:v1.12.0
    k8s.gcr.io/kube-scheduler:v1.12.0
    k8s.gcr.io/kube-proxy:v1.12.0
    k8s.gcr.io/etcd:3.2.24
    k8s.gcr.io/pause:3.1

    # network and dns
    quay.io/coreos/flannel:v0.10.0-amd64
    k8s.gcr.io/coredns:1.2.2

    # helm and tiller
    gcr.io/kubernetes-helm/tiller:v2.11.0

    # nginx ingress
    quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
    k8s.gcr.io/defaultbackend:1.4

    # dashboard and metric-sever
    k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
    gcr.io/google_containers/metrics-server-amd64:v0.3.0

3.使用docker拉取镜像到本地  

    #拉取镜像
    docker pull 
    #为镜像创建标签
    docker tag 
    #删除镜像
    docker rmi
    
    sudo docker pull registry.cn-beijing.aliyuncs.com/flashpig8014/kube-apiserver-amd64:v1.12.1
    sudo docker pull registry.cn-beijing.aliyuncs.com/flashpig8014/kube-controller-manager-amd64:v1.12.1
    sudo docker pull registry.cn-beijing.aliyuncs.com/flashpig8014/kube-scheduler-amd64:v1.12.1
    sudo docker pull registry.cn-beijing.aliyuncs.com/flashpig8014/kube-proxy-amd64:v1.12.1
    sudo docker pull registry.cn-beijing.aliyuncs.com/flashpig8014/pause-amd64:v3.1
    sudo docker pull registry.cn-beijing.aliyuncs.com/flashpig8014/etcd-amd64:v3.2.24
    sudo docker pull registry.cn-beijing.aliyuncs.com/flashpig8014/coredns:1.2.2
    sudo docker pull registry.cn-beijing.aliyuncs.com/flashpig8014/flannel:0.10.0

    sudo docker tag registry.cn-beijing.aliyuncs.com/flashpig8014/kube-apiserver-amd64:v1.12.1 k8s.gcr.io/kube-apiserver:v1.12.1
    sudo docker tag registry.cn-beijing.aliyuncs.com/flashpig8014/kube-controller-manager-amd64:v1.12.1 k8s.gcr.io/kube-controller-manager:v1.12.1
    sudo docker tag registry.cn-beijing.aliyuncs.com/flashpig8014/kube-scheduler-amd64:v1.12.1 k8s.gcr.io/kube-scheduler:v1.12.1
    sudo docker tag registry.cn-beijing.aliyuncs.com/flashpig8014/kube-proxy-amd64:v1.12.1 k8s.gcr.io/kube-proxy:v1.12.1
    sudo docker tag registry.cn-beijing.aliyuncs.com/flashpig8014/pause-amd64:v3.1 k8s.gcr.io/pause:3.1
    sudo docker tag registry.cn-beijing.aliyuncs.com/flashpig8014/etcd-amd64:v3.2.24 k8s.gcr.io/etcd:3.2.24
    sudo docker tag registry.cn-beijing.aliyuncs.com/flashpig8014/coredns:1.2.2 k8s.gcr.io/coredns:1.2.2
    sudo docker tag registry.cn-beijing.aliyuncs.com/flashpig8014/flannel:0.10.0 quay.io/coreos/flannel:v0.10.0-amd64

    sudo docker rmi registry.cn-beijing.aliyuncs.com/flashpig8014/kube-apiserver-amd64:v1.12.1
    sudo docker rmi registry.cn-beijing.aliyuncs.com/flashpig8014/kube-controller-manager-amd64:v1.12.1
    sudo docker rmi registry.cn-beijing.aliyuncs.com/flashpig8014/kube-scheduler-amd64:v1.12.1
    sudo docker rmi registry.cn-beijing.aliyuncs.com/flashpig8014/kube-proxy-amd64:v1.12.1
    sudo docker rmi registry.cn-beijing.aliyuncs.com/flashpig8014/pause-amd64:v3.1
    sudo docker rmi registry.cn-beijing.aliyuncs.com/flashpig8014/etcd-amd64:v3.2.24
    sudo docker rmi registry.cn-beijing.aliyuncs.com/flashpig8014/coredns:1.2.2
    sudo docker rmi registry.cn-beijing.aliyuncs.com/flashpig8014/flannel:0.10.0
    
4.配置系统,为kubeadmin init做准备  

    swapoff -a
    vm.swappiness=0
    
    /etc/hosts内容
    192.168.122.242         master
    192.168.122.157         node-1
    192.168.122.16          node-2
    
5.初始化kubernetes  

    kubeadm init \
      --kubernetes-version=v1.12.1 \
      --pod-network-cidr=10.244.0.0/16 \
      --apiserver-advertise-address=192.168.122.242
