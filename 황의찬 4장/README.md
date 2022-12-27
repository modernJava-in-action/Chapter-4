# Chapter 4 - 스트림 소개
SQL에서는 질의를 어떻게 구현해야 할지 명시할 필요가 없으며 구현은 자동으로 제공됩니다.  
많은 요소를 포함하는 커다란 컬렉션의 경우, 성능을 높이려면 멀티코어 아키텍처를 활용해서 `병렬`로 컬렉션의 요소를 처리해야 합니다.  

## 4.1 스트림이란 무엇인가?
**선언형 = 데이터를 처리하는 임시 구현 코드 대신 질의로 표현할 수 있다**  
스트림을 이용하면 선언형으로 컬렉션 데이터를 처리할 수 있습니다.  
또한 스트림을 이용하면 멀티스레드 코드를 구현하지 않아도 데이터를 **투명하게** 병렬로 처리할 수 있습니다.  
  
다음은 저칼로리의 요리명을 반환하고, 칼로리를 기준으로 요리를 정렬하는 자바 7 코드입니다.  
```java
package com.me.modernJavainAction.chapter4;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class LowCalories {

  public static final List<Dish> menu = Arrays.asList(
      new Dish("pork", false, 800, Type.MEAT),
      new Dish("beef", false, 700, Type.MEAT),
      new Dish("chicken", false, 400, Type.MEAT),
      new Dish("french fries", true, 530, Type.OTHER),
      new Dish("rice", true, 350, Type.OTHER),
      new Dish("season fruit", true, 120, Type.OTHER),
      new Dish("pizza", true, 550, Type.OTHER),
      new Dish("prawns", false, 400, Type.FISH),
      new Dish("salmon", false, 450, Type.FISH)
  );

  public static void main(String[] args) {
    List<Dish> lowCaloricDishes = new ArrayList<>(); //'가비지 변수'
    for (Dish dish : menu) { //filtering
      if (dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
      }
    }
    Collections.sort(lowCaloricDishes, new Comparator<Dish>() { //익명 클래스로 요리 정렬
      @Override
      public int compare(Dish dish1, Dish dish2) {
        return Integer.compare(dish1.getCalories(), dish2.getCalories());
      }
    });
    List<String> lowCaloricDishesName = new ArrayList<>();
    for (Dish dish : lowCaloricDishes) {
      lowCaloricDishesName.add(dish.getName());
    }
    System.out.println(lowCaloricDishesName);
  }

}
```
위 코드에서는 lowCaloricDishes라는 '가비지 변수'를 사용했습니다. 즉, lowCaloricDishes는 컨테이너 역할만 하는 중간 변수입니다.  
Java8에서 이러한 세부 구현은 라이브러리 내에서 모두 처리합니다.  
```java
 List<String> lowCaloricDishesNameWithStream =
        menu.stream()
            .filter(d -> d.getCalories() < 400) //400칼로리 이하의 요리 선택
            .sorted(comparing(Dish::getCalories)) //칼로리로 요리 정렬
            .map(Dish::getName)
            .collect(Collectors.toList());
```
stream()을 parallelStream()으로 바꾸면 이 코드를 **멀티코어 아키텍처에서 병렬로** 실행할 수 있습니다.  
  
filter, sorted, map, collect 같은 여러 빌딩 블록 연산을 연결해서 복잡한 데이터 처리 파이프라인을 만들 수 있습니다.  
filter의 결과는 sorted 메서드로, sorted 결과는 map 메서드로, map 메서드의 결과는 collect로 연결됩니다.  
  
filter 같은 연산은 **고수준 빌딩 블록** 으로 이루어져 있으므로 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있습니다.  
또한 멀티코어 아키텍처를 최대한 투명하게 활용할 수 있게 구현되어 있습니다. 결과적으로 우리는 데이터 처리 과정을 병렬화하면서  
스레드와 락을 걱정할 필요가 없습니다.  
  
+ 선언형 : 더 간결하고 가독성이 좋아집니다.  
+ 조립할 수 있음 : 유연성이 좋아집니다.  
+ 병렬화 : 성능이 좋아집니다.  
  
