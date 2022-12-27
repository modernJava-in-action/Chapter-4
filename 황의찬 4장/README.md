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

  
