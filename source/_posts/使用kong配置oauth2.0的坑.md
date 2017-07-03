---
title: 使用kong0.10.x配置oauth2.0的坑
date: 2017-07-03 17:33:37
categories: [kong]
tags: [kong]
---
kong的安装和配置启动之前一篇博文已经讲过，这里推荐一个UI管理:kong-dashboard，功能还不是很强大，但是可以省去一些比较基础的配置（api，consumers，plugins的管理），安装配置地址如下： [官方链接](https://www.npmjs.com/package/kong-dashboard)

### 正文
1.oauth2.0插件使用配置的[官方文档](https://getkong.org/plugins/oauth2-authentication/)，但是做本地开发时还是踩了一些坑的，这里记录一下。首先使用oauth2.0需要kong配置https协议，由于是本地开发，随便自己生成就可以了，没有openssl的自己装
**生成私钥**：
```
openssl genrsa -out server.key 1024
```
执行结果
```
Generating RSA private key, 1024 bit long modulus
.++++++
...............++++++
e is 65537 (0x10001)
```
**生成证书**
```
openssl req -new -x509 -key server.key -out server.pem -days 365
```
执行结果（这里是我的示例，要改成自己需要的，其实只有Common Name是重点，其它可有可无，这个表示的是域名，由于是本地开发，所以就用localhost）
```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Guangdong
Locality Name (eg, city) []:Guangzhou
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Test  
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:
```
至此，会在当前路径下生成两个文件，我的是在/usr/local/ssl下，根据官方文档配置ssl分别填写（注意版本）
```
$ curl -i -X POST http://localhost:8001/certificates \
    -F "cert=@/usr/local/ssl/server.pem" \
    -F "key=@/usr/local/ssl/server.key" \
    -F "localhost"
```
创建api添加ssl（注意是host模式，不能用uris（低于0.10版本的叫request_path）模式）
```
curl -i -X POST http://localhost:8001/apis \
    -d "name=sslapi" \
    -d "upstream_url=http://www.xxx.com" \
    -d "hosts=localhost"
```
至此配置完成，测试一下
```
$ curl -i https://localhost:8443/
```
可以访问，ok，建议配置上把https_only属性也配置上，这样的话http形式就不能访问了，反正oauth一定要https，接下来创建一个oauth2.0插件
这里有四种模式一般来说都是用enable_authorization_code，但是这里测试起来复杂，就用password模式
```
$ curl -X POST http://kong:8001/apis/{api}/plugins \
    --data "name=oauth2" \
    --data "config.enable_password_grant=true" \
    --data "config.scopes=email,phone,address,password" \
    --data "config.mandatory_scope=true"
```
这样sslapi就用上了oauth2.0插件，再次访问的话
```
$ curl -i https://localhost:8443/
```
报如下错误：
```
{"error_description":"The access token is missing","error":"invalid_request"}
```
所以，现在需要一个认证服务器，在本地机器上创建一个脚本，我用的是java，运行在tomcat（这里的坑可就多了，看注释），在此之前，还需要创建一个consumer，因为验证的时候有一些配置需要用到
```
$ curl -X POST http://localhost:8001/consumers/ \
    --data "username=user123" \
    --data "custom_id=SOME_CUSTOM_ID"
```
为consumer创建应用来管理oauth和api（这里password模式不需要redirect_uri属性，但是又不能为空，随便填，比如code模式就需要用到）
```
$ curl -X POST http://kong:8001/consumers/{consumer_id}/oauth2 \
    --data "name=Testapplication" \
    --data "redirect_uri=http://some-domain/endpoint/"
```
再创建一个api来转发到验证服务器
```
curl -i -X POST http://localhost:8001/apis \
    -d "name=auth" \
    -d "upstream_url=http://www.xxx.com/auth" \
    -d "uris=/auth"
```
开始写验证服务（用户填写用户名密码发到kong，kong转发到验证服务器）
```
// 接收验证请求
@ResponseBody
@RequestMapping("auth")
public Result auth(@RequestParam("username") String username, 
	@RequestParam("password") String password, 
	@RequestParam("url") String url) {
	Result result = new Result();
	try {
		// 这里写死验证条件，根据需要去验证
		if (username.equals("zfs") && password.equals("123456")) {
			// url不能写死，要当成参数传进来，这个url是你要访问的api再加端点，如我这里要访问的是https://localhost:8443下的资源，所以这里url为https://localhost:8443/oauth2/token（kong提供了/oauth2/authorize和/oauth2/token两个，这里不是code模式，不需要/oauth2/authorize，这些都是需要用post访问的）
			URL realUrl = new URL(url);
			Map<String,Object> param = new HashMap<String,Object>();
			// 这些在创建consumers的application的时候会生成
			param.put("client_id", "9b120e441c664526a179e64f5fae9d61");
			param.put("client_secret", "f272ad1b876e48358062893300b14d37");
			// 这个可以看这里https://tools.ietf.org/html/rfc6749#section-7.1
			param.put("grant_type", "password");
			// 这些在创建oauth2.0插件的时候生成
			param.put("scope", "password");
			param.put("provision_key", "function");
			// 这些是自己的用户信息
			param.put("authenticated_userid", "233");
			param.put("username", "zfs");
			param.put("password", "123456");
			// 必须是post请求
			String res = sendHttps(url, param);
			result.setSuccessResult(res);
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
	return result;
}
```
验证服务器验证通过后发送请求到kong的OAuth2 接口，kong会下发token给请求的用户
```
public static String sendHttps(String SYS_VULLN_URL_JSON, Map<String,Object> param) throws Exception {  
        InputStream in = null;  
        OutputStream out = null;  
        String returnValue = "";  
		BufferedReader br= null;	
		String result = "";
		
        StringBuffer tempStr = new StringBuffer();  
        String responseContent="";  
        HttpURLConnection conn = null;  
        
        try {
	        // 这里ssl要自己写，因为这是一个SSL版本问题。服务器只支持SSLv3，而Java将从v2开始，并尝试向上协商，但并非所有服务器都支持这种类型的协商。在网上找答案说使用SSLv3是目前唯一的解决方案，默认是TLS1，会报connect reast的错误，具体的tomcat对应关系见https://blogs.oracle.com/java-platform-group/diagnosing-tls,-ssl,-and-https，我这里是tomcat7，所以设置为TLSv1.2
            System.setProperty("https.protocols", "TLSv1.2");  
            SSLContext sc = SSLContext.getInstance("TLSv1.2");  
            sc.init(null, new TrustManager[] { new MyX509TrustManager() }, new java.security.SecureRandom());  
            URL url = new URL(SYS_VULLN_URL_JSON);  
            
            HttpsURLConnection https = (HttpsURLConnection)url.openConnection();  
           if (url.getProtocol().toLowerCase().equals("https")) {  
               https.setHostnameVerifier(
               		new HostnameVerifier() {  
							@Override
							public boolean verify(String hostname,
									SSLSession session) {
								return false;
							}  
               		});  
               conn = https;  
           } else {  
               conn = (HttpURLConnection) url.openConnection();  
           }  
           ((HttpsURLConnection) conn).setSSLSocketFactory(sc.getSocketFactory());  
           ((HttpsURLConnection) conn).setHostnameVerifier(new TrustAnyHostnameVerifier()); 
           
         //设置请求的参数
         String post = "";
         if(null != param && param.size() > 0) {
	         Set<Entry<String, Object>> entrys = param.entrySet();
	         for(Entry<String, Object> entry : entrys){
		         String key = StringUtil.objToStr(entry.getKey());
		         String val = StringUtil.objToStr(entry.getValue());
		         post += key + "=" + val + "&";
			}
			post = post.substring(0, post.length() - 1);
		}
           
           conn.setRequestMethod("POST");// 提交模式
           // 发送POST请求必须设置如下两行
           conn.setDoOutput(true);
           conn.setDoInput(true);
           // 获取URLConnection对象对应的输出流
           PrintWriter printWriter = new PrintWriter(conn.getOutputStream());
           // 发送请求参数
           printWriter.write(post);//post的参数 xx=xx&yy=yy
           // flush输出流的缓冲
           printWriter.flush();
           conn.connect();
           //开始获取数据
           BufferedInputStream bis = new BufferedInputStream(conn.getInputStream());
           ByteArrayOutputStream bos = new ByteArrayOutputStream();
           int len;
           byte[] arr = new byte[1024];
           while((len=bis.read(arr))!= -1){
               bos.write(arr,0,len);
               bos.flush();
           }
           bos.close();
           return bos.toString("utf-8");
        } catch (Exception e) {  
            throw e;  
        } finally {  
            try {  
                in.close();  
            } catch (Exception e) {  
            }  
            try {  
                out.close();  
            } catch (Exception e) {  
            }  
        }  
    }
```

```
// 这个方法的作用是信任所有的证书，因为上面申请的证书并不是合规的，实际开发是不需要这样的
private static void trustAllHosts() {
	TrustManager[] trustAllCerts = new TrustManager[] { new X509TrustManager() {
		public java.security.cert.X509Certificate[] getAcceptedIssuers() {
			return new java.security.cert.X509Certificate[] {};
		}
		public void checkClientTrusted(X509Certificate[] chain,
			String authType) {
		}
		public void checkServerTrusted(X509Certificate[] chain,
			String authType) {
		}
	} };
	try {
		SSLContext sc = SSLContext.getInstance("TLS");
		sc.init(null, trustAllCerts, new java.security.SecureRandom());
		HttpsURLConnection
				.setDefaultSSLSocketFactory(sc.getSocketFactory());
	} catch (Exception e) {
		e.printStackTrace();
	}
} 
```
认证服务器写完，开始验证
```
$ curl -i https://localhost:8443/auth?username=zfs&password=123456&url=https://localhost:8443/oauth2/token
```
返回结果（这是我自己的json结构，即上面的Result）
```
{"data":"{\"refresh_token\":\"673871402d0f45fe962be3b9fa2eb68a\",\"token_type\":\"bearer\",\"access_token\":\"924e79bd9fa941d08cc5b96fe8eba44b\",\"expires_in\":7200}\n","err":0,"msg":"操作成功！"}
```
拿到access_token之后就可以访问https://localhost:8443下的资源了，试一下
```
$ curl -k 'https://localhost:8443/test' -H 'Authorization: Bearer 924e79bd9fa941d08cc5b96fe8eba44b'
```
这样整个流程就走完了，其实这里因为用了虚拟机，传进去的url参数不是https://localhost:8443/oauth2/token，这个地址是不对的，因为在主机里访问虚拟机不能直接用localhost，而是得用ip+port，所以我传的是https://myip:8443/oauth2/token，这里就有一个问题了，证书绑的是域名，ip请求并不能被识别，所以再创了一个api，心好累
```
curl -i -X POST http://localhost:8001/apis \
    -d "name=zhuanfa" \
    -d "upstream_url=https://localhost:8443/oauth2/token" \
    -d "uris=/zhuanfa"
```
所以验证的请求就会变成
```
$ curl -i https://localhost:8443/auth?username=zfs&password=123456&url=https://myip:8443/zhuanfa/oauth2/token
```