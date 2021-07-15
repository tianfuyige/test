# test
一、基本结构
package main
 
import (
    "fmt"
)
 
func main() {
    fmt.Printf("hello world!!")
}
1、一个完整的go文件包含包名、import包（可选） 、业务代码（可选）。

2、每个项目必须包含main包，且main包中，必须有一个文件中实现入口main()方法。

3、同一个目录下只能包含一个包。即同一目录下的文件包名必须相同。

二、++i、与swich的优化简直大快人心
1、去掉了++i，再也不用纠结 ++i和i++了。

2、switch 命中case后，不再需要手动break。

func main() {
    var i int
    i++
    switch i {
    case 1:
        fmt.Print("step 1")
    case 2:
        fmt.Print("step 2")
    default: /* 可选 */
        fmt.Print("step 3")
    }
    fmt.Printf("i:%d", i)
}
三、只有一种循环的方法
for i := 0; i < 10; i++ {
        fmt.Printf("i:%d\n", i)
    }
没有while、do while，纠结用什么样方式循环还不如多写两行代码。

如何迭代数组？  range 关键字了解下？

四、Go的面向对象
package main
 
import "fmt"
 
type Base struct {
    Platform string
}
 
type Payment struct {
    Type int
    Base //内嵌实现结构的组合，达到了继承的目的
}
 
func (self *Payment) Pay() {
    self.Platform = "alipay"
    fmt.Print("pay...", self.Platform)
}
func main() {
    pay := Payment{}
    pay.Pay()
}
1、通过struct定义结构，而不是类。go语言发明人认为结构要优先于算法，而且go是c++的精简，所以用struct代替class关键字，看起来更c语言。

2、类方法的定义过程其实是结构与函数的“绑定”过程。其他语言的类方法定义看起来很面向对象，其实是内部隐藏了内部的绑定细节。

3、go语言通过方法、成员名称的首字母大小写实现了封装（小写隐藏，大写暴露），所以省略掉了关键字private、public。

4、没有继承，只有组合。

五、Go的接口
package main
 
import "fmt"
 
type Payment interface {
    Pay()
}
 
type AliPay struct {
    Type int
}
 
func (self *AliPay) Pay() { //仅仅实现这个Pay方法，并没有看到任何implement关键字
    fmt.Print("alipay...")
}
func main() {
    var pay Payment
    pay = &AliPay{}
    pay.Pay()
}
1、go中的类型是鸭子类型：看起来是鸭子，叫起来是鸭子，那就是鸭子。即：任何类型只要实现了某个接口A（interface）中的方法，那么这个类型就可以看做是接口A类型。这样一来又省略了关键字implement。

2、进一步理解是：go中的interface只管实现，无需声明。

六、异常捕获
package main
import "fmt"
func OccuerError() {
    defer func() {
        err := recover() //捕获异常，类似php的 catch
        if err != nil {
            fmt.Print(err)
        }
    }()
    panic("test panic") // 抛异常，类似 php的throw
}
func main() {
    OccuerError()
}
1、通过recover、defer实现异常的捕获。

2、defer在go的方法中类似压栈。先被defer的函数，将在函数返回前最先被调用。这里其实就隐藏了一个潜规则，go的方法返回前（正常返回或者异常返回）是允许写代码的人手动通过defer“塞”自定义函数的。

七、数据类型三剑客：CMS
Chan ：绝对安全的并发通道

var  ch chan  int 

ch = make(chan int,10)

ch  <-   10  //通过   <- 右侧运算符往chan中写数据

i :=   <-ch   //通过左侧的  <-  从通道读数据    

创建一个有10缓冲区的channel 。



Map：程序员的万能数据结构

var hm map[string]string
hm = make(map[string]string)
hm["name"] = "frank"
fmt.Println(hm)



Slice：成精后的数组

var ary []string
ary = make([]string, 0)
ary = append(ary, "frank")

fmt.Println(ary)



以上三种内置数据类型在使用前均需要make一下进行初始化。

八、好算法不如好结构之：struct
1、
import (
    "encoding/json"
    "fmt"
)
func main() {
    type Packet struct {
        From        string      `json:"from" validate:"required,max=32"`         //来源地区域代号
        Destination string      `json:"destination" validate:"required,max=32"`  //目的地区域代号
        ServiceName string      `json:"service_name" validate:"required,max=64"` //待处理服务
        ApiUrl      string      `json:"api_url" validate:"required,max=128"`     //待处理服务接口地址
        Data        interface{} `json:"data" validate:"required"`                //原始数据
    }
    packet := Packet{}
    jsonStr, _ := json.Marshal(packet)
    fmt.Print(string(jsonStr))
}
输出：{"from":"","destination":"","service_name":"","api_url":"","data":null}
 
2、
import (
    "encoding/json"
    "fmt"
)
func main() {
    type Packet struct {
        From        string      `json:"from" validate:"required,max=32"`         //来源地区域代号
        Destination string      `json:"destination" validate:"required,max=32"`  //目的地区域代号
        ServiceName string      `json:"service_name" validate:"required,max=64"` //待处理服务
        ApiUrl      string      `json:"api_url" validate:"required,max=128"`     //待处理服务接口地址
        Data        interface{} `json:"data" validate:"required"`                //原始数据
    }
    jsonStr := "{\"from\":\"\",\"destination\":\"\",\"service_name\":\"data-sync-service\",\"api_url\":\"\",\"data\":null}"
    packet := Packet{}
    json.Unmarshal([]byte(jsonStr), &packet)
    fmt.Printf("%+v", packet)
}
 
