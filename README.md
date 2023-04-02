# -linux-
一键获取linux内存、cpu、磁盘IO等信息脚本编写

1. 获取要监控的本地服务器IP地址
IP=`ifconfig | grep inet | grep -vE 'inet6|127.0.0.1' | awk '{print $2}'`
echo "IP地址："$IP

ifconfig | grep inet过滤出如下结果包含字符串inet的行，如下图红色圈起来的两行 其中| 是管道的意思，将前面命令的结果作为输入给| 右边的命令
grep -vE 'inet6|127.0.0.1'将第一步结果通过命令grep -vE过滤掉包含inet6和127.0.0.1的行
将第二步结果通过命令awk 将字符串分割，$n(0~N) 对应相应的参数,如下图多少$2对应地址：192.168.0.125，**'{print 
2}'**打印出2的值
将第三步的结果赋值给变量IP
echo "IP地址："$IP打印出变量IP的值，【后面脚本中赋值和打印的语句功能相同，不再重复解释】


2. 获取cpu总核数
cpu_num=`grep -c "model name" /proc/cpuinfo`
echo "cpu总核数："$cpu_num

在linux的/proc目录下存放了系统运行的很多系统资源信息，其中**/proc/cpuinfo**存放了系统运行时cpu的很多重要信息。
所有的cpu核信息由model name字符串给出，
通过命令**grep -c "model name" /proc/cpuinfo** 可以计算出文件 /proc/cpuinfo中出现字符串model name出现的次数，就可以得到cpu总核数。-c 表示统计字符串出现次数。


3. 获取CPU利用率
top命令经常用来监控linux的系统状况，是常用的性能分析工具，能够实时显示系统中各个进程的资源占用情况。
# 获取用户空间占用CPU百分比
cpu_user=`top -b -n 1 | grep Cpu | awk '{print $2}' | cut -f 1 -d "%"`
echo "用户空间占用CPU百分比："$cpu_user
 
# 获取内核空间占用CPU百分比
cpu_system=`top -b -n 1 | grep Cpu | awk '{print $4}' | cut -f 1 -d "%"`
echo "内核空间占用CPU百分比："$cpu_system
 
# 获取空闲CPU百分比
cpu_idle=`top -b -n 1 | grep Cpu | awk '{print $8}' | cut -f 1 -d "%"`
echo "空闲CPU百分比："$cpu_idle
 
# 获取等待输入输出占CPU百分比
cpu_iowait=`top -b -n 1 | grep Cpu | awk '{print $10}' | cut -f 1 -d "%"`
echo "等待输入输出占CPU百分比："$cpu_iowait

# top -b -n 1显示系统的信息并以格式化打印，结果只刷新一次
n 设置退出前屏幕刷新的次数
b 将top输出编排成适合输出到文件的格式，可以使用这个选项创建进程日志
grep Cpu提取出字符串Cpu所在的行
awk '{print $2}'将第二步得到的字符串分割，并调用方法print 打印出**$2**对应的第二个字符串，0.5%us
cut -f 1 -d "%" 表示以%为分隔符，将第三步的结果分隔开，并显示分割后的记过的第一个字符串即0.5
-d  "%" 是以%作为分隔符， 
-f 1显示以：分割每一行的第一段内容
其他脚本以此类推


4.获取CPU上下文切换和中断次数
# 获取CPU中断次数
cpu_interrupt=`vmstat -n 1 1 | sed -n 3p | awk '{print $11}'`
echo "CPU中断次数："$cpu_interrupt
 
# 获取CPU上下文切换次数
cpu_context_switch=`vmstat -n 1 1 | sed -n 3p | awk '{print $12}'`
echo "CPU上下文切换次数："$cpu_context_switch

# 获取任务队列(就绪状态等待的进程数)
cpu_task_length=`vmstat -n 1 1 | sed -n 3p | awk '{print $1}'`
echo "CPU任务队列长度："$cpu_task_length

vmstat是Virtual Meomory Statistics（虚拟内存统计）的缩写，可对操作系统的虚拟内存、进程、CPU活动进行监控。是对系统的整体情况进行统计，不足之处是无法对某个进程进行深入分析。vmstat -n 1 1只显示一次各字段名称。
-n：只在开始时显示一次各字段名称。

sed -n 3p将第一步的结果打印出第3行
参数说明：
    -n或--quiet或--silent 取消自动打印模式空间,仅显示script处理后的结果。
动作说明：
    p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～

