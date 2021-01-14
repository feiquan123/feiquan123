Golang 实现简单的rsa加密算法

##### NOTE:  

- gcd 是求最大公约数
- Golang 中已经有`crypto/rsa`实现加密`EncryptPKCS1v15()`、解密`DecryptPKCS1v15()`，本文只是自己实现简单的ras 加密算法

##### rsa 算法

	1. 选择两个 p,q 素数，n = p * q
	2. 计算 Q = (p-1) * (q-1)
	3. 找出满足条件的e:
		1 < e < Q, gcd(e,Q) = 1
		方法:
			在 (1,Q) 中选择一个素数, 使 gcd(e,Q) = 1, 既使得 gcd(e,p-1) =1 && gcd(e,q-1) = 1
		推论:
			gcd(e,Q) = 1 	=> 	gcd(e,p-1) = 1 && gcd(e,q-1) = 1
			如:	gcd(4,7) = 1 && gcd(4,9) = 1 	=> 	gcd(4,36)
	4. 找出满足条件的d
		1< d < Q, ed % Q = 1 	=> 	1< Q/e < d < Q, ed % Q = 1
		方法:
			 => (ed - 1)  / Q = 0
	5. 公钥: (n,e) , 私钥: (n,d) . p,q,Q 的值不公开

##### 加密:

​	c = exp(m,e) % n

##### 解密:

​	m = exp(c,d) % n

##### 实现：

```go
package rsa

import (
	"fmt"
	"math/big"
	"strings"
	"time"

	"github.com/feiquan123/algorithm/prime"
)

var rsa *RSA

func init() {
	rsa = New(0)
}

// GCD 求两个数的最大公约数
func GCD(m, n uint64) uint64 {
	if m < n {
		m, n = n, m
	}

	if n == 0 {
		return m
	}

	return GCD(n, m%n)
}

var split = "\xff\xfe\xff"

// PublicKey public key
type PublicKey struct {
	N uint64
	E uint64
}

// NewPublicKey new public key
func NewPublicKey(n, e uint64) *PublicKey {
	return &PublicKey{n, e}
}

// String get public key
func (p PublicKey) String() string {
	return fmt.Sprintf("%d,%d", p.N, p.E)
}

// Check check pubilc key
func (p PublicKey) Check() error {
	if p.N < 6 {
		return fmt.Errorf("public kye  error, N too small")
	}
	if p.E < 2 {
		return fmt.Errorf("public kye error, E too small")
	}
	return nil
}

// PrivateKey private key
type PrivateKey struct {
	N uint64
	D uint64
}

// NewPrivateKey creates a new private key
func NewPrivateKey(n, d uint64) *PrivateKey {
	return &PrivateKey{n, d}
}

// String get private key
func (p PrivateKey) String() string {
	return fmt.Sprintf("%d,%d", p.N, p.D)
}

// Check check private key
func (p PrivateKey) Check() error {
	if p.N < 6 {
		return fmt.Errorf("public kye  error, N too small")
	}
	if p.D < 2 {
		return fmt.Errorf("public kye error, E too small")
	}
	return nil
}

// RSA msg encode and decode
type RSA struct {
	p uint64
	q uint64
	o uint64

	N uint64
	E uint64
	D uint64

	pub PublicKey
	pri PrivateKey
}

// New creates a new RSA
func New(N uint64) *RSA {
	n := time.Now().UnixNano()/1e6%(1000+int64(N)) + 13
	ps := prime.NPrimeEratosthenes(uint64(n))
	l := len(ps)
	if l == 0 {
		panic(fmt.Sprintf("l is 0, n=%d", n))
	}
	psm := make(map[uint64]struct{}, l)
	for _, v := range ps {
		psm[v] = struct{}{}
	}

	p, q := uint64(0), uint64(0)
	for k := range psm {
		if k < 5 {
			continue
		}
		p = k
	}
	for k := range psm {
		if k < 5 || k == p {
			continue
		}
		q = k
	}

	rsa := &RSA{
		p: p,
		q: q,
		o: (p - 1) * (q - 1),
		N: p * q,
	}

	tps := prime.NPrimeEratosthenes(rsa.o)
	tpsm := make(map[uint64]struct{}, len(tps))
	for _, t := range tps {
		tpsm[t] = struct{}{}
	}
	for t := range tpsm {
		if GCD(t, p-1) == 1 && GCD(t, q-1) == 1 {
			rsa.E = t
			break
		}
	}

	for i := rsa.o / rsa.E; i < rsa.o; i++ {
		if rsa.E*i%rsa.o == 1 {
			rsa.D = i
			break
		}
	}

	rsa.pub = PublicKey{N: rsa.N, E: rsa.E}
	rsa.pri = PrivateKey{N: rsa.N, D: rsa.D}

	return rsa
}

func (r RSA) PublicKey() string {
	return r.pub.String()
}

func (r RSA) PrivateKey() string {
	return r.pri.String()
}

// Encode encode bytes
func Encode(bs []byte, pub *PublicKey) (c []byte) { return rsa.Encode(bs, pub) }
func (r RSA) Encode(bs []byte, pub *PublicKey) (be []byte) {
	e, n := r.E, r.N
	if pub != nil && pub.Check() == nil {
		e = pub.E
		n = pub.N
	}
	bet := make([]string, 0)
	for _, b := range bs {
		m := new(big.Int).SetBytes([]byte{b})                                // source
		c := new(big.Int).Exp(m, big.NewInt(int64(e)), big.NewInt(int64(n))) // encode: m ** e % n
		bet = append(bet, string(c.Bytes()))
	}
	return []byte(strings.Join(bet, split))
}

// Decode decode bytes
func Decode(be []byte, pri *PrivateKey) (bs []byte, err error) { return rsa.Decode(be, pri) }
func (r RSA) Decode(be []byte, pri *PrivateKey) (bs []byte, err error) {
	d, n := r.D, r.N
	if pri != nil && pri.Check() == nil {
		d = pri.D
		n = pri.N
	}

	bs = make([]byte, 0)
	for _, b := range strings.Split(string(be), split) {
		c := new(big.Int).SetBytes([]byte(b))                                // encode
		m := new(big.Int).Exp(c, big.NewInt(int64(d)), big.NewInt(int64(n))) // decode: c ** d % n
		bs = append(bs, m.Bytes()[0])
	}

	return bs, nil
}
```

