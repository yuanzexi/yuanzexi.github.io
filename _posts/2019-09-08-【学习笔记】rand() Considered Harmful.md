# 【学习笔记】 rand() Considered Harmful

> 版权声明：欢迎各位转载，但是未经作者本人同意，转载文章之后必须在文章页面明显位置给出作者和原文连接，否则保留追究法律责任的权利。 {{site.url}}{{page.url}}

该笔记学习自 GoingNative 2013 的一个演讲视频 [rand() Considered Harmful, Aug 17, 2013 at 11:04AM  by Stephan T. Lavavej](https://channel9.msdn.com/Events/GoingNative/2013/rand-Considered-Harmful)

## rand() 的问题

```c++

int main() {
    srand(time(NULL));

    for (int i = 0; i < 16; ++i){
        printf("%d ", rand() % 100);
    }

    printf("\n");
}

```

- `srand(time(NULL))`
    - NULL is ABOMINATION.
    - `time(NULL)`: Frequency = 1 Hz，若同一秒内多次调用，产生的是同一个时间值。
    - `srand()`: 32-bit Seed.
- `rand() % 100`
    - `rand()`
        - Range is [0, 32767]
        - Linear congruential -> low quality.
    - `%`: modulo -> Non-uniform distribution
        ```c++
        int src = rand(); // Assume uniform [0, 32767]
        int dst = src % 100; // Non-uniform [0, 99]
        ```
        - Floating-Point Treachery
        ```c++
        int src = rand();
        int dst = static_cast<int>(
            (src * 1.0 / RAND_MAX) * 99
        ); // Hilariously non-uniform [0, 99]
        ```
        ```c++
        int src = rand();
        int dst = static_cast<int>(
            (src * 1.0 / (RAND_MAX + 1)) * 100
        ); // Subtly non-uniform [0, 99]
        ```
        - Same problem as `src % 100`: Nothing can uniformly map 32768 inputs to 100 outputs.


## 建议使用 <random.h> 

```c++
int main(){
    std::random_device rd;
    // std::mt19937 mt(2748);   // deterministic  32-bit seed.
    std::mt19937 mt(rd());      // non-deterministic 32-bit seed, different when running new program.
    std::uniform_int_distribution<int> dist(0, 99);  // Distribution: [0, 99],  Note:[inclusive, inclusive]
    for (int i = 0; i < 16; ++i){
        std::cout << dist(mt) << " ";  // Run engine, viewed through distribution
    }
    std::cout << std::endl;
}
```

- mt19937 
    - Fast(499 MB/s = 6.5 cycles/byte)
    - Extremely high quality, but not cryptographically secure
    - Seedable (with more than 32 bits if you want)
    - Reproducible (Standard-mandated algorithm)
- random_device
    - Possibly slow (1.93 MB/s = 1683 cycles/byte)
        - Strongly platform-dependent
    - Possibly crypto-secure (check documentation, true for VC)
    - Non-seedable
    - Non-reproducible
- random_shuffle() - Considered Harmful
    - May call rand()
    - Knuth shuffle needs r(n) to return [0, n), not inclusive
- shuffle() - Considered Awesome
    - Takes URNGs directly (e.g. mt19937)
    - Shuffles perfectly
    - Invokes the URNG in-place (can't copy)

## <random.h> 注意事项:

- Running mt19937 is fast, constructing/copying isn't
    - Constructing/copying engines often is already undesirable.
- URNG/distribution function call ops are non-const
    - Multiple threads cannot simultaneously call a single object. Better using lock or thread_local.
- When is it safe to skip uniform_int_distribution?
    - mt19937's [0, 2^32) or mt19937_64's [0, 2^64) -> [0, 2^N)
    - In this case, masking is safe, simple, and efficient
    - In all other cases, use uniform_int_distribution.