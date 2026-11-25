# HTTP와 TCP 소켓 통신 (HTTP & TCP Socket)

## 개념

HTTP는 응용 계층의 프로토콜이고 TCP는 전송 계층의 프로토콜이다. 따라서 둘은 같은 계층에서 선택하는 대안이 아니다. 실무에서 말하는 'HTTP 방식 vs TCP 방식'은 보통 표준 HTTP 통신과 자체 프로토콜을 사용하는 TCP 소켓 직접 통신을 비교하는 표현이다.

| 구분 | HTTP 통신 (HTTP/1.1·HTTP/2) | TCP 소켓 직접 통신 |
| --- | --- | --- |
| 응용 계층 규칙 | 표준 HTTP | 자체 정의하거나 별도 프로토콜 사용 |
| 전송 계층 | TCP | TCP |
| 메시지 경계 | HTTP 규격에 따라 처리 | 길이 필드, 구분자 등 별도 규칙 필요 |
| 개발 및 유지보수 | 표준 라이브러리 활용이 쉬움 | 자체 메시지 처리, 오류 처리 등이 필요 |
| 사용 사례 | 웹 API, 일반 시스템 간 연계 | 금융 전문 연계, 장비 연동 등 |

HTTP/3는 TCP가 아닌 UDP 기반의 QUIC을 사용한다.

## 응용 프로토콜별 전송 방식

| 응용 프로토콜 | 일반적인 하위 전송 방식 |
| --- | --- |
| HTTP/1.1, HTTP/2 | TCP |
| HTTP/3 | QUIC over UDP |
| FTP | TCP (제어 연결과 데이터 연결 모두) |
| TFTP | UDP |
| DNS | UDP 또는 TCP (일반 DNS) |
| SSH | TCP |

HTTP 요청의 정상 응답은 HTTP 형식이어야 한다. FTP 응답을 그대로 보내면 HTTP 클라이언트가 정상적으로 해석할 수 없다. 다만 WAS가 별도의 FTP 연결로 파일을 가져와 HTTP 응답으로 전달하는 구조는 가능하다.

## 코드 예시

HTTP 통신:

```java
URL url = new URL("http://127.0.0.1:1111/customer");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.setConnectTimeout(5000);
conn.setReadTimeout(10000);
try {
    int status = conn.getResponseCode(); // 실제 연결 및 요청 수행
    System.out.println(status);
} finally {
    conn.disconnect();
}
```

TCP 소켓 직접 통신 (줄바꿈을 메시지 구분자로 사용하는 자체 규칙):

```java
try (Socket socket = new Socket("127.0.0.1", 9000)) {
    OutputStream out = socket.getOutputStream();
    out.write("CUSTOMER|100\n".getBytes("UTF-8"));
    out.flush();
}
```

TCP는 바이트 스트림이므로 `write()`를 두 번 호출했다고 수신 측의 `read()`도 두 번으로 나뉘는 것은 아니다. 메시지 길이나 구분자 같은 응용 계층 규칙이 필요하다.

## 실제 통신 프로토콜을 확인하는 방법

- 호출 URL의 스킴 확인: `http://`, `https://`, `ftp://` 등.
- 호출 소스 및 공통 라이브러리 확인: `HttpURLConnection`, HTTP Client, `Socket` 등. 단, Socket을 사용해 직접 HTTP를 구현할 수도 있다.
- 인터페이스 정의서, EAI/PO 설정, 웹서버/WAS 설정 확인.
- 필요 시 승인받은 환경에서 Wireshark 또는 tcpdump로 패킷 분석. HTTPS의 HTTP 메시지는 TLS로 암호화되어 있다.

포트 번호만으로 응용 프로토콜을 확정할 수 없으며, L4를 거친다는 사실만으로 HTTP 통신이라고 판단할 수도 없다.

## 면접 질문

**Q. HTTP와 TCP는 어떻게 다른가요?**

A. HTTP는 응용 계층의 요청·응답 규칙이고 TCP는 전송 계층의 신뢰성 있는 전송 규칙입니다. HTTP/1.1과 HTTP/2는 TCP를 사용합니다.

**Q. HTTP 요청에 FTP로 응답할 수 있나요?**

A. 정상적인 HTTP 응답은 HTTP 규격을 따라야 합니다. 서버 내부에서 별도 FTP 통신으로 파일을 가져와 HTTP로 응답하는 것은 가능합니다.

**Q. TCP 소켓 통신에서 메시지 경계는 어떻게 구분하나요?**

A. TCP는 바이트 스트림이므로 길이 필드 또는 줄바꿈 등 응용 계층에서 메시지 경계를 정의해야 합니다.
