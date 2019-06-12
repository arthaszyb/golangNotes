# golang学习
---
## 基础知识
### 指针
a为变量，则&a为其指针，即是一个指向a的内存地址。*&a为该指针的值。

## 包知识
### flag包
* flag包负责处理cmd的命令行参数。
* flage.Args()返回所有非标志参数的命令行参数的string类型集合切片。
* flag.String('s','','something'）创建一个string类型的参数变量，s为标志参数的名字，中间的微默认值，最后是该参数的描述信息。其本身是一个指针。
* flag.Bool('s',false,'something')创建一个布尔类型的参数变量，同上。
* 当用户输入无效的标志参数或-h或-help时，就会显示help信息给用户。
* flag.Parse()用于获取并更新所有的args值。
* os.Args()也用于获取命令行参数，os.Args[1]与flag.Args()[0]都表示第一个参数，而非文件本身。

### strings包
* strings.join(someSlince,p)表示将一个切片转化为一个string，分割标签为p

### 代码实例:

```
//模拟echo命令的-s和-n功能。
package main

import (
	"flag"
	"fmt"
	"strings"
)

var n = flag.Bool("n", false, "忽略行尾的换行符")
var spe = flag.String("s", "", "制定分隔符")

func main() {
	flag.Parse()
	fmt.Print(strings.Join(flag.Args(), *spe))
	if !*n {
		fmt.Println()
	}
}
```

### 小tips
* golang的字符串表达与python不一样，go中用双银行或反引号表示字符，如果用单引号，就会报错如下：  

> cannot use 'smt' (type rune) as type string in array or slice literal

因为在go中，单引号一般用来表示「rune literal」 ，即——码点字面量。
* range(x)每次循环返回的是一对值，包含序号和值，正规表示应该是:  
> x, y := range(10)
> 
> _, y := range(10) //只需要值时.
> 
> x, _ := range(10) //只需要序号.
> 
>x := range(10) //只需要序号.

* 单独文件需要独立编译和运行的话，就需要定义包名为main。
* main函数不能有入参出参，但可以从命令行参数获取外部参数，使用os.args或flag包获取。

### new函数
* new函数创建变量与var变量创建没有不一样，只是不用指定变量名。
* new(T)创建了一个T类型的变量，返回该变量的内存地址，返回的指针类型为*T  

### slince
* slince底层是引用了一个数组或者其切片。slince包含三部分：指针+长度（len）+容量（cap），len不能大雨cap。slince发生变化时，如果要求的cap大于底层数组的len，则会生成一个新的底层数组，新数组cap会略大于原数组，将原数组的元素copy过去，然后append新的元素。
* 因为不知道slince变更后是否有底层数组数组空间的变化或内存的重新分配，因此在变更时我们通常直接将变更后的结果赋值给slince变量：
`sli = append(sli,strx)`
* slince的元素是间接引用的，只有其指针是不变的，除了与nil做比较外，其他比较均不算合法，当然强制的话也是可以比较的，比如Bytes.Equal就可以比较两个byte类型的slince。

### Map
* map类型为map[K]V。创建方法通常两种：
```
ages := make(map[string]int)
```
```
   ages := map[string]int{  
   "alice": 33,  
   "jim": 24,  
   }
   ```
* 与slince一样，map除了与nil比较，其他比较均不合法。
* go没有提供set类型，可以用map去实现set的应用场景，因为map的每个key都是不同的。 

### 结构体
- 结构体不能包含其自身，但可以是自身指针类型，以此可以创建递归的数据结构，例如二叉树。注意其元素可导出性还是一样遵守大小写。  
```
type tree struct{  
    Value int  
    Left, Right *tree    
}
```

- 结构体是一种数据类型，创建后可以这样来使用，struc_type{...},注意是大括号，如创建一个type s truct{A,B int}结构体类型的变量：  
```
*p := s{2,3}
```
- 考虑到效率，一般较大的结构体一般用指针的方式传入和传出。如果要在函数内部修改结构体成员，则必须用指针，因为go语言中，函数传入参数都是值拷贝传入，而结构体本身是一个指定type的变量。  
```
func changestruc(e *SomeStruc){  
    e.Id = e.Id * 22
    }
```
- 如果结构体内成员都可以比较，则结构体也可以比较。


### 包与依赖
- 在 $GOPATH 之外使用 go modules, 如果是现有项目的话可以直接 go mod init生成模块, 现有项目会根据 git remote 自动识别 module 名, 但是新项目的话就会报 go: cannot determine module path for source directory, 需要带上 module名。
- 执行完go mod init后再执行go mod tidy以同步依赖的具体文件。其下载好的文件默认会在$GOPATH/pkg/mod/cache/download/下。

---
