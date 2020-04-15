# 一次设置HTTP隧道代理的问题排查
##问题
通过用户名和密码使用VIP代理时，指定了代理域名，代理端口，用户名，密码，但是代理仍然返回“HTTP/1.1 407 Proxy Authentication Required”。代码如下：

认证器

    package zone.guo.ipcheck;
    import lombok.Setter;
    import java.net.Authenticator;
    import java.net.PasswordAuthentication;

    @Setter
    public class IpsProxyAuth extends Authenticator {
    
        private String user;
        private String pwwd;
    
        protected PasswordAuthentication getPasswordAuthentication(){
            return new PasswordAuthentication(user, pwwd.toCharArray());
        }
    }
使用方法

        ......
        SocketAddress inetAddress = new InetSocketAddress(ip, port);

        auth.setUser("userName");
        auth.setPwwd("pwwd");
        Authenticator.setDefault(auth);

        Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress(proxyIp, proxyPort));

        Socket socket = new Socket(proxy);
        try{
            socket.connect(inetAddress, 5000);
            socket.close();
            isAlive = true;
            log.error("ip check - connect to {}:{} successfully", ip, port);
        }catch (SocketTimeoutException e){
            log.error("ip check - connect to {}:{} timeout", ip, port, e);
        }catch (IOException e){
            log.error("ip check - connect to {}:{} error", ip, port, e);
        }catch (InternalError e){
            log.error("ip check - connect to {}:{} error", ip, port, e);
        }
        
        ......
##分析
首先，这里的使用代理的方式是为Socket指定代理，不同于一般的为HTTP请求设置代理。返回407的直接原因是我们自定义的认证器没有被触发。那么，先试一下为HTTP请求设置代理时，认证器会不会被触发。

    @Test
    public void testRawHttpAuth() throws Exception{
      IpsProxyAuth auth = new IpsProxyAuth();
      Authenticator.setDefault(auth);
    
      String proxyIp = "proxyIp";
      int proxyPort = 8080;
      String user = "userName";
      String pwwd = "pwwd";
    
      auth.setPwwd(pwwd);
      auth.setUser(user);
    
      Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress(proxyIp, proxyPort));
    
      URL url = new URL("http://www.baidu.com");
      HttpURLConnection conn = (HttpURLConnection)url.openConnection(proxy);
      BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream()));
    
      String line = null;
      while((line=br.readLine())!=null){
        System.out.println(line);
      }
      br.close();
    }

结果发现一切正常，认证器执行正常。

那么为什么我们为sokcet指定代理时，认证器没有被触发呢？答案就在源码里，就看我们是否有耐心找。翻看`sun.net.www.protocol.http.HttpURLConnection`
的源码，看看哪里会使用`Authenticator`。发现方法调用链路是：

    doTunneling->resetProxyAuthentication->getHttpProxyAuthentication->privilegedRequestPasswordAuthentication->Authenticator.requestPasswordAuthentication

    getInputStream->getInputStream0->resetProxyAuthentication->getHttpProxyAuthentication->privilegedRequestPasswordAuthentication->Authenticator.requestPasswordAuthentication

这两条调用链路，分别对应着我们上面为http请求和socket设置代理的情况。

我们运行两种情况的代码，在`resetProxyAuthentication`入口处打断点看看，有何不同。

通过上图显而易见，`resetProxyAuthentication`的第二参数`AuthenticationHeader`不同，主要在于一个是`prefer Basic realm="Restricted Files"`,一个是`prefer null`。

那是什么原因导致了这个不同？我们再在`AuthenticationHeader`的构造方法处打断点看看。

通过上图，发现AuthenticationHeader的构造方法参数的最后一个参数值不同。

在`HttpURLConnection`源码中，共有2处调用此构造方法。

    AuthenticationHeader authhdr = new AuthenticationHeader (
                            "Proxy-Authenticate",
                            responses,
                            new HttpCallerInfo(url,
                                               http.getProxyHostUsed(),
                                http.getProxyPortUsed()),
                            dontUseNegotiate,
                            disabledProxyingSchemes
                    );



    AuthenticationHeader authhdr = new AuthenticationHeader (
                            "Proxy-Authenticate",
                            responses,
                            new HttpCallerInfo(url,
                                               http.getProxyHostUsed(),
                                http.getProxyPortUsed()),
                            dontUseNegotiate,
                            disabledTunnelingSchemes
                    );

至此真相大白，`disabledProxyingSchemes` `disabledTunnelingSchemes`的不同导致了我们的问题。来看看这2个是什么。

原来，他们分别对应着两个jvm选项。

进一步探究，原来从jdk8u111开始，oracle把basic加入到了`jdk.http.auth.tunneling.disabledSchemes`中，也就是“Now, proxies requiring Basic authentication when setting up a tunnel for HTTPS will no longer succeed by default.”怎么解决？还是看文档“If required, this authentication scheme can be reactivated by removing Basic from the jdk.http.auth.tunneling.disabledSchemes networking property, or by setting a system property of the same name to "" ( empty ) on the command line.”

https://bugs.openjdk.java.net/browse/JDK-8210814

##总结
反常的问题看源码，总能找到你要的答案。这次认识了一下jvm的这个选项：jdk.http.auth.tunneling.disabledSchemes。