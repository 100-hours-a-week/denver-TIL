# :books: 실습 및 네트워크 구조도 설계

# 실습
> 이전 실습 링크: [2025-02-04](./2025-02/2025-02-04/theory.md) - 이론 보충 및 실습

## 5. ARP

### 네트워크 구조

![image.png](/images/2025-02/2025-02-06/05-01.png)

### 스위치의 MAC 테이블 확인

![image.png](/images/2025-02/2025-02-06/05-02.png)

> ping을 수행 하지 않았는데, 자동적으로 MAC 테이블이 생성되어 있어서 당황했다. 강의자료에 주기적으로 ARP 프로토콜로 반복적으로 MAC 주소를 교환하고 최신 정보로 채워진 듯!
> 

### ARP 동작 원리

> 일단 처음 스위치와 노드들을 연결했을 때, MAC 테이블이 완성되어 있지 않다고 가정해보자.
> 

**IP 주소를 이용해, PC0 → Laptop0으로 ping**

1. PC0이 ARP-요청 패킷을 Switch0으로 전송 (Laptop0의 MAC 주소 요청)
    1. 출발지 IP 주소와 MAC 주소: PC0의 IP 주소와 MAC 주소
    2. 목적지 IP 주소와 MAC 주소: Laptop0의 IP 주소와 0
2. Switch0이 나머지 노드들에게 MAC 주소 요청
    1. Switch0은 PC0의 MAC 주소를 저장
    2. 수신한 패킷을 모든 인터페이스의 노드들에게 전달하고 응답 요청
3. Laptop0 응답
    1. 목적지 IP주소를 가진 Laptop0이 PC0의 MAC 주소를 저장
    2. ARP-응답 패킷을 Switch0에 전달
        1. 출발지 IP 주소와 MAC 주소: Laptop0의 IP 주소와 MAC 주소
        2. 목적지 IP 주소와 MAC 주소: PC0의 IP 주소와 MAC 주소
    3. Switch0이 수신한 패킷 처리
        1. 수신한 패킷을 PC0으로 전달
        2. PC0과 Laptop0의 MAC 주소를 테이블에 저장
    4. PC0이 Laptop0의 MAC 주소 획득

### ARP를 이용한 MAC 주소 테이블 단점

- ARP 프로토콜은 보통 매우 짧은 시간 간격으로 반복 실행 → 노드들이 많은 네트워크에서는 통신량으로 부하 가중. 특히, 스위치가 모든 노드에게 브로드캐스트하는 ARP-요청 패킷의 경우
- 보안상의 문제
    - 모든 노드로 전달된 ARP-요청 패킷에 대해 수신 노드가 아니면 응답하지 않음
    - 악의적인 공격자가 Laptop1에서 Laptop0의 IP에 Laptop1 MAC 주소를 실어 ARP-응답 패킷을 보낸다면 → 테이블이 갱신되기 전까지 Laptop0으로 가는 모든 패킷이 공격자인 Laptop1으로 전달

→ 따라서, 해결 방법은 VLAN이다

MAC 주소를 이용한 라우팅은 LAN에서 스위치를 통해서만 발생한다. 따라서 LAN을 논리적으로 분리된 가상의 VLAN으로 관리하면 부하 가중과 보안의 해결할 수 있다.

이에 대한 실습은 실습1에 있다.

## 6. 목표 네트워크 구성

![image.png](/images/2025-02/2025-02-06/06-01.png)

### 기본 설정

- 라우터 3대: [Network Devices] → [Routers] → [2811]
- 스위치 1대: [Network Devices] → [Switches] → [2960]
- PC 3대, 노트북 1대, 서버 1대: [End Devices] → [PC], [Laptop], [Server]

### 라우터 인터페이스

라우터들 간의 연결은 PC나 Laptop과 달리, FastEthernet이 아닌 시리얼 인터페이스 사용

그러나 디폴트 인터페이스로 지원 안하므로 모듈 추가 필요!

- Router0 클릭 → [Config] → [INTERFACE]에 FastEthernet만 있음
- [Physical] 탭
    - 전원 off → [MODULES] / HWIC-2T drag → 0번 슬롯에 drop → 전원 on → 부팅

![image.png](/images/2025-02/2025-02-06/06-02.png)

### 노드 연결

![image.png](/images/2025-02/2025-02-06/06-03.png)

**연결 포트**

![image.png](/images/2025-02/2025-02-06/06-04.png)

**참고**

- Laptop0은 통신용 단말이 아니라 Router0을 설정하기 위한 콘솔 → FastEthernet 케이블이 아닌 RS232케이블로 직접 라우터에 연결
    - 그렇지 않으면, 원격 접속 프로토콜인 telnet이나 ssh를 이용해야 함
