# :fire: Deep Understanding of Average Load in Linux
<br/>

## 🔔 개요
**리눅스 평균 부하(Load Average)는 시스템의 성능을 평가하는 중요한 지표**로, 특정 시간 동안 CPU와 프로세서의 부하를 나타냅니다. 이 값은 시스템의 안정성과 처리 능력을 이해하는 데 필수적이며, 특히 서버 관리 및 성능 최적화에 있어 중요한 역할을 합니다.
 서비스를 중단 없이 사용하기 위해 자원관리가 필요하기 때문에 평균 부하에 관하여 테스트 해보았습니다.
<br/><br/>

## 👥 팀원 소개

| 노솔리 | 구동길 |
|:-----------:|:-----------:|
| <img width="100px" src="https://avatars.githubusercontent.com/soljjang777" /> | <img width="100px" src="https://avatars.githubusercontent.com/dkac0012"/> |
| [@soljjang777](https://github.com/soljjang777) | [@dkac0012](https://github.com/dkac0012) |

<br/>

## 💻 시스템 환경 및 사전 작업
### OS Version
```bash
username@servername:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.5 LTS
Release:        22.04
Codename:       jammy
```

### CPU 
 - CPU 갯수 확인 : CPU의 개수를 알게 되면 평균 부하가 CPU 개수를 초과할 때 시스템이 과부하 상태라는 것을 알 수 있음.
```bash
username@servername:~$ cat /proc/cpuinfo
...
cpu cores       : 2
...
```
### RAM
-  RAM이 용량 확인 : RAM이 부족하면 스왑 공간을 사용하게 되어 성능 저하가 발생하고, 프로세스가 메모리 대기 상태에 빠질 수 있습니다. 따라서 평균 부하와 RAM을 함께 모니터링하기 위해 RAM 확인 필수.
```bash
username@servername:~/test$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       369Mi       6.4Gi       1.0Mi       989Mi       7.1Gi
Swap:             0B          0B          0B
```

### 사전 설치해야 될 패키지
- **stress**: 시스템에 부하를 주는 도구로, CPU, 메모리, I/O 등을 인위적으로 사용하여 시스템의 성능을 테스트하는 데 사용됨. 이 도구를 통해 시스템의 안정성과 성능을 평가할 수 있음.
- **sysstat**: 시스템 성능 모니터링 도구 모음으로, 다양한 성능 데이터를 수집하고 분석할 수 있는 유틸리티(예: `iostat`, `mpstat`, `pidstat`, `sar` 등)를 포함함. 이 도구를 사용하면 시스템의 자원 사용 현황을 모니터링하고 성능 병목 현상을 진단할 수 있음.

사전 작업으로 다음 패키지를 설치:

```bash
apt install stress sysstat
```
<br/><br/>

## ⚡ 리눅스 평균 부하(Load Average)
### 평균 부하의 정의
평균 부하는 단위 시간 동안 실행 가능하거나 중단 불가능한 상태의 프로세스의 평균 수를 나타냄. 이는 CPU 사용률과는 다름.

### 프로세스 상태
 - **실행 가능한 프로세스:** CPU를 사용 중이거나 CPU 시간을 기다리는 프로세스. 일반적으로 R 상태로 표시됨.
 - **무중단 프로세스:** 중단될 수 없는 프로세스. 일반적으로 I/O 작업을 수행 중이며 D 상태로 표시됩니다. 이 프로세스는 하드웨어 장치의 응답을 기다림.
   
### 평균 부하의 의미
평균 부하는 활성 프로세스의 평균 수로, 시스템의 CPU 수에 따라 다르게 해석될 수 있습니다. <br/> <br/>
예시: <br/>
 - 평균 부하가 2이고 CPU가 2개인 시스템: 모든 CPU가 완전히 사용 중.
 - 평균 부하가 2이고 CPU가 4개인 시스템: 50%의 유휴 CPU 용량.
 - 평균 부하가 2이고 CPU가 1개인 시스템: 프로세스의 절반이 CPU 시간을 놓고 경쟁.
 <br/> <br/>

## #️⃣ 시스템의 부하 확인 명령어
### uptime
 - ```12:18:16```: 현재 시간
 - ```up 2 days, 5:19```: 시스템이 2일 5시간 19분 동안 가동 중
 - ```4 users```: 현재 시스템에 로그인한 사용자 수
 - ```load average```: 0.00, 0.00, 0.00: 1분, 5분, 15분 동안의 평균 부하. (0.00은 대기 중인 작업이 없다는 것을 의미)
```bash
username@servername:~$ uptime
 12:18:16 up 2 days,  5:19,  4 users,  load average: 0.00, 0.00, 0.00
```
<br>

### top
 - CPU 사용량, 메모리 사용량, 프로세스 목록 등이 포함된 실시간 화면 확인 가능
```bash
username@servername:~$ top
```
<img src="https://github.com/user-attachments/assets/1d823100-1321-4e15-abc8-2d89fc7617c4" width="650">
<br>

### mpstat
 - 시스템의 CPU 사용 현황을 모니터링 확인 가능
   
| 옵션        | 설명                                         |
|-------------|----------------------------------------------|
| `-P [cpu]`  | 특정 CPU 또는 CPU 코어의 사용 현황 표시 (예: `all`로 모든 CPU) |
| `-u`        | CPU 사용률 정보 표시 (기본적으로 활성화)   |
| `-I`        | CPU 인터럽트 및 컨텍스트 스위치 정보 포함 표시 |
| `-o [format]` | 출력 형식 지정 (예: `CSV`, `JSON`, `HTML`) |
| `-r`        | 메모리 사용 정보 포함하여 표시              |
| `-h`        | 사용 가능한 옵션 및 도움말 정보 출력       |
| `[interval]` | 데이터 출력 간격을 초 단위로 지정          |
| `[count]`   | 지정한 횟수만큼 데이터 출력                 |

```bash
username@servername:~$ mpstat
Linux 5.15.0-122-generic (servername)   09/23/2024      _x86_64_        (2 CPU)

06:55:56 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
06:55:56 PM  all    0.20    0.00    0.16    0.07    0.00    0.24    0.00    0.00    0.00   99.33
```
<br/><br/>

## 시스템의 부하 테스트

### 시나리오 1 : cpu 변동 모니터링
<br/>

**1. CPU 사용률이 100%인 시나리오를 시뮬레이션** <br/>

 - ```--cpu 1``` :  CPU 부하를 1개의 CPU 코어에서 발생시킴
 - ```timeout 600``` : 부하 테스트를 600초(10분) 동안 실행
```bash
stress --cpu 1 --timeout 600
06:59:33 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
06:59:38 PM  all   49.85    0.00    0.10    0.20    0.00    0.50    0.00    0.00    0.00   49.35
06:59:38 PM    0    0.20    0.00    0.20    0.40    0.00    0.99    0.00    0.00    0.00   98.21
06:59:38 PM    1  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
```
<br/>

**2. uptime 모니터링** <br/>

 - ```watch```: 주어진 명령어를 일정 간격으로 반복 실행하여 그 출력을 보여주는 유틸리티. (기본적으로 2초 간격으로 실행)

 - ```d```: 출력의 변경된 부분을 강조 표시.

 - ```uptime```: 시스템의 현재 동작 시간을 보여주는 명령어로, 시스템이 켜져 있는 시간, 현재 사용자 수, 평균 로드(최근 1, 5, 15분 동안의 평균 CPU 사용률)를 표시함.
```bash
watch -d uptime
```
<img src="https://github.com/user-attachments/assets/6810ddae-4957-4760-ac0e-8fd53ac73729" width="550">
<br/><br/>

**3. mpstat 모니터링** <br>
 - ```-P ALL``` : 모든 CPU를 모니터링.
 - ```5``` :  5초마다 데이터 세트가 출력됨을 나타냄.
```bash
mpstat -P ALL 5 
```
<br>

**3-1. stess 시작 시** <br/>
<img src="https://github.com/user-attachments/assets/20dd258f-8071-4e94-b4ee-de5f6f19e369" width="550">

**3-2. stess 종료 시**  <br/>
- CPU코어 0번 확인 시 CPU가 실제로 100% 사용 중이지만 iowait가 0인 것을 볼 수 있음. 이는 평균 부하가 증가한 것이 CPU가 100% 사용 중이기 때문임을 나타냄
<img src="https://github.com/user-attachments/assets/fca7cb10-4466-4f80-8c4c-6bec412bd8b8" width="550">
<br><br/>

**pidstat 모니터링** <br/>
✨ 어떤 프로세스가 CPU 사용량을 100% 확인 작업 <br/>
 - ```pidstat```: 프로세스 통계 정보를 제공하는 명령어.
 - ```-u```: CPU 사용률 정보를 표시하도록 지정.
 - ```5``` : 5초 간격으로 정보를 출력.
 - ```1``` : 총 1회 데이터를 출력. (5초 후 한 번만 데이터가 출력됨)
```bash
pidstat -u 5 1
Linux 5.15.0-122-generic (servername)   09/23/2024      _x86_64_        (2 CPU)

07:36:12 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
07:36:17 PM     0       599    0.00    0.20    0.00    0.00    0.20     0  kworker/0:3-events
07:36:17 PM  1000      1340    0.20    0.00    0.00    0.00    0.20     0  node
07:36:17 PM  1000      6700    0.00    0.20    0.00    0.00    0.20     0  watch
07:36:17 PM  1000      7925   99.60    0.00    0.00    0.00   99.60     1  stress

Average:      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
Average:        0       599    0.00    0.20    0.00    0.00    0.20     -  kworker/0:3-events
Average:     1000      1340    0.20    0.00    0.00    0.00    0.20     -  node
Average:     1000      6700    0.00    0.20    0.00    0.00    0.20     -  watch
Average:     1000      7925   99.60    0.00    0.00    0.00   99.60     -  stress

```
CPU 사용률이 100%라는것 확인 가능
<br>

### 시나리오 2 : IO 변동 모니터링
<img src="https://github.com/user-attachments/assets/c40e46af-2500-450b-88f4-41bdd2b911a7" width="650">


### 시나리오 3: 시나리오 1,2 비교분석

```bash username@servername:~$ mpstat -P ALL 5 ```

#### 시나리오 1
```bash 
07:31:36 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
07:31:41 PM  all   75.85    0.00    0.20    0.10    0.00    0.00    0.00    0.00    0.00   23.85
07:31:41 PM    0   75.65    0.00    0.20    0.20    0.00    0.00    0.00    0.00    0.00   23.95
07:31:41 PM    1   76.06    0.00    0.20    0.00    0.00    0.00    0.00    0.00    0.00   23.74
````

#### 시나리오 2
```bash
07:21:11 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
07:21:16 PM  all    1.06    0.00   45.34   48.94    0.00    4.13    0.00    0.00    0.00    0.53
07:21:16 PM    0    0.77    0.00   42.00   51.64    0.00    4.62    0.00    0.00    0.00    0.96
07:21:16 PM    1    1.41    0.00   49.41   45.65    0.00    3.53    0.00    0.00    0.00    0.00
```

##### 분석
시나리오 1의 경우 CPU 부하발생 ```bash stress --cpu 2 --timeout 600 or stress -c 2 --timeout 600``` 커맨드를 활용하여 CPU에서 순수 연산작업을 하게되어 USR가 과부하가 발생하게 된다.
시나리오 2의 경우 IO 부하발생 ```bash stress -i 20 --timeout 600``` 커맨드를 활용하여 IO작업을 하게 된다. 입출력 장치와의 직접적인 상호작용이 필요하기 때문에 커널영역에서 부하를 발생시키므로 SYS영역과 iowait에서 과부하가 발생하게 된다.

##### 결과
시나리오 1의 경우 CPU 성능 테스트, 연산 과부하 상황에서의 시스템 안정성 확인, CPU 온도나 냉각 시스템의 동작을 모니터링 할때 사용하며 시나리오 2의 경우 디스크, 네트워크 I/O 성능 평가, 스토리지 시스템 테스트, I/O 병목 현상 분석, I/O 부하 상황에서 시스템 성능을 모니터링 할때 사용할 수 있다.



## 🧪 References

 [Deep Understanding of Average Load in Linux](https://blog.devgenius.io/deep-understanding-of-average-load-in-linux-74822e1dbcb1) 컬럼를 참고하여 영감을 받았습니다. 이 컬럼의 통찰력이 이 프로젝트에서 사용된 평균 부하 개념의 이해와 구현에 큰 도움이 되었습니다.
<br/><br/>

