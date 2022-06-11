### Go
- 全局变量使用`:=`去初始化，跳出作用域变量资源会及时清理，所以在作用域外要使用该值，不能使用`:=`初始化

	```golang
	package main

	import "fmt"

	type Person struct {
		name string
	}

	var p Person

	func main() {
		Test()
		fmt.Println(p)
	}

	func Test() {
		p := Person{name: "hello world"}
		fmt.Println(p.name)
	}

	```
	输出结果:
	```shell
	hello world
	{}
	```
	如果要在main中使用全局变量p在Test中的初始化结果，可以将Test中的`p:=`改成`p=`,如下:
	```golang
	func Test() {
		p = Person{name: "hello world"}
		fmt.Println(p.name)
	}
	```
- 全局变量不能在函数外先定义，然后再初始化，但是可以定义的时候指定初始化值。如下代码:定义string类型变量name,然后初始化值为hello，结果报错

	```golang
	package main

	import "fmt"

	var name string
	name = "hello"

	func main() {

	}
	```

	```shell
	syntax error: non-declaration statement outside function body
	```

	但是可以声明的时候指定初始值:
	```golang
	package main

	var name = "hello"
	//或者
	//var name string = "hello"
	func main() {

	}
	```
- 只能在本包为本包的类型扩展方法，不能在其它包扩展方法，举个例子

	```golang

	package test

	import "fmt"

	type Person struct {
		Name string
	}

	func (p *Person) say() {
		fmt.Println(p.Name);
	}
	```
	```golang
	package hello

	import "fmt"

	func (p *Person) write() {
		fmt.Println(p.Name);
	}
	```
	上面的代码是错误的，Person类型属于test包，不能在hello包里面为Person类型添加方法。

- 下划线`_`的使用场景

    - 使用在import中，因为go中import一个包之后，如果不使用会编译失败，所以使用下划线来表示只是导入其他package，执行其它包中的init函数
    
    ```golang
        import (
            _ "net/http"
        )
    ```

    - 忽略或者抛弃不使用的变量，仅仅作为一个占位

    ```golang
    package main

    import "fmt"

    func main() {
        sum, err := Add(1, 2)
        fmt.Println(sum)
    }

    func Add(a, b int) (int, error) {
        return a + b, nil
    }
    ```
    上面的代码或报错，因为err变量没用使用，go语言是一个干净的语言，不允许有闲置不适用的变量存在，如果变量值没有使用，那就不要存在这个变量。为了解决这种问题，go中使用占位符`_`类抛弃不使用的变量，上面改成下面的代码就不报错了

     ```golang
    //sum, err := Add(1, 2)
    sum, _ := Add(1, 2)
    ```

    - 用在变量(特别是接口断言)

    看开源代码的使用看到过如下类似的代码:
    ```golang
    package main

    type Test interface {
        Result() string
    }

    type T struct {
        Name string
    }

    func (t *T) Result() string {
        return t.Name
    }

    func main() {
        var _ Test = &T{Name: "Json"}
    }

    ```

    这段代码`var _ Test = &T{Name: "Json"}`的意思是判断&T是否实现了Test接口,如果&T没有实现Test,则会报编译错误,`_`的作用也是个占位，类型断言，但是后续不使用


- golang中的三个点(...)

    - 根据元素实际数量识别数组的长度

    ```golang
    package main

    import "fmt"

    func main() {
        arr := [...]string{"java", "python", "php", "js"}
        fmt.Println(arr, len(arr))
    }

    ```

    - slice可以被打散进行传递

    ```golang
    package main

    import "fmt"

    func main() {
        arr := []int{1, 2, 3, 4, 5}
        var lst []int
        lst = make([]int, 0)
        lst = append(lst, arr...)
        fmt.Println(lst)
    }

    ```

    - 函数接收不定长参数的情况

    ```golang
    package main

    import "fmt"

    func main() {
        arr := []int{1, 2, 3, 4, 5}
        fmt.Println(Sum(arr...))
        fmt.Println(Sum(1, 2, 3, 4, 5, 6))
    }

    func Sum(args ...int) int {
        sum := 0
        for _, value := range args {
            sum += value
        }
        return sum
    }

    ```