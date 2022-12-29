# 스트림이란?

스트림은 자바 8 API에 새로 추가된 기능이다. 스트림을 이용하면 선언형으로 컬렉션 데이터를 처리할 수 있다. 여기서 말하는 선언형이란:
  - **선언형** : 데이터를 처리하는 임시 구현 코드 대신 질의로 표현할 수 있다

또 스트림을 사용하면 멀티스레드 코드를 구현하지 않고 병렬로 처리할 수 있다.

우선 요리를 칼로리를 기준으로 정렬하는 코드를 살펴보자.

우선 아래는 **Dish 클래스** 코드이다:
```java
  public class Dish {

  private final String name;
  private final boolean vegetarian;
  private final int calories;
  private final Type type;

  public Dish(String name, boolean vegetarian, int calories, Type type) {
    this.name = name;
    this.vegetarian = vegetarian;
    this.calories = calories;
    this.type = type;
  }

  public String getName() {
    return name;
  }

  public boolean isVegetarian() {
    return vegetarian;
  }

  public int getCalories() {
    return calories;
  }

  public Type getType() {
    return type;
  }

  public enum Type {
    MEAT,
    FISH,
    OTHER
  }

  @Override
  public String toString() {
    return name;
  }

  public static final List<Dish> menu = Arrays.asList(
      new Dish("pork", false, 800, Dish.Type.MEAT),
      new Dish("beef", false, 700, Dish.Type.MEAT),
      new Dish("chicken", false, 400, Dish.Type.MEAT),
      new Dish("french fries", true, 530, Dish.Type.OTHER),
      new Dish("rice", true, 350, Dish.Type.OTHER),
      new Dish("season fruit", true, 120, Dish.Type.OTHER),
      new Dish("pizza", true, 550, Dish.Type.OTHER),
      new Dish("prawns", false, 400, Dish.Type.FISH),
      new Dish("salmon", false, 450, Dish.Type.FISH)
  );
}
```

**기존 코드 (자바 7)** :
```java
  //JAVA 7 Code
  List<Dish> lowCaloricDishes = new ArrayList<>();
  for (Dish dish : menu) {
    if (dish.getCalories() < 400) {
      lowCaloricDishes.add(dish);
    }
  }
  Collections.sort(lowCaloricDishes, new Comparator<Dish>() {   //익명 클래스로 요리 정렬
    @Override
    public int compare(Dish o1, Dish o2) {
      return Integer.compare(o1.getCalories(), o2.getCalories());
    }
  });
  List<String> lowCaloricDishesName = new ArrayList<>();
  for (Dish dish : lowCaloricDishes) {
    lowCaloricDishesName.add(dish.getName());   //정렬된 리스트를 처리하면서 요리 이름 선택
  }
```

**최신 코드 (자바 8)** :
```java
  List<String> lowCaloricDishesName =
  menu.stream()
    .filter(d -> d.getCalories() < 400)     //400 칼로리 이하의 요리 추출
    .sorted(comparing(Dish::getCalories))   //칼로리 순으로 요리 정렬
    .map(Dish::getName)                     //요리명 추출
    .collect(toList());                     //모든 요리명을 리스트에 저장
```

위 코드를 병령로 실행하려면 **stream()** 을 **parallelStream()** 로 변경해주기만 하면 된다:
```java
  List<String> lowCaloricDishesNameParallelSort =
  menu.parallelStream()
    .filter(d -> d.getCalories() < 400)     //400 칼로리 이하의 요리 추출
    .sorted(comparing(Dish::getCalories))   //칼로리 순으로 요리 정렬
    .map(Dish::getName)                     //요리명 추출
    .collect(toList());                     //모든 요리명을 리스트에 저장
```

위처럼 스트림이라는 새로운 기능을 사용하면 다음의 다양한 이득을 얻을 수 있다:
  - 선언형으로 코드를 구현할 수 있다.
  - *filter, sorted, map, collect* 같은 빌딩 블록 연산들을 연결해서 데이터 처리 파이프라인을 만들 수 있다 (유연성이 좋아진다).
  - 기존 자바 7 코드에 비해서 간결하고 가독성이 좋아진다.
  - 쉬운 병렬화가 가능하다 (성능이 좋아진다).

**스트림이란 정화히 뭘까?**
  - **연속된 요소** : 스트림은 연속된 값 집함의 인터페이스를 제공한다. 켤렉션은 자료구조이므로 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다. 반면 스트림은 *filter, sorted, map* 처럼 표현 계싼식이 주를 이룬다. 한마디로 컬렉션의 주제는 데이터고 스트림의 주제는 계산이다. 
  - **소스** : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다.
  - **데이터 처리 연산** : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원한다. 스트림 연산은 순차적으로 또는 병렬로 실행할 수 있다.
  - **파이프라이닝** : 스트림 연산은 스트림 연산끼리 연결해서 파이프라인을 구상할 수 있도록 자신을 반환한다.
  - **내부 반복** : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원한다.

아래는 위 특징을 이용한 스트림 파이프라인이다:
```java
  //파이프라인 연산 만들기
  List<String> threeHighCaloricDishNames =
    menu.stream()                               //메뉴에서 스트림을 얻는다
      .filter(dish -> dish.getCalories() > 300) //고칼로리 요리 필터링
      .map(Dish::getName)                       //요리명 추출
      .limit(3)                                 //선착순 세 개만 선택
      .collect(toList());                       //리스트로 반환
```

## 스트림과 컬렉션

컬렉션과 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다. 여기서 **연속된 (sequenced)** 이라는 표현은 순차적으로 값에 전근한다는 것을 의미한다. 

**스트림과 컬렉션의 차이**
데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이다. 컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장한다. 한마디로 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다. 

하지만 스트림은 요청할 때만 요소를 계산하는 고정된 자료구조이다 (스트림에 요소를 추가하거나 제거할 수 없다). 반복자와 마찬가지로 스트림도 한 번만 탐색할 수 있다 (탐색된 스트림의 요소는 소비된다). 반복자와 마찬가지로 한 번 탐색한 요소를 다시 탐색하려면 초기 데이터 소스에서 새로운 스트림을 생성해야 한다.

#### 외부 반복과 내부 반복

컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복해야 한다, 이를 **외부 반복 (external iteration)** 이라고 한다. 반면 스트림 라이브러리는 반복을 알아서 처리하고 스트림값을 어딘가에 저장해주는 **내부 반복 (internal iteration)** 을 사용한다.

외부 반복은 명시적으로 컬렉션 항목을 하나씩 가져와서 처리한다. 반면 내부 반복을 이용하면 작업을 투명하게 병렬로 처리하거나 더 최적화된 다양한 순서로 처리할 수 있다. 그리고 스트림 라이브러리의 내부 반복은 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다 (물론 이 이점을 누리려면 filter나 map같은 반복을 숨겨주는 연산 리스트가 미리 정의되어 있어야 한다).