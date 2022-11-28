# Object와 Overriding

<br/>

> **Object 클래스?**

- `java.lang.Object
- 모든 클래스의 최상위 클래스
- 아무것도 상속받지 않으면 자동으로 `Object`를 상속
- `Object`가 가지고 있는 메소드는 모든 클래스에서 다 사용할 수 있다는 것을 의미.



<br/><br/>

> **메서드**

**1. `equals()` 객체가 가진 값을 비교할 때 사용**
  ```java
    package javaStudent;

    public class Student {

      String name;
      String number;
      int birthYear;

      public static void main(String[] args) {
        Student s1 = new Student();
        s1.name = "juok";
        s1.number = "1234";
        s1.birthYear = 1998;

        Student s2 = new Student();
        s2.name = "juok";
        s2.number = "1234";
        s2.birthYear = 1998;

        if(s1.equals(s2))
          System.out.println("s1 == s2");
        else
          System.out.println("s1 != s2");
      }
    }
  ```
  ```console
  s1 != s2
  ```
  - `equals()`는 주소값이 다른 객체는 서로 다른 객체로 판단한다.
  - 두 객체를 같도록 하려면 재정의(override)해야 한다.
  
  <br/>
  
**2. `hashCode()` 객체의 해시코드 값 반환**
  ```java
    package javaStudent;

    public class Student {

      String name;
      String number;
      int birthYear;

      public static void main(String[] args) {
        Student s1 = new Student();
        s1.name = "juok";
        s1.number = "1234";
        s1.birthYear = 1998;

        Student s2 = new Student();
        s2.name = "juok";
        s2.number = "1234";
        s2.birthYear = 1998;

        System.out.println("s1.hashCode: " + s1.hashCode());
        System.out.println("s2.hashCode: " + s2.hashCode());
      }
    }
  ```
  ```console
    s1.hashCode: 1365202186
    s2.hashCode: 1651191114
  ```
  
  <br/>
  
  **+)Override: eclipse를 통해서 생성**
  
    ![image](https://user-images.githubusercontent.com/72849620/204213266-b169e69b-9524-4b53-8981-a49a023f8577.png)
  
  ```java
    @Override
    public int hashCode() {
      return Objects.hash(number);
    }

    @Override
    public boolean equals(Object obj) {
      if (this == obj)
        return true;
      if (obj == null)
        return false;
      if (getClass() != obj.getClass())
        return false;
      Student other = (Student) obj;
      return Objects.equals(number, other.number);
    }
  ```
  ```console
    s1 == s2
    s1.hashCode: 1509473
    s2.hashCode: 1509473
  ```
  - `equals`를 재정의한 클래스는 `hashcode`도 반드시 재정의한다.
  - equals가 두 객체를 같다고 판단했으면, 두 객체의 hashCode 값은 항상 같다.
  
  <br/>
  
**3. `toString()` 객체가 가진 값을 문자열로 반환**
  **Override: eclipse를 통해서 생성**
  
    ![image](https://user-images.githubusercontent.com/72849620/204213786-011a6f15-9c50-4569-a822-39c3d01178ab.png)
  
  ```java
    @Override
    public String toString() {
      return "Student [name=" + name + ", number=" + number + ", birthYear=" + birthYear + "]";
    }
  ```
  <br/>
  
  
  

<br/><br/>

> **참고**

[자바 중급 강의](https://school.programmers.co.kr/learn/courses/9)
[[Java] equals() & hashcode() 메서드는 언제 재정의해야 할까?](https://velog.io/@sonypark/Java-equals-hascode-메서드는-언제-재정의해야-할까)

<br/>

_2022.11.28_
