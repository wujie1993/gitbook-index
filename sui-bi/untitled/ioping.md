# ioping

ioping主要用于测试磁盘io延迟

### 安装

centos环境

```text
yum install -y epel-release
yum install -y ioping
```

ubuntu环境

```text
apt install ioping
```

### 使用

例子：测试/dev/sda磁盘的io延迟

```text
ioping -c 20 /dev/sda
```

### 可用参数

```text
 Usage: ioping [-ABCDRLWYykq] [-c count] [-i interval] [-s size] [-S wsize]
               [-o offset] [-w deadline] [-pP period] directory|file|device
        ioping -h | -v

      -c <count>      stop after <count> requests
      -i <interval>   interval between requests (1s)
      -l <speed>      speed limit in bytes per second
      -t <time>       minimal valid request time (0us)
      -T <time>       maximum valid request time
      -s <size>       request size (4k)
      -S <wsize>      working set size (1m)
      -o <offset>     working set offset (0)
      -w <deadline>   stop after <deadline> time passed
      -p <period>     print raw statistics for every <period> requests
      -P <period>     print raw statistics for every <period> in time
      -A              use asynchronous I/O
      -C              use cached I/O (no cache flush/drop)
      -B              print final statistics in raw format
      -D              use direct I/O (O_DIRECT)
      -R              seek rate test
      -L              use sequential operations
      -W              use write I/O (please read manpage)
      -G              read-write ping-pong mode
      -Y              use sync I/O (O_SYNC)
      -y              use data sync I/O (O_DSYNC)
      -k              keep and reuse temporary file (ioping.tmp)
      -q              suppress human-readable output
      -h              display this message and exit
      -v              display version and exit
```

