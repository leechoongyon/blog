## spring batch JpaCursorItemReader

## 결론
- PagingItemReader 보다 CursorItemReader 가 더 빠른 이유는 PagingItemReader 는 Paging 을 읽을 때마다 connection 이 발생해서 데이터를 가져옴.
- 즉, reader 에서 페이지를 읽을 때마다 connection 관련 추가 작업이 들어가기에 CursorItemReader 보다 느리다.
- CursorItemReader 의 경우 streaming 방식으로 계속 connection 을 열어두기에 따로 connection 관련 리소스 감소가 없음.
- 단, 조심해야할 건 streaming 방식으로 인해 connection 을 계속 열어둬야 하니 query timeout 을 조심해야 함.
- 쿼리 타임아웃 같이 찾아보기.

## Reference
- https://jojoldu.tistory.com/551?category=902551