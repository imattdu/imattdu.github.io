





### 容器安装及配置



```sh
Starting program: /csapp/test/a.out
warning: Error disabling address space randomization: Operation not permitted
warning: Could not trace the inferior process.
warning: ptrace: Function not implemented
During startup program exited with code 127
```







```sh
docker run --platform=linux/amd64 -it -v /Users/matt/workspace/cs/csapp:/csapp --network host ubuntu:20.04 /bin/bash




docker只有以--security-opt seccomp=unconfined的模式运行container才能利用GDB调试
# --security-opt seccomp=unconfined
# privileged root权限


docker run --platform=linux/amd64 --privileged=true --cap-add=CAP_SYS_PTRACE --security-opt seccomp=unconfined --name=csapp3 -it -v /Users/matt/workspace/cs/csapp:/csapp --network host ubuntu:20.04 /bin/bash



```

-----



```sh

apt update

apt install sudo
sudo apt install build-essential



```





这将安装一些必要的编译工具，如gcc（GNU C编译器）、g++（GNU C++编译器）和make等。



```sh
sudo apt install gcc-multilib
```



```sh
sudo apt install gdb

sudo apt install vim
```





gcc --version