**awk '{print $1}'`**将第2步结果得出的字符串分割，并打印第一个字符串


5、获取CPU负载信息
# 获取CPU15分钟前到现在的负载平均值
cpu_load_15min=`uptime | awk '{print $11}' | cut -f 1 -d ','`
echo "CPU 15分钟前到现在的负载平均值："$cpu_load_15min
 
# 获取CPU5分钟前到现在的负载平均值
cpu_load_5min=`uptime | awk '{print $10}' | cut -f 1 -d ','`
echo "CPU 5分钟前到现在的负载平均值："$cpu_load_5min
 
# 获取CPU1分钟前到现在的负载平均值
cpu_load_1min=`uptime | awk '{print $9}' | cut -f 1 -d ','`
echo "CPU 1分钟前到现在的负载平均值："$cpu_load_1min

uptime 命令可以用来查看服务器已经运行了多久，当前登录的用户有多少，以及服务器在过去的1分钟、5分钟、15分钟的系统平均负载值。系统负载是处于可运行runnable或不可中断uninterruptable状态的进程的平均数。可运行状态的进程要么正在使用 CPU 要么在等待使用 CPU。不可中断状态的进程则正在等待某些 I/O 访问，例如等待磁盘 IO。有三个时间间隔的平均值。负载均值的意义根据系统中 CPU 的数量不同而不同，负载为 1 对于一个只有单 CPU 的系统来说意味着负载满了，而对于一个拥有 4 CPU 的系统来说则意味着 75% 的时间里都是空闲的。
参考之前脚本分析，**awk '{print $9}' | cut -f 1 -d ','**将第一步的结果分割开，并得到第9个字符串，然后用‘,’分隔开，并得到分割后的第一个字符串


6、获取内存信息
# 获取物理内存总量
mem_total=`free | grep Mem | awk '{print $2}'`
echo "物理内存总量："$mem_total
 
# 获取操作系统已使用内存总量
mem_sys_used=`free | grep Mem | awk '{print $3}'`
echo "已使用内存总量(操作系统)："$mem_sys_used
 
# 获取操作系统未使用内存总量
mem_sys_free=`free | grep Mem | awk '{print $4}'`
echo "剩余内存总量(操作系统)："$mem_sys_free
 
# 获取应用程序已使用的内存总量
mem_user_used=`free | sed -n 3p | awk '{print $3}'`
echo "已使用内存总量(应用程序)："$mem_user_used
 
# 获取应用程序未使用内存总量
mem_user_free=`free | sed -n 3p | awk '{print $4}'`
echo "剩余内存总量(应用程序)："$mem_user_free
 
 
# 获取交换分区总大小
mem_swap_total=`free | grep Swap | awk '{print $2}'`
echo "交换分区总大小："$mem_swap_total
 
# 获取已使用交换分区大小
mem_swap_used=`free | grep Swap | awk '{print $3}'`
echo "已使用交换分区大小："$mem_swap_used
 
# 获取剩余交换分区大小
mem_swap_free=`free | grep Swap | awk '{print $4}'`
echo "剩余交换分区大小："$mem_swap_free

free 命令显示系统内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存。
grep Swap将第一步的结果过滤只显示包含字符串Swap的行
**awk '{print $4}'**将第二步结果分割，并打印出第四个字符串的值


7. 获取磁盘I/O统计信息
echo "指定设备(/dev/sda)的统计信息"
# 每秒向设备发起的读请求次数
disk_sda_rs=`iostat -kx | grep sda| awk '{print $4}'`
echo "每秒向设备发起的读请求次数："$disk_sda_rs
 
# 每秒向设备发起的写请求次数
disk_sda_ws=`iostat -kx | grep sda| awk '{print $5}'`
echo "每秒向设备发起的写请求次数："$disk_sda_ws
 
# 向设备发起的I/O请求队列长度平均值
disk_sda_avgqu_sz=`iostat -kx | grep sda| awk '{print $9}'`
echo "向设备发起的I/O请求队列长度平均值"$disk_sda_avgqu_sz
 
# 每次向设备发起的I/O请求平均时间
disk_sda_await=`iostat -kx | grep sda| awk '{print $10}'`
echo "每次向设备发起的I/O请求平均时间："$disk_sda_await
 
# 向设备发起的I/O服务时间均值
disk_sda_svctm=`iostat -kx | grep sda| awk '{print $11}'`
echo "向设备发起的I/O服务时间均值："$disk_sda_svctm
 
# 向设备发起I/O请求的CPU时间百分占比
disk_sda_util=`iostat -kx | grep sda| awk '{print $12}'`
echo "向设备发起I/O请求的CPU时间百分占比："$disk_sda_util

iostat命令被用于监视系统输入输出设备和CPU的使用情况。它的特点是汇报磁盘活动统计情况，同时也会汇报出CPU使用情况。
-k：显示状态以千字节每秒为单位，而不使用块每秒
-x：显示扩展状态
** grep sda用于过滤第一步得到的结果，只显示包含字符串sda**的哪一行
**awk '{print $4}'**将第二步的结果分割，并只显示第4个字符串
iostat 由 Red Hat Enterprise Linux AS 发布。同时 iostat 也是 Sysstat 的一部分。所以我们安装要安装sysstat。

安装 sysstat 包：

sudo apt-get install sysstat 


