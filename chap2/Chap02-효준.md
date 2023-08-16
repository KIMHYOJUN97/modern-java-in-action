# CHAP 2 동적 파라미터화 코드 전달하기
개발자들은 반복되는 코드는 물론이고 엔지니어링 비용을 가장 최소화 시키는 것을 항상 고민한다. 또한 새로 추가한 기능은 쉽게 구현할 수 있어야 하며 장기적인 관점에서 유지보수가 쉬워야 한다.\
**동적 파라미터화**를 사용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.\
동적 파라미터화란 아직 어떻게 실행할 것인지 결정하지 않은 코드 블록이다. 이 코드 블록은 프로그램에서 호출한다.
## 2.1 변화하는 요구사항에 대응하기
기존에 빨간색 사과만을 필터랑 하던 농장 재고목록 애플리케이션 리스트에서 녹색 사과도 필터링하는 기능을 추가해보자.
### 녹색사과 필터링
```Java
public static List<Apple> fileterGreenApples(List<Apples> inventory){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory){
        if (GREEN.equals(apple.getColor())){
            result.add(apple);
        }
    }
    return result;
}
```
`GRREN.equals(apple.getColor())`를 보면 녹색 사과를 선택하는 데 필요한 조건을 가리킨다. 이때 빨간색 사과도 필터링하고 싶으면 어떻게 할까?\
가장 쉬운 방법은 filterRedApples라는 메소드를 하나 더 생성하는 것이다. 하지만 이게 최선인지는 뒤에서 조금 더 공부해보자
### 2.1.2 색을 파라미터화
어떻게 해야 filterGreenApples의 코드를 반복하지 않고 filterRedApples를 만들 수 있을까? 색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 조금 더 유연해진다!
```Java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if(apple.getColor().equals(color)){
            result.add(apple);
        }
    }
    return result;
}
```
이제 다음과 같이 메서드를 호출할 수 있다.
```Java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> greenApples = filterApplesByColor(inventory, RED);
```
그런데 갑자기 농부가 색 이외에도 무게를 추가하고 싶다고 요구사항을 늘렸다. 대응하기 위해 무게 정보 파라미터도 추가해보자.
```Java
public static List<Apple> filterApplesByColor(List<Apple> inventory, int weight){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if(apple.getWeight()>weight){
            result.add(apple);
        }
    }
    return result;
}
```
위 코드도 좋은 예시일 수 있지만 개발자에게 중요한 건 같은 것을 반복하지 않는 것 `DRY(dont repeat yourself)`이다.\
색과 무게를 filter라는 메서드로 합치는 방법도 있지만, 어떤 기준으로 필터링할지 구분하는 또 다른 방법이 필요하다. 따라서 색이나 무게 중 어떤 것을 기준으로 필요할 지 플래그를 추가할 수도 있지만 절대 이 방법을 사용하면 안된다.
### 2.1.3 가능한 모든 속성으로 필터링
안좋은 예시를 봐보자
```Java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color,int weight, boolean flag){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if((flag && apple.getColor().equals(color))|| (!flag && apple.getWeight() > weight)){
            result.add(apple);
        }
    }
    return result;
}
```
다음과 같은 메소드 호출이 일어난다
```Java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> redApples = filterApples(inventory, null, 150, false);
```
형편없는 코드이다. true와 false는 무엇을 의미하는지 가시성이 보이지 않고 유연한 대응도 불가능하다. 예를 들어 사과의 크기, 모양, 출하지 등으로 사과를 필터링 하고 싶다면 어떻게 해야할까? 또 똑같은 방식으로 매개변수를 추가해 나가야 한다.
## 2.2 동작 파라미터화
위의 방식처럼 파라미터를 추가하는 방법이 아닌 좀 더 유연하게 대응할 수 있는 방법이 절실하다.\
우리의 선택 조건을 다음처럼 결정할 수 있다. 사과의 어떤 속성에 기초해서 불리언값을 반환하는 방법이 있다. 참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 한다.\
**선택 조건을 결정하는 인터페이스**를 정의하자
```Java
public class AppleHeavyWeightPredicate implements ApplePredicate{
    public boolean test(Apple apple){
        return apple.getWeight()>150;
    }
}

public class AppleGreenColorPredicate implements ApplePredicate{
    public boolean test(Apple apple){
        return GREEN.equals(apple.getColor());
    }
}
```
위 조건에 따라 filter 메서드가 다르게 종작할 것이라고 예상할 수 있다. 이를 전략 디자인 패턴(strategy design pattern)이라고 한다.\
전략 디자인 패턴은 각 알고리즘(전략이라 부르는)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 **런타임**에 알고리즘을 선택하는 기법이다.\
그런데 ApplePredicate는 어떻게 다양한 동작을 수행할 수 있을까? filterApples에서 ApplePredicate 객체를 받아 애플의 조건을 검사하도록 메서드를 고쳐야 한다. 이렇게 동작 파라미터화, 즉 메서드가 다양한 동작을 받아서 내부적으로 다양한 도앚긍ㄹ 수행할 수 있다.
### 추상적 조건으로 필터링
다음은 ApplePredicate를 이용한 필터 메서드이다.
```Java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if(p.test(apple)){
            result.add(apple);
        }
    }
    return result;
}
```
첫 번째 코드에 비해 훨씬 유연하며 가독성도 좋아졌다. 이제 150이 넘는 빨간 사과를 검색해달라고 부탁하면 우리는 ApplePredicate를 적절하게 구현하는 클래스를 만들기만 하면 된다!
```Java
public class AppleRedAndHeavyPredicate implements ApplePredicate{
    public boolean test(Apple apple){
        return RED.equals(apple.getColor()) && apple.getWeight() > 150;
    }
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```
우리가 전달한 ApplePredicaate 객체에 의해 filterApples 메서드의 동작이 결정된다! 즉, 우리는filterApples 메서드의 동작을 파라미터화 했다.\
지금까지 살펴본 것처럼 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 동적 파라미터화의 강점이다. 따라서 유연한 API를 만들 때 동작 파라미터화가 중요한 역할을 한다\
하지만 여러 클래스를 구현해서 인스턴스화하는 과정이 조금은 거추장스럽게 느껴질 수 있다. 이를 어떻게 개선할까?
## 2.3 복잡한 과정 간소화
현재 filterApples 메소드로 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화해야 한다. 이는 상당히 번거로운 작업이다. 로직과 관련 없는 많은 코드들을 어떻게 줄일 수 있을까?\
바로 **익명 클래스**라는 기법을 제공받아 사용하면 된다.
### 2.3.1 익명클래스
익명 클래스는 자바의 지역 클래스와 비슷한 개념이지만 이름이 없는 클래스이다. 익명 클래스를 이용하면 클래스 선언과 동시에 인스턴스화를 할 수 있다. 즉, 즉석에서 필요한 구현을 만들 수 있다
### 익명 클래스 사용
```Java
List<Apple> redApples = filterApples(inventory, new ApplePredicate(){
    public boolean test(Apple apple){
        return RED.equals(apple.getColor());
    }
})
```
이와 같이 사용하면 ApplePredicate의 클래스를 따로 생성할 필요가 없다. 하지만 익명 클래스도 여전히 많은 공간을 차지한다. 코드의 장황함은 나쁜 특성이다. 이를 어떻게 간결하게 정리할 수 있을까?
### 람다 표현식 사용
자바 8의 람다 표현식을 이용해서 위 예제 코드를 다음처럼 간단하게 재구현할 수 있다.
```Java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```
이전 코드와 다르게 매우 간결해졌다!
### 리스트 형식으로 추상화
```Java
public interface Predicate<T>{
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){
    List<T> result = new ArrayList<>();
    for(T e : list){
        if(p.test(e)){
            result.add(e);
        }
    }
    return result;
}
```
이제 여러 타입을 `<T>`를 사용한 덕분에 리스트 내에서 필터 메서드를 사용할 수 있다. 다음 예제를 보자
```Java
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));

List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```
이렇게 하면 유연함과 간결함 둘 다 잡을 수 있다!
## 2.4 실전예제
위와 같은 작업들을 사용 예제를 통해 확인해 보자!
### Comparator로 정렬
```Java
public interface Comparator<T>{
    int compare(T o1, T o2);
}
```
comparetor를 구현해서 sort 메서드의 동작을 다양화할 수 있다.
```Java
inventory.sort(new Comparator<Apple>()){
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
```
### Runnable로 코드 블록 실행
자바 8까지는 Thread 생성자에 객체만을 전달할 수 있었으므로 보통 결과를 반환하지 않는 void run 메소드를 포함하는 익명 클래스가 Runnable 인터페이스를 구현하도록 하는 것이 일반적이였다.
```Java
public interface Runnable{
    void run();
}

//Runnable을 이용해서 다양한 동작을 스레드로 실행할 수 있다.
Thread t = new Thread(new Runnable(){
    public void run(){
        System.out.println("Hello world")
    }
})
```
람다 표현식을 이용하면 더 쉽게 표현할 수 있다
```Java
Thread t = new Thread(()->System.out.println("Hello world"));
```
### Callable을 결과로 받기
ExecutorService 인터페이스는 태스크 제출과 실행 과정의 연관성을 끊어준다. ExecutorService를 이용해 태스크를 스레드 풀로 보내고 결과를 Future에 저장할 수 있다. 이것이 Runnable과의 차이점이다.
```Java
ExecutorService executorService = Executors.newCachedThreadPool();
Future<String> threadName = executorService.submit(new Callable<String>()){
    @Override
    public String call() throws Exception{
        return Thread.currentThread().getName();
    }
};

//람다 활용
Future<String> threadName = executorService.submit(
    () -> Thread.currentThread().getName()
);
```
## 정리
- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있다 -> 엔지니어링 비용 감소
- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달. 익명 클래스를 사용해서 더욱 간결한 코드 사용 가능