# String과 불변성

## 개념
String은 기본형이 아닌 참조형이며 불변(Immutable) 객체이다. 생성된 String 객체의 문자열 내용은 변경할 수 없다.

## 코드 예시
```java
String a = "Hello";
String b = a;
b = "World";
System.out.println(a); // Hello
System.out.println(b); // World
```
b의 참조 대상이 바뀐 것이며 기존 Hello 객체가 수정된 것은 아니다.

```java
String x = "Hello";
String y = new String("Hello");
System.out.println(x == y); // false
System.out.println(x.equals(y)); // true
```
- `==`: 참조형에서는 동일 객체 참조 여부를 비교한다.
- `equals()`: String에서는 문자열 내용을 비교한다.

## 실무 연관성
문자열 내용 비교에는 equals()를 사용한다. null 가능성이 있다면 `"Hello".equals(value)`처럼 비교할 수 있다.

## 면접 질문
Q. String은 왜 여러 변수에서 공유해도 안전한가?

A. 불변 객체이므로 한 변수가 다른 문자열을 할당받더라도 기존 객체의 내용은 변경되지 않는다.
