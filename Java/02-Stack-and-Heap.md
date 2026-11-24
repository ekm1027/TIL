# Stack과 Heap

## 개념
| 구분 | Stack | Heap |
|---|---|---|
| 저장 대상 | 메서드 실행 정보, 지역변수, 매개변수 | 객체, 배열, 인스턴스 필드 |
| 공유 | 스레드별 독립 | 스레드 간 공유 |
| 해제 | 메서드 종료 시 프레임 제거 | GC에 의한 객체 메모리 회수 |
| 대표 오류 | StackOverflowError | OutOfMemoryError |

## 코드 예시
```java
class Person {
    int age = 29;
}
// 메서드 내부
int num = 10;
Person p = new Person();
```
```text
Stack                   Heap
num = 10
p (참조값) -----------> Person 객체
                        age = 29
```
기본형도 인스턴스 필드라면 객체의 일부로 Heap에 저장된다. 위 구조는 JVM의 개념적 모델이며 실제 최적화에 따라 달라질 수 있다.

## 동작 원리
메서드 호출 시 Stack Frame이 생성되고 종료 시 제거된다. Heap 객체는 메서드가 종료되어도 다른 곳에서 접근할 수 있다면 유지된다.

## 면접 질문
Q. Stack과 Heap의 차이는?

A. Stack은 스레드별 메서드 실행 정보를 저장하고, Heap은 객체를 저장하는 공유 영역이다. Heap의 객체 메모리는 GC가 관리한다.
