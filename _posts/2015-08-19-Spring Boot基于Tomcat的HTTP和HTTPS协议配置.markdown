---
layout:     post
title:      "Spring Boot基于Tomcat的HTTP和HTTPS协议配置"
date:       2015-08-19 20:51
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - SSL
    - HTTPS
    - Spring Boot
---

按照官方的说法，Spring Boot无法使用配置同时支持http和https两种协议访问应用。替代方案是可以配置一种协议，然后另外一种协议的支持，通过编码来实现。推荐的方案是使用配置支持https，而写代码支持http，原因是编码写http会比较复杂一些。官方的samples示例中演示了如果通过代码支持https（也就是官方认为比较复杂的那种情况。）

下面我们来使用另外一种实现方案，就是通过配置支持https，通过编码支持http。

# 1. 生成并安装证书

首先生成SSL证书，这里选择P12格式。

	keytool -genkey -alias xgsdk -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 3650
	
按提示输入各项之后，可以得到一个keystore.p12文件。可以通过以下命令验证其中的信息。

	keytool -list -v -keystore keystore.p12 -storetype pkcs12
	
如果ssl链接仅通过浏览器访问，可以跳过下面这步，如果需要通过客户端访问，可以从浏览器中导出cer或者crt格式的公钥，然后通过以下命令添加证书到信任列表。

	keytool -import -alias xgsdk -keystore $JAVA_HOME/jre/lib/security/cacerts -file xgsdk.com.crt
	
# 2. 配置SSL支持

将keystore.p12放到工程根目录下，并在application.properties文件配置即可：

	server.ssl.key-store = keystore.p12
	server.ssl.key-store-password = your_password
	server.ssl.keyStoreType = PKCS12
	server.ssl.keyAlias = xgsdk
	
还可以额外定义端口，例如到8443：

	server.port=8443
	
此时服务器已经打开8443端口，需要用https访问。访问http端口会发现已经不通了。

# 3. 配置HTTP支持

接下来通过编码的方式支持HTTP协议访问，可以在application.java入口中加入以下代码创建一个新德Connector：

	@Value("${xgsdk.http.port}")
    private int httpPort;
    
	@Bean
    public EmbeddedServletContainerFactory servletContainer() {
        TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
        tomcat.addAdditionalTomcatConnectors(createHttpConnector());
        return tomcat;
    }
    
    private Connector createHttpConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(httpPort);
        connector.setSecure(false);
        return connector;
    }
    
另外在application.properties文件配置http端口即可：

	xgsdk.http.port=8090
	
此时系统已经完全支持http和https同时访问。

# 4. 配置HTTP自动跳转到HTTPS

我们希望用户访问http端口的时候会自动跳转到https协议来访问，可以在创建connect的时候进行端口重定向。（当然还有其他方式，比如可以在应用中通过filter进行重定向处理）。

	@Bean
    public EmbeddedServletContainerFactory servletContainer() {
        TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory() {
        	  SecurityConstraint securityConstraint = new SecurityConstraint();
            securityConstraint.setUserConstraint("CONFIDENTIAL");
            SecurityCollection collection = new SecurityCollection();
            collection.addPattern("/");
            securityConstraint.addCollection(collection);
            context.addConstraint(securityConstraint);
        };
        tomcat.addAdditionalTomcatConnectors(createHttpConnector());
        return tomcat;
    }
    
    private Connector createHttpConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(httpPort);
        connector.setSecure(false);
        connector.setRedirectPort(httpsPort);
        return connector;
    }
    
# 5. 配置部分链接允许http访问

这个需求源自一些静态资源，使用http协议访问可以获得更高效率。这里需要再增加一个SecurityConstraint对象进行处理，我们先设置一些url-pattern，然后将这些pattern加入到NONE策略的SecurityConstraint中，以便允许这部分链接通过http访问。而剩下的仍然走CONFIDENTIAL策略。

	private static final String HTTP_URL_PATTERNS[] = {
            "/static/*", 
            "/download/*", 
            "/doc/*"
            };
	@Bean
    public EmbeddedServletContainerFactory servletContainer() {
        TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("NONE");
                SecurityCollection collection = new SecurityCollection();
                for (String pattern : HTTP_URL_PATTERNS) {
                    collection.addPattern(pattern);
                }
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
                
                SecurityConstraint securityConstraint2 = new SecurityConstraint();
                securityConstraint2.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection2 = new SecurityCollection();
                collection2.addPattern("/");
                securityConstraint2.addCollection(collection2);
                context.addConstraint(securityConstraint2);
                
                LOGGER.info("Constraints length = " + context.findConstraints().length);
            }
        };
        tomcat.addAdditionalTomcatConnectors(createHttpConnector());
        return tomcat;
    }
    
    private Connector createHttpConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(httpPort);
        connector.setSecure(false);
        connector.setRedirectPort(httpsPort);
        return connector;
    }