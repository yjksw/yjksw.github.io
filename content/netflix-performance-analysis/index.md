---
emoji: 📈
title: 넷플릭스에서 60000ms 만에 리눅스 서버 성능을 진단하는 방법 10가지
date: '2021-10-15 23:10:00'
author: 코다
tags: 웹 성능테스트 
categories: 웹 성능테스트
---

> 이 글은 다음 [링크]([https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55))를 번역하며 공부한 글입니다 🙌   

<br>

## 💡 Intro

- 성능 테스트에 관련한 공부 및 적용을 하면서 좋은 아티클을 추천 받았다. (Thanks to 제리 👍)
- 관련 명령어들에 대해서 공부하고 각 칼럼이 의미하는 os 및 네트워크 기초 지식을 메꾸보자.

<br>

## 1. uptime

```bash
$ uptime 
23:51:26 up 21:31, 1 user, load average: 30.02, 26.43, 19.02
```

- 실행되기를 기다리는 프로세스의 갯수를 출력한다. 리눅스 시스템에서는 CPU를 기다리는 프로세스와 uninterruptible I/O (disk I/O) 에 의해 프로세스가 막혀있을 수 있다. 따라서 이 수치를 통해서 리소스 부하를 간편하게 확인 할 수 있다.
- 위 세개의 번호는 각각 1분, 5분, 15분 간 실행되지 못하고 대기 중인 프로세스 갯수를 나타낸다. 시간 추이에 따른 부하 상태를 통해 상황을 유추할 수도 있다.

<br>

## 2. dmesg | tail

```bash
$ dmesg | tail
[1880957.563150] perl invoked oom-killer: gfp_mask=0x280da, order=0, oom_score_adj=0
[...]
[1880957.563400] Out of memory: Kill process 18694 (perl) score 246 or sacrifice child
[1880957.563408] Killed process 18694 (perl) total-vm:1972392kB, anon-rss:1953348kB, file-rss:0kB
[2320864.954447] TCP: Possible SYN flooding on port 7001. Dropping request.  Check SNMP counters.
```

- 마지막 10개의 시스템 메세지를 출력한다. 여기서 성능에 이슈를 일으킨 에러 메세지를 확인할 수 있다. oom-killer나 TCP 요청 드랍 같은 경우를 확인할 수 있으므로 필수다.

<br>

## 3. vmstat 1

```bash
procs ---------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
34  0    0 200889792  73708 591828    0    0     0     5    6   10 96  1  3  0  0
32  0    0 200889920  73708 591860    0    0     0   592 13284 4282 98  1  1  0  0
32  0    0 200890112  73708 591860    0    0     0     0 9501 2154 99  1  0  0  0
32  0    0 200889568  73712 591856    0    0     0    48 11900 2459 99  0  0  0  0
32  0    0 200890208  73712 591860    0    0     0     0 15898 4840 98  1  1  0  0
^C
```

- 가상 메모리 stat를 보여준다. 서버의 중요한 statistics를 출력한다.
- vmstat 명령어와 1을 클릭하면 1초마다 서버의 statictics를 출력한다. (단 첫번째 행은 서버가 부팅되었을 때부터의 평균 수치를 보여준다)

### 중요한 칼럼

- **r**: CPU에서 실행되는 프로세스와 기다리고 있는 프로세스 숫자이다. I/O에 의해 생기는 부하를 제외하고 CPU에 대해서만 보여주기 때문에 load average의 원인이 CPU인지 아닌지를 확인 할 수 있다.
    
    참고로, "r" 칼럼 값이 CPU 코어 갯수보다 많다면 saturation 상황이다.
    
- **free**: 비어있는 메모리를 kilobytes 단위로 보여준다.
- **si, so**: swap-ins, swap-outs를 보여준다. 이 값이 0이 아니라면 메모리 부족이다.
- **us, sy, id, wa, st**: CPU 시간을 분할하여 모든 CPU 시간의 평균 정보를 보여준다.
    - user time, system time(kernel), idle, wait I/O, stolen time(다른 게스트, Xen 등등) 지표를 보여준다.

