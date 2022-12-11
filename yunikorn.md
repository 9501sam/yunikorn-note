# 下載 yunikorn
以下指令都是在 master 上執行

1. 下載程式碼
2. make image
3. 使用 helm 把 image 佈署到 kubernetes 上

### 1. 下載程式碼
先把這幾個 repo clone 下來到一個資料夾裡

[https://github.com/apache/yunikorn-core](https://github.com/apache/yunikorn-core)
[https://github.com/apache/yunikorn-scheduler-interface](https://github.com/apache/yunikorn-scheduler-interface)
[https://github.com/apache/yunikorn-k8shim](https://github.com/apache/yunikorn-k8shim)
[https://github.com/apache/yunikorn-release](https://github.com/apache/yunikorn-release)

```sh
mkdir yunikorn
cd yunikorn/
git clone https://github.com/apache/yunikorn-core
git clone https://github.com/apache/yunikorn-scheduler-interface
git clone https://github.com/apache/yunikorn-k8shim
git clone https://github.com/apache/yunikorn-release
```

### 2. make image
參考 [https://github.com/apache/yunikorn-k8shim](https://github.com/apache/yunikorn-k8shim)
2.1 更改 ```yunikorn-k8shim/``` 資料夾裡的 ```go.mod```
```go
// ... //
require (
  // ... //
	github.com/apache/yunikorn-core v1.0.0-1
  // ... //
)
replace (
  github.com/apache/yunikorn-core => ../yunikorn-core // 新增這一行！
  // ... //
)
```
新增那一行，在編譯時才會使用電腦裡的程式碼做編譯，否則他會使用 github 上面的程式碼
因為之後可能會修改 yunikorn-core 裡的程式碼，所以會希望他用電腦裡的程式碼做編譯

2.2
使用
```sh
make image
```
可以編譯出一個 image  
編譯出來的 image 可以用 ```docker images``` 察看

### 3. 使用 helm 把 image 佈署到 kubernetes 上
參考 [https://github.com/apache/yunikorn-release](https://github.com/apache/yunikorn-release)

3.1
先[下載 helm](https://helm.sh://helm.sh/) (可以選擇 snap 的方式)

3.2 修改 ```value.yaml```
進入到 ```yunikorn-release/helm-charts/yunikorn``` 修改裏面的 ```value.yaml```
把檔案中
```
pullPolicy: Always
```
的地方，改成
```
pullPolicy: Never
```
原因：原本的 ```pullPolicy: Always``` 代表使用 docker hub 上面的 image，可是這裡是希望他用剛剛在 ```yunikorn-k8shim/``` 中使用 ```make image``` 編譯出來的 image
所以要使用 ```pullPolicy: Never```
註: ```web``` 的那一個可以使用 ```pullPolicy: Always``` 沒關係

3.3 
[修改 yunikorn-release/helm-charts/yunikorn/templates/ 裡面的檔案](https://github.com/9501sam/yunikorn-release/commit/dda631f71ea25370ac6e6b3807521b6b251e99e5)
[https://github.com/9501sam/yunikorn-release/tree/project/helm-charts/yunikorn/templates](https://github.com/9501sam/yunikorn-release/tree/project/helm-charts/yunikorn/templates)
新增 ```nodeName: lab``` 那裡 (```lab``` 要改成 host name)
原因：在 ```yunikorn-k8shim/``` 中 make 出來的 image 只有存在於一台電腦(master)，所以要指定 ```yunikorn``` 佈署在 master 上

3.4
```
helm install yunikorn . -n yunikorn
```
就可以把 yunikorn 佈署上去了
可以用
```
kubectl get pod -n yunikorn
```
察看

```
helm uninstall yunikorn -n yunikorn
```
可以 uninstall

可以參考
[https://yunikorn.apache.org/docs/developer_guide/build](https://yunikorn.apache.org/docs/developer_guide/build)
