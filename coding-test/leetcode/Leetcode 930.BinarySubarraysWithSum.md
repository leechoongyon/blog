





# 풀이 (brute-force)

- for loop 2번 돌아서 sum 이 goal 과 일치할 경우 output 을 계산해줬습니다.
- 시간 복잡도 : O(n²) , 공간 복잡도 : 1

```kotlin
    fun numSubarraysWithSum1(nums: IntArray, goal: Int): Int {
        var output = 0
        var sum = 0
        for (i in 0 until nums.size) {
            sum += nums[i]
            if (sum == goal) {
                output++
            }

            for (j in i + 1 until nums.size) {
                sum += nums[j]
                if (sum == goal) {
                    output++
                }
            }
            sum = 0
        }
        return output
    }
```





# 풀이 (sliding window)