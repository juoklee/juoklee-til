# java.lang Package

<br/>

> **java.lang**

자바는 기본적으로 다양한 패키지를 지원한다.
1. 문자열과 관련된 String, StringBuffer, StringBuilder
2. 화면에 값을 출력할 때 사용하는 System
3. 수학과 관련된 Math
4. 스레드와 관련된 중요 클래스들
5. 이외에도 다양한 클래스와 인터페이스


<br/>

> **Wrapper Class**

기본형 데이터 타입의 객체화를 가능하게 도와주는 클래스로, java.lang 패키지에 포함되어 있다. <br/>
`Boolean, Byte, Short, Integer, Long, Float, Double` <br/>
하지만 산술 연산을 위해 정의된 클래스가 아니므로, 인스턴스에 저장된 값을 변경할 수 없다. <br/>
단지, 값을 참조하기 위해 새로운 인스턴스를 생성하고, 생성된 인스턴스의 값만을 참조할 수 있다.

<br/>

> **박싱(Boxing) / 오토박싱(Auto Boxing)**
 1. 박싱

  기본 타입의 데이터를 wrapper 클래스의 인스턴스로 변환하는 과정 
  
 2. 오토박싱

  JDK 1.5부터 지원하는 자동화된 박싱 


<br/>

> **언박싱(Unboxing) / 오토 언박싱(Auto Unboxing)**

 1. 언박싱

   wrapper 클래스의 인스턴스에 저장된 값을 다시 기본 타입의 데이터로 꺼내는 과정
  
 2. 오토 언박싱

  JDK 1.5부터 지원하는 자동화된 오토 언박싱 


<br/>

```java
public class WrapperEx {
  public static void main(String[] args) {
    int i = 5;
    Integer i2 = new Integer(5); //박싱
    Integer i3 = 5; //오토박싱
    int i4 = i2.intValue(); //언박싱
    int i5 = i2; //오토언박싱
```



<br/><br/>

> **참고**

[자바 중급 강의](https://school.programmers.co.kr/learn/courses/9)

[Wrapper 클래스](http://www.tcpschool.com/java/java_api_wrapper)

<br/>

_2022.12.19_
