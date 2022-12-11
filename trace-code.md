# 如何追蹤程式碼
* 建議不要單純用看的，嘗試修該程式碼並且觀察行為上的改變會比較容易理解
* 雖然寫程式時會希望模組之間的相依性降低，但讀程式碼時卻要儘量去觀察模組之間的互動(個人感想)
* 一些粗略的流程可以參考我之前的筆記

## 追程式碼的工具
1. 使用 github
2. 有在用 ```vim``` 了話可以搭配 ```ctags``` 以及 ```cscope```，我覺得非常好用但是學習成本有點高
  * 想學 ```vim``` 可以參考[即將失傳的古老技藝 Vim](https://www.youtube.com/watch?v=mPVwS8gjDVI&list=PLBd8JGCAcUAH56L2CYF7SmWJYKwHQYUDI&index=1)，個人覺得好用，但非必要

## 如何觀察行為上的改變
1. 修改程式碼並且重新執行 [yunikorn.md](./yunikorn.md) 的 ```make image``` 以及 ```helm install``` 的步驟  
程式中有很多像是  
```go
log.Logger().Info(......)
```
的寫法，用這個語法可以新增一行 log  

2. 取得 pod 的名稱：(每次的名稱都不同)  
```sh
$ kubectl get pod -n yunikorn
yunikorn-scheduler-xxxxxxxx   1/1     Running       2 (4h20m ago)   2d23h
```

3. 取得 log：
```sh
$ kubectl logs yunikorn-scheduler-xxxxxxxx -n yunikorn yunikorn-scheduler-k8s
```
