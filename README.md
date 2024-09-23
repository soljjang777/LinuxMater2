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

### 사전 작업
 - stress : 시스템에 부하를 주는 도구로, CPU, 메모리, I/O 등을 인위적으로 사용하여 시스템의 성능을 테스트하는 데 사용됨
 - sysstat: 시스템 성능 모니터링 도구 모음으로, 다양한 성능 데이터를 수집하고 분석할 수 있는 유틸리티(예: iostat, mpstat, pidstat, sar 등)를 포함하고 있음.
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
 - 
### 평균 부하의 의미
평균 부하는 활성 프로세스의 평균 수로, 시스템의 CPU 수에 따라 다르게 해석될 수 있습니다. <br/> <br/>
예시: <br/>
 - 평균 부하가 2이고 CPU가 2개인 시스템: 모든 CPU가 완전히 사용 중.
 - 평균 부하가 2이고 CPU가 4개인 시스템: 50%의 유휴 CPU 용량.
 - 평균 부하가 2이고 CPU가 1개인 시스템: 프로세스의 절반이 CPU 시간을 놓고 경쟁.
 <br/> <br/>

## #️⃣ 시스템의 부하 확인 명령어
### uptime
 - **12:18:16:** 현재 시간
 - **up 2 days, 5:19:** 시스템이 2일 5시간 19분 동안 가동 중
 - **4 users:** 현재 시스템에 로그인한 사용자 수
 - **load average:** 0.00, 0.00, 0.00: 1분, 5분, 15분 동안의 평균 부하. (0.00은 대기 중인 작업이 없다는 것을 의미)
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

### mpstat

## 시스템의 부하 테스트

### 시나리오 1 : cpu 변동 모니터링
### uptime 모니터링
<img src="https://github.com/user-attachments/assets/d733c2cc-3f49-4b16-af75-30e29b1fd2f0" width="650">

### mpstat 모니터링
#### stess 시작시
<img src="https://github.com/user-attachments/assets/20dd258f-8071-4e94-b4ee-de5f6f19e369" width="650">

#### stess 종료시
<img src="https://github.com/user-attachments/assets/fca7cb10-4466-4f80-8c4c-6bec412bd8b8" width="650">

### pidstat 모니터링
<img src="https://github.com/user-attachments/assets/5b2bcc6c-a07c-4d1a-9f52-0b45bed33f6e" width="650">

### 시나리오 2 : IO 변동 모니터링
<img src="https://github.com/user-attachments/assets/c40e46af-2500-450b-88f4-41bdd2b911a7" width="650">

## 🧪 References

 [Deep Understanding of Average Load in Linux](https://blog.devgenius.io/deep-understanding-of-average-load-in-linux-74822e1dbcb1) 컬럼를 참고하여 영감을 받았습니다. 이 컬럼의 통찰력이 이 프로젝트에서 사용된 평균 부하 개념의 이해와 구현에 큰 도움이 되었습니다.
<br/><br/>

