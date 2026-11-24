# String Constant Pool

## 개념
동일한 문자열을 대표하는 String 객체를 재사용하는 JVM의 문자열 상수 풀이다. HotSpot Java 7부터 풀에 등록된 String 객체는 Heap에 위치한다.

## 코드 예시
```java
String a = "Hello";
String b = "Hello";
String c = new String("Hello");

System.out.println(a == b); // true
System.out.println(a == c); // false
System.out.println(a.equals(c)); // true
```
```text
a ----+
      +----> "Hello" (풀에 등록된 객체)
b ----+
c ---------> "Hello" (별도 객체)
```
문자열 리터럴은 풀의 객체를 재사용한다. new String()은 별도의 String 객체를 생성한다. 모든 실행 중 생성 문자열이 자동으로 풀에 등록되는 것은 아니다.

## intern()
```java
c = c.intern();
System.out.println(a == c); // true
```
intern()은 동일한 내용의 대표 문자열 객체를 반환한다.

## 사용 목적
중복 객체 생성과 메모리 사용을 줄인다. String의 불변성 덕분에 객체 공유가 안전하다.

## 면접 질문
Q. 문자열 리터럴과 new String()의 차이는?

A. 문자열 리터럴은 풀에 등록된 대표 객체를 재사용하고, new String()은 별도 객체를 생성한다.
