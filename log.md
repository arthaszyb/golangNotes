## log.Panicf,log.Fatalf和log.Printf区别
测试源码如下：
```
func main() {
	var c config.ServerConf
	//c.GetConf(serverConfPath)
	c.GetConf(confFilePath)
	//print content of struct type
	fmt.Printf("%+v\n", c)
	fmt.Println(c.UserName)
	fmt.Println(c.DeptId)
	deptID, err := strconv.Atoi(c.DeptId)
	if err != nil {
		log.Fatalf("DeptId %s can't change to int. %v", c.DeptId, err)
	}
	fmt.Println(deptID)
	fmt.Println(c.FormalEnv.ApiServer)
}
```
执行结果。
```
// 这个是用的Fatalf，触发后程序执行退出，code为1
[Running] go run "d:\work\code\go\yuntihandle\cmd\main.go"
{YuntiConf:{UserPropty:{UserId:1111 DeptId:64s UserName:yaungzhou} ServerPropty:{FormalEnv:{Env:3 ApiServer:http://exp.yunti.oa.com/apply/api/?api_key=} TestEnv:{Env:2 ApiServer:http://yunti.oa.com/apply/api/?api_key=}}}}
yaungzhou
64s
2019/06/26 12:11:37 DeptId 64s can't change to int. strconv.Atoi: parsing "64s": invalid syntax
exit status 1

[Done] exited with code=1 in 2.091 seconds

// 这个是用的Printf，触发后程序继续执行。
[Running] go run "d:\work\code\go\yuntihandle\cmd\main.go"
{YuntiConf:{UserPropty:{UserId:1111 DeptId:64s UserName:yaungzhou} ServerPropty:{FormalEnv:{Env:3 ApiServer:http://exp.yunti.oa.com/apply/api/?api_key=} TestEnv:{Env:2 ApiServer:http://yunti.oa.com/apply/api/?api_key=}}}}
yaungzhou
64s
2019/06/26 12:12:11 DeptId 64s can't change to int. strconv.Atoi: parsing "64s": invalid syntax
0
http://exp.yunti.oa.com/apply/api/?api_key=

[Done] exited with code=0 in 2.008 seconds

// 这个是用的Panicf，触发后程序触发panic，可以看到具体出错的代码文件和行数。执行退出，code为2。
[Running] go run "d:\work\code\go\yuntihandle\cmd\main.go"
{YuntiConf:{UserPropty:{UserId:1111 DeptId:64s UserName:yaungzhou} ServerPropty:{FormalEnv:{Env:3 ApiServer:http://exp.yunti.oa.com/apply/api/?api_key=} TestEnv:{Env:2 ApiServer:http://yunti.oa.com/apply/api/?api_key=}}}}
yaungzhou
64s
2019/06/26 12:12:35 DeptId 64s can't change to int. strconv.Atoi: parsing "64s": invalid syntax
panic: DeptId 64s can't change to int. strconv.Atoi: parsing "64s": invalid syntax

goroutine 1 [running]:
log.Panicf(0x540ab5, 0x21, 0xc000081ef8, 0x2, 0x2)
	C:/Go/src/log/log.go:340 +0xc7
main.main()
	d:/work/code/go/yuntihandle/cmd/main.go:23 +0x3f5
exit status 2

[Done] exited with code=1 in 2.165 seconds
```