##### 安装

```sh
go install github.com/feiquan123/algorithm/example/rsa
```

##### 测试

1. 生成key

   ```
   rsa -gen
   ```

   ```txt
   public  key: 474731,172867
   private key: 474731,356659
   ```

   

2. 加密string

   ```sh
   echo "GOLANG" | rsa -pub 474731,172867
   ```

   ```txt
   �����[����#����������9����������
   ```

   

3. 解密string

   ```sh
   echo "GOLANG" | rsa -pub 474731,172867 | rsa -e=false -pri=474731,356659 
   ```

   ```
   GOLANG
   ```

   

4. 加密文件

   ```sh
   rsa -pub 474731,172867  -f go.mod
   ```

   ```txt
   ����������Ψ�������������X����R����:������M��������������U���x����/����������������������X�������4�������������ԧ���+-���y�����������������������:��������5�������M������������������������:��������R����+-���x����+-���Ŧ�������������5����X����4�����������5����X����R����:������M��������������U���x����/������������������zm�������������5����M���������zm���M����5����X����X����M����zm��������:��������/��������ԧ����X���X���������R�����X���+-���x���������x����Ŧ�����
   ```

   

5. 解密文件

   ```sh
   rsa -pub 474731,172867  -f go.mod | rsa -e=false -pri=474731,356659 
   ```

   ```txt
   module github.com/feiquan123/algorithm
   
   go 1.14
   
   require github.com/smartystreets/goconvey v1.6.4
   ```

   