- 라우터의 인터페이스
    - 먼저 클릭한 쪽에 시계 모양
    - 두 라우터간에 데이터를 주고 받기 위한 동기화가 필요
    - 시계 있는 쪽이 시간 동기화를 주도하는 노드 (DCE: Data Communication Equipment)
    - 시계 없는 쪽이 그에 따라 동기화하는 노드 (DTE: Data Terminal Equipment)

### IP 설정

![image.png](/images/2025-02/2025-02-06/06-05.png)

> **추가 공부**
Default Gateway는 네트워크 내에서 다른 네트워크로 패킷을 전달할 때 거쳐 가는 라우터의 IP주소.
같은 네트워크 내 장치들은 Default Gateway를 거치지 않고 직접 통신.
그러나, 다른 네트워크일 경우 패킷은 Default Gateway로 전달. 따라서 Default Gateway 설정이 없으면 같은 네트워크 안에서는 문제 없이 통신 가능하지만, 다른 네트워크로는 나갈 수 없음.

위의 예시를 보면, PC0과 Server0이 연결되어 있는 라우터의 IP 주소와 PC0과 Server0의 게이트웨이가 같은 것을 볼 수 있다.
> 

**PC 및 서버 IP설정**

GUI를 이용해 설정했다.

**Router 및 Switch IP 설정**

- Switch IP 설정: 이전 실습에서 배웠던 내용을 토대로 표에 있는 정보 기입
- Router IP 설정
    - 실제 라우터에서는 GUI 제공을 안한다하니, Router0은 강의자료에 있는 Laptop0를 이용해서 IP를 설정했다.
    - 다른 라우터들은 GUI를 이용해 IP 설정

## 중간 연결 확인

![image.png](/images/2025-02/2025-02-06/06-06.png)

대부분 연결이 되어 있지만, 아직까지 Router1 → PC1과 Router2 → PC2의 연결은 안되어 있다.

먼저 PC0으로부터의 통신을 통해 동일 랜에 속한 노드 먼저 확인해보자.

1. “ping 127.0.0.1”: localhost 통신 확인
    
    ![image.png](/images/2025-02/2025-02-06/06-07.png)
    
2. “ping 203.237.10.100”: Switch0 통신 확인
    
    ![image.png](/images/2025-02/2025-02-06/06-08.png)
    
3. “ping 203.237.10.101”: Server0 통신 확인
    
    ![image.png](/images/2025-02/2025-02-06/06-09.png)

4. “ping 203.237.10.254”: Router0 통신 확인
    
    ![image.png](/images/2025-02/2025-02-06/06-10.png)
    

동일한 랜 내에서는 통신이 잘 되는 것을 확인할 수 있다. 그러나, 아직까지는 라우터 너머 다른 랜에 있는 PC2으로의 통신을 불가할 것이다. 한번 확인해보자.

![image.png](/images/2025-02/2025-02-06/06-11.png)

이는 PC0으로부터 패킷을 받은 Router0이 목적지 IP 주소를 보고도 어느 인터페이스로 보내야할지 모르기 때문이다. 따라서 라우팅 테이블을 생성할 필요가 있다.

### 라우팅 테이블 생성

라우팅 테이블을 생성하기 전에 먼저 다음과 같은 개념을 알아보자.

**라우팅이란?**

네트워크 상에서 IP 주소 등을 이용하여 출발지에서 목적지까지 패킷을 체계적으로 전달하기 위해 전달 경로를 선택하는 과정을 뜻한다.

- 정적 라우팅
    - 관리자가 수동으로 경로를 미리 설정하는 방법
    - 간단하고 소규모 네트워크에서 사용되는 방식
    - 네트워크 상태가 변할 경우 관리자가 갱신
- 동적 라우팅
    - 네트워크 상태에 따라 경로를 결정하는 방식
    - 복잡하지만 자동으로 최적의 경로를 탐색
    - RIP, OSPF, IGRP, EIGRP, BGP 등

**초기 라우팅 테이블**

![image.png](/images/2025-02/2025-02-06/06-12.png)

FastEthernet이나 Serial 인터페이스는 잘 연결이 되어 있지만, Serial 인터페이스의 경우 서브넷팅 정보만 있고, PC1(203.237.20.1)이나 PC2(203.237.30.1)의 정보는 없다

→ PC1, PC2로의 통신 불가!

**정적 라우팅 설정**

- Router0의 정적 라우팅 설정
    
    ![image.png](/images/2025-02/2025-02-06/06-13.png)
    
- Router1의 정적 라우팅 설정

    ![image.png](/images/2025-02/2025-02-06/06-14.png)

- Router2의 정적 라우팅 설정
    
    ![image.png](/images/2025-02/2025-02-06/06-15.png)
    

**통신 테스트**

- PC0 → PC1
    
    ![image.png](/images/2025-02/2025-02-06/06-16.png)
    
- PC0 → PC2
    
    ![image.png](/images/2025-02/2025-02-06/06-17.png)
    

# 네트워크 구조도 설계

## 본사 네트워크 설계

