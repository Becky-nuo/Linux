## RawCap简介
>RawCap是针对windows的免费开源软件，并且能够抓取windows本地回环接口127.0.0.1 (localhost）的数据包
>在windowns上可以使用RAWCAP进行数据抓取
>使用RawCap命令需要使用管理员权限打开CMD，然后进入到RawCap.exe的目录
## RawCap使用
>格式：RawCap.exe [选项] <接口> <数据包名>

>抓取127.0.0.1的三种方式
>>RawCap.exe 5 dumpfile.pcap
>>RawCap.exe 127.0.0.1 dumpfile.pcap
>>通过交互式的方式：在命令行中直接输入RawCap.exe

## RawCap的局限性
>无法抓取IPV6的数据包
>XP系统，只能抓取UDP和ICMP的数据包，不能抓取TCP包
>windows 7系统，无法捕获传入的数据包
>Windows Vista系统，无法捕捉传出的数据包