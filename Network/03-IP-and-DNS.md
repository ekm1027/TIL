# IP와 DNS (Internet Protocol & Domain Name System)

## 개념

| 구분 | IP | DNS | TCP |
| --- | --- | --- | --- |
| 정식 명칭 | Internet Protocol | Domain Name System | Transmission Control Protocol |
| 계층 | 인터넷 계층 | 응용 계층 | 전송 계층 |
| 핵심 역할 | 목적지까지 패킷 전달 | 도메인 이름에 대응하는 IP 주소 등 조회 | 신뢰성 있는 데이터 전송 |
| 관련 정보 | IP 주소, 라우팅 | 도메인, DNS 레코드 | 포트, SEQ, ACK |
| 기억할 내용 | 어디로 보낼까? | 주소가 어디일까? | 어떻게 정확히 보낼까? |

IP 자체는 패킷의 도착이나 순서를 보장하지 않는다. IP는 프로토콜이고 IP 주소는 그 프로토콜에서 사용하는 주소다.

DNS는 분산된 이름 조회 시스템이다. 일반 DNS는 주로 UDP 53번 포트를 사용하며 TCP 53번도 사용한다. DNS over HTTPS/TLS 등의 방식도 존재한다. 캐시에 조회 결과가 있다면 매번 DNS 서버에 요청할 필요는 없다.

예시: `api.example.com`의 IP가 `10.0.0.100`이라고 가정한다.

- 도메인 호출: `https://api.example.com/customer` → DNS 조회(캐시 포함) 후 IP 주소로 연결.
- IP 직접 호출: `http://10.0.0.100:8080/customer` → 해당 목적지 이름에 대한 DNS 조회 불필요.

HTTPS에서는 인증서와 호스트 이름 검증, HTTP Host 헤더 등이 중요하므로 도메인 호출을 단순히 IP 호출로 바꾸면 정상 동작하지 않을 수 있다.

## 면접 질문

**Q. IP와 DNS의 차이는 무엇인가요?**

A. DNS는 도메인 이름에 대응하는 IP 주소 등을 조회하고 IP는 주소를 기반으로 패킷을 전달합니다.

**Q. IP 주소로 직접 호출하면 DNS 조회가 필요한가요?**

A. 해당 목적지의 이름 조회는 필요하지 않습니다. 단 HTTPS 인증서 검증과 가상 호스트 설정 때문에 IP 직접 호출이 실패할 수 있습니다.