## 4.2 스트림 시작하기 
Java8에는 스트림을 반환하는 stream 메서드가 추가됐습니다. `java.util.stream.Stream` 참고  
숫자 범위나 I/O 자원에서 스트림 요소를 만드는 등 stream 메서드 이외에도 다양한 방법으로 스트림을 얻을 수 있습니다.  
  
스트림이란 **데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소** 로 정의할 수 있습니다.  
+ 연속된 요소 : 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공합니다. 컬렉션의 주제는 데이터이고, 스트림의 주제는 계산입니다.  
+ 소스 : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비합니다. 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지됩니다.  
+ 데이터 처리 연산 : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 데이터베이스와 비슷한 연산을 지원합니다.  
  
또한 스트림에는 다음과 같은 두 가지 중요 특징이 있습니다.  
+ 파이프라이닝 : 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 **스트림 자신을 반환**합니다.  
+ 내부 반복 : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원합니다.  
  
```java
 List<String> threeHighCaloricDishNames =
       menu.stream() //메뉴(요리 리스트)에서 스트림을 얻는다.
           .filter(dish -> dish.getCalories() > 300) //파이프라인 연산 만들기. 첫 번째로 고칼로리 요리 필터링
           .map(Dish::getName) //요리명 추출
           .limit(3)//선착순 세 개만 선택
           .collect(toList()); //결과를 다른 리스트로 저장
    System.out.println(threeHighCaloricDishNames); // [pork, beef, chicken]
```
여기서 **데이터 소스**는 요리 리스트(메뉴)입니다. 데이터 소스는 **연속된 요소**를 스트림에 제공합니다.  
다음으로 스트림에 filter, map, limit, collect로 이어지는 일련의 데이터 처리 연산을 적용합니다.  
collect를 제외한 모든 연산은 서로 **파이프라인**을 형성할 수 있도록 스트림을 반환합니다.  
  
파이프라인은 소스에 적용하는 질의 같은 존재입니다.  
마지막으로 collect 연산으로 파이프라인을 처리해서 결과를 반환합니다.(collect는 스트림이 아니라 List를 반환합니다.)  
  
마지막에 collect를 호출하기 전까지는 menu에서 무엇도 선택되지 않으며 출력 결과도 없습니다.  
즉, collect가 호출되기 전까지 메서드 호출이 저장되는 효과가 있습니다.  
  
+ filter : `람다를 인수로 받아 스트림에서 특정 요소를 제외시킵니다`. 예제에서는 d -> d.getCalories() > 300이라는 람다를 전달해서 300칼로리 이상의 요리를 선택합니다.  
+ map : `람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출합니다.` 예제에서는 메서드 참조 Dish::getName, d -> d.getName() 을 전달해서 각각의 요리명을 추출합니다.  
+ limit : `정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소합니다`  
+ collect : `스트림을 다른 형식으로 변환`합니다.  
  
## 4.3 스트림과 컬렉션 
자바의 기존 컬렉션과 새로운 스트림 모두 **연속된** 요소 형식의 값을 지정하는 자료구조의 인터페이스를 제공합니다.  
여기서 '연속된' 이란 표현은 순서와 상관없이 아무 값에나 접속하는 것이 아니라 순차적으로 값에 접근한다는 것을 의미합니다.  
  
데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이입니다.  
컬렉션은 현재 자료구조가 포함하는 **모든** 값을 메모리에 저장하는 자료구조입니다. 즉, 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 합니다.  
(컬렉션에 요소를 추가하거나 컬렉션의 요소를 삭제하는 연산을 수행할 때마다 컬렉션의 모든 요소를 메모리에 저장해야 하며 컬렉션에 추가하려는 요소는 미리 계산되어야 합니다.)  
  
반면 스트림은 이론적으로 **요청할 때만 요소를 계산**하는 고정된 자료구조입니다. 스트림에 요소를 추가하거나 스트람에서 요소를 제거할 수 없습니다.  
사용자가 요청하는 값만 스트림에서 추출하는 것이 핵심입니다. 또한, 스트림은 게으르게 만들어지는 컬렉션과 같습니다.  
즉, 사용자가 데이터를 요청할 때만 값을 계산합니다.  
  
