# redis的持久化

Redis作为一个内存数据库，数据都是存储在内存中，那么一旦Redis服务器进程退出，服务器中的数据也会消失。为了解决这个问题，Redis提供了持久化机制，也就是把内存中的数据保存到磁盘当中，防止服务器崩溃时数据丢失。

## Redis持久化方案：
* RDB（snapshot）：在指定的时间间隔能对你的数据进行快照存储。
* AOF（append on file）：记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据。

## RDB持久化
1. RDB文件的创建：分为手动创建和自动创建。
    * SAVE命令：会阻塞服务器进程，阻塞期间服务器不能处理任何命令请求。
    * BGSAVE命令：fork出一个子进程，子进程负责创建RDB文件，服务器进程（父进程）继续处理命令请求。执行流程如下图：
    <br/><div align=center>![image](https://github.com/WangXing17/redis/blob/main/redis%E6%8C%81%E4%B9%85%E5%8C%96/img/bgsave.png)
    
2. RDB文件的载入：服务器启动时自动执行，在启动时检测到RDB文件存在，就会自动载入。
<br/><div align=center>![image](https://github.com/WangXing17/redis/blob/main/redis%E6%8C%81%E4%B9%85%E5%8C%96/img/redis%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%BD%BD%E5%85%A5%E6%96%87%E4%BB%B6%E5%88%A4%E6%96%AD%E6%B5%81%E7%A8%8B.png)

3. 自动间隔性保存：save [seconds] [changes]
    * 服务器默认配置如下：
<br/><div align=center>![image](https://github.com/WangXing17/redis/blob/main/redis%E6%8C%81%E4%B9%85%E5%8C%96/img/autoSave.png)
    * 实现原理：
  redis服务器周期性操作函数serverCron默认每隔100毫秒就会执行一次，检查save选项所设置的保存条件是否满足，如果满足的话，执行BGSAVE命令。

4. RDB文件结构：RDB文件是一个经过压缩的二进制文件，有多个部分组成，如下图。
<br/><div align=center>![image]()


## AOF持久化: 实时写入，AOF重写
1. 前置条件：配置文件中开启AOF持久化开关，默认为关。如下图：
<br/><div align=center>![image](https://github.com/WangXing17/redis/blob/main/redis%E6%8C%81%E4%B9%85%E5%8C%96/img/aof%E5%BC%80%E5%85%B3.png)
2. 实时写入：
    * 追加：每执行完一个写命令，将被执行的写命令追加到服务器的aof_buf缓冲区末尾。
    * 写入：把aof_buf缓冲区中的内容写入到AOF文件中。
    * 同步：
  <br/><div align=center>![image](https://github.com/WangXing17/redis/blob/main/redis%E6%8C%81%E4%B9%85%E5%8C%96/img/aofFsync.png)
3. AOF重写：为了解决AOF文件过大的问题，将多条写记录合并为1条写记录。执行流程如下：
  <br/><div align=center>![image](https://github.com/WangXing17/redisNote/blob/main/redis%E6%8C%81%E4%B9%85%E5%8C%96/img/bgrewriteaof.png)
  
