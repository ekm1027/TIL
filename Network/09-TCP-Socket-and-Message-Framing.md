# TCP 소켓과 메시지 경계

## 개념

TCP는 바이트 스트림을 제공한다. 여러 번 write()한 데이터는 수신 측에서 한 번에 읽힐 수도 있고, 한 번 write()한 데이터가 여러 read()로 읽힐 수도 있다. TCP는 애플리케이션 메시지 경계를 보존하지 않는다.

## 코드 실습: 4000 Byte 전송

두 파일을 같은 폴더에 저장한다. 기존 WAS와 충돌하지 않도록 비어 있는 실습용 TCP 포트 18080을 가정한다.

### TcpServer.java

```java
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.InetAddress;

public class TcpServer {
    public static void main(String[] args) throws Exception {
        try (ServerSocket server = new ServerSocket(18080, 50,
                InetAddress.getByName("127.0.0.1"));
             Socket socket = server.accept();
             InputStream in = socket.getInputStream()) {
            byte[] buffer = new byte[1024];
            int total = 0;
            int length;
            while ((length = in.read(buffer)) != -1) {
                total += length;
                System.out.println("수신 크기: " + length);
            }
            System.out.println("총 수신 크기: " + total);
        }
    }
}
```

### TcpClient.java

```java
import java.io.OutputStream;
import java.net.Socket;
import java.util.Arrays;

public class TcpClient {
    public static void main(String[] args) throws Exception {
        try (Socket socket = new Socket("127.0.0.1", 18080);
             OutputStream out = socket.getOutputStream()) {
            byte[] data = new byte[4000];
            Arrays.fill(data, (byte) 'A');
            out.write(data);
            out.flush();
        }
    }
}
```

```bash
javac -encoding UTF-8 TcpServer.java TcpClient.java
# 터미널 1: 서버 실행
java TcpServer
# 터미널 2: 클라이언트 실행
java TcpClient
```

출력 예시 (실제 read() 크기는 달라질 수 있음):

```text
수신 크기: 1024
수신 크기: 1024
수신 크기: 1024
수신 크기: 928
총 수신 크기: 4000
```

`byte[1024]`는 한 번에 읽을 최대 크기이며 TCP 세그먼트 크기가 아니다. 이 예제는 클라이언트가 소켓을 닫아 EOF가 전달되면 서버가 읽기를 끝낸다. 지속 연결에서 EOF까지 기다리면 다음 메시지를 처리할 수 없으므로 별도의 메시지 규격이 필요하다.

## 동작 원리: 메시지 경계

고객 정보 4000 Byte와 계약 정보 2000 Byte를 연속으로 전송하면 TCP는 총 6000 Byte의 연속 스트림을 제공한다. 수신 프로그램에서 각각의 메시지를 구분하는 방법이 필요하다.

| 방식 | 설명 |
| --- | --- |
| 고정 길이 | 약속한 바이트 수를 읽음 |
| 길이 헤더 | 메시지 앞에 데이터 길이를 표시 |
| 구분자 | 메시지 끝을 특정 문자 등으로 표시(이스케이프 규칙 필요) |

길이 헤더 예시(연결된 소켓을 가정):

```java
// 송신
java.io.DataOutputStream output = new java.io.DataOutputStream(socket.getOutputStream());
output.writeInt(data.length);
output.write(data);
output.flush();

// 수신 측의 별도 코드
java.io.DataInputStream input = new java.io.DataInputStream(socket.getInputStream());
int size = input.readInt();
if (size < 0 || size > 1_048_576) {
    throw new java.io.IOException("Invalid message size");
}
byte[] message = new byte[size];
input.readFully(message);
```

이 예시는 길이 값에 대한 최대 제한을 두고 메시지 바이트를 모두 읽는다. 실제 운영 코드에는 타임아웃, 오류 처리, 연결 관리 등도 필요하다.

## HTTP 개발과의 차이

HTTP/1.1에는 Content-Length, Transfer-Encoding: chunked 등의 메시지 경계 규칙이 있다. 일반적인 웹개발에서는 브라우저, HTTP 라이브러리, 웹 서버/WAS가 이를 처리한다. 개발자가 TCP 분할 단위를 직접 설정할 필요는 없다.

## 면접 질문

**Q. write()와 read()의 크기는 왜 일치하지 않나요?**

A. TCP는 바이트 스트림 프로토콜이며 애플리케이션의 호출 경계를 보존하지 않습니다.

**Q. 일반적인 HTTP 개발에서 MSS를 직접 계산해야 하나요?**

A. 아닙니다. TCP 스택이 분할과 재전송을 담당하며 HTTP 라이브러리는 메시지 경계 처리를 담당합니다.