<aside>
:bulb: **목표**
    
1. 인터넷 연결 및 내부망 구축
2. 사무실 내 유선/무선 네트워크 구성
3. 서버 및 보안 장비 배치
4. VLAN을 활용한 부서별 네트워크 분리

</aside>

### 1. 본사 네트워크 기본 개요

1. 인터넷 연결 (ISP)
    1. ISP로부터 인터넷 연결을 받음
    2. 광케이블 또는 전용 회선 사용 가능
2. 네트워크 핵심 장비
    1. 라우터: 인터넷과 내부망 연결
    2. 방화벽: 내부망 보호, VLAN 50/200 차단
    3. 코어 스위치: 네트워크 중앙 제어, VLAN 트래픽 처리
    4. 액세스 스위치: 각 부서의 PC 및 장비 연결
    5. 무선 AP: 직원용 & 게스트 Wi-Fi 제공
3. 사내 서버 구성
    1. DHCP 서버: 사내 IP 자동 할당 (192.168.x.x)
    2. DNS 서버: 내부 도메인 네임 시스템 관리
    3. 파일 서버 (NAS): 직원들이 공유 파일을 저장하는 스토리지
    4. 메일 서버: 회사 이메일 관리
    5. 웹 서버: 내부 웹사이트
4. 무선 네트워크(Wi-Fi)
    1. 사내 직원용 Wi-Fi (SSID: Company-WiFi)
    2. 방문객용 Wi-Fi (SSID: Guest-WiFi, 인터넷만 허용)

### 2. 본사 네트워크 구성도

![image.png](/images/2025-02/2025-02-06/02-01.png)

### 3. 네트워크 장비 및 IP 설계

1. 라우터 & 인터넷 연결
    1. 공인 IP: 203.0.113.1 (ISP 제공)
    2. 라우터 내부 IP: 192.186.1.1
    3. 기능: NAT(Network Address Translation) 적용, 내부 네트워크 연결
2. 방화벽
    1. 외부에서 내부망으로의 접근 차단
    2. DMZ(비무장 지대) 설정 (공용 웹서버는 DMZ에 배치)
    3. VPN 서버와 연동 가능 (추후 원격 근무 시 필요)
    
    **VLAN 50, 200 → 내부망 차단**
    
    | 출발지 VLAN | 목적지 VLAN | 액션 |
    | --- | --- | --- |
    | VLAN 50 (게스트) | VLAN 10~40 (내부망) | 차단 |
    | VLAN 50 (게스트) | 인터넷 (0.0.0.0/0) | 허용 |
3. VLAN 설계 (부서별 네트워크 분리)

| 부서 | VLAN ID | 서브넷 | IP 대역 |
| --- | --- | --- | --- |
| IT팀 | 10 | 255.255.255.0 | 192.168.10.0 |
| HR팀 | 20 | 255.255.255.0 | 192.168.20.0 |
| 마케팅팀 | 30 | 255.255.255.0 | 192.168.30.0 |
| 개발팀 | 40 | 255.255.255.0 | 192.168.40.0 |
| 게스트 Wi-Fi | 50 | 255.255.255.0 | 192.168.50.0 (내부망 차단) |
| DMZ (웹 서버) | 200 |  | 192.168.200.50 (내부망 차단) |
- 부서별 네트워크를 VLAN으로 나눠서 보안과 관리 효율성을 높임
- VLAN 50은 내부망 접근을 차단하고 인터넷만 사용 가능하도록 설정

### 4. 서버 구성

| 서버 종류 | 역할 | IP 주소 |
| --- | --- | --- |
| DHCP 서버 | 내부 IP 자동 할당 | 192.168.100.10 |
| DNS 서버 | 내부 도메인 관리 | 192.168.100.20 |
| 파일 서버 | 사내 데이터 공유 | 192.168.100.30 |
| 메일 서버 | 사내 이메일 관리 | 192.168.100.40 |
| 웹 서버 (DMZ) | 외부에서 접근 가능 | 192.168.200.50 |
- 웹 서버는 따로 DMZ 안에 배치 (VLAN 200)

### 5. 무선 네트워크 (Wi-Fi)

- 사내 Wi-Fi (SSID: Company-WiFi, VLAN 10~40과 연결)
    - 대역폭 → 500Mbps ~ 1Gbps
- 방문객 Wi-Fi (SSID: Guest-WiFi, VLAN 50에 연결)
    - 내부 네트워크 차단, 인터넷 전용
    - 대역폭 → 50Mbps ~ 100Mbps

### 6. 본사 네트워크 보안 정책

- 방화벽에서 내부망 보호
- VPN을 통한 원격 접속 지원 (추후 원격 근무 시 반영)
- 부서별 VLAN 적용하여 보안 강화
- 게스트 Wi-Fi와 사내망 분리하여 보안 유지
- 서버 접근은 관리 네트워크(VLAN 100)에서만 허용
