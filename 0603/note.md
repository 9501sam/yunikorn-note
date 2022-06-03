## shim <--> interface <--> rmproxy -> UpdateAllocation
* suppose that after application there would be an allocation(?
* so leave it alone

## 期望可以達到的功能
1. 在建立好 application 之後，還可以再繼續修改這個 application 的 resource 分配的量
2. 可以先嘗試看看：在拿到了 application 的 request 之後，直接給他兩倍的 resource
* 應該要嘗試切入的程式碼片斷  
    * 針對第 2. 點，應該可以先假建立好 application 之後，之後的 allocation rmproxy 會幫忙處理好  
        * 而針對建立 application 的過程，應嗨往 partition 的地方去看
    * 針對第 1. 點，應開要考慮
        * shim 以及 core 的溝通過程
        * 應該由 shim 還是 core 去做修改 allocation 的動作?

## partition 在 application 過來時所扮演的角色？
* 應該要從 handleRMUpdateApplicationEvent() 開始看  
- [ ] 就很直接的在 partition 附近給 appllication 兩倍的 看看
* should be at placement manager

## yunikorn 的一些功能
* 先看看 quota 如何使用
