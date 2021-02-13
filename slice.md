### slice

#### golang中的array

资料参考地址：

- https://mp.weixin.qq.com/s/gUkYwCodYI0iAD5LMYXdMA      Go面试必考题目之slice篇
- https://mp.weixin.qq.com/s?__biz=MzI2Nzk1ODgwMw==&mid=2247483687&idx=1&sn=06443e8d4876074aa38e3d6531281675&chksm=eaf7ae5bdd80274d56942d8dac6f8a06f45f8b1edd24af7d00c59866ccf6a6ca9af1eee7aa68&scene=21#wechat_redirect      Go是否有引用？

 Go的数组array底层和C的数组一样，是==一段连续的内存空间==，通过下标访问数组中的元素。array只有长度len属性而且是==固定长度==的。

array的赋值是==值拷贝==的，看以下代码：

```go
type Array struct {
	len  int64
	elem Type
}
```



```go
func main() {
    c := [3]int{1, 2, 3}
    d := c
    c[0] = 999
    fmt.Println(d) // 输出[1, 2, 3]
}
```

#### slice底层

slice是一个特殊的引用类型，但他本身也是一个结构体。

属性==len==表示可用元素数量,读写操作不能超过这个限制,不然就会panic

属性==cap==表示最大扩张容量,当然这个扩张容量也不是无限的扩张,它是受到了底层数组array的长度限制,超出了底层array的长度就会panic

slice的数据结构：

 slice的底层结构由一个指向==数组的指针ptr和长度len，容量cap==构成，也就是说slice的数据存在数组当中。

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/DBP3HAicOpFVoTbBH0r3qBBvOScibjGRxI7gJEhjzzhSHh5mp6mqNWFLNgRxmj6TiaZkDtLk0UEplnfj4Jo6c7d8w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

slice的赋值是==指针赋值==，看以下代码：

```go
	a := []int{1,2,3,4,5}
	b := a
	b[0] = 123
	fmt.Println(a)     //输出[123 2 3 4 5]
	fmt.Println(b) 	   //输出[123 2 3 4 5]
```

**slice的重要知识点**

1. slice的底层是数组指针。
2. 当append后，slice长度不超过容量cap，新增的元素将直接加在数组中。
3.  当append后，slice长度超过容量cap，将会返回一个新的slice。

==**对于slice的底层是数组指针**==

```go
func main() {
    s := []int{1, 2, 3} // len=3, cap=3
    a := s
    s[0] = 888
    s = append(s, 4)

    fmt.Println(a, len(a), cap(a)) // 输出：[888 2 3] 3 3
    fmt.Println(s, len(s), cap(s)) // 输出：[888 2 3 4] 4 6
}
```

因为slice的底层是==数组指针==，所以slice `a`和`s`指向的是==同一个底层数组==，所以当修改s[0]时，a也会被修改。

但是 当s进行append时，因为长度len和容量cap是int值类型，所以不会影响到a。

==**对于slice长度不超过容量cap**==

```go
func main() {
    s := make([]int, 0, 4)
    s = append(s, 1, 2, 3)
    fmt.Println(s, len(s), cap(s)) // 输出：[1, 2, 3] 3 4
    s = append(s, 4)
    fmt.Println(s, len(s), cap(s)) // 输出：[1, 2, 3] 4 4
}
```

  当s进行append后，长度没有超过容量，所以底层数组的指向并没有发生变化，只是将值添加到数组中。

```go
	s := make([]int, 0, 4)
	a := s
	s = append(s, 1, 2, 3)
	a = append(a, 88)
	fmt.Println(s, len(s), cap(s)) // 输出：[88 2 3] 3 4
	fmt.Println(a, len(a), cap(a)) // 输出：[88] 1 4
	s = append(s, 4)
	fmt.Println(s, len(s), cap(s)) // 输出：[88 2 3 4] 4 4
```

a , s使用的是同一片连续的内存空间，他们的开头都是一样的，都是指向数组的开头指针。但是因为各自的len，cap不同，所以输出会有所不同。

但是==在相同的len中，他俩是完全一样的（必须强调**未扩容**）==！！！！

==**slice扩容**==

```go
func main() {
    s := []int{1, 2, 3}
    fmt.Println(s, len(s), cap(s)) // 输出：[1, 2, 3] 3 3
    a := s

    s = append(s, 4) // 超过了原来数组的容量
    s[0] = 999
    fmt.Println(s, len(s), cap(s)) // 输出：[999 2 3 4] 4 6  ！！！！此时，s和a指向的已经不是同一个数组了！！！！
    fmt.Println(a，len(s)，cap(s)) // 输出：[1, 2, 3] 3 3
}
```

  上面代码中，当对s进行append后，它的长度和容量都发生了变化，==最重要的是它的底层数组指针指向了一个新的数组==，然后将旧数组的值复制到了新的数组当中。

==**关于slice的扩容**==

append的时候发生扩容的动作

- append单个元素，或者append少量的多个元素，这里的少量指double之后的容量能容纳，这样就会走以下扩容流程，不足1024，双倍扩容，超过1024的，1.25倍扩容。
- 若是append多个元素，且double后的容量不能容纳，直接使用预估的容量。

**敲重点**！！！！此外，以上两个分支得到新容量后，均需要根据**slice的类型size**，算出新的容量所需的内存情况`capmem`，然后再进行`capmem`向上取整，得到新的所需内存，除上类型size，得到真正的最终容量,作为新的slice的容量。

```go
e := []int32{1,2,3}
fmt.Println("cap of e before:",cap(e))//输出cap of e before: 3
e = append(e,4)
fmt.Println("cap of e after:",cap(e))//输出cap of e after: 8

f := []int{1,2,3}
fmt.Println("cap of f before:",cap(f))//输出cap of f before: 3
f = append(f,4)
fmt.Println("cap of f after:",cap(f))//输出cap of f after: 6
```

#### golang函数传参

一个很重要的知识点是：**Go的函数传参，都是以值的形式传参。**而且**Go是没有引用**的，可以看下[这篇文章](http://mp.weixin.qq.com/s?__biz=MzI2Nzk1ODgwMw==&mid=2247483687&idx=1&sn=06443e8d4876074aa38e3d6531281675&chksm=eaf7ae5bdd80274d56942d8dac6f8a06f45f8b1edd24af7d00c59866ccf6a6ca9af1eee7aa68&scene=21#wechat_redirect)。

> 什么是引用？

```c++
#include <stdio.h>
int main() {        
    int a = 10;           
    int &b = a;        
    int &c = b;        
    printf("%p %p %p\n", &a, &b, &c);

    // 0x7ffe114f0b14 0x7ffe114f0b14 0x7ffe114f0b14

    return 0;
}
```

你可以给一个变量声明一个别名，这种就叫引用。

a，b，c的地址都是同一个，说明它们使用着同一块内存；如果对a进行修改，那么b，c的值也会被同时修改；

而且引用作为参数时，***在函数调用时在内存中不会生成副本\***。

==引用与指针的区别是==，指针通过某个指针变量指向一个对象后，对它所指向的变量间接操作。程序中使用指针，程序的可读性差；而引用本身就是目标变量的别名，对引用的操作就是对目标变量的操作。

> 在go语言中，所有的变量都有属于自己的地址

```go
package main
import "fmt"
func main() {        
    var a int
    var b, c = &a, &a
    fmt.Println(b, c)    // 0x1040a124 0x1040a124
    
    fmt.Println(&b, &c)    // 0x1040c108 0x1040c110
}
```

















