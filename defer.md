## defer

#### defer基础

> defer是什么？

defer 就像它的字面意思一样，就是延迟执行的意思，但是需要注意的是 defer ==只能作用于函数==，像变量赋值defer i = 10这种编译是会报错的。

> defer的执行顺序

被 defer 的函数会放入一个栈中，所以是先进后出的执行顺序，而被 defer 的函数在 return 之后执行。

> 用defer来清理资源

**假如不用defer**，只用close()手动关闭

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)            //加入在这里出现错误！！！！
    if err != nil {							//会导致之前打开的src没有被关闭，从而泄露	
        //当然可以在这里加一行代码进行解决，但是会很臃肿
        //src.close()
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

**当利用defer的时候**,在函数return之后，defer便开始了他的行动。

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

#### defer与return的关系

==return不是原子操作==，被拆成了两步

```go
rval = xxx // 返回值赋值给rval
ret // 函数返回
```

==defer在两句之间进行==

```go
rval = xxx // 返回值赋值给rval
defer_func  // 执行defer函数
ret // 函数返回
```

> 闭包是什么？

 简单来说，Go 语言中的闭包就是==在函数内引用函数体之外的数据==，这样就会产生一种结果，虽然==数据定义是在函数外==，但是在==函数内部操作数据也会对数据产生影响==。

```go
func foo() {
    i := 1
    func() {
        i++
    }
    fmt.Println(i) // 输出2    上面的 i 就是一个闭包引用，当匿名函数执行时，i 也会被修改。
}
```

#### 关于defer的一些题目

```go
func f1() {
    for i := 0; i < 5; i++ {
        defer fmt.Println(i)
    }
}
// 因为defer的调用是先进后出的顺序
// 所以输出：4, 3, 2, 1, 0


func f2() {
    for i := 0; i < 5; i++ {
    	defer func() {
    	    fmt.Println(i)
    	}()
    }
}
// 上面说到，i是一个闭包引用
// 所以当执行defer时，i已经是5了
// 所以输出：5，5，5，5，5


func f3() {
    for i := 0; i < 5; i++ {
    	defer func(n int) {
    	    fmt.Println(n)
    	}(i)
    }
}
// Go的函数参数是值拷贝，所以这是普通的函数传值
// 所以输出：4，3，2，1，0


func f4() int {
    t := 5
    defer func() {
    	t++
    }()
    return t
}
// 注意：f4函数的返回值是没有声明变量的
// 所以t虽然是闭包引用，但返回值rval不是闭包引用
// 可以拆解为
// rval = t
// t++
// return rval
// 所以输出是5


func f5() (r int) {
    defer func() {
        r++
    }()
    return 0
}
// 注意：f5函数的返回值是有声明变量的
// 所以返回值r是闭包引用
// 可以拆解为
// r = 0
// rval = r
// r++
// return rval
// 所以输出：1


func f6() (r int) {
    t := 5
    defer func() {
    	t = t + 5
    }()
    return t
}
// 这里t虽然是闭包引用，但返回值r不是闭包引用
// 可以拆解为
// r = t
// rval = r
// t = t + 5
// return rval
// 所以输出：5


func f7() (r int) {
    defer func(r int) {
	r = r + 5
    }(r)
    return 1
}
// 因为匿名函数的参数也是r，所以相当于是
// 匿名函数的参数r = r + 5，不影响外部
// 所以输出：1
```





































