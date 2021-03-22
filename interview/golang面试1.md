### 问答部分

---

1. 描述go语言的优点，为什么使用go
    1. go 是一门静态的强类型、支持高并发、自带GC的语言，

    2. go 内置完整的工具链,构建、编译、格式化代码等。

    3. 高性能

    4. 简洁

       

2. go语言的select是随机的还是顺序的？为什么？
   	随机的
      	select 如果没有default 会阻塞, case 会监听从通道中返回数据，那个先返回那个分支就先执行，如果同时返回，随机选一个分支执行。如果有default, 在其他 case 没有返回阻塞时，执行 default 分支

   

3. go语言的局部变量分配在栈还是堆？
   	不一定， go 会做逃逸分析，看这个局部变量是否逃除了当前函数的作用域，如果逃出了放在堆上，没有的话在栈上

   


4. 描述go语言的make(T) 和 new(T) 的区别
	make 用来创建 slice,map,chan 等返回值是某个类型的引用，都可以在创建时指定容量
	new 用来创建某个类型的实例,返回值是这个类型的指针，初始化是此类型的零值
	
	


5. go语言如何保护共享变量？
	1. 加 Mutex 锁
	2. 加 RWtex 锁
	3. 如果是map 类型可以使用,使用 sync.map 
	4. 如果是单个值可以使用CSP



6. 描述go语言的gc原理
	标记清除
	1. gc 在某个时间点上触发时会导致 STW ,其他 M 停止工作
	2. 标记: 给停止的 M 分配标记任务，然后唤醒这些 M 开始执行标记任务，全部标记完成后，M 再次休眠
	3. 清除: 起一个单独的协程清理之前标记的内存块
	4. 重启，唤醒所用的 M，开始正常工作




7. 描述go语言的协程调度模型和原理
	M: 内核线程 	P: 用户态的调度器  G: 用户态协程
	1. 维护全局的G队列，全局 M 空闲队列
	2. 一个 M 绑定一个 P, 每个P维护自己的 G 队列
	3. 创建：当一个 M 创建 G 时, 如果本地 P 中的 G 队列未满直接添加; 如果满了加入到全局 G 队列等待调度
	4. 执行：M 从本地 P 中的 G 队列直接取一个 G 
		存在：开始执行
			未阻塞：取下一个G，不断循环
			阻塞： M 与本地 P 解绑，然后这个 P 去查看有没有空的 M ，如果有直接绑定；如果没有从全局 M 空闲队列中取一个。 
			     当这个 M 执行完 G 后，去查看有没有空闲的 P 可以绑定，如果可以就绑定执行，如果不可以，直接进 全局的 M 空闲队列
		不存在：
			从其他 P 中偷一半放在本地 P 执行； 如果偷不到，从全局 G 中偷，然后放在本地 P 队列； 如果全局中也没有，直接放入全局 M 空闲队列




8. 描述go语言的runtime机制
	
	1. 启动时传入环境变量: 如 GOMAXPROCS等
	
	2. 创建主 M0, 主 G0 , 然后绑定 M0 和 GO
	
	3. G0 设定每个 goroutine 能申请的最大值，如果有某个 goroutine 大于这个值会发生栈溢出，发生 painc
	
	4. G0 启动系统检测任务，方便调度器查漏补缺
	
	5. 检测当前 M 是不是 M0， 如果不是异常
	
	6. 创建特殊的 defer 语句
	
	7. 创建 GC 协程，设置 GC 可用标识
	
	8. 调用 init 
	
	9. runtime.main 调用 main.mian
	
	10. 执行 main 中的语句
	
	11. 执行之前创建的特殊的 defer
	
	   


9. 怎么调试golang的bug，跟踪性能问题
	1. 编写对应的`bench`函数测试
	2. 开启 `GODEBUG='gctrace=1'`
	3. 使用 pprof 工具
	4. 生成 火焰图
5. linux 常用命令： top、/etc/prco、strace 等
	
	



### 编程部分

---



1. 下面这段代码输出什么，说明原因

   ```go
   func main() {
 slice := []int{0, 1, 2, 3}
    m := make(map[int]*int)
   
    for key, val := range slice {
     m[key] = &val
    }
   
    for k, v := range m {
 fmt.Println(k, "->", *v)
 }
	}
	```
	
	答：

	```
	0->3
	1->3
	2->3
	3->3
	
	range 遍历时发生的是值拷贝，使用的是同一个变量
	```
	
	

2. 下面这段代码输出什么，说明原因

  ```go
  func main() {
   a := []int{7, 8, 9}
   fmt.Printf("%+v\n", a)
   ap(a)
   fmt.Printf("%+v\n", a)
   app(a)
   fmt.Printf("%+v\n", a)
  }
  
  func ap(a []int) {
      a = append(a, 10)
  }
  
  func app(a []int) {
      a[0] = 1
  }
  ```

  答：

  ```
  7,8,9
  7,8,9  // slice 扩容，新开辟内存地址
  1,8,9  // slice 是数组的引用
  ```

  




