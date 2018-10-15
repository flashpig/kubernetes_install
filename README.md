# kubernetes_install
通过kubeadm安装kubernetes,使用阿里云资源

1.使用阿里云的仓库
    https://opsx.alibaba.com/mirror
    安装docker
    安装kubernetes组件: kubelet, kubeadm, kubectl
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