- 여기서 user + system time을 통해 CPU가 바쁜 상태인지 확인할 수 있다.
- wait I/O에 특정 값이 유지된다면 disk 병목 현상이 있다고 볼 수 있다. 이 경우는 작업이 disk I/O 작업을 기다리느라 CPU 가 유휴 상태에 있는 때이다.
- System time은 I/O 작업을 하기 위해서 필수이다. high system time 평균은 20% 이상이며 이런 수치를 보이면 커널이 I/O 작업을 비효율적으로 처리하고 있다고 볼 수 있다.
- CPU 사용률(user-level)은 평균 90% 이상일 수도 있으나 이것이 꼭 문제를 뜻하는 것은 아니다. CPU에 부하가 걸리고 있는지는 "r" 칼럼을 통해서 판단해야한다.
- [여기서]([https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=ggaibi1004&logNo=221398356656](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=ggaibi1004&logNo=221398356656)) 각 칼럼에 대한 자세한 설명을 확인할 수 있다. 개인적으로 매우 유용하게 참고하고 있다. [여기도..]([https://im-recording-of-sw-studies.tistory.com/30](https://im-recording-of-sw-studies.tistory.com/30))

<br>

## 4. mpstat -P ALL 1

```bash
$ mpstat -P ALL 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015  _x86_64_ (32 CPU)

07:38:49 PM  CPU   %usr  %nice   %sys %iowait   %irq  %soft  %steal  %guest  %gnice  %idle
07:38:50 PM  all  98.47   0.00   0.75    0.00   0.00   0.00    0.00    0.00    0.00   0.78
07:38:50 PM    0  96.04   0.00   2.97    0.00   0.00   0.00    0.00    0.00    0.00   0.99
07:38:50 PM    1  97.00   0.00   1.00    0.00   0.00   0.00    0.00    0.00    0.00   2.00
07:38:50 PM    2  98.00   0.00   1.00    0.00   0.00   0.00    0.00    0.00    0.00   1.00
07:38:50 PM    3  96.97   0.00   0.00    0.00   0.00   0.00    0.00    0.00    0.00   3.03
[...]
```

- CPU 코어 별 CPU 시간을 출력한다. 이것을 통해 한 코어에 프로세스가 집중되어 비효율적이지는 않은지 확인할 수 있다.
- 만일 하나의 CPU에 부하가 심하다면 single thread application에 의한 부하일 수 있다.

<br>

## 5. pidstat 1

```bash
$ pidstat 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015    _x86_64_    (32 CPU)

07:41:02 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
07:41:03 PM     0         9    0.00    0.94    0.00    0.94     1  rcuos/0
07:41:03 PM     0      4214    5.66    5.66    0.00   11.32    15  mesos-slave
07:41:03 PM     0      4354    0.94    0.94    0.00    1.89     8  java
07:41:03 PM     0      6521 1596.23    1.89    0.00 1598.11    27  java
07:41:03 PM     0      6564 1571.70    7.55    0.00 1579.25    28  java
07:41:03 PM 60004     60154    0.94    4.72    0.00    5.66     9  pidstat

07:41:03 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
07:41:04 PM     0      4214    6.00    2.00    0.00    8.00    15  mesos-slave
07:41:04 PM     0      6521 1590.00    1.00    0.00 1591.00    27  java
07:41:04 PM     0      6564 1573.00   10.00    0.00 1583.00    28  java
07:41:04 PM   108      6718    1.00    0.00    0.00    1.00     0  snmp-pass
07:41:04 PM 60004     60154    1.00    4.00    0.00    5.00     9  pidstat
^C
```

- pidstat는 프로세스 별 top 명령어 같은 느낌이다. 하지만 rollling 방식으로 매 시간마다의 모니터링 결과를 출력해준다.
- 이 명령을 통해 시간에 흐름에 따른 패턴을 확인할 수 있고, 현상을 기록할 수 있는 장점이 있다.
- 위 예시를 보면 2개의 자바 프로세스가 CPU를 과하게 소비하고 있음을 알 수 있다.
- 위에서 `%CPU` 칼럼은 서버의 모든 CPU를 포함한 수치이다. 따라서 1591%라는 수치는 해당 자바 프로세스가 거의 16개의 CPU를 소비하고 있음을 나타낸다. (리눅스에서 CPU 사용률은 주로 각 CPU 당 100%의 수치로 그 합을 의미한다)

<br>

## 6. iostat -xz 1

```bash
$ iostat -xz 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015  _x86_64_ (32 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          73.96    0.00    3.73    0.03    0.06   22.21

Device:   rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
xvda        0.00     0.23    0.21    0.18     4.52     2.08    34.37     0.00    9.98   13.80    5.42   2.44   0.09
xvdb        0.01     0.00    1.02    8.94   127.97   598.53   145.79     0.00    0.43    1.78    0.28   0.25   0.25
xvdc        0.01     0.00    1.02    8.86   127.79   595.94   146.50     0.00    0.45    1.82    0.30   0.27   0.26
dm-0        0.00     0.00    0.69    2.32    10.47    31.69    28.01     0.01    3.23    0.71    3.98   0.13   0.04
dm-1        0.00     0.00    0.00    0.94     0.01     3.78     8.00     0.33  345.84    0.04  346.81   0.01   0.00
dm-2        0.00     0.00    0.09    0.07     1.35     0.36    22.50     0.00    2.55    0.23    5.62   1.78   0.03
[...]
^C
```

- 블록 디바이스 (disks)를 확인하기 매우 좋은 명령어다. 해당 디바이스의 부하와 퍼포먼스까지 확인할 수 있다. 다음 칼럼들을 유의해서 보자.

1. **r/s, w/s, rkB/s, wkB/s** 
    - 위 수치들은 1초 동안 해당 디바이스에게 전달된 읽기, 쓰기, Kbytes 읽기, Kbytes 쓰기이다.
    - 디바이스의 작업량 정도를 확인할 수 있다. 이 수치가 지나치게 높다면 과부하에 의한 성능 문제가 발생할 수 있다.
2. **await**
    - ms 단위의 I/O 평균 시간이다. 이것은 어플리케이션이 queue에 있는 시간과 서비스되는 시간이 모두 포함된 시간이다.
    - 예상보다 긴 평균시간은 디바이스 포화 여부 혹은 디바이스에 오류 가능성을 의미할 수 있다.
3. **avgqu-sz**
    - 디바이스의 평균 요청 수이다. 1 보다 큰 수치는 포화상태의 근거일 수 있다. (여러 디스크에 대한 가장 디바이스 같은 경우 요청을 병렬로 처리할 수 있긴 하다는 것을 참고)
4. **%util**
    - 디바이스 사용률이다. 매 초마다 디바이스가 처리하고 있는 퍼센트를 나타닌다.
    - 디바이스 마다 상이하지만 60% 보다 큰 수치는 주로 좋지 않은 성능을 나타낸다. (await 칼럼에서 함께 확인할 수 있는 상태이다)
    - 100%에 가까운 것은 포화상태임을 뜻한다.
    
- 만일 storage device가 여러 물리 디스크 앞에 있는 논리 디스크라면 100% 의 사용률은 어떤 I/O 프로세스가 100% 시간동안 처리되고 있으나 실제 물리 디스크는 포화상태가 아닐 수도 있다.
- 참고할 것은 disk I/O의 낮은 성능이 어플리케이션의 성능을 저하시키는 요인이 아닐 수도 있다는 것이다. 많은 기술들은 I/O 작업을 비동기로 처리하여 어플리케이션이 봉쇄상태에 머물거나 지연시간(latency)에 영향이 가지 않도록 한다.

<br>

## 7. free -m

```bash
$ free -m
             total       used       free     shared    buffers     cached
Mem:        245998      24545     221453         83         59        541
-/+ buffers/cache:      23944     222053
Swap:            0          0          0
```

- 중요 칼럼 (가장 오른쪽 칼럼)
    1. **buffers**
        - 버퍼 캐시이며 block device I/O 에 사용
    2. **cached** 
        - 페이지 캐시이며 file systems에 사용
- 위 두 칼럼의 수치가 `0`이 아니도록 주의하자. 0 이라면 disk I/O가 빈번하게 발생하며(`iostat`으로 확인) 가장 느린 연산이므로 최악의 성능을 낸다. 위 경우 Mbytes 이상의 여유 공간을 가지고 있으니 양호한 상태이다.
- `/+ buffers/cache` 은 `free` 수치에 대해서 더 명확하게 알려준다.
    - 운영체제의 물리 메모리는 그 빈 공간을 캐싱을 하기 위해서 사용한다. 하지만 프로세스가 필요로 하다면 곧바로 회수하여 필요한 프로세스에게 할당한다. 따라서 엄밀히 말하먄 캐시 데이터가 차지하고 있는 메모리의 용량도 free에 포함되어야 마땅하다.
    - 이 수치의 free는 캐시 데이터 메모리 용량까지 포함한 수치이다.
    - 이 부분에 대한 많은 혼란이 있기에 관련 [사이트]([https://www.linuxatemyram.com/](https://www.linuxatemyram.com/))가 따로 있을 정도이다. (ㅋㅋㅋㅋ)
- 리눅스에서 [ZFS]([https://itsfoss.com/what-is-zfs/](https://itsfoss.com/what-is-zfs/)) 라는 향상된 file system을 사용하고 있다면 위 수치가 더 혼란스러울 수 있다. ZFS는 별도의 캐시가 존재하며 `free -m` 에 제대로 반영이 되지 않기 때문이다.
    - 시스템이 가용 가능한 메모리 공간이 적어보이지만 ZFS 캐시에 의해 가용 가능한 메모리가 존재할 수도 있다는 것을 참고하자.

<br>

## 8. sar -n DEV 1

```bash
$ sar -n DEV 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015     _x86_64_    (32 CPU)

12:16:48 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
12:16:49 AM      eth0  18763.00   5032.00  20686.42    478.30      0.00      0.00      0.00      0.00
12:16:49 AM        lo     14.00     14.00      1.36      1.36      0.00      0.00      0.00      0.00
12:16:49 AM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

12:16:49 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
12:16:50 AM      eth0  19763.00   5101.00  21999.10    482.56      0.00      0.00      0.00      0.00
12:16:50 AM        lo     20.00     20.00      3.25      3.25      0.00      0.00      0.00      0.00
12:16:50 AM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
^C
```

- 이 명령어를 통해 네트워크 인터페이스 처리량을 확인할 수 있다.
    - `rxkB/s` 와 `txkB/s` 를 통해 작업량을 확인하고 한계치에 다다랐는지 확인할 수 있다.
    - 위 예시를 보면 eth0 는 22 Mbytes/s (176 Mbits/sec) 이다. (1 Gbit/sec 제한 보다 낮은 수치이다)
- `%ifutil` 칼럼을 통해 디바이스 사용률을 확인할 수 있다. (full-duplex인 경우 양쪽 방향의 최대값이다) 이것은 정확한 수치를 알기 어렵고 사용하지 않는 0.00 수치를 보이기도 한다.

<br>

## 9. sar -n TCP,ETCP 1

```bash
$ sar -n TCP,ETCP 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015    _x86_64_    (32 CPU)

12:17:19 AM  active/s passive/s    iseg/s    oseg/s
12:17:20 AM      1.00      0.00  10233.00  18846.00

12:17:19 AM  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
12:17:20 AM      0.00      0.00      0.00      0.00      0.00

12:17:20 AM  active/s passive/s    iseg/s    oseg/s
12:17:21 AM      1.00      0.00   8359.00   6039.00

12:17:20 AM  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
12:17:21 AM      0.00      0.00      0.00      0.00      0.00
^C
```

- 중요한 TCP 수치에 대해서 보여준다.
    1. **active/s** 
        - 로컬에서 시작된 초당 TCP 커넥션 개수 (`connect()`를 명령어로 시작된 커넥션)
    2. **passive/s**
        - 리모트에서 시작된 초당 TCP 커넥션 개수 (`accept()`로 연결된 커넥션)
    3. **retrans/s**
        - 초당 TCP 재전송량
- active와 passive 수는 서버 부하를 대략적으로 산출할 수 있는 좋은 수치이다.
    - 새롭게 들어온 passive 커넥션 개수와 내보내지고 있는 active 커넥션 개수
    - active를 outbound, passive을 inbound 수치로 판단할 수 있으나 정확히 그렇지만은 않다. ([localhost](http://localhost)와 localhost connection의 차이를 고려해보라) [링크]([https://velog.io/@lky9303/127.0.0.1-과-localhost의-차이](https://velog.io/@lky9303/127.0.0.1-%EA%B3%BC-localhost%EC%9D%98-%EC%B0%A8%EC%9D%B4))
- 네트워크 재전송량은 네트워크 혹은 서버 이슈 일 수 있다.
    - 네트워크 문제라면 네트워크가 안정적이지 않은 네트워크 일 수 있다.
    - 혹은 서버 과부화로 인해 패킷이 유실되는 문제일 수도 있다.
    - 위 예시에서는 초당 1개의 TCP 커넥션이 재전송되고 있다.

<br>

## 10. top

```bash
$ top
top - 00:15:40 up 21:56,  1 user,  load average: 31.09, 29.87, 29.92
Tasks: 871 total,   1 running, 868 sleeping,   0 stopped,   2 zombie
%Cpu(s): 96.8 us,  0.4 sy,  0.0 ni,  2.7 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:  25190241+total, 24921688 used, 22698073+free,    60448 buffers
KiB Swap:        0 total,        0 used,        0 free.   554208 cached Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 20248 root      20   0  0.227t 0.012t  18748 S  3090  5.2  29812:58 java
  4213 root      20   0 2722544  64640  44232 S  23.5  0.0 233:35.37 mesos-slave
 66128 titancl+  20   0   24344   2332   1172 R   1.0  0.0   0:00.07 top
  5235 root      20   0 38.227g 547004  49996 S   0.7  0.2   2:02.74 java
  4299 root      20   0 20.015g 2.682g  16836 S   0.3  1.1  33:14.42 java
     1 root      20   0   33620   2920   1496 S   0.0  0.0   0:03.82 init
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.02 kthreadd
     3 root      20   0       0      0      0 S   0.0  0.0   0:05.35 ksoftirqd/0
     5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H
     6 root      20   0       0      0      0 S   0.0  0.0   0:06.94 kworker/u256:0
     8 root      20   0       0      0      0 S   0.0  0.0   2:38.05 rcu_sched
```

- `top` 명령어는 이전에 다룬 다른 명령어로 확인할 수 있는 수치를 전반적으로 포함한 지표들을 보여준다. 때문에 편리하게 사용된다.
- `top` 명령어의 단점은 시간의 흐름에 따른 패턴 확인이 어렵고 당시 전반적인 서버의 상태만을 확인할 수 있다는 것이다. 시간의 흐름에 따른 패턴을 확인하고 싶다면 `vmstat` 혹은 `pidstat` 으로 확인할 수 있다. (rolling output을 보여준다)

 <br>
 <br>

## 🌩 느낀 점

- 확실히 os 에 대한 지식이 필요한 것을 느꼈다. os와 네트워크 공부와 병행하며 지표를 살피니 의미하는 바를 잘 이해할 수 있었다.
- 서버의 부하를 살펴보려면 크게 CPU부하, I/O 디바이스 부하, 네트워크 부하를 확인해야한다.
- 어플리케이션을 구현하고 제대로 서비스 하기 위해서는 서버를 제대로 모니터링하고 미리 어느 정도를 감당할 수 있는지 확인할 수 있어야 한다.
- 또한 지표를 보고 어느 부분을 개선하여 성능을 개선시킬 수 있을지도 판단할 수 있어야 한다.
- 결국 경험이 답이다...!!

<br>
<br>

**[참고자료]**

- [[https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55)]([https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55))


```toc
```