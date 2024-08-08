## 为什么可以url编码才可以访问到上传文件

虽然泛微ebridge使用了jfinal,但是使用url编码来访问文件不是利用的JFinal的DenyAccessJsp绕过，因为在使用tomcat除了编码还可以使用.jsp;来访问文件(https://forum.butian.net/share/1899)，但是这个在使用.jsp;访问时却依然会跳转

![image-20240808161652102](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808161652102.png)

直接可以打开OutSysProxyHandler

illocalrequesturl首先会从proxy.xml里取出view标签里的值，当取尽之后再从excludes中取

Proxy.xml中的view标签和excludes如下

```
<view>
    <pattern>*.jsp</pattern>
    <pattern>/weaver/*</pattern>
</view>
	<excludes>
        <pattern>/</pattern>
		<pattern>/login.do</pattern>
		<pattern>/*</pattern>
		<pattern>/druid/*</pattern>
		
    </excludes>
```



![image-20240808153926950](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808153926950.png)

往下执行可以看到有一个循环语句，他会从proxy.xml中取出列表里的元素来对requestURL进行匹配，在这个循环中如果匹配到了.jsp结尾的文件路径或者/weaver/开头的路径都会返回false

![image-20240808155437440](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808155437440.png)

在返回false之后会来到，由于islocalrequest为false所以无法进入第一个if语句，继续执行会来到else语句中，

![image-20240808160440541](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808160440541.png)



这里会有一个getLastAccessOutSys

![image-20240808160614353](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808160614353.png)

但是返回为null所以直接来到else这里 err code = -15，在经过一系列处理后会重定向到/wxapi/erropage

![image-20240808160754158](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808160754158.png)

![image-20240808161124649](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808161124649.png)





而在我们使用url编码时当pattern.marcger来匹配requesturl会匹配不到

![image-20240808154619136](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808154619136.png)

由于匹配不到所以会继续从excludes中取值匹配，直到来到/*

![image-20240808154924008](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808154924008.png)

此时islocalrequest会为true

![image-20240808154239117](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808154239117.png)

随后经过if语句判断之后直接可以访问到我们的文件

![image-20240808154502466](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808154502466.png)

![image-20240808161139134](/Users/a1kaid/Library/Application Support/typora-user-images/image-20240808161139134.png)