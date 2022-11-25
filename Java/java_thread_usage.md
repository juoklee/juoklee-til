# Java Thread Usage

<br/>

> **Thread?**

하나의 프로세스 내부에서 독립적으로 실행되는 하나의 작업 단위
JVM에 의해 하나의 프로세스가 발생하고, `main()` 안의 실행문들이 하나의 thread가 된다.

<br/><br/>

> **Thread 생성하기**

`main()` 이외의 또 다른 thread를 만들려면 **A) Thread class**를 상속하거나 **B) Runnable interface**를 구현한다

**A-0. Thread class 상속받아서 run() 메소드 오버라이딩**
```java
public class MyThread extends Thread {

    public void run(){
       System.out.println("MyThread running");
    }
}
```
```java
MyThread myThread = new MyThread();
myTread.start();
```
start()는 호출이 이루어지면 run()이 끝나기를 기다리지 않고 바로 종료된다.

<br/>

**A-1. Thread의 Anonymous class**
```java
public class MyThread {
    public static void main(String[] args) {
        new Thread() {
            public void run() {}
        }.start();
    }
}
```

<br/>

**B-0. Runnable Interface 구현**
```java
public class MyRunnable implements Runnable {

    public void run(){
       System.out.println("MyRunnable running");
    }
}
```
```
Thread thread = new Thread(new MyRunnable());
thread.start();
```
Thread 자신의 `run()` 대신 생성자로 전달된 MyRunnable 인스턴스의 `run()`을 호출한다.

<br/>

**B-1. Runtnable Interface Anonymous class**
```java
public class MyRunnable {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            public void run() {
     
            }
        }).start();
    }
}
```
 
<br/><br/>
 
> **start(), run(), join()**

`void start()` : thread를 시작하고 자동으로 run()을 호출한다.

`void run()` : 오버라이딩 된 run()을 실행한다.

`void join()` : 다른 thread가 종료될 때까지 기다리게 하는 메서드이다.
 
 
 

<br/><br/>

> **참고**

[자바 Thread(스레드) 사용법 & 예제](https://coding-factory.tistory.com/279)

[멀티쓰레드 프로그래밍](https://wisdom-and-record.tistory.com/48) 

[Thread(쓰레드)의 정의와 사용법 정리](https://blog.naver.com/zion830/221393808512)

[자바 쓰레드 시작하기](https://parkcheolu.tistory.com/10)

[점프 투 자바 07-05 쓰레드(Thread)](https://wikidocs.net/230)
