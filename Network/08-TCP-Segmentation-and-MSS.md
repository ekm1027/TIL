# TCP 데이터 분할: MTU와 MSS

## 개념

개발자는 전송할 데이터를 작성하고, TCP 스택은 바이트 스트림을 실제 전송할 세그먼트로 나눈다. 이 과정을 Segmentation이라고 한다. 일부 분할 작업은 NIC 오프로딩으로 수행될 수도 있다.

| 용어 | 의미 |
| --- | --- |
| MTU (Maximum Transmission Unit) | 해당 링크에서 전송할 수 있는 최대 IP 패킷 크기 (일반적인 IP over Ethernet 기준) |
| MSS (Maximum Segment Size) | TCP 세그먼트의 데이터 부분에 대한 최대 크기 |

## 동작 원리

일반적인 Ethernet MTU 1500 Byte, IPv4 기본 헤더 20 Byte, TCP 기본 헤더 20 Byte를 가정한다.

```text
IP 패킷 최대 1500 Byte
[IPv4 헤더 20B][TCP 헤더 20B][TCP 데이터 1460B]
```

이 환경에서 광고할 수 있는 MSS의 대표적인 값은 1460 Byte이다. TCP/IP 옵션을 사용하면 실제 세그먼트 데이터 크기는 더 작아질 수 있다. 경로 MTU, 상대의 MSS, 흐름 제어와 혼잡 제어, 전송할 데이터 양 등도 영향을 준다.

### 3-Way Handshake에서의 MSS

SYN과 SYN+ACK에 MSS 옵션을 넣어 각자의 수신 가능한 최대 TCP 데이터 크기를 알려줄 수 있다. 양방향 MSS는 같을 필요가 없다.

| 방향 | 상대가 광고한 MSS | 송신 측 한도 (예시) |
| --- | --- | --- |
| 클라이언트 → 서버 | 1360 B | 1360 B 이하 |
| 서버 → 클라이언트 | 1460 B | 1460 B 이하 |

실제 크기는 이보다 작을 수 있다. MSS는 최대 크기이지 고정된 전송 크기가 아니다.

## 코드 예시

```java
byte[] data = new byte[4000];
out.write(data); // out은 연결된 TCP 소켓의 OutputStream
```

MSS 1460 Byte로 최대 크기씩 나누는 단순한 예시:

```text
애플리케이션: 4000 B
       ↓ TCP 세그먼테이션
[1460 B] [1460 B] [1080 B]
       ↓ 수신 측 TCP의 순서 복원
연속 바이트 스트림: 총 4000 B
```

실제 로컬 루프백 MTU 및 MSS는 Ethernet 예시와 다를 수 있다. 애플리케이션의 write() 호출 경계는 TCP 세그먼트 경계와 일치할 필요가 없다.

```java
socket.setSendBufferSize(8192);
socket.setReceiveBufferSize(8192);
byte[] readBuffer = new byte[1024];
```

| 설정 | 의미 |
| --- | --- |
| data.length | 애플리케이션이 전송하려는 바이트 수 |
| setSendBufferSize / setReceiveBufferSize | OS 소켓 버퍼 크기 설정 요청(힌트) |
| MSS | TCP 세그먼트의 최대 데이터 크기 |
| readBuffer.length | 애플리케이션이 한 번에 읽으려는 최대 크기 |

버퍼 크기 설정은 MSS를 직접 설정하는 것이 아니다. UDP의 데이터그램 크기·경계는 애플리케이션이 정하며, 너무 큰 데이터그램은 IP 단편화가 발생하거나 전송에 실패할 수 있다.

## 면접 질문

**Q. TCP 데이터 분할 크기는 누가 결정하나요?**

A. 일반적으로 운영체제 TCP 스택이 상대방의 MSS와 경로 MTU, 전송 상태 등을 고려해 자동으로 결정합니다.

**Q. MTU와 MSS의 차이는?**

A. 일반적인 IP over Ethernet에서 MTU는 IP 패킷 전체의 최대 크기이고, MSS는 TCP 세그먼트 데이터 부분의 최대 크기입니다.
