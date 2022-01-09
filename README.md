**It works well in Tomcat `7.x`, and breaks from Tomcat `8.0.x`.**

Related bug: https://bugs.eclipse.org/bugs/show_bug.cgi?id=493277

## Reproduce steps in Eclipse

1. Import the project as the Gradle project in Eclipse for Java EE.
1. Add the tomcat server and add the `app` module to the configured section.

   <img src="https://user-images.githubusercontent.com/3297602/148646558-769360e5-9d2a-4aa9-93d0-cf0096c783f1.png" width="436">
1. Double-click the tomcat server and change the `Server Options` as below, save the configuration.

   <img src="https://user-images.githubusercontent.com/3297602/148646716-b3626670-aa88-4f1c-8432-988172dee446.png" width="433">
   
   **It is requried to check `Serve modules without publishing`**
1. Start the tomcat server.
1. Visit http://localhost:8080/wtp-bug and error occurrs here.

## Troubleshooting

It throws an exception:

```
org.apache.jasper.JasperException: The absolute uri: [http://www.springframework.org/tags/form] cannot be resolved in either web.xml or the jar files deployed with this application
```

It complains that the server couldn't find the tld in web.xml or the jar files. While, actually, the tld the server couldn't find locates the `spring-webmvc-5.3.14.jar!/META-INF/spring-form.tld`.

The reason why the server couldn't find it is the jar files are mounted to the `/WEB-INF/classes` folder in the tomcat context XML (see below),  
which means the `META-INF` folder inside the jar files are ignored.

```xml
<Context docBase="C:\Users\admin\Desktop\work\eclipse-test\wtp-tomcat-jar-resource-bug\app\src\main\webapp" path="/wtp-bug" reloadable="true" source="org.eclipse.jst.j2ee.server:app">
    <Resources>
        <PreResources base="C:\Users\admin\Desktop\work\eclipse-test\wtp-tomcat-jar-resource-bug\app\bin\main" classLoaderOnly="false" className="org.apache.catalina.webresources.DirResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
        <JarResources base="C:\Users\admin\.gradle\caches\modules-2\files-2.1\org.springframework\spring-webmvc\5.3.14\beb6dc57abf6685878b824d8ab0af39ebd1cfbae\spring-webmvc-5.3.14.jar" classLoaderOnly="true" className="org.apache.catalina.webresources.JarResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
        <JarResources base="C:\Users\admin\.gradle\caches\modules-2\files-2.1\org.springframework\spring-context\5.3.14\ce6042492f042131f602bdc83fcb412b142bdac5\spring-context-5.3.14.jar" classLoaderOnly="true" className="org.apache.catalina.webresources.JarResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
        <JarResources base="C:\Users\admin\.gradle\caches\modules-2\files-2.1\org.springframework\spring-aop\5.3.14\f049146a55991e89c0f04b9624f1f69e1763d80f\spring-aop-5.3.14.jar" classLoaderOnly="true" className="org.apache.catalina.webresources.JarResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
        <JarResources base="C:\Users\admin\.gradle\caches\modules-2\files-2.1\org.springframework\spring-web\5.3.14\801d96f3914ace2e347ee3f6d29e21073e4f50ed\spring-web-5.3.14.jar" classLoaderOnly="true" className="org.apache.catalina.webresources.JarResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
        <JarResources base="C:\Users\admin\.gradle\caches\modules-2\files-2.1\org.springframework\spring-beans\5.3.14\24cc27af89edc1581a57bb15bc160d2353f40a0e\spring-beans-5.3.14.jar" classLoaderOnly="true" className="org.apache.catalina.webresources.JarResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
        <JarResources base="C:\Users\admin\.gradle\caches\modules-2\files-2.1\org.springframework\spring-expression\5.3.14\5cd4c568522b7084afac5d2ac6cb945b797b3f16\spring-expression-5.3.14.jar" classLoaderOnly="true" className="org.apache.catalina.webresources.JarResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
        <JarResources base="C:\Users\admin\.gradle\caches\modules-2\files-2.1\org.springframework\spring-core\5.3.14\d87ad19f9d8b9a3f1a143db5a2be34c61751aaa2\spring-core-5.3.14.jar" classLoaderOnly="true" className="org.apache.catalina.webresources.JarResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
        <JarResources base="C:\Users\admin\.gradle\caches\modules-2\files-2.1\org.springframework\spring-jcl\5.3.14\ffcf745ed5ba32930771378316fd08e97986bec2\spring-jcl-5.3.14.jar" classLoaderOnly="true" className="org.apache.catalina.webresources.JarResourceSet" internalPath="/" webAppMount="/WEB-INF/classes"/>
    </Resources>
</Context>
```

There is a similar issue reported to Tomcat years ago. https://bz.apache.org/bugzilla/show_bug.cgi?id=57791. The solution provided by this ticket is to configure the jar files as the `org.apache.catalina.webresources.FileResourceSet` and mount it to `/WEB-INF/lib/xxx.jar`.

The related code in Eclipse server plugin could be https://git.eclipse.org/c/gerrit/servertools/webtools.servertools.git/tree/plugins/org.eclipse.jst.server.tomcat.core/tomcatcore/org/eclipse/jst/server/tomcat/core/internal/Tomcat90PublishModuleVisitor.java#n97
