# golang处理http的json数据方法
通常http接口开发是最常见最基本的开发场景，这里包括三个基本部分：发送请求，接收请求和处理回包。这里假定通信的基本数据格式为json。

## 接收数据方法
按照己方接口的入参格式构造入参结构体，用json标注来进行匹配。json标注包含两个常用的基本flag：
* ``json:"clusterId" binding:"required"``: 表示必须，如果入参json中不包含该项，则直接返回失败。
* ``json:"instanceIds,omitempty"``:表示非必须，如果入参不包含，则生成的结构体中不包含该项。

## 处理数据方法
接收到服务端返回的回包后，一般根据http包回包数据为[]byte类型，如果数据结构为json，有三种方法处理：
* json包解析到struct
根据自己对数据的需要构造结构体，不需要的数据可以不写到结构体中，需要的则严格按照数据层次构造，根据json标注进行匹配。如下示例：
完整返回数据如下：
``
{"status":"1",
"count":"1",
"info":"OK",
"infocode":"10000",
"lives":[{"province":"广东","city":"深圳市","adcode":"440300","weather":"小雨","temperature":"28","winddirection":"东北","windpower":"≤3","humidity":"80","reporttime":"2019-07-28 17:14:14"}]
}
``
而我只需要weather和temperature两个值。则构造如下struct：
``
// GetWeather 获取天气回包结构体  
type GetWeather struct{  
	Lives []WeatherDesc `json:"lives"`  
}  

// WeatherDesc 天气详情  
type WeatherDesc struct{  
	Weather string `json:"weather"`  
	Temperature string `json:"temperature"`  
}  
``
然后用格式化数据即可。
``var weatherStruBody GetWeather  
	json.Unmarshal(resp,&weatherStruBody)  
	weather = weatherStruBody.Lives[0].Weather  
	temperature = weatherStruBody.Lives[0].Temperature``
  
* 采用断言来层析
即避免构造结构体。需要构造一个万能结构体(如map[string]interface{})来格式化任何结构的json数据，然后通过对每一层的interface进行断言来确认其数据类型并获取其中数据。
``
//利用断言对复杂数据进行解析. 通常方法是用创建结构体并赋值解决.
	var respJSON map[string]interface{}  
	json.Unmarshal(responseBody,&respJSON)  
	InstanceID,ok := respJSON["result"].(map[string]interface{})["data"].(map[string]interface{})["deviceList"].([]interface{})[0].(map[string]interface{})["instanceId"].(string)  
	if !ok{  
		elog.Errorf("%v assertion failed.",respJSON["result"])  
	}  
	return InstanceID  
  ``
  这种方法实际并不会更简单，一层层断言本身就不简单同时适应性也差，一旦断言失败，则直接panic。不建议生产环境使用。
  
  * 采用外部包实现[]byte到json转换的处理。如github.com/bitly/go-simplejson。
  使用示例可参见[这里](https://www.cnblogs.com/pluse/p/9157599.html)

##发送数据方法
以net/http包中为常用手段。根据服务端要求的入参格式创建对应的结构体。
