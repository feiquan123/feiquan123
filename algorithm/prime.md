golang 查找素数

1. ##### 素数的定义

   - 素数又称为质数，指在大于1的自然数中，除了1和它本身以外不再有其他因数的自然数
   - 2是最小的质数

2. ##### 算法
	
	1. 暴力枚举
	2. 开方	因数都是成对出现,
		比如，100的因数有：1和100、2和50、4和25、5和20、10和10
		即成对的因数，其中一个必然小于等于100的开平方，另一个大于等于100的开平方
		因此只要判断2~sqrt(n)的因数即可
	3. 埃拉托斯特尼(Eratosthenes)
		1. 创建长度为 (n -2 + 1) 的数组, 初始值都为 true
		2. 遍历[2-n] , 每次将 i 的倍数的下标设置为 false
			- 倍数筛选 [j=2, i*j<=n, j++]
			- 二次筛选 [j=i*i, j<=n, j+=i]
			- 线性筛选
	3. 最后遍历数组中为 true 的元素就是素数
	
3. ##### 素数定理
	
	素数的分布是越往后越稀疏。或者说，素数的密度是越来越低。而素数定理，
	说白了就是数学家找到了一些公式，用来估计某个范围内的素数，大概有几个。
	在这些公式中，最简洁的就是 x/ln(x).假设要估计1,000,000以内有多少质数，
	用该公式算出是72,382个，而实际有78,498个，误差约8个百分点。
	该公式的特点是：估算的范围越大，偏差率越小。
	一般用x/ln x来估计某个范围内素数的个数(误差小于15%)
	
4. ##### 实现

```go
package prime

import (
	"math"
)

// IsPrime judge whether the given number is prime
type IsPrime func(n uint64) bool

// IsPrimeByRangeEnum judge whether the given number is prime
func IsPrimeByRangeEnum(n uint64) bool {
	if n < 2 {
		return false
	}
	for i := uint64(2); i < n; i++ {
		if n%i == 0 {
			return false
		}
	}
	return true
}

// IsPrimeBySqrtRangeEnum judge whether the given number is prime
func IsPrimeBySqrtRangeEnum(n uint64) bool {
	if n < 2 {
		return false
	}
	for i, l := uint64(2), uint64(math.Sqrt(float64(n))); i <= l; i++ {
		if n%i == 0 {
			return false
		}
	}
	return true
}

// NPrime 获取一个[2-n] 内的素数
func NPrime(n uint64, isPrime IsPrime) (ps []uint64) {
	ps = make([]uint64, 0)
	if n < 2 {
		return
	}

	for i := uint64(2); i <= n; i++ {
		if isPrime(i) {
			ps = append(ps, i)
		}
	}

	return
}

// NPrimeEratosthenes 埃拉托斯特尼, 最优  + 倍数筛选
func NPrimeEratosthenes(n uint64) (ps []uint64) {
	ps = make([]uint64, 0)
	if n < 2 {
		return ps
	}

	N := make([]bool, n+1) // false: 素数, true: 不是素数
	for i, l := uint64(2), uint64(math.Sqrt(float64(n))); i <= l; i++ {
		if N[i] == false {
			for j := uint64(2); i*j <= n; j++ {
				N[i*j] = true // 倍数筛选: 同一元素会重复删除
			}
		}
	}

	for i, l := uint64(2), n+1; i < l; i++ {
		if N[i] == false {
			ps = append(ps, i)
		}
	}
	return ps
}

// NPrimeEratosthenes2 埃拉托斯特尼 + 二次筛选法
func NPrimeEratosthenes2(n uint64) (ps []uint64) {
	ps = make([]uint64, 0)
	if n < 2 {
		return ps
	}

	N := make([]bool, n+1) // false: 素数, true: 不是素数
	for i, l := uint64(2), uint64(math.Sqrt(float64(n))); i <= l; i++ {
		if N[i] == false {
			for j := i * i; j <= n; j += i {
				N[j] = true // 二次筛选法: 假设每次i是素数，则下一个起点是 i*i ,把后面 [i*i, i*(i+1), i*(i+2), ..., n] 全部移除
			}
		}
	}

	for i, l := uint64(2), n+1; i < l; i++ {
		if N[i] == false {
			ps = append(ps, i)
		}
	}
	return ps
}

// NPrimeEratosthenesLine 埃拉托斯特尼 + 线性筛选
func NPrimeEratosthenesLine(n uint64) (ps []uint64) {
	ps = make([]uint64, 0)
	if n < 2 {
		return ps
	}

	N := make([]bool, n+1)  // false: 素数, true: 不是素数
	P := make([]uint64, n+1) // 保存素数
	count := uint64(0)       // 素数的个数

	for i := uint64(2); i <= n; i++ {
		if N[i] == false {
			P[count] = i
			count++
		}
		for j := uint64(0); j < count && P[j]*i <= n; j++ {
			N[P[j]*i] = true
			a := make([]uint64, n+1)
			for i, v := range N {
				if v {
					a[i] = 1
				}
			}
			if i%P[j] == 0 { // 保证每个合数只会被它的最小质因数筛去，因此每个数只会被标记一次，所以时间复杂度是O(n)
				break
			}
		}
	}

	for i, l := uint64(2), n+1; i < l; i++ {
		if N[i] == false {
			ps = append(ps, i)
		}
	}
	return ps
}

```

```go
// 基准测试
var N = uint64(100000)

func BenchmarkNprimeIsPrime(b *testing.B) {
	for i := 0; i < b.N; i++ {
		NPrime(N, IsPrimeByRangeEnum)
	}
}

func BenchmarkNprimeIsPrimeBySqrtRangeEnum(b *testing.B) {
	for i := 0; i < b.N; i++ {
		NPrime(N, IsPrimeBySqrtRangeEnum)
	}
}

func BenchmarkNPrimeEratosthenes(b *testing.B) {
	for i := 0; i < b.N; i++ {
		NPrimeEratosthenes(N)
	}
}

func BenchmarkNPrimeEratosthenes2(b *testing.B) {
	for i := 0; i < b.N; i++ {
		NPrimeEratosthenes2(N)
	}
}

func BenchmarkNPrimeEratosthenesLine(b *testing.B) {
	for i := 0; i < b.N; i++ {
		NPrimeEratosthenesLine(N)
	}
}
```

测试结果:

```go
goos: darwin
goarch: amd64
pkg: github.com/feiquan123/algorithm/prime
// 暴力计算 n*n
BenchmarkNprimeIsPrime-8                               1        3182596148 ns/op
// 暴力计算 n*sqrt(n)
BenchmarkNprimeIsPrimeBySqrtRangeEnum-8               55          20153547 ns/op
// 埃拉托斯特尼 - 倍数筛选
BenchmarkNPrimeEratosthenes-8                       3261            334561 ns/op
// 埃拉托斯特尼 - 二次筛选
BenchmarkNPrimeEratosthenes2-8                      3382            338191 ns/op
// 埃拉托斯特尼 - 线性筛选
BenchmarkNPrimeEratosthenesLine-8                      1        23504143006 ns/op
PASS
ok      github.com/feiquan123/algorithm/prime   30.136s
```