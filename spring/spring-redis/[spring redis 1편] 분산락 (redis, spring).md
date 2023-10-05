> 목차는 [spring redis 목차](https://insanelysimple.tistory.com/category/Spring/redis) 에 있습니다.



# [spring redis 1편] 분산락 (redis, spring, lettuce)





# RedisLockRegistry

- spring integration 에서 제공해주는 Redis 분산락 입니다.
- spin lock, pub-sub lock 옵션을 제공하며, pub-sub 이 성능상 이유로 선호 됩니다.
  - spin lock 은 주기적으로 lock 을 획득하며, pub-sub 은 pub-sub 구조로 lock 을 획득하기에 성능상 유리합니다.





# 어떻게 동작하는지?

- LockRegistry 는 UUID 와 lock 의 map 을 메모리에 가지고 있습니다. lockRegistry 는 일종의 lock 저장소입니다.
- lockRegistry 에서 lock 을 획득하려고 할 때, map 에 lock key 가 존재하면 lock 을 return 합니다.
- 만약 없다면 redis 와 통신하여 lock 을 획득할려고 시도합니다. (Registry key:lock name)
  - lock 이 존재하면 return 하고 없으면 새로 만듭니다.

- lock 을 획득했다면 lock.tryLock() 을 호출하며, 기본적으로 lockRegistry.obtain 과 유사한 스텝을 가집니다. 처음에는 map 의 locks 에 lock 이 있는지 체크하고, 없으면 Redis 에서 체크합니다.






# Reference
- https://medium.com/@egorponomarev/distributed-lock-with-redis-and-spring-boot-2c3f51a44c65