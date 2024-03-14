# 配置文件

<show-structure depth="3"/>

前面我们初探 SpringBoot 应用时，提到了 application.properties 这个配置文件，现在我们来看一下通过这个文件进行 SpringBoot 应用的配置。

> 至于前面提及的 pom.xml 文件，那是 SpringBoot 应用的依赖项配置文件，后续再研究。

## 1. application.properties

关于 application.properties 的配置项，我们可以参阅[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html)。该文档配置项比较多且比较详细，建议有时间的话全读一遍。

SpringBoot 服务启动的默认端口是 8080，下面我们修改一下端口号以及虚拟目录

```Python
server.port=9090
server.servlet.context-path=/start
```

重新启动服务后，我们需要通过 http://localhost:9090/start/hello 来访问应用了。


此外，我们也可以使用 application.yml 文件来进行配置，只不过将配置项写成 YAML 格式而已。实际使用中，使用 application.yml 更多，因为这种数据存储结构更清晰且易读，折叠也更方便。


## 2. YAML 文件配置信息以及读取

YAML 文件主要用于书写三方技术配置信息以及自定义配置信息，例如我们需要将 Redis 接口整合进 SpringBoot 应用中，可以这么做:
1. 添加依赖项

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    ```

2. 在 application.yml 文件中配置 Redis 连接信息

    ```yaml
    spring:
      data:
        redis:
          host: localhost
          port: 6379
          password: 123456
          connect-timeout: 3000
    ```

### 2.1 书写 YAML 配置信息

如果 SpringBoot 已经提供了相关组件的话，那整合组件就非常简单，但是有些第三方应用并没有相应的组件，例如我们想整合阿里图床到 SpringBoot 中，我们就得重新编写代码，一些账号、密码以及秘钥信息可能会被以 Hard Code 方式写进代码中。当然，我们可以将其写入单独的配置文件中，然后通过程序进行加载该配置文件，后续账号信息发生变化时，我们只需要更改配置文件就好，而不需要重新更新代码以及打包程序。

<tabs>
<tab title="阿里云图床">
<code-block lang="python">
<![CDATA[
class AliYunUploadUtil {
    public static void upload(String objectName, String content) {
        String endpoint,
        String accessKeyId,
        String accessKeySecret,
        String bucketName
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
        try {
            PutObjectRequest putObjectRequest = new PutObjectRequest(
                objectName, new ByteArrayInputStream(content.getBytes());
            PutObjectResult result = ossClient.putObject(putObjectRequest);
            System.out.println(result);
            );
        } catch (ClientException ce) {
            System.out.println(ce.getMessage());
        } finally {
            if (ossClient != null) {
                ossClient.shutdown();
            }
        }
    }
}
]]>
</code-block>
</tab>
<tab title="配置文件">
<code-block lang="yaml">
<![CDATA[
ali-yun:
    oss:
        endpoint: https://oss-cn-hangzhou.aliyuncs.com
        access-key-id: asd12ax12
        access-key-secret: zmlk123
        bucket-name: gcf
]]>
</code-block>
</tab>
</tabs>

### 2.2 发送邮件案例

接下来我们再以一个发送邮件的实际案例，演示喜下置文件的编写以及读取。


<tabs>
<tab title="pom.xml">
<code-block lang="xml">
<![CDATA[
<!-- 引入发送邮件组件 -->
<dependency>
    <groupId>org.eclipse.angus</groupId>
    <artifactId>jakarta.mail</artifactId>
</dependency>
]]>
</code-block>
</tab>
<tab title="MathUtil.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootconfigfile.utils;
import com.aifun.springbootconfigfile.pojo.EmailProperties;
import jakarta.mail.*;
import jakarta.mail.internet.InternetAddress;
import jakarta.mail.internet.MimeMessage;
import java.util.Properties;

public class MailUtil {
    /**
     * 发送邮件
     * @param emailProperties 发件人信息(发件人邮箱、授权码)以及邮件服务器信息
     * @param to 收件人邮箱
     * @param title 邮件标题
     * @param content 邮件正文
     * @return
     */
    public static boolean sendMail(EmailProperties emailProperties, String to, String title String content) {
        // ...
    }
}
]]>
</code-block>
</tab>
<tab title="EmailService.java">
<code-block lang="java">
package com.aifun.springbootconfigfile.service;
public interface EmailService {
    boolean send(String to, String title, String content);
}
</code-block>
</tab>
<tab title="EmailServiceImpl.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootconfigfile.service.impl;
import com.aifun.springbootconfigfile.service.EmailService;
import com.aifun.springbootconfigfile.utils.MailUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class EmailServiceImpl implements EmailService {
    // 注入 Email 配置信息实体类
    @Autowired
    private EmailProperties emailProperties;
    public boolean send(String to, String title, String content) {
        System.out.println(emailProperties);
        boolean sendStatus = Mailutil.sendMail(emailProperties, to, title, content);
        return sendStatus;
    }
}
]]>
</code-block>
</tab>
<tab title="EmailProperties.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootconfigfile.pojo;

import org.springframework.stereotype.Component;

@Component
public class EmailProperties {
    // 发件人邮箱
    public String user = "1234@qq.com";
    // 发件人邮箱授权
    public String code = "abc";
    // 邮箱服务器域名
    public String host = "smtp.qq.com";
    // 身份验证
    private boolean auth = true;
    public String getHost() { return host; }
    public void setHost(String host) { this.host = host; }
    public boolean isAuth() { return auth; }
    public void setAuth(boolean auth) { this.auth = auth; }
    public String getUser() { return user; }
    public void setUser(String user) { this.user = user; }
    public String getCode() { return code; }
    public void setCode(String code) { this.code = code; }
}
]]>
</code-block>
</tab>

<tab title="EmailController.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootconfigfile.controller;
import com.aifun.springbootconfigfile.utils.MailUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EmailController {
    @Autowired
    private EmailService emailService;

    @RequestMapping("/send")
    public Boolean send() {
       // 注入 email 配置信息实体类
        String to = "1234@qq.com";
        String title = "测试邮件";
        String content = "这是正文信息...";
        boolean flag = emailService.send(to, title, content);
        return flag; 
    }
}
]]>
</code-block>
</tab>
</tabs>


上面示例中，我们将邮件信息已经写死在 EmailController.java 中了，但实际开发时，这个正确操作应该是从配置文件中读取邮件信息，并在进行读取。

<tabs>
<tab title="application.properties">
<code-block lang="python">
<![CDATA[
email.user=1234@qq.com
email.code=a1b2c3
email.host=smtp.qq.com
emai.auth=true
]]>
</code-block>
</tab>
<tab title="application.yml">
<code-block lang="python">
<![CDATA[
email:
    user: 1234@qq.com
    code: a1b2c3
    host: smtp.qq.com
    auth: true
]]>
</code-block>
</tab>
</tabs>


我们使用上面 application.yml 来存储邮件信息，删除 EmailProperties.java 中相关邮件信息。在 SpringBoot 可以通过 `@Value` 注解来获取配置文件中的值，也可以通过 `@ConfigurationProperties` 注解完成。

<tabs>
<tab title="EmailProperties.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootconfigfile.pojo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class EmailProperties {
    // 使用 `@Value("${键名}")` 来获取配置文件对应的值

    @Value("${email.user}")
    public String user;

    @Value("${email.code}")
    public String code;

    @Value("${email.host}")
    public String host;

    @Value("${email.auth}")
    private boolean auth = true;

    public String getHost() { return host; }
    public void setHost(String host) { this.host = host; }
    public boolean isAuth() { return auth; }
    public void setAuth(boolean auth) { this.auth = auth; }
    public String getUser() { return user; }
    public void setUser(String user) { this.user = user; }
    public String getCode() { return code; }
    public void setCode(String code) { this.code = code; }
}
]]>
</code-block>
</tab>
<tab title="@ConfigurationProperties">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootconfigfile.pojo;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix="email")
public class EmailProperties {
    public String user;
    public String code;
    public String host;
    private boolean auth = true;

    public String getHost() { return host; }
    public void setHost(String host) { this.host = host; }
    public boolean isAuth() { return auth; }
    public void setAuth(boolean auth) { this.auth = auth; }
    public String getUser() { return user; }
    public void setUser(String user) { this.user = user; }
    public String getCode() { return code; }
    public void setCode(String code) { this.code = code; }
}
]]>
</code-block>
</tab>
</tabs>


> 1. 当配置变量比较多时，推荐使用 `@ConfigurationProperties` 注解
> 2. 使用 `@ConfigurationProperties` 时需要注意 `prefix` 以及变量名需要与配置文件中完全能对应上
> 
{style="warning"}


> 这里提出一个问题，就是我们将配置信息都是放在 application.yml 或者 application.properties 文件中的，这两个文件实际上都是被 SpringBoot 应用自动读取并加载的，但是如果文件名为 application.yml 的情况下，我们如何指定所加载文件？

