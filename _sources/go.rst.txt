golang(够浪)
===========================

install
------------

官方下载msi文件安装即可，配置环境变量::

 GOROOT go的安装path

 在src目录下创建src/hello/hello.go,并写入以下文件
 // 测试安装
 package main

 import "fmt"

 func main() {
	fmt.Printf("hello, world\n")
 }

 // 编译运行
 cd src/hello
 go build hello.go
 // 编译完成后生成hello.exe，直接运行即可
 // 你可以go install 安装,go clean -i清除

操作
-------------------

格式输入输出::
 
 %d int
 %f float
 %t bool
 %c byte
 %s string
 %x 十六进制显示
 %T 打印对应变量的类型
 %p 打印地址
 
 fmt.Printf("%T",a) // 打印变量类型

 a := 10 // :=推断为int
 fmt.printf("%2d",a)

 a:= 10.1
 fmt.printf("%.3f",a) //保留三位有效数字

 var a int
 fmt.Scan(&a)
 // fmt.Scanf("%d",&a)
 fmt.println(a)

go的随机数生成::
 
 rand.Intn(100) // 生成小于100的整型数据

常规的数据操纵::
 
 b:=[]int{1,2,4} // 一维数组
 b = append(b,2) // 一维数组添加数据
 for i,v:= range b{
	fmt.Println(i,v)
	} // 遍历数组
 // 切片位于堆区,不指定长度;数组需指定长度
 c:=[][]int{{1,2},{3,4}} // 二维数组
 // len(s) 切片的使用大小 cap(s) 容量大小
 s:=make([]int,5) // int 为定义的数据类型,5为大小
 w:=map[string]int{"s1":1} // 定义map,map自动扩容
 w["s2"] = 3 // map 中添加数据
 fmt.println(w["s2"]) // 取出s2的值,如果没有键值返回0
 delete(w,"s1") // 删除数据

  // 遍历map
 for k,v range m{
 fmt.println(k,v)
 }

 // 创建指针
 var p *int // int 型指针空指针nil
 p:=new(int)
 *p=3 // 为指针赋值

  // 数组指针
 a:=[3]int{1,2,3}
 fmt.Printf("%T\n",a)
 fmt.Println(a)
 var p *[3]int // 
 p=&a
 p[0]=100
 fmt.Println(p)



逻辑与(&&)高于逻辑或(||)

定义结构体::

 type Student struct {
	id int // 字段名 类型
	name string
	age int
 }
 // 结构体赋值
 	s:=Student{id:1,
		name:"ff",
		age:4}
	fmt.Println(s)

 // 结构体作为map的value
 m:=make(map[int],Student)
 m[103]=Student{id:1,name="S"}


 // 结构体切片作为map的值
 m:=make(map[int][]Student)  // 结构体传参,值传递


|  为结构体添加方法
::
 
 type person struct {
	name string
	age int
 }
 // 继承自person
 type Student1 struct {
	person
	id int
	score int
 }
 
 func (方法接收者)方法名(参数列表)返回值类型
 // 操作两个对象并返回值
 func (a Student1)add(b Student1)  int{
	t:=a.age + b.age
	return t
 }
 // 打印学生信息的方法
 func (a Student1)printInfo()  {
	fmt.Println(a.score)
	fmt.Println(a.age)
 }

|  接口定义
::

 type 接口名 interface{方法列表}
 // 方法名(参数列表)(返回值列表)
 type Hum interface {
	sayHello()
 }
 // 接口继承
 type Hum1 interface {
	Hum
	Sing(song string)
 }

|  面向对象实例
::

 package main

 import "fmt"

 type AddOperation struct {
	num1 int
	num2 int
 }

 func (a *AddOperation)opera()  int{
	return a.num1 + a.num2
 }

 type SubOperation struct {
	num1 int
	num2 int
 }

 func (s *SubOperation)opera()  int{
	return s.num1 - s.num2
 }
 type Calculate interface {
	opera() int
 }

 type Factory struct {

 }

 func (f *Factory)reckon(num1 int,num2 int,op string)  (value int){
	var interFace Calculate
	switch op {
	case "+":
		a:=AddOperation{num1,num2}
		interFace=&a
	case "-":
		a:=SubOperation{num1,num2}
		interFace=&a
	}
	//value = interFace.opera()
	value = Fs(interFace) // 多态实现
	return

 }
 // 多态
 func Fs(o Calculate)  int{
	return o.opera()
 }

 func main() {
	var s Factory
	d:=s.reckon(7,2,"-")
	fmt.Println("rsult is ",d)
 }


 // 类型断言
 	arr:=make([]interface{},3)
	arr[0] = 1
	arr[1] = "2"
	arr[2] = "hello"
	for i,v :=range arr{
		fmt.Println(i)
		d,ok:=v.(int) // 进行类型断言
		if ok{
			fmt.Println(d,"is int")
		}else {
			fmt.Println("is not a int")
		}
	}



管道
----------------------

channel::

 定义channel
 // 无缓冲channel
 channel := make(chan string) // string 为类型chinnel传输类型
 // 有缓冲channel 
 channel1 := make(chan string,5) // 缓冲区有五个数据
 go func() {channel <- "hello"}()
 str := <-channel
 fmt.Println(str)
 // 无缓冲通道,通道容量为0,应用于两个go程,同步
 // 有缓冲通道,通道容量非0,应用于两个go程,异步
 // 无缓冲通道关闭后,读端无法读到数据
 // 有缓冲通道关闭后,读端可以读到缓存数据
 // 单向写channel var sendch chan <- int make(chan <- int)
 // 单向读channel var sendch <- chan int make(<-chan int)


函数定义
---------------------

函数的定义及参数传递::

 func test1(a ...int)  {
	fmt.Println(a)
 }

 func RandValue(args ...int)  {
	fmt.Println(args[1:])
	test1(args[:]...) // 传递不定参数

 func main(){
 	RandValue(1,4,3,2) // 传递不定参数
 }


生成ras并写文件::

 func GenerateRsaKey(keySize int) {
	// 1. 使用rsa中的GenerateKey方法生成私钥
	privateKey, err := rsa.GenerateKey(rand.Reader, keySize)
	if err != nil {
		panic(err)
	}
	// 2. 通过x509标准将得到的ras私钥序列化为ASN.1 的 DER编码字符串
	derText := x509.MarshalPKCS1PrivateKey(privateKey)
	// 3. 要组织一个pem.Block(base64编码)
	block := pem.Block{
		Type : "rsa private key", // 这个地方写个字符串就行
		Bytes : derText,
	}
	// 4. pem编码
	file, err := os.Create("private.pem")
	if err != nil {
		panic(err)
	}
	pem.Encode(file, &block)
	file.Close()

	// ============ 公钥 ==========
	// 1. 从私钥中取出公钥

	publicKey := privateKey.PublicKey
	// 2. 使用x509标准序列化
	derstream, err := x509.MarshalPKIXPublicKey(&publicKey)
	if err != nil {
		panic(err)
	}
	// 3. 将得到的数据放到pem.Block中
	block = pem.Block{
		Type : "rsa public key",
		Bytes : derstream,
	}
	// 4. pem编码
	file, err  = os.Create("public.pem")
	if err != nil {
		panic(err)
	}
	pem.Encode(file, &block)
	file.Close()

| 错误处理::

1. defer func() // 延时调用
2. defer func() {recover()}() // recover 拦截panic错误,错误发生前使用
   - 捕获错误 err:=recover()
3. errors.New() // 返回错误信息

