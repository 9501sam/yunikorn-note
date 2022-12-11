# 下載 docker
按照[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)安裝即可  

安裝完成之後，應該要用 ```sudo docker``` 才能夠執行 docker  

注意 **Use Docker as a non-privileged user, or install in rootless mode?** 中的描述
按照 [post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)做完，就可以讓 non-root user 也可以使用 docker 了
