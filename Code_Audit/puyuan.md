# 普元EOS Platform

Fofa语句：body="普元" || (body="ame.primeton.com" && body="eos-web")



应用目录结构

```
├── BPS Work Client.url
├── Coframe.url
├── Governor.url
├── LICENSE
├── LICENSE_en
├── apache-tomcat-8.5.57
├── apps_config
├── components
├── ico
├── scripts
├── startServer.cmd
├── startServer.sh
├── stopServer.cmd
├── stopServer.sh
└── workspace.url
```



### Primeton EOS Platform jmx反序列化



修复方式是在该文件中查找并删除以下配置项：

```
<handler id="JmxServiceProcessor" suffix=".jmx" sortIdx="0" class="com.primeton.access.client.impl.processor.JmxServiceProcessor" />
```

直接打开文件

```
<?xml version="1.0" encoding="UTF-8"?>
<handlers>
	<!--
	handlers that are added to processors
	id: the id of the handler, must be unique;
	suffix: the suffix that this handler will be matched to,this attribute can hold multiple suffix string that
	   are separated by ','.
	sortIdx: sort order,when multiple handler are matched to a suffix, the hander with a smaller sortIdx will
	     be executed first;
	class：processor impl class;
	Note: there will be different processor instance for different suffix string.
	-->
	<handler id="flowProcessor" suffix=".flow" sortIdx="0"
		class="com.primeton.ext.engine.core.processor.HttpPageFlowProcessor" />

	<handler id="actionProcessor" suffix=".action" sortIdx="0"
		class="com.primeton.ext.engine.core.processor.ActionProcessor" />

	<handler id="downloadProcessor" suffix=".download"
		sortIdx="0"
		class="com.primeton.access.http.impl.processor.DownloadProcessor" />

	<handler id="downloadConfigProcessor" suffix=".configdownload"
		sortIdx="0"
		class="com.primeton.access.http.impl.processor.DownloadConfigProcessor" />	
		
	<handler id="ConfigurationDownloadProcessor" suffix=".governordownload" sortIdx="0"
		class="com.primeton.access.http.impl.processor.GovernorDownloadConfigProcessor" />
	<handler id="GovernorDownloadConfigurationProcessor" suffix=".governordownload" sortIdx="0"
		class="com.primeton.access.http.impl.processor.GovernorDownloadConfigProcessor" />
	<!--
	<handler id="ajaxFlowProcessor" suffix=".flow.ajax,.flowx.ajax"
		sortIdx="0"
		class="com.primeton.ext.engine.core.processor.AjaxPageflowProcessor" />
 	-->

	<handler id="commonServiceProcessor" suffix=".remote" sortIdx="0"
		class="com.primeton.access.client.impl.processor.CommonServiceProcessor" />

	<handler id="ajaxBizProcessor" suffix=".biz.ajax"
		sortIdx="0"
		class="com.primeton.ext.engine.core.processor.AjaxBizProcessor" />

	<handler id="precompileProcessor" suffix=".precompile"
		sortIdx="0"
		class="com.primeton.ext.engine.core.processor.PrecompileProcessor" />

	<handler id="debugBizProcessor" suffix=".bizx.debug,.biz.debug"
		sortIdx="0"
		class="com.primeton.ext.engine.core.processor.DebugProcessor" />

	<handler id="gzipProcessor" suffix=".gzip" sortIdx="0"
		class="com.primeton.ext.engine.core.processor.GzipProcessor" />

	<handler id="jspDebugProcessor" suffix=".jsp.debug" sortIdx="0"
		class="com.primeton.ext.engine.core.processor.JspDebugProcessor" />

	<handler id="ajaxServiceProcessor" suffix=".service.ajax" sortIdx="0"
		class="com.primeton.sca.http.service.processor.AjaxServiceProcessor" />

	<handler id="ajaxJavaBeanProcessor" suffix=".java.beanx" sortIdx="0"
		class="com.primeton.ext.engine.core.processor.BeanCServiceProcessor" />
	
	
	
	<handler id="ajaxSpringBeanProcessor" suffix=".spring.beanx" sortIdx="0"
		class="com.primeton.spring.processor.SpringCServiceProcessor" />

	<handler id="extBizProcessor" suffix=".biz.ext" sortIdx="0"
		class="com.primeton.ext.engine.core.processor.ExtBizProcessor" />
 
	<handler id="ternimateDebugProcessor" suffix=".thread.terminate" sortIdx="0"
		class="com.primeton.ext.engine.core.processor.TernimateDebugProcessor" />
		
	<handler id="JmxServiceProcessor" suffix=".jmx" sortIdx="0"
		class="com.primeton.access.client.impl.processor.JmxServiceProcessor" />
</handlers>
```

跟进之后

```
    public void process(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
        if (!"POST".equalsIgnoreCase(request.getMethod())) {
            throw new ServletException("Caucho protocol requires POST");
        } else {
            try {
                ResultDefinition resultDef = new ResultDefinition();

                try {
                    ObjectInputStream requestIn = new ObjectInputStream(request.getInputStream());
                    MethodDefinition methodDef = (MethodDefinition)requestIn.readObject();
                    this.service.service(request.getRequestURL().toString(), methodDef, resultDef);
                } catch (InvocationTargetException var6) {
```



利用的话这个文章有写

```
https://forum.butian.net/share/2745
```