반면 컬렉션은 적극적으로 생성됩니다.(생산자 중심 : 팔기도 전에 창고를 가득 채움)  
소수 예제에 적용하면, 컬렉션은 끝이 없는 모든 소수를 포함하려 할 것이므로 무한 루프를 돌면서 새로운 소수를 계산하고 추가하기를 반복할 것입니다.  
결국 소비자는 영원히 결과를 볼 수 없게 됩니다.  
  
### 4.3.1 딱 한번만 탐색할 수 있다
반복자와 마찬가지로 스트림도 한 번만 탐색할 수 있습니다. 즉, 탐색된 스트림의 요소는 소비됩니다.  
반복자와 마찬가지로 한 번 탐색한 요소를 다시 탐색하려면 초기 데이터 소스에서 새로운 스트림을 만들어야 합니다.  
(그러려면 컬렉션처럼 반복 사용할 수 있는 데이터 소스여야 합니다. 만일 데이터 소스가 I/O 채널이라면 소스를 반복 사용할 수 없으므로 새로운 스트림을 만들 수 없습니다.)  
```java
 public static void main(String[] args) {
    List<String> title = Arrays.asList("Java8", "In", "Action");
    Stream<String> s = title.stream();
    s.forEach(System.out::println);
    try {
      s.forEach(System.out::println);
    } catch (Exception e) {
      e.printStackTrace(); // java.lang.IllegalStateException: stream has already been operated upon or closed
      //스트림이 이미 작동되었거나 닫혔습니다.
    }
  }
```
스트림은 단 한 번만 소비할 수 있다는 점을 명심하자!  
컬렉션과 스트림의 또 다른 차이점은 데이터 반복 처리 방법입니다.  
  
### 4.3.2 외부 반복과 내부 반복
컬렉션 인터페이스를 사용하면 사용자가 직접 요소를 반복해야 합니다.(예를 들면 for-each). 이를 외부 반복(external iteration)이라고 합니다.  
반면 스트림 라이브러리는 (반복을 알아서 처리하고 결과 스트림값을 어딘가에 저장해주는) 내부 반복(internal iteration)을 사용합니다.  
```java
public static void main(String[] args) {
  List<String> names = new ArrayList<>();
    for (Dish dish : menu) {
    names.add(dish.getName());
  }
}
```
for-each 구문은 반복자를 사용하는 불편함을 어느 정도 해결해줍니다. for-each를 이용하면 Iterator 객체를 이용하는 것보다 더 쉽게 컬렉션을 반복할 수 있습니다.  
```java
 List<String> names = new ArrayList<>();
    for (Dish dish : menu) {
      names.add(dish.getName());
    }
```
Iterator 객체를 이용하는 것보다 더 쉽게 컬렉션을 반복할 수 있습니다.  
```java
List<String> names = new ArrayList<>();
    Iterator<Dish> iterator = menu.iterator();
    while (iterator.hasNext()) {
      Dish dish = iterator.next();
      names.add(dish.getName());
    }
```
다음은 스트림을 이용한 내부 반복입니다.  
```java
List<String> names = menu.stream()
       .map(Dish::getName)//map 메서드를 getName 메서드로 파라미터화해서 요리명을 추출합니다.
       .collect(Collectors.toList());
```
컬렉션은 외부적으로 반복, 즉 **명시적으로 컬렉션 항목을 하나씩 가져와서 소비**합니다.  
내부 반복을 이용하면 작업을 투명하게 병렬로 처리하거나 더 최적화된 다양한 순서로 처리할 수 있습니다.  
  
스트림 라이브러리의 내부 반복은 `데이터 표현`과 `하드웨어를 활용한 병렬성 구현`을 자동으로 선택합니다.  
반면 for-each를 이용하는 외부 반복에서는 병렬성을 스스로 관리해야 합니다.(병렬성을 포기 or Synchronized 이용)  
  
스트림은 내부 반복을 사용하므로 반복 과정을 우리가 신경 쓰지 않아도 됩니다.  
하지만 이와 같은 이점을 누리려면 (filter나 map 같이) 반복을 숨겨주는 연산 리스트가 미리 정의되어 있어야 합니다.  
  
반복을 숨겨주는 대부분의 연산은 람다 표현식을 인수로 받으므로 3장에서 배운 동작 파라미터화를 활용할 수 있습니다.  
  
