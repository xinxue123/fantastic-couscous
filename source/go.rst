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

格式输入输出::
 
 %d int
 %f float
 %t bool
 %c byte
 %s string
 %x 十六进制显示
 %T 打印对应变量的类型
 
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

逻辑与(&&)高于逻辑或(||)



chinnel::

 定义chinnel
 channel := make(chan string) // string 为类型chinnel传输类型
 go func() {channel <- "hello"}()
 str := <-channel
 fmt.Println(str)
 // 无缓冲通道,通道容量为0,应用于两个go程,同步
 // 有缓冲通道,通道容量非0,应用于两个go程,异步
 // 无缓冲通道关闭后,读端无法读到数据
 // 有缓冲通道关闭后,读端可以读到缓存数据
 // 单向写channel var sendch chan <- int make(chan <- int)
 // 单向读channel var sendch <- chan int make(<-chan int)


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

