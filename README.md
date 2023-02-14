# mongodb-preliminary

## 总结

~~~sh
# 复制
echo 文件1 文件2 文件3 | xargs -t -n 1 cp 要被复制的文件路径+文件名
	例 echo s2 s3 | xargs -t -n 1 cp s1/sentinel.conf
	-n, --max-args=MAX_ARGS
	表示命令在执行的时候一次使用参数的最大个数
# 单个文件修改内容
sed -i -e 's/原来的值/新值/g' 文件路径+文件名
	例 sed -i -e 's/6379/7001/g' 7001/redis.conf
	-i        编辑文件内容
	-e script 将脚本中指定的命令添加到处理输入时执行的命令中,  多条件，一行中要有多个操作
~~~

##### 每个目录都要改，我们一键完成修改（在/tmp目录执行下列命令）：

```sh
# 在7001，7002，7003的配置文件顶部添加主节点ip
printf '%s\n' 7001 7002 7003 | xargs -I{} -t sed -i '1a replica-announce-ip 192.168.41.129' {}/redis.conf
	%s\n 占位符换行
	-I REPLACE_STR	将 xargs 输出的每一项参数单独赋值给后面的命令，参数需要用指定的替代字符串 REPLACE_STR 代替。REPLACE_STR 可以使用 {} $ @ 等符号，其主要作用是当 xargs command 后有多个参数时，调整参数位置。例如备份以 txt 为-后缀的文件：find . -name "*.txt" | xargs -I {}  cp {} /tmp/{}.bak
	-t， --verbose	先打印命令到标准错误输出，然后再执行
```

```sh
# 如果要一键停止，可以运行下面命令：
printf '%s\n' 7001 7002 7003 | xargs -I{} -t redis-cli -p {} shutdown
# 一键启动所有服务
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t redis-server {}/redis.conf
# 修改配置文件
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t sed -i 's/6379/{}/g' {}/redis.conf
```









### redis集群

3）拷贝配置文件到每个实例目录

然后将redis-6.2.4/redis.conf文件拷贝到三个目录中（在/tmp目录执行下列命令）：

```sh
# 方式一：逐个拷贝
cp redis-6.2.4/redis.conf 7001
cp redis-6.2.4/redis.conf 7002
cp redis-6.2.4/redis.conf 7003

# 方式二：管道组合命令，一键拷贝
echo 7001 7002 7003 | xargs -t -n 1 cp /usr/local/src/redis-6.2.4/redis.conf
echo s2 s3 | xargs -t -n 1 cp s1/sentinel.conf
```

4）修改每个实例的端口、工作目录

修改每个文件夹内的配置文件，将端口分别修改为7001、7002、7003，将rdb文件保存位置都修改为自己所在目录（在/tmp目录执行下列命令）：

```sh
sed -i -e 's/6379/7001/g' -e 's/dir .\//dir \/tmp\/7001\//g' 7001/redis.conf
sed -i -e 's/6379/7002/g' -e 's/dir .\//dir \/tmp\/7002\//g' 7002/redis.conf
sed -i -e 's/6379/7003/g' -e 's/dir .\//dir \/tmp\/7003\//g' 7003/redis.conf
```

5）修改每个实例的声明IP

虚拟机本身有多个IP，为了避免将来混乱，我们需要在redis.conf文件中指定每一个实例的绑定ip信息，格式如下：

```properties
# redis实例的声明 IP
replica-announce-ip 192.168.150.101
```



每个目录都要改，我们一键完成修改（在/tmp目录执行下列命令）：

```sh
# 逐一执行
sed -i '1a replica-announce-ip 192.168.150.101' 7001/redis.conf
sed -i '1a replica-announce-ip 192.168.150.101' 7002/redis.conf
sed -i '1a replica-announce-ip 192.168.150.101' 7003/redis.conf
# 本机
printf '%s\n' 7001 7002 7003 | xargs -I{} -t sed -i '1a replica-announce-ip 192.168.41.129' {}/redis.conf
```

```sh
# 如果要一键停止，可以运行下面命令：
printf '%s\n' 7001 7002 7003 | xargs -I{} -t redis-cli -p {} shutdown
# 一键启动所有服务
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t redis-server {}/redis.conf
```

修改每个目录下的redis.conf，将其中的6379修改为与所在目录一致：

```sh
# 进入/tmp目录
cd /tmp
# 修改配置文件
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t sed -i 's/6379/{}/g' {}/redis.conf
```

