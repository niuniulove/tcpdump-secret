# tcpdump-secret
dive into more tcpdump sniffer options

# Option -C, -G and -W. All of is UPPERCASE
-C: 配合-w使用，限制保存文件的大小，单位为1M(10^6)，当超过大小限制时，重新开始下一个文件保存，文件名+1递增

-W: 保存文件的数目，超过后，视情况而定：
    配合-G使用时，结束
    配合-C使用时，rotate
    
-G: 多少秒写一个文件，文件名可以是strftime格式，如果是strftime格式，按时间来命名文件名

# Example
1. tcpdump -i eth0 -s0 -w /root/test/Trace%Y-%m-%d_%H%M%S.pcap -W 5  -G 60 -z dump.sh

result:
-rw-r--r--  1 root root 18401 Sep 13 05:01 Trace2019-09-13_050029.pcap.gz
-rw-r--r--  1 root root 45739 Sep 13 05:02 Trace2019-09-13_050129.pcap.gz
-rw-r--r--  1 root root 19185 Sep 13 05:03 Trace2019-09-13_050229.pcap.gz
-rw-r--r--  1 root root 24602 Sep 13 05:04 Trace2019-09-13_050329.pcap.gz
-rw-r--r--  1 root root 19015 Sep 13 05:05 Trace2019-09-13_050429.pcap.gz
说明-W和-G使用时，超过-W设置文件数目后终止

如果不按照strftime格式命名文件名，会出问题
tcpdump -i eth0 -s0 -w /root/test/Trace.pcap -W 5  -G 60 -z /root/test/dump.sh
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
gzip: /root/test/Trace.pcap.gz already exists; do you wish to overwrite (y or n)? 

-rw-r--r--  1 root root    0 Sep 13 05:18 Trace.pcap
-rw-r--r--  1 root root   31 Sep 13 05:17 Trace.pcap.gz

保存一个后就会出现覆盖，且后面保存的文件也有问题

2. tcpdump -i eth0 -s0 -w /root/test/Trace.pcap -W 5  -C 1 -z /root/test/dump.sh

tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
gzip: /root/test/Trace.pcap0.gz already exists; do you wish to overwrite (y or n)? gzip: /root/test/Trace.pcap1.gz already exists; do you wish to overwrite (y or n)? gzip: /root/test/Trace.pcap2.gz already exists; do you wish to overwrite (y or n)? 

-rw-r--r--  1 root root 18401 Sep 13 05:01 Trace2019-09-13_050029.pcap.gz
-rw-r--r--  1 root root 45739 Sep 13 05:02 Trace2019-09-13_050129.pcap.gz
-rw-r--r--  1 root root 19185 Sep 13 05:03 Trace2019-09-13_050229.pcap.gz
-rw-r--r--  1 root root 24602 Sep 13 05:04 Trace2019-09-13_050329.pcap.gz
-rw-r--r--  1 root root 19015 Sep 13 05:05 Trace2019-09-13_050429.pcap.gz

保存了5个文件后，又重新开始rotate文件，从第一个开始覆盖，出现提示

3. -z 一定要个 -C or -G 使用，就是rotate files的情形

tcpdump -i eth0 -s0 -w /root/test/Trace.pcap  -C 1 -z /root/test/dump.sh

-rw-r--r--  1 root root 946592 Sep 13 05:40 Trace.pcap1.gz
-rw-r--r--  1 root root 956543 Sep 13 05:40 Trace.pcap2.gz
-rw-r--r--  1 root root 951573 Sep 13 05:41 Trace.pcap3.gz
-rw-r--r--  1 root root 947949 Sep 13 05:41 Trace.pcap4.gz
-rw-r--r--  1 root root 950802 Sep 13 05:41 Trace.pcap5.gz
-rw-r--r--  1 root root 949052 Sep 13 05:41 Trace.pcap6.gz
-rw-r--r--  1 root root 945995 Sep 13 05:41 Trace.pcap7.gz
-rw-r--r--  1 root root 487424 Sep 13 05:41 Trace.pcap8
-rw-r--r--  1 root root 953074 Sep 13 05:40 Trace.pcap.gz

注意，最后一个文件，应为不是-C的限制产生的，没有full，因此没有调用-z的命令

-z and -G
tcpdump -i eth0 -s0 -w /root/test/Trace.pcap  -G 10 -z /root/test/dump.sh
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
gzip: /root/test/Trace.pcap.gz already exists; do you wish to overwrite (y or n)? gzip: /root/test/Trace.pcap.gz already exists; do you wish to overwrite (y or n)? 

文件名一定要strftime的格式，不然overwrite


tcpdump -i eth0 -s0 -w /root/test/Trace%Y%m%d-%H%M%S.pcap  -G 5 -z /root/test/dump.sh

-rw-r--r--  1 root root    1442 Sep 13 05:46 Trace20190913-054632.pcap.gz
-rw-r--r--  1 root root 3024045 Sep 13 05:46 Trace20190913-054637.pcap.gz
-rw-r--r--  1 root root 5052776 Sep 13 05:46 Trace20190913-054642.pcap.gz
-rw-r--r--  1 root root    3334 Sep 13 05:46 Trace20190913-054648.pcap.gz
-rw-r--r--  1 root root     417 Sep 13 05:46 Trace20190913-054653.pcap.gz
-rw-r--r--  1 root root    5274 Sep 13 05:47 Trace20190913-054659.pcap.gz
-rw-r--r--  1 root root    6487 Sep 13 05:47 Trace20190913-054704.pcap.gz
-rw-r--r--  1 root root     312 Sep 13 05:47 Trace20190913-054709.pcap.gz
-rw-r--r--  1 root root     444 Sep 13 05:47 Trace20190913-054714.pcap.gz
-rw-r--r--  1 root root     370 Sep 13 05:47 Trace20190913-054719.pcap.gz
-rw-r--r--  1 root root     101 Sep 13 05:47 Trace20190913-054724.pcap.gz
-rw-r--r--  1 root root       0 Sep 13 05:47 Trace20190913-054729.pcap


# 结论
1. -C and -G 可以rotate write file
2. -C & -W file-num, exceed file-num, stop
3. -G & -W file-num, exceed file-num, rotate the first one sequently, overwirte or not
4. -z & -C, exec -z cmd
5. -z & -G, exec -z cmd