输出：{From: Destination: ServiceName:data-sync-service ApiUrl: Data:<nil>}
1、其中通过反引号 “  ·  ” 为字段打tag，让struct的字段与json关联，包括mongo、mysql都可以通过这种方式。

九、万能的类型：interface{}与类型断言
package main
import("fmt")
func main() {
    var mix interface{}
    mix = 100
    a, _ := mix.(int)
    a = a + 10
    fmt.Printf("%d", a)
}
1、interface{} 类似C中的void，可以接收任意数据类型。

2、实际使用时，仍然需要知道原来的数据类型。必须通过类型断言实现： 变量.(类型)。

十、可以多通道监听的select
package main
 
import (
    "fmt"
)
 
func main() {
    var ch1 chan int
    var ch2 chan int
    ch1 = make(chan int, 1)
    ch2 = make(chan int, 1)
    ch1 <- 1
    ch2 <- 6
    for {
        select {
        case i := <-ch1:
            fmt.Println("ch1:", i)
        case i := <-ch2:
            fmt.Println("ch2:", i)
        default:
        }
    }
}
1、通过select实现多通道读取，当有数据同时到达时，go内部是随机读取。

十一、Go中实现并发编程
package main
 
import (
    "fmt"
    "time"
)
 
func producer(ch chan int) {
    for i := 0; i <= 1000; i++ {
        ch <- i
    }
}
 
func consumer(ch chan int) {
    for {
        select {
        case i := <-ch:
            fmt.Printf("got i :%d\n", i)
        default:
            time.Sleep(time.Millisecond * time.Duration(1))
        }
    }
}
 
func main() {
    var ch chan int
    ch = make(chan int, 10000)
    go producer(ch)
    time.Sleep(time.Millisecond * time.Duration(1000))
    for i := 0; i < 5; i++ { //创建10个消费者
        go consumer(ch)
    }
    select {} //阻塞当前主进程
}
1、对比python或者java中针对并发编程时需要创建线程、绑定线程函数、创建同步队列，这里仅仅通过go、chan、select三个关键字就实现了生产者、消费者模式的多线程模型。

十二、包的引用
一、使用gopath的方式

引入一个第三方包之前，都需要先将包下到本地。可以手动按照包名放到gopath目录。如：

gopath目录是：I:\workspace\golib ，需要引用的包是github.com/aliyun/aliyun-oss-go-sdk/oss，则将包从github上下载下来后保存进 I:\workspace\golib\src\github.com\aliyun\aliyun-oss-go-sdk\oss 目录。或者最简单的  go get  github.com/aliyun/aliyun-oss-go-sdk/oss  。

然后代码里  import("github.com/aliyun/aliyun-oss-go-sdk/oss")

二、使用go module的方式：

直接在代码里 import("github.com/aliyun/aliyun-oss-go-sdk/oss") ，go build时将自动为你下载并保存。 

如何开启go module ？版本必须大于 go 1.11，然后通过命令：  go env -w GO111MODULE=on

十三、包中的init()方法
package main
 
import (
    "fmt"
)
 
func main() {
    fmt.Println("step 02")
}
 
func init() {
    fmt.Println("step 01")
}
输出：
step 01
step 02
1、在包被导入时，init方法将默认被最先执行。

2、所以通过init方法可以实现包内的一些资源初始化。

十四、关于基本语法的最后总结
1、整体风格是类C的，所以有大括号形成语句块，可以有分号(一行多个语句时，用来分隔)，也可以省略。

2、由于小括号()是方法调用，所以if 语句后没有小括号。毕竟if()这样看起来是调用了一个叫if的函数。

3、go中有指针，但是指针不能运算。因为go的定位是系统语言，还没底层到向C那样，需要对某个明确的内存地址进行写操作。

4、函数可以返回多个值，这样一来，你就不需要返回一个结构了。

5、变量声明是从左向右，名称在前，类型在后。看一个变量的类型，按照最近原则。变量的类型可以提前声明，如:var i int； 也可以使用内置的推导： i := 0; 这种定义都是合法的。

6、import与php的include类似，import的是路径。不同的是，go的import引入的是整个目录。import 引入的包可以取别名。如： import  log  "github.com/cihub/seelog"

7、变量的作用域是块级的。即大括号内的变量和大括号外的变量虽然同名，但是并不会被覆盖。

func main() {
        a := 0
        {
            a := 2
            fmt.Print(a, "\n")
        }
        fmt.Print(a)
}
8、go中对不会用到的变量与包是深恶痛绝的。所以，引入了不用的包、定义了不用的变量，在go中都零容忍，编译时将会被啪啪打脸！

9、go build main.go 与go install 的区别是：go build会在当前目录生成一个可执行文件，而go install 则会在gopath/bin目录生成可执行文件。

10、go path与go module的区别：go 11之前是gopath，和php的include path类似，当我们在一个项目中import一个包时，都是先从本项目找，然后在gopath里找。

go module 相比 gopath，引入了版本管理机制，并统一将包放置到 gopath/pkg/mod 默认目录。和php的composer相似，不过不需要手动安装依赖包，编译时会自动下载并缓存起来。



十五、知识拓展
1、map在并发写时会crash？ 同步专用包： sync 。 import ("sync")    var map  sync.Map

2、如何知道一个变量的类型？import("reflect")

3、go的垃圾回收机制：逃逸分析，变量是到堆里了还是在栈里？

4、go的并发模型：csp 。https://www.cnblogs.com/sunsky303/p/9115530.html

5、go的抢占式调度机制：https://zhuanlan.zhihu.com/p/30353139







