# 下載 kubernetes

## 最後載完 kubernetes 可以做的事情
在終端機中輸入 ```kubectl``` 可以使用 kubernetes，例如 ```kubectl get node``` 可以看到叢集中有哪些 node

```sh
$ kubectl get node
NAME   STATUS   ROLES                  AGE   VERSION
lab    Ready    control-plane,master   80d   v1.23.0
lab1   Ready    <none>                 80d   v1.23.1
lab3   Ready    <none>                 80d   v1.23.1
```

## 方法一：使用 minikube
只有一台主機時，會建議使用的方法，但我們之後用的都是方法二，老實說我也不太記得怎麼裝了
印象中是按照[https://minikube.sigs.k8s.io/docs/start://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start://minikube.sigs.k8s.io/docs/start/)
並且也有印象[drivers page](https://minikube.sigs.k8s.io/docs/drivers/) 那個連結有點重要
使用 minikube 時，yunikorn 的圖形化介面可能會跑不出來

## 方法二：使用實體電腦
這個方法需要有兩台以上的實體電腦
可以參考 [https://ithelp.ithome.com.tw/articles/10235069](https://ithelp.ithome.com.tw/articles/10235069) 這篇文章
以下有幾點注意事項：
* 文章中的 Step 1-4, 9 做完之後，需要重開機
* 這篇文章要使用的版本是 1.23 版，所以下載時要指定版本 1.23，也就是```sudo apt-get install -y kubelet kubeadm kubectl``` 要特別注意一下，可以參考
  * [How can I see all versions of a package that are available in the archive?](https://askubuntu.com/questions/447/how-can-i-see-all-versions-of-a-package-that-are-available-in-the-archive)
  * [How to install specific version of some package? [duplicate]](https://askubuntu.com/questions/428772/how-to-install-specific-version-of-some-package)
* Step 7 建議到 [kubernetes 的官網](https://v1-23.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
  * 官網的右上角的 versions 可以選擇版本 1.23
  * 按照 **Installing kubeadm, kubelet and kubectl** 下載
* 文章中的**初始化Control Plane**
  * 使用 Calico CNI: 注意 ```/16``` 要改成 ```/24``` 才行(可能是因為是在使用實驗室的內部網路，但我也不是很確定原因)
  ```
sudo kubeadm init --pod-network-cidr=120.108.204.0/24 --apiserver-advertise-address=120.108.204.xx
  ```
  * Flannel CNI 可能要問學長
