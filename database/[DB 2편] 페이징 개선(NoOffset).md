> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 2편] 페이징 개선(NoOffset)




## 기존 페이지 방식
- 기본적으로 offset 과 limit 을 기반으로 페이징 쿼리를 조회합니다.
- 아래 예시와 같은 페이징이 느린 이유는 offset 부분을 계산하는 부분이 들어가기 때문입니다.
  - offset 을 구하기 위해서는 앞에서부터 읽어나가야하는데 마지막으로 갈수록 앞에 읽어야하는부분이 많기에 읽는 속도가 점점 느려집니다.

```sql

select * from test
where 
    xxx
order by seq
offset 0 limit 10

```

## 페이지 방식 개선 (No Offset)
- 조회 시작부분을 인덱스로 빠르게 찾아 offset 계산을 생략하는 것입니다.
- 이렇게 하기 위해서는 페이지 번호가 없고, 더보기, 더 찾아보기와 같은 방식을 이용하는 것입니다.
  - 기존 페이징 방식은 1,2,3, ... 10 이런식으로 페이지 번호가 나타난다면 더보기, 더 찾아보기 방식은 해당 버튼을 누를 시 다음 데이터를 더 가져오는 것입니다.
  - 대부분의 사람들은 첫페이지만 읽으며, 뒤에 페이지는 읽지 않기에 이 방법이 많이 사용됩니다.

```sql

select * from test
where
    xxx
    and seq < 마지막 조회 seq 
order by seq desc
limit 10

```

#### 동작 흐름
- 기존에는 웹페이지에서 pageNo, pageSize 만 주면 서버에서 offset, limit 을 조회해서 데이터를 응답해줬습니다.
- NoOffSet 을 사용하기 위해선 client 에서 몇 번째 부터 검색하면 되는지 정보를 주면 됩니다.
- client 에서 100번째부터 읽어서 10개의 데이터를 달라고하는 요건이 있다고 가정하겠습니다.
  - client 에서 100번째라는 단서와 10개의 데이터라는 정보를 서버로 보내주면 서버에서는 100번째에 대한 조건을 걸고 limit 10 을 걸어 데이터를 가져오게 됩니다.
  - 이렇게 할 경우 offset 을 계산할 필요가 없기에 더욱 빠르게 조회가 가능합니다.

## NoOffset 한계
- 더보기를 사용할 수 없을 경우 NoOffset 은 이용할 수 없습니다.
  - 왜냐하면 더보기 (more) 기능은 다음페이지 이동만 가능하지만, 번호가 붙은 페이지 기능은 사용하기 어렵습니다.
  - 첫번째 페이지를 읽다가 끝페이지를 읽어버리면 어디서부터 시작해야할지 계산의 복잡성이 따르기 때문입니다. 물론 화면 또는 client 에서 다음에 읽어야 할 데이터 seq 를 서버로 넘겨줄수도 있으나 복잡할 것입니다. 
- offset 을 계산하지 않기 위해 조건문을 사용한다고 했는데요. 이 조건문에 사용되는 조건이 group by 에도 사용될 경우 문제가 생길 수 있습니다.
  - 아래와 같이 group by seq 했는데 seq 조건문에 따라 결과 값이 바뀔수 있기에 사용할 수 없습니다.

```sql

select * from test
where
  xxx
  and seq < 마지막 조회 seq
group by seq
ordeer by seq desc
limit 10

```


## reference
- https://jojoldu.tistory.com/528