# 实验内容

* 安装KLEE，完成官方tutorials。至少完成前三个，有时间的同学可以完成全部一共7个。

# 实验过程

1. 安装docker

   ```shell
   $ sudo apt-get update
   #安装以下包以使apt可以通过HTTPS使用存储库（repository）：
   $ sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
   #添加Docker官方的GPG密钥：
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   #使用下面的命令来设置stable存储库：
   $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   $ sudo apt-get update
   #安装最新版本的Docker CE
   $ sudo apt-get install -y docker-ce
   ```

   <img src="pic\1.png" style="zoom:80%;" />

2. 验证docker安装成功【docker内部运行过慢时参考文末解决办法】

   ```shell
   #查看docker服务是否启动：
   $ systemctl status docker
   #若未启动，则启动docker服务：
   $ sudo systemctl start docker
   #用hello world来测试是否成功
   $ sudo docker run hello-world
   ```

<img src="pic\2.png" style="zoom:80%;" />

3. 在docker中安装klee

```shell
# 启动 docker
systemctl start docker
# 安装 KLEE
docker pull klee/klee:2.0
# 创建一个临时容器(为了测试实验用)
docker run --rm -ti --ulimit='stack=-1:-1' klee/klee:2.0

# 创建一个长期容器
sudo docker run -ti --name=klee_cwx --ulimit='stack=-1:-1' klee/klee
# 退出后可通过名字再次进入
sudo docker start -ai klee_cwx
# 删除长期容器
docker rm klee_container
```

<img src="pic\3.png" style="zoom:80%;" />

4. 【mission1】Testing a small function.

* 在以下目录里能看到一个.c文件，用来判断一个整数的正，负，或者为0.

<img src="pic\4.png" style="zoom:80%;" />

* 其中，klee_make_sybolic是KLEE自带的函数，用来产生符号化的输入。因为KLEE是在LLVM字节码上进行工作，所以我们首先需要将.c编译为LLVM字节码。首先，我们进入到该文件目录（~/klee_src/examples/get_sign）下执行命令【此时会报空间不足的错，参见参考部分解决方法】

```shell
clang -I ../../include -emit-llvm -c -g -O0 -Xclang -disable-O0-optnone get_sign.c
#参数-I是为了编译器找到头文件klee/klee.h,-g是为了在字节码文件中添加debug信息，还有后面的，具体不深究，按照官网推荐来。
```

<img src="pic\5.png" style="zoom:80%;" />

* 同目录下我们会生成一个get-sign.bc的字节码文件，然后进行测试：

```shell
$ klee get_sign.bc
```

<img src="pic\6.png" style="zoom:80%;" />

- 可以看到结果中KLEE给出了总指令数，完整路径和生成的测试案例数。

- 最后，我们看当前目录下多生成了两个文件：**klee-last** 和 **klee-out-0。**其中klee-out-0是本次测试结果，klee-last是最新测试结果，每次测试后覆盖。

- 从中可以看出，该测试函数有3条路径，并且为每一条路径都生成了一个测试例。KLEE执行输出信息都在文件klee-out-N中，不过最近一次的执行生成的目录由klee-last快捷方式指向。查看生成的文件：

  <img src="pic\7.png" style="zoom:80%;" />

* 利用测试例运行程序, 用生成的测试例作为输入运行程序，命令及结果如下：

  ```shell
  # 设置除默认路径外查找动态链接库的路径
  $ export LD_LIBRARY_PATH=~/klee_build/lib/:$LD_LIBRARY_PATH
   # 将程序与 libkleeRuntest 库链接
  $ gcc -I ../../include -L /home/klee/klee_build/lib/ get_sign.c -lkleeRuntest
          #gcc编译生成a.out，一个可执行程序，然后用下面的方式指定其输入为test000001.ktest。
  $ KTEST_FILE=klee-last/test000001.ktest ./a.out
  $ echo $?
  ```

<img src="pic\8.png" style="zoom:80%;" />

5. 【mission 2】Testing a simple regular expression library

* 示例代码Regexp.c

* 将 C 语言文件编译转化为 LLVM bitcode

  ```shell
  clang -I ../../include -emit-llvm -c -g -O0 -Xclang -disable-O0-optnone Regexp.c
  ```

* 使用 KLEE 运行代码：

  ```shell
  klee --only-output-states-covering-new Regexp.bc
  ```

<img src="pic\9.png" style="zoom:80%;" />

* 报错。出现内存错误，测试驱动程序有一个错误。因为输入的正则表达式序列完全是符号的，但是match函数期望它是一个以null结尾的字符串。

* 解决方式：将' \0 '符号化后存储在缓冲区的末尾。修改代码如下

  

  ```c
  int main() {
    // The input regular expression.
    char re[SIZE];
  
    // Make the input symbolic.
    klee_make_symbolic(re, sizeof re, "re");
    re[SIZE - 1] = '\0';
  
    // Try to match against a constant string "hello".
    match(re, "hello");
  
    return 0;
  }
  ```

* 之后再编译链接应该就会有正确输出。但是docker内的apt-get update过慢，按照网上的教程也没有变快，一直卡在这里无法推进。

6. 【mission 3】Solving a maze with KLEE

* KLEE通过符号执行找到了所有的解（包括陷阱）。通过这个例子可以完全看到KLEE符号执行的过程，首先是按照符号变量的size每一个字节都是符号值，然后从第一个字节开始一个一个地试验具体值

* 首先要下载迷宫的程序

  

  ```shell
  # Update aptitude 
  sudo apt-get update
  # Install git 
  sudo apt-get install -y git-core
  # Download maze 
  git clone https://github.com/grese/klee-maze.git ~/maze
  
  
  # Build & Run Maze
  # Source is in maze.c.
  cd ~/maze
  
  #Build: 
  gcc maze.c -o maze
  #Run manually: 
  ./maze
  #Input a string of "moves" and press "enter"
  #Allowed moves: w (up), d (right), s (down), a (left)
  #Example solution: ssssddddwwaawwddddssssddwwww
  #Run w/solution: 
  cat solution.txt | ./maze
  ```

经过mission2的一顿操作之后，源变得乱七八糟，apt-get update时出现错误，在网上查到的也仍然是换源操作，但不成功，气死俺了。

<img src="pic\10.png" style="zoom:80%;" />

# 实验参考

> 谌雯馨同学的仓库：https://github.com/chencwx/Software-and-system-security/tree/master/符号执行
>
> docker太卡：https://www.jianshu.com/p/34d3b4568059
>
> tutorials网页地址：https://klee.github.io/tutorials/testing-function/
>
> tutorials指导：https://blog.csdn.net/vincent_nkcs/article/details/85224491
>
> 虚拟机扩容：https://blog.csdn.net/acdefghb/article/details/80103817
>
> 删除用户：https://blog.csdn.net/ninnyyan/article/details/88059092
>
> 安装vim时为了加速，出现permission deny问题：https://blog.csdn.net/Let_me_tell_you/article/details/80786881



