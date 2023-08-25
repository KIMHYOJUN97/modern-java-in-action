# CHAP 03 람다 표현식
## 3.1 람다란 무엇일까?
람다 표현식은 메소드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다.\
람다 표현식에는 이름은 없지만, 파라미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트는 가질 수 있다.
```Java

//람다식을 사용하지 않은 코드
Comparator<Apple> byWeight = new Comparator<Apple>(){
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight);
    }
}

//람다식을 사용한 코드
Comparator<Apple> byWeight = 
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
코드가 훨씬 간결해짐을 볼 수 있다.\
람다는 세 부분으로 나뉘어진다.(Apple a1, Apple a2)와 같은 **람다 파라미터**, ->인 **화살표** a1.getWeight().compareTo(a2.getWeight())인 **람다 바디**로 볼 수 있다.
## 3.2 람다를 어디에 사용할까?
2장의 개념인 함수형 인터페이스의 대표인 Predicate<T>에 대해 다시 짚고 가자.\
Predicate는 함수형 인터페이스로 오직 하나의 추상 메서드만 지정할 수 있다.
```Java
public interface Predicate<T>{
    boolean test (T t);
}
```
함수형 인터페이스로는 뭘 할 수 있을까? 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**할 수 있다.
```Java
Runnable r1 = () -> System.out.println("Hello World 1"); //람다 사용

Runnable r2 = new Runnable(){                          //익명 클래스 사용
    public void run(){
        System.out.println("Hello World 2"); 
    }
};

public static void process(Runnable r){
    r.run();
}
process(r1);
process(r2);
process(() -> System.out.println("Hello World 3"));
```
### 함수 디스크립터
함수형 인터페이스의 추상 메서드 **시그니처**는 람다 표현식의 시그니처를 가리킨다.\
람다 표현식의 시그니처를 서술하는 메서드를 **함수 디스크립터**라고 부른다. 예를 들어 Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로 Runnable 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다.

자바의 모든 형식은 참조형(Reference Type = Byte, Integer, Object, List ...)과 기본형(Primitive Type = int, double, char, byte)로 이루어져 있다. 하지만 제네릭 파라미터에는 **참조형**만 사용할 수 있다. 제네릭의 내부 구현 때문에 어쩔 수 없다. 자바에서는 기본형을 참조형으로 변환하는 기능을 제공한다. 이 기능을 **박싱**이라고 한다. 참조형을 기본형으로 변환하는 반대 동작을 **언박싱** 이라고 한다.\
***하지만 이러한 변환 과정은 비용이 소모된다. 박싱한 값은 기본형을 감싸는 래퍼며 힙에 저장된다. 따라서 박싱한 값은 메모리를 더 소비하며 기본형을 가져올 때도 메모리를 탐색하는 과정이 필요하다.***\
자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.
```Java
public interface IntPredicate{
    boolean test(int t);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000); //참(박싱 없음)

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
oddNumbers.test(1000); //거짓(박싱)
```

일반적으로 특정 형식을 입력받는 함수형 인터페이스의 이름 앞에는 DoubleFunction, IntConsumer 등의 형식명이 붙는다.
## 3.5 형식 검사, 형식 추론, 제약
람다가 사용되는 콘텍스트를 이용해서 람다의 형식을 추론할 수 있다. 어떤 콘텍스트에서 기대되는 람다 표현식의 형식을 **대상 형식(target type)**이라고 한다.
### 지역 변수 사용
람다 표현식에는 익명 함수가 하는 것처럼 자유 변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 이와 같은 동작을 **람다 캡처링**이라고 부른다.
```Java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```
하지만 자유 변수에도 약간의 제약이 있다. 람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처(자신의 바디에서 참조할 수 있도록)할 수 있다. 그러기 위해 지역변수는 final로 선언되어 있꺼나, 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.\
즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다. 위 코드에서 portNumber가 한번 더 값을 할당 한다면 컴파일 에러가 발생한다.

왜 이런 제약이 있을까?
```
내부적으로 인스턴스 변수와 지역 변수는 태생부터 다르다. 인스턴스 변수는 힙에 저장되는 반면에 지역 변수는 스택에 위치한다. 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다.
따라서 자바 구현에서는 원래 변수에 접근 허용을 하지 않고 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생겼다.
```
## 메서드 참조
메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.
```Java
inventory.sort((Apple a1, Apple a2) ->
                a1.getWeight().compareTo(a2.getWeight()));

//메서드 참조
inventory.sort(comparing(Apple::getWeight));
```
**명시적으로 메서드명을 참조함으로써 가독성을 높일 수 있다.**

## 정리
- 람다 표현식은 익명함수의 일종이다.
- 람다 표현식으로 간결한 코드를 구현할 수있다.
- 함수형 인터페이스는 한아ㅢ 추상 메서드만을 정의하는 인터페이스이다
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다
- 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공할 수 있으며 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다
- IntPredicate, IntToLongFunction 등과 같은 기본형 특화 인터페이스도 제공한다
- 람다 표현식의 기대 형식을 대상 형식이라고 한다
- 메서드 참조를 이용하면 기존 메서드 구현을 재사용하고 직접 전달할 수 있다
- 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다.
  