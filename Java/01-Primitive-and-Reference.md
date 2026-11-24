# 기본형과 참조형 (Primitive & Reference Type)

## 개념
- 기본형: 실제 값을 저장한다. `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`의 8가지가 있다.
- 참조형: 객체를 가리키는 참조값을 저장한다. String, 배열, 클래스 및 인터페이스 타입 등이 해당한다.

## 코드 예시
```java
int a = 10;
int b = a;
b = 20;
System.out.println(a); // 10

int[] x = {10, 20};
int[] y = x;
y[0] = 100;
System.out.println(x[0]); // 100
```
기본형은 실제 값이 복사되며, 참조형은 참조값이 복사된다. x와 y는 같은 배열을 가리킨다.

```java
String s = null; // 객체를 참조하지 않음
String t = new String("Hello"); // 새 객체 생성
String u = t; // 기존 객체 참조
```
참조형 변수 선언과 객체 생성은 서로 다른 개념이다. Java는 메서드 인자 전달 시 항상 Call by Value를 사용한다.

## 면접 질문
Q. 기본형과 참조형의 차이는?

A. 기본형은 실제 값을 저장하고 참조형은 객체의 참조값을 저장한다. 복사할 때 기본형은 값이, 참조형은 참조값이 복사된다.
