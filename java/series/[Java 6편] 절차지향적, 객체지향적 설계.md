> 목차는 [Java series](https://insanelysimple.tistory.com/category/Java/series) 에 있습니다.



> 유튜브 보고 공부한 내용을 정리했습니다.



# [Java 6편] 절차지향적, 객체지향적 설계

- 도메인을 설계하는 방법인 절차지향적, 객체지향적에 대해 공부해서 정리했습니다.



# 절차지향적 설계

- 데이터와 프로세스가 별도로 분리돼있는 설계 방식입니다.
- 아래 예시와 같이 데이터와 처리가 분리돼있는 방식이며, 비즈니스 flow 가 잘 나와있습니다.

```java

public void reserve(xxxxx) {
  
  1) Showing (상영) 데이터 조회
    
  2) Movie (영화) 데이터 조회
    
  3) Discount(할인) 데이터 조회
    
  4) Rule 데이터 조회
    
  5) 상영 데이터를 가지고 가격 계산 + 할인 계산 해서 예매를 한다. (행위)
  
}

```





# 객체지향적 설계
- 데이터 + 행위에 대한 것을 객체로 묶어서 처리하며, 객체 간의 협력(메시지) 를 통해 비즈니스 요구사항을 처리하는 방식입니다. 즉, 책임을 위임하고 분산시키는 설계 방법입니다.
- 특정 기능을 어떤 객체에 할당해야하는지에 대해서 기준이 되는점은 많은 데이터를 보유하고 있는 객체가 어떤 것인지 고민하는 것입니다.



### 설계하면서 생각난 고민

- 영화를 예매하는 기능의 (reserve) 처음 시작점이 왜 Showing 인가?
  - 객체에 어떤 코드를 둘지 고민할 때, 기준이 되는 것은 해당 책임에 대한 데이터를 어떤 객체에서 많이 다루고 있는지를 고민해보는 것입니다.
  - 영화를 예매하기 위해 가장 많은 데이터를 가지고 있는 것은 Showing (상영) 이기에 Showing 이 시작점이 됩니다.
- Reservation 정보를 만들기 위해 Fee 를 계산해야하는데 왜 Showing 이 시작점이 되는가? Movie 가 가장 많은 정보를 가지고 있으니 Movie.calculateFee 가 되야 하지 않는가?
  - Reservation --> Showing --> Movie --> Discount --> Rule 이와 같은 방향으로 흐르고 있습니다.
  - 만약 요금을 계산할 때, Movie.calculateFee 를 호출해버리면 
    - Reservation --> Movie --> Showing 객체 흐름이 되고, 이는 사이클을 만듭니다. (Movie 에서 요금계산을 위해 Showing 이 필요합니다.)
    - 사이클이 생성되면 소스가 변경될 때, 서로 영향을 받기에 유지보수성에 좋지 않습니다.



```java
public class Showing { 
	public Reservation reserve(xxxxxx) {
    return Reservation.create(this);
  }
  
  public Fee calculateFee() {
    	return movie.calculateFee(this);
  }
}

public clsss Moving {
  public Fee calculateFee() {
    	return xxxxx;
  }
}

@Builder
@AllArgsConstructor
public class Reservation {
  
  private Showing showing;
  private Fee fee;
  
  public static Reservation create(Showing showing) {
		return Reservation.builder()
      .showing(showing)
      .fee(showing.calculate(xxx))
      .build();
  } 
}


```



# 객체지향 vs 절차지향

- 소스 추적은 객체지향이 편리합니다. 왜냐하면 특정 객체에 하나의 책임을 할당하기에 찾고자 하는 기능에 맞는 객체를 찾아가면 되기 때문입니다.
- 기존 코드를 수정할 때도 객체지향이 편리합니다. 수정하고자 하는 기능이 무엇인지 파악하고 해당 객체를 찾아가서 수정하면 되기 때문입니다. 또한, 객체지향으로 구현하였으면, 상속, interface 등을 구현하여 기존 코드를 건드리지 않고 수정이 가능합니다.
- 절차지향은 소스 코드의 flow 를 보는데 좋습니다. 객체지향은 flow 를 볼려면 관련 객체를 모두 탐색해야 합니다.





# Service Layer

- 어플리케이션 로직 이며, 경계를 의미합니다.
- 기술적인 (Transactional) 것을 사용할 수 있습니다.
- 도메인 로직을 재사용할 수 있습니다.
- 도메인 로직을 처리하기 위한 준비를 하는 Layer 입니다.




# Reference
- https://www.youtube.com/watch?v=26S4VFUWlJM