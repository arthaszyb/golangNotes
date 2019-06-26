#Go实战--go语言中使用YAML配置文件(与json、xml、ini对比)
2017年06月27日 12:55:43 一蓑烟雨1989 阅读数 28377更多
所属专栏： Go从不放弃到实战
 版权声明：本文为博主原创文章，未经博主允许不得转载。 https://blog.csdn.net/wangshubo1989/article/details/73784907
生命不止，继续 go go go !!!

golang中如何使用json在前面介绍过了： 
《Go语言学习之encoding/json包(The way to go)》

golang中如何使用xml在前面也有介绍过： 
《Go语言学习之encoding/xml(The way to go)》

###json使用
JSON(JavaScript Object Notation, JS 对象标记) 是一种轻量级的数据交换格式。它基于 ECMAScript (w3c制定的js规范)的一个子集，采用完全独立于编程语言的文本格式来存储和表示数据。简洁和清晰的层次结构使得 JSON 成为理想的数据交换语言。 易于人阅读和编写，同时也易于机器解析和生成，并有效地提升网络传输效率。

新建一个文件名为conf.json，键入内容：
```
{
    "enabled": true,
    "path": "/usr/local"
}
```
新建main.go，键入内容：
```
package main

import (
    "encoding/json"
    "fmt"
    "os"
)

type configuration struct {
    Enabled bool
    Path    string
}

func main() {
    file, _ := os.Open("conf.json")
    defer file.Close()

    decoder := json.NewDecoder(file)
    conf := configuration{}
    err := decoder.Decode(&conf)
    if err != nil {
        fmt.Println("Error:", err)
    }
    fmt.Println(conf.Path)
}
```

###xml使用
可扩展标记语言，标准通用标记语言的子集，是一种用于标记电子文件使其具有结构性的标记语言。

新建一个文件名为conf.xml，键入内容：
```
<?xml version="1.0" encoding="UTF-8" ?>
<Config>
   <enabled>true</enabled>
   <path>/usr/local</path>
</Config>
```
新建main.go，键入内容：
```
package main

import (
    "encoding/xml"
    "fmt"
    "os"
)

type configuration struct {
    Enabled bool   `xml:"enabled"`
    Path    string `xml:"path"`
}

func main() {
    xmlFile, err := os.Open("conf.xml")
    if err != nil {
        fmt.Println("Error opening file:", err)
        return
    }
    defer xmlFile.Close()

    var conf configuration
    if err := xml.NewDecoder(xmlFile).Decode(&conf); err != nil {
        fmt.Println("Error Decode file:", err)
        return
    }

    fmt.Println(conf.Enabled)
    fmt.Println(conf.Path)

}
```

###ini的使用
INI文件格式是某些平台或软件上的配置文件的非正式标准，以节(section)和键(key)构成，常用于微软Windows操作系统中。这种配置文件的文件扩展名多为INI，故名。

新建一个文件名为conf.ini，键入内容：

```
[Section]
enabled = true
path = /usr/local # another comment
1
2
3
4
使用第三方库：go get gopkg.in/gcfg.v1

新建main.go，键入代码：

package main

import (
    "fmt"

    "gopkg.in/gcfg.v1"
)

func main() {
    config := struct {
        Section struct {
            Enabled bool
            Path    string
        }
    }{}

    err := gcfg.ReadFileInto(&config, "conf.ini")
    if err != nil {
        fmt.Println("Failed to parse config file: %s", err)
    }
    fmt.Println(config.Section.Enabled)
    fmt.Println(config.Section.Path)
}
```

输出： 
> true 
> /usr/local

###yaml使用
之前介绍json和xml都是为了抛砖引玉，现在才开始正式介绍golang中的yaml。 
何为yaml? 
百度百科上这么说的： 
YAML是“另一种标记语言”的外语缩写[1] （见前方参考资料原文内容）；但为了强调这种语言以数据做为中心，而不是以置标语言为重点，而用返璞词重新命名。它是一种直观的能够被电脑识别的数据序列化格式，是一个可读性高并且容易被人类阅读，容易和脚本语言交互，用来表达资料序列的编程语言。

wiki上这么说的： 
> YAML is a human-readable data serialization language. It is commonly used for configuration files, but could be used in many applications where data is being stored (e.g. debugging output) or transmitted (e.g. document headers). YAML targets many of the same communications applications as XML, but has taken a more minimal approach which intentionally breaks compatibility with SGML.[1] YAML 1.2 is a superset of JSON, another minimalist data serialization format where braces and brackets are used instead of indentation.

通俗的说，也就是一种标记语言。

golang的标准库中暂时没有给我们提供操作yaml的标准库，但是github上有很多优秀的第三方库开源给我们使用。

新建一个文件名为conf.yaml，键入内容：
```
enabled: true
path: /usr/local
```
> https://github.com/kylelemons/go-gypsy

> go get -u github.com/kylelemons/go-gypsy

新建文件main.go，键入内容：
```
package main

import (
    "fmt"

    "github.com/kylelemons/go-gypsy/yaml"
)

func main() {
    config, err := yaml.ReadFile("conf.yaml")
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(config.Get("path"))
    fmt.Println(config.GetBool("enabled"))
}
```

> gopkg.in/yaml.v2

>go get gopkg.in/yaml.v2

修改main.go代码：
```
package main

import (
    "fmt"
    "io/ioutil"
    "log"

    "gopkg.in/yaml.v2"
)

type conf struct {
    Enabled bool   `yaml:"enabled"`
    Path    string `yaml:"path"`
}

func (c *conf) getConf() *conf {

    yamlFile, err := ioutil.ReadFile("conf.yaml")
    if err != nil {
        log.Printf("yamlFile.Get err   #%v ", err)
    }
    err = yaml.Unmarshal(yamlFile, c)
    if err != nil {
        log.Fatalf("Unmarshal: %v", err)
    }

    return c
}

func main() {
    var c conf
    c.getConf()

    fmt.Println(c)
}
```

> https://github.com/ghodss/yaml

> go get -u github.com/ghodss/yaml

修改main代码，这里主要是使用了yaml和json的相互转换：
```
package main

import (
    "fmt"

    "github.com/ghodss/yaml"
)

func main() {
    j := []byte(`{"name": "John", "age": 30}`)
    y, err := yaml.JSONToYAML(j)
    if err != nil {
        fmt.Printf("err: %v\n", err)
        return
    }
    fmt.Println(string(y))
    /* Output:
    name: John
    age: 30
    */
    j2, err := yaml.YAMLToJSON(y)
    if err != nil {
        fmt.Printf("err: %v\n", err)
        return
    }
    fmt.Println(string(j2))
    /* Output:
    {"age":30,"name":"John"}
    */
}
```