3. 下面这段代码输出什么（选择)，说明原因

  ```go
  type Math struct {
      x, y int
     }
  
  var m = map[string]Math{
      "foo": Math{2, 3},
  }
  
  func main() {
      m["foo"].x = 4
      fmt.Println(m["foo"].x)
  }
  
  // A. 4
  // B. compilation error
  ```

  答：

  ```
  B // map 中存放的是Math 的值类型，不能直接修改其中的字段。应该直接为 m["foo"] 赋一个修改后的值
  ```

  ​	




4. 下面这段代码输出什么，说明原因

  ```go
  type Ttt struct {
      Daemon string
     }
  
  func (c *Ttt) String() string {
      return fmt.Sprintf("print: %v", c)
  }
  
  func main() {
      c := &Ttt{}
      c.String()
  }
  ```

  答:

  ```
  没有输出, 没有 fmt.Println
  
  异常，c.String() 会调用自身，栈超过 1G[64位]、 250M[32位]后, 栈溢出
  ```



5. 下面的代码有什么问题？如何改好

   ```go
func main() {
    wg := sync.WaitGroup{}
   
    for i := 0; i < 5; i++ {
        go func(wg sync.WaitGroup, i int) {
            wg.Add(1)
            fmt.Printf("i:%d\n", i)
            wg.Done()
     }(wg, i)
    }
   
   ```

 wg.Wait()
 fmt.Println("exit")
	}
```
	
	答：
	
```
	wg.Add(1) 来不及设置，G0 就执行完了	
	
	1. wg.Add(1) 应该移动到for 循环外部,并修改为 wg.Add(5)
	2. wg 应该共用同一个wg,只传入 i 就可以
	```




6. 下面这段代码输出什么，说明调用过程

  ```go
  func fibonacci(n int, c chan int) {
     x, y := 0, 1
     for i := 0; i < n; i++ {
     	c <- x
     	x, y = y, x+y
     }
     close(c)
     }
  
  func main() {
  	c := make(chan int, 5)
  	go fibonacci(cap(c), c)
  	for i := range c {
  		fmt.Println(i)
  	}
  }
  ```

  答：

  ```
  0
  1
  1
  2
  3
  
  1. goroutine 调用 fibonacci(5,c)
  2. 在 main 的 for 循环中阻塞等待 c 中数据的输入
  3. goroutine 中发送数据[1,2,3,4,5]，并执行计算，发送完数据后关闭通道
  4. main 中的 for 打印计算的结果
  ```

  


7. 下面这段代码输出什么，说明调用过程

  ```go
  func fibonacci(c, quit chan int) {
     x, y := 0, 1
     for {
     	select {
     	case c <- x:
     		x, y = y, x+y
     		
  
     	case d :=<-quit:
     		fmt.Printf("--%v--\n", d)
     		fmt.Println(0, " quit")
     		return
     	}
  
     }
     }
  
  func main() {
  	c := make(chan int)
  	quit := make(chan int, 2)
  	go func() {
  		for i := 0; i < 5; i++ {
  			fmt.Println(<-c)
  		}
  		quit <- 3
  		println("111")
  		quit <- 1
  		println("222")
  	}()
  	fibonacci(c, quit)
  }
  ```

  答：

  ```go
  0
  1
  1
  2
  3
  --3--
  0 quite
  
  1. 启动一个 goroutine, 内部阻塞在  <-c ，此时 c 没有数据输入
  2. 执行 fibonacci(c, quit)
  3. fibonacci 中执行 select 第一个分支，给 c 中一次写入 [ 0 1 1 2 3 ]
  4. goroutine 中依次读出 c 中的值[ 0 1 1 2 3 ]
  5. 给 quit 输入 3
  6. fibonacci 中执行 select 第二个分支, 打印完后，执行return，退出 fibonacci 函数
  7. 退出 main 函数
  ```

  

机面：

Golang工程师上机题

 

1.写一个函数处理文件路径：
输入任何一个路径字符串，把它转换成不含.和..的绝对文件路径，对不合法的路径/错误的路径给出提示
    比如：c:\1\2\..\3\.\1.txt 输出c:\1\3\1.txt
    比如：c:\1\3\1.txt 输出c:\1\3\1.txt

2.把一个文本文件中的内容做如下转换后（假定文件大小不大于10M字节），生成另一个新的文件
    把文件中的所有*号提前，放在文件的最前面，其它内容不变，保存为一个新文件
    比如：hello *wor*ld* 处理后新文件的内容为:***hello world
    
要求：
1.实现上面2个功能
2.注意程序执行效率、可维护性（易维护/易读懂/
3.代码风格好，注释合理到位，完成时间不得超过2小时
4.思路清晰易懂，时间和空间开销尽量最优化
5.使用go语言实现



终面：

1. 10人10个座位的组合
2. 10个箱子如何放苹果实现组合出1000内的任意一个数
3. 投篮90%，投20次全中的概率
4. 3天后兔子成熟，然后成熟的兔子每年下一只兔子，50年后多少只