## 4.4 스트림 연산 
`java.util.stream.Stream` 인터페이스는 많은 연산을 정의합니다. 스트림 인터페이스의 연산을 크게 두 가지로 구분할 수 있습니다.  
```java
 List<String> threeHighCaloricDishNames =
    menu.stream() //요리 리스트에서 스트림 얻기
        .filter(dish -> dish.getCalories() > 300)  //중간 연산
        .map(Dish::getName) //중간 연산
        .limit(3)//중간 연산
        .collect(toList()); //스트림을 리스트로 변환 
```
위 예제에서 연산을 두 그룹으로 구분할 수 있습니다.  
- filter, map, limit은 서로 연결되어 파이프라인을 형성합니다.  
- collect로 파이프라인을 실행한 다음에 닫습니다.  
  
연결할 수 있는 스트림 연산을 `중간 연산` 이라고 하며, 스트림을 닫는 연산을 `최종 연산` 이라고 합니다.  

### 4.4.1 중간 연산
`filter`나 `sorted` 같은 중간 연산은 다른 스트림을 반환합니다. 따라서 여러 중간 연산을 연결해서 질의를 만들 수 있습니다.  
중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다는 것, 즉 게으르다는 것입니다.  
  
중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리하기 때문입니다.  
  
스트림 파이프라인에서 어떤 일이 일어나는지 쉽게 확인할 수 있도록 람다가 현재 처리 중인 요리를 출력해봅니다.  
```java
public static void main(String[] args) {
    List<String> names = menu.stream()
        .filter(dish -> {
          System.out.println("filtering = " + dish.getName());
          return dish.getCalories() > 300;
        })
        .map(dish -> {
          System.out.println("mapping:" + dish.getName());
          return dish.getName();
        })
        .limit(3)
        .collect(Collectors.toList());
    System.out.println(names);
  }
```
다음은 프로그램 실행 결과입니다.  
filtering = pork  
mapping:pork  
filtering = beef  
mapping:beef  
filtering = chicken  
mapping:chicken  
[pork, beef, chicken]  

300칼로리가 넘는 요리는 여러 개지만 오직 처음 3개만 선택되었습니다. 이는 limit 연산 그리고 **쇼트서킷**이라 불리는 기법 덕분입니다.  
둘째, filter와 map은 서로 다른 연산이지만 한 과정으로 병합되었습니다. (이 기법을 **루프 퓨전** 이라고 합니다.)  
### 4.4.2 최종 연산
최종 연산은 스트림 파이프라인에서 결과를 도출합니다.  
보통 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환됩니다. 예를 들어 다음 파이프라인에서 forEach는 소스에 각 요리에  
람다를 적용한 다음에 void를 반환하는 최종 연산입니다.  
```java
menu.stream().forEach(System.out::println);
```
System.out.println을 forEach에 넘겨주면 menu에서 만든 스트림의 모든 요리를 출력합니다.  

### 4.4.3 스트림 이용하기
스트림 이용 과정은 다음과 같이 세 가지로 요약할 수 있습니다.  
- 질의를 수행할 (컬렉션 같은) 데이터 소스  
- 스트림 파이프라인을 구성할 중간 연산 연결  
- 스트림 파이프라인을 실행하고 결과를 만들 최종 연산  
스트림 파이프라인의 개념은 빌더 패턴과 비슷합니다.  
  
중간 연산의 예 - filter, map, limit, sorted, distinct  
최종 연산의 예 - forEach, count, collect  
forEach - 스트림의 각 요소를 소비하면서 람다를 적용합니다.  
count - 스트림의 요소 개수를 반환합니다.  
collect - 스트림을 리듀스해서 리스트, 맵, 정수 형식의 컬렉션을 만듭니다.  

## 4.6 마치며 
+ 스트림은 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소입니다.  
+ 스트림은 내부 반복을 지원합니다.  
+ 스트림에는 중간 연산과 최종 연산이 있습니다.  
+ 중간 연산은 filter나 map처럼 `스트림을 반환`하면서 다른 연산과 연결되는 연산입니다.  
+ 중간 연산을 이용해서 파이프라인을 구성할 수 있지만 중간 연산으로는 어떤 결과도 생성할 수 없습니다.  
+ forEach나 count처럼 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환하는 연산을 최종 연산이라고 합니다.  
+ 스트림의 요소는 요청할 때 게으르게 계산됩니다.  


