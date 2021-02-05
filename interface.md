## interface

#### 声明interface

java：class  &&  interface

golang：struct  &&  interface

Java的**class**/golang的**struct**：对模型的定义和封装

Java的**interface**/golang的**interface**：对行为的抽象描述和封装

```go
type Birds interface {           //声明，一个叫Birds的接口
    Twitter() string		//行为Twitter && Fly
    Fly(high int) bool		//行为（函数方法）包括：函数名，参数，返回值
}
```

#### 实现interface

在java中实现接口：通过类来实现，同时通过implement来说明实现的接口

在==golang中实现接口==：通过==struct==来实现，但是不进行声明，而是默默==实现所有接口中的方法==，然后就认为你实现了这个接口。

> 如果它走起步来像鸭子,并且叫声像鸭子, 那个它一定是一只鸭子

![image-20210205162237096](C:\Users\Cristiano-Ronaldo\AppData\Roaming\Typora\typora-user-images\image-20210205162237096.png)

> 上面这段代码，声明了一个叫做==Sparrow==的struct，同时声明了两个方法。

> ```go
> func (s *Sparrow) Fly(hign int) bool        //(s *Sparrow)
> //(s *Sparrow)这个声明，在golang中称为接受者声明。其中s代表这个方法的接收者，*Sparrow代表这个接收者的类型。
> //接收者的类型可以为一个数据类型的指针类型，也可以是数据类型本身，比如我们针对Sparrow再实现一个方法：
> func (s Sparrow) Walk() {
>     // ...
> }
> ```

> 接收者为数据类型的方法称为==值方法==，接收者为指针类型的方法称之为==指针方法==

#### 使用interface

> 利用struct去实现接口之后，我们就可以==用这个struct作为接口参数==，==**使用那些接收接口参数的方法完成我们的功能**==。这也是面向接口编程的方式，我们的功能依据接口来实现，而不用关心实现接口的是什么，这样大大提供了功能的通用性可扩展性。

```go
func BirdAnimation(bird Birds, high int) {
    fmt.Printf("BirdAnimation of %T\n", bird)
    bird.Twitter()
    bird.Fly(high)
}

func main() {
    var bird Birds
    sparrow := &Sparrow{}
    bird = sparrow
    BirdAnimation(bird, 1000)
    // 或者将sparrow直接作为参数
    BirdAnimation(sparrow, 1000)
}
```

**上面这段代码中，我们声明了一个`Birds`接口类型的变量`bird`，由于`*Sparrow`实现了`Birds`接口的所有方法，所以我们可以将`*Sparrow`类型的变量`sparrow` 赋值给`bird`。或者直接将`sparrow`作为参数调用`BirdAnimation`，运行结果如下：**

#### 关于空interface

**interface何时才为nil？**

- eface：当_type和data都为nil的时候。
- iface：当itab和data都为nil的时候。

> 简要来说，一个interface结构包含两部分：1.这个接口值的类型；2.指向这个接口值的指针。（详情见底层interface）

```go
type Chicken interface {     //接口Chicken
	Birds              //实现继承效果
	Walk()  bool
}

func NilInterfaceTest(chicken Chicken) {         
    if chicken == nil {
        fmt.Println("Sorry,It’s Nil")
    } else {
        fmt.Println("Animation Start!")
        fmt.Printf("type:%v,value:%v\n", reflect.TypeOf(chicken), reflect.ValueOf(chicken))
        ChickenAnimation(chicken)
    }
}
//输出
Animation Start!
ChickenAnimation of *main.Sparrow
panic: value method main.Sparrow.Walk called using nil *Sparrow pointer

func main() {
  var sparrow3 *Sparrow         //这里的sparrow3是一个nil
    NilInterfaceTest(sparrow3)	//但是当sparrow3传到(chicken Chicken)中的时候，reflect.TypeOf(chicken) ！= nil,
    												//reflect.ValueOf(chicken)) == nil
}

```

#### 关于方法列表

![image-20210205165100400](C:\Users\Cristiano-Ronaldo\AppData\Roaming\Typora\typora-user-images\image-20210205165100400.png)

