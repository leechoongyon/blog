## DTO 정리
- Data Transfer Object = 데이터를 전송해주는 객체
- 계층 간 데이터를 전송해주는 객채입니다.
  - service, controller 간 데이터를 전송해줄수 있고, 통신할 때도 dto 를 사용할 수 있습니다.

## DTO 장점

#### 비용 절감
- client 와 server 간 통신은 시간이 오래 걸리는 작업입니다.
- 이 때, DTO 없이 데이터를 보낼려고 하면 여러 번의 통신이 발생할 수 있습니다.
  - 이는 resource 낭비이며, 성능이 안좋습니다.
- dto 를 통해 여러 데이터를 한꺼번에 보낼 수 있기에 좋습니다.

#### 직렬화에 대한 책임
- 통신을 할 떄, 데이터를 직렬화 해야 합니다.
  - 직렬화란 object to byte[] 를 의미합니다.
- 이 떄, dto 가 없다면 직렬화하기 위해서 여러 계층에 직렬화 코드가 있을 수 밖에 없습니다.
- dto 에 직렬화 코드가 있다면 이를 통해 유지 보수가 가능할 것입니다.
  - 단일책임원칙의 (SOLID 의 S) 예라고 할 수 있을 것 같습니다.

## VO 란? (Value Object)
- 도메인 객체의 일종이고, 한 개 이상의 속성들을 묶어서 특정 도메인을 가리키고자 사용합니다.
  - ex) x,y,z 좌표를 하나의 클래스로 만들어서 사용하는 것이 VO 예시입니다.
- 이렇게 VO 를 사용함으로써 다른 entity 나 클래스에 많은 변수를 매번 등록할 필요없이 VO 한 번만 사용하면 되니 편리합니다. 

## VO 특징

#### 불변성 (setter 가 없습니다) 입니다.
- 불변성에 따른 장점은 값을 공유해도 thread-safe 합니다.

#### 객체 간 동등성 
- 객체 간 동등성을 검사할 때 속성을 검사합니다.
- 동등성을 검사하기 위해 equals, hashCode 를 재정의해줘야합니다. 현실세계를 기준으로 2개 객체를 비교하기 위해선 속성 값을 비교해서 같은지를 비교해야 합니다.
- 아래 예제를 보면 == 를 통해 비교하면 주소 값을 비교하게 됩니다.
- 현실 세계에서 아래 두개의 객체는 달라야 됩니다. 그렇기 위해 equals, hashCode 를 구현해야합니다.
  - equals 는 값을 비교하도록 override 를 해야합니다.
  - hashCode 도 재정의해줘야 하는데 아래와 같이 구현해주면 객체 간의 비교가 가능해집니다.

```java


public class Coordinates {
    
    private String x;
    private String y;
    private String z;
    
    public Coordinates(String x, String y, String z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }
}



  public static void main(String[] args) {
    Coordinates coordinates = new Coordinates("1","2", "3");    
    Coordinates coordinates2 = new Coordinates("3","2", "1");

    System.out.println(coordinates == coordinates2);
  }

```

```java

    @Override
    public boolean equals(final Object o) {
        
        // 값 비교 로직
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y, z);
    }
```



## reference
- https://kafcamus.tistory.com/13
- https://tecoble.techcourse.co.kr/post/2020-06-11-value-object/