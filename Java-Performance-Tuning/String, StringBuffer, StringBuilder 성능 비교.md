# String, StringBuffer, StringBuilder

JDK 5.0 기준으로 문자열을 만드는데 사용되는 클래스는 String, StringBuilder, StringBuffer 이렇게 세가지가 가장 많이 사용된다.

## String

String 클래스는 문자열을 저장하기 위해서 문자열 배열 참조변수를 멤버변수(인스턴스 변수)로 정의해놓고 있으며 인스턴스 초기화(생성) 시 생성자의 
매개변수로 받는 문자열로 인스턴스 변수를 초기화한다.

이 인스턴스 변수를 살펴보면 다음과 같이 final 로 선언되어 있으며 보다 싶이 String 클래스 자체가 final 로 선언되어 있기 때문에 상속이 불가능하다.
즉 생성된 String 클래스의 인스턴스는 한번 생성되는 시점에 그 값을 읽을 수만 있고, 변경할 수는 없는 불변 객체인데 이러한 특성으로 인해 연산을 할 때 불필요한 메모리공간의 낭비가 발생할 수 있다.

```java
class Ex{
    String a = "a";
    String b = "b";
    public static void main(String[] args) {
        // 새로운 메모리공간에 "ab"이 담긴 String 인스턴스가 생성 
        a = a + b;
        
        // 계속 새로운 String 인스턴스 생성... 메모리 공간 낭비
        for(int i = 0; i<100000; i++){
            a += b;
        }
    }
}
```
다음과 같은 + 연산을 하게 되면 문자열이 결합되게 되는데, 아까 말했듯이 String 인스턴스는 불변 객체로 한번 생성된 문자열은 변경할 수 없다. 
즉, + 연산을 하면 인스턴스 내부의 멤버 변수 value 가 변경되는게 아니라 새로운 인스턴스가 생성되어 새로운 메모리공간을 차지하게 된다.

## StringBuffer

StringBuffer 클래스는 String 클래스와 비슷하게 문자열을 저장하기 위한 char 형 배열의 참조변수를 인스턴스 변수로 선언해놓고 있다. 그래서
인스턴스를 생성할 때 적절한 길이의 배열이 생성되고, 이 배열은 문자열을 저장하고 편집하기 위한 공간(Buffer) 로 사용된다.

String 클래스가 인스턴스를 생성할 때, 지정된 문자열을 변경할 수 없는 것과 달리, StringBuffer 클래스는 내부적으로 문자열 편집을 위한 
버퍼(buffer) 를 가지고 있기 때문에 변경이 가능하다. 물론 변경이 가능하지만 모두가 알듯이 배열의 크기는 정적이다. 그렇기 때문에 길이가 부족하면 
새로운 길이의 배열을 다시 생성한 이후 이전 배열의 값을 복사하는 과정을 거쳐야하며 이는 매우 비효율적이다. 
그렇기 떄문에 편집할 문자열의 길이를 고려하여 버퍼의 길이를 충분히 잡아주는 것이 좋다. 
> 만약 버퍼의 크기를 지정해주지 않으면 16개의 문자를 저장할 수 있는 크기의 버퍼를 생성한다.

String 과 달리 StringBuffer 는 buffer 를 가지고 있기 때문에 문자열을 추가하는 작업 등이 발생할 때마다 새롭게 인스턴스를 생성하는 것이 아닌
버퍼를 이용하여 (추가/삭제 등의)작업이 수행된다. 이러한 동작 원리 덕분에 반복적인 작업을 수행할 때 String 보다 GC 성능에 합리적이다.
> GC 를 하면 할 수록 CPU 를 많이 사용하게 되며, 시간도 많이 소요된다.

## StringBuilder

StringBuffer 는 Safe Thread 하지만, 불필요한 동기화로 인한 성능 저하가 일어난 여지가 있다. 
> 멀티쓰레드로 작성된 프로그램이 아닌 경우, 동기화는 불필요하게 성능만 저하시킬 뿐이다.

StringBuilder 는 StringBuffer 에서 동기화만 뺀, 완전히 똑같은 기능으로 작성되어 있는 클래스라고 생각하면 된다. 
> StringBuffer 도 충분히 뛰어난 성능을 지니고 있기 때문에 유의미한 차이가 발생하지 않는 이상, 굳이 StringBuilder 로 변경할 이유는 없다.

## 성능 비교 (by JMH)

```java
@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class JmhTest {

    private static String str = "A";

    @Benchmark
    public String getByString(){
        String result = str;
        for(int i = 0; i<10000; i++){
            result += str;
        }
        return result;
    }

    @Benchmark
    public String getByStringBuffer(){
        StringBuffer result = new StringBuffer(str);
        for(int i = 0; i<10000; i++){
            result.append(str);
        }
        return result.toString();
    }

    @Benchmark
    public String getStringBuilder(){
        StringBuilder result = new StringBuilder("A");
        for(int i = 0; i<10000; i++){
            result.append(str);
        }
        return result.toString();
    }

}

```
```
10번 반복

Benchmark                  Mode  Cnt   Score   Error  Units
JmhTest.getByString        avgt       ≈ 10⁻⁴          ms/op
JmhTest.getByStringBuffer  avgt       ≈ 10⁻⁴          ms/op
JmhTest.getStringBuilder   avgt       ≈ 10⁻⁴          ms/op
```
```

10000번 반복 

Benchmark                  Mode  Cnt  Score   Error  Units
JmhTest.getByString        avgt       4.223          ms/op
JmhTest.getByStringBuffer  avgt       0.219          ms/op
JmhTest.getStringBuilder   avgt       0.078          ms/op
```

위와 같이 반복횟수가 크면 처리하는데 많은 시간 차이가 발생하는 것을 볼 수 있다.