> 关于方法列表的概念：
>
> **首先**，一个指针类型的方法列表必然包含所有接收者为指针接收者的方法，同理非指针类型的方法列表也包含所有接收者为非指针类型的方法。在我们例子中==*Sparrow==首先包含：`Fly(high int)`和`Twitter()`；==Sparrow==包含`Walk()`。
>
> 
>
> **其次**，当我们拥有一个指针类型的时候，因为有了这个变量的地址，我们得到这个具体的变量，所以**==一个指针类型的方法列表还可以包含其非指针类型作为接收者的方法==**。
>
> 1
>
> **但是**一个非指针类型却并不总是能取到它的地址，从而获取它接收者为指针接收者的方法。所以**==非指针类型的方法列表中只有接收者为非指针类型的方法==**。



#### 标准库中的interface

```go
package sort
// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package.  The methods require that the
// elements of the collection be enumerated by an integer index.
type Interface interface {
  // Len is the number of elements in the collection.
  Len() int

  // Less reports whether the element with
  // index i should sort before the element with index j.
  Less(i, j int) bool

  // Swap swaps the elements with indexes i and j.
  Swap(i, j int)
}
//在这里，将int，string...均实现上面interface中的行为，那么均可以使用下面的Sort(data Interface)
...
// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {           
  // Switch to heapsort if depth of 2*ceil(lg(n+1)) is reached.
  n := data.Len()
  maxDepth := 0
  for i := n; i > 0; i >>= 1 {
    maxDepth++
  }
  maxDepth *= 2
  quickSort(data, 0, n, maxDepth)
}
```

**在设计函数时的一个准则**

> Be ***conservative\*** in what you send, be ***liberal\*** in what you accept.   — Robustness Principle
>
> 发送要简单点，接收要自由

**对应与Golang就是**

> Return ***concrete types\***, receive ***interfaces\*** as parameter.  — Robustness Principle applied to Go
>
> 返回具体，接收抽象

**这样带来的问题**

> interface 底层实现的时候会动态的检测。这样也会引入一些问题:
>
> 1. 性能下降。使用 interface 作为函数参数，runtime 的时候会动态的确定行为。而使用 struct 作为参数，编译期间就可以确定了。
> 2. 不知道 struct 实现哪些 interface。这个问题可以使用 guru 工具来解决。

#### 底层interface

Go的interface是由两种类型来实现的：`iface`和`eface`。

其中，==iface表示的是包含方法的interface==，例如：

```go
type Person interface {
	Speak()
}
```

而==eface表示的是不包含方法的interface==，即

```go
type Person interface {}
```

**eface具体结构**

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/DBP3HAicOpFVcSI6EE8kFHdLL9YhmqFwicv4Q0iaoR9KOrmia5ByQwJGia4iaZIf34tEVUM1uDIePmSDqW31Ve6HFtyg/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="图片" style="zoom:80%;" />

`eface`一共由两个属性构成，一个是类型信息`_type`，一个是数据信息`data`。

`_type`是描述数据类型的，Go语言中几乎所有的数据结构都可以抽象成`_type`，是所有类型的表现，可以说是万能类型。

`data`是指向具体==数据的指针==。

**iface底层分析**

iface的源代码是：

```go
type iface struct {
	tab  *itab
	data unsafe.Pointer
}
```

**iface的具体结构是：**

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/DBP3HAicOpFVcSI6EE8kFHdLL9YhmqFwic0FPbSlRc05W7k4pCugPrMgCLQXCrp1smeMB8zsATSj9wC7j8GamqyQ/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="图片" style="zoom:80%;" />

==itab==是iface不同于eface比较关键的数据结构。其可包含两部分：一部分是==确定唯一的包含方法的interface的具体结构类型==，一部分是==指向具体方法集的指针==。

**itab具体结构为：**

<img src="https://mmbiz.qpic.cn/mmbiz_png/DBP3HAicOpFVcSI6EE8kFHdLL9YhmqFwicd7C4FcLZEFK3QM6CayOuEgjqDP7UURI7HFJ9cSI3LjUbDsXh8GgxyA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="图片" style="zoom:57%;" />

属性itab的源代码是：

```go
type itab struct {
	inter *interfacetype //此属性用于定位到具体interface
	_type *_type //此属性用于定位到具体interface
	hash  uint32 // copy of _type.hash. Used for type switches.
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}
```

































