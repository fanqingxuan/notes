### Go
# 误用短声明导致变量覆盖

```go
var remember bool = false
if something {
    remember := true //错误
}
// 使用remember
```

在此代码段中，`remember`变量永远不会在`if`语句外面变成`true`，如果`something`为`true`，由于使用了短声明`:=`，`if`语句内部的新变量`remember`将覆盖外面的`remember`变量，并且该变量的值为`true`，但是在`if`语句外面，变量`remember`的值变成了`false`，所以正确的写法应该是：

```go
if something {
    remember = true
}
```

此类错误也容易在`for`循环中出现，尤其当函数返回一个具名变量时难于察觉，例如以下的代码段：

```go
func shadow() (err error) {
	x, err := check1() // x是新创建变量，err是被赋值
	if err != nil {
		return // 正确返回err
	}
	if y, err := check2(x); err != nil { // y和if语句中err被创建
		return // if语句中的err覆盖外面的err，所以错误的返回nil！
	} else {
		fmt.Println(y)
	}
	return
}
```


# 何时使用 new() 和 make()
    - 切片、映射和通道，使用make
    - 数组、结构体和所有的值类型，使用new 
	
# 不需要将一个指向切片的指针传递给函数

切片实际是一个指向潜在数组的指针。我们常常需要把切片作为一个参数传递给函数是因为：实际就是传递一个指向变量的指针，在函数内可以改变这个变量，而不是传递数据的拷贝。

因此应该这样做：
```go
func findBiggest( listOfNumbers []int ) int {}
```
而不是：
```go
func findBiggest( listOfNumbers *[]int ) int {}
```


## 不要使用布尔值检测错误：

像下面代码一样，创建一个布尔型变量用于测试错误条件是多余的：

```go
var good bool
    // 测试一个错误，`good`被赋为`true`或者`false`
    if !good {
        return errors.New("things aren’t good")
    }
```

正确方式如下,立即检测一个错误：

```go
err1 := api.Func1()
if err1 != nil { … }
```

# defer 会推迟资源的释放，所以尽量不要在 for 中使用
```go
for {
    time.Sleep(time.Second)
    // ......
    conn, err := grpc.Dial(address, grpc.WithInsecure())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
        //...
}
```
由于这是一个死循环，defer代码不会被执行到，所以申请的内存得不到释放，然后会导致程序占满整个内存，死机