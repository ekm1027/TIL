# 전송 계층: TCP와 UDP

## 개념

응용 계층에서 전달받은 데이터를 상대방의 어떤 소켓(서비스)에 어떤 방식으로 전달할 것인지 담당한다. TCP와 UDP 모두 출발지 포트와 목적지 포트를 사용한다.

## TCP / UDP 비교

| 구분 | TCP | UDP |
| --- | --- | --- |
| 연결 | 연결 지향형 | 비연결형 |
| 연결 수립 | 3-Way Handshake | 없음 |
| 수신 확인 및 재전송 | 제공 | 자체 기능 없음 |
| 순서 보장 | 제공 | 없음 |
| 흐름·혼잡 제어 | 제공 | 자체 기능 없음 |
| 헤더 | 20~60 Byte | 8 Byte |
| 전달 방식 | 바이트 스트림 | 데이터그램 |
| 메시지 경계 | 보존하지 않음 | 보존함 |
| 예시 | 웹, 파일 전송, DB 연결 | 영상 통화, 게임, 일반 DNS 조회 |

## 동작 원리

TCP는 연결 상태, 순서 번호, ACK 등을 관리해 신뢰성 있는 바이트 스트림을 제공한다. UDP는 연결 수립 절차 없이 독립적인 데이터그램을 전송한다. 수신 여부를 확인하거나 유실을 복구하는 기능은 UDP 자체에 없다.

영상 통화에서 지나간 영상의 복구보다 최신 영상의 전달이 중요하다면 UDP를 활용할 수 있다. UDP가 항상 더 빠른 것은 아니며, QUIC처럼 UDP 위에 신뢰성과 혼잡 제어를 구현하는 프로토콜도 있다.

## 포트의 역할

| 정보 | 예시 |
| --- | --- |
| 출발지 IP / 포트 | 192.168.0.10:52341 |
| 목적지 IP / 포트 | 10.0.0.20:8080 |
| 전송 프로토콜 | TCP |

출발지 포트는 보통 OS가 임시 포트로 할당한다. TCP 연결은 출발지 IP·포트와 목적지 IP·포트의 4-Tuple로 구분한다. 프로토콜을 포함하면 5-Tuple로 표현한다. TCP 53번과 UDP 53번은 별개이므로 동시에 사용할 수 있다.

## 코드 예시

```java
// TCP: 접속 과정에서 연결 수립 (실습 서버가 먼저 실행되어 있어야 함)
try (java.net.Socket socket = new java.net.Socket("127.0.0.1", 18080)) {
    socket.getOutputStream().write(new byte[] {65, 66, 67});
}
```

```java
// UDP: 상대와 TCP 방식의 연결 수립 없이 데이터그램 전송
try (java.net.DatagramSocket socket = new java.net.DatagramSocket()) {
    byte[] data = "Hello".getBytes("UTF-8");
    java.net.DatagramPacket packet = new java.net.DatagramPacket(
        data, data.length, java.net.InetAddress.getByName("127.0.0.1"), 18080);
    socket.send(packet);
}
```

두 코드는 비교용 스니펫이다. UDP 전송 성공은 상대가 데이터를 받았다는 의미가 아니다. 전체 TCP 실습은 [소켓과 메시지 경계](09-TCP-Socket-and-Message-Framing.md) 참고.

## 면접 질문

**Q. TCP와 UDP의 차이는?**

A. TCP는 연결 지향형으로 재전송과 순서 보장을 제공하고, UDP는 비연결형으로 메시지 경계를 보존하지만 자체적인 수신 보장이나 재전송은 제공하지 않습니다.

**Q. 응용 프로토콜이 정해진 상태에서 전송 프로토콜을 자유롭게 바꿀 수 있나요?**

A. 해당 프로토콜의 규격과 양쪽 구현이 지원해야 합니다. FTP의 TCP를 임의로 UDP로 변경할 수는 없습니다.
