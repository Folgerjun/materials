---
title: SpringBoot2 实现邮件发送功能
date: 2019-06-04 16:53:42
categories: [开发,总结]
tags: [SpingBoot2,Java]
---

> springboot2 实现邮件发送功能，QQ/Gmail/163/126..
> 
> 个人博客：[DoubleFJ の Blog](http://putop.top/2019/06/04/springboot2-mail/)


效果图如下：![springboot 实现邮件发送](https://raw.githubusercontent.com/Folgerjun/materials/master/blog/img/spring-boot-mail.png)

### 技术选型
- **Spring Boot 2.1.3.RELEASE** （原本官网推荐 2.1.5.RELEASE，可是搭建途中发现部分注解未生效，故改之）
- **Thymeleaf** (用作邮件模板)
- **JDK 1.8**

### 简要讲解
#### 依赖以及配置
这里还是用的 Spring Boot 来整合，自带了模块依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```
随着 Spring Boot 热度越来越大，现在从事 Java 的要是不知道 Spring Boot 的存在那就真的很不应该了。本来只是为了取代繁琐的 EJB，一直发展到了如今无所不在的地步。

然后在配置文件中进行对应的配置
```
server:
  port: 8887

spring:
  mail:
    host: smtp.163.com
    username: ffj0721@163.com
    password: xxxx # 授权码
    protocol: smtp
    properties.mail.smtp.auth: true
    properties.mail.smtp.port: 994
    properties.mail.display.sendmail: DoubleFJ
    properties.mail.display.sendname: Spring Boot Email
    properties.mail.smtp.starttls.enable: true
    properties.mail.smtp.starttls.required: true
    properties.mail.smtp.ssl.enable: true
    default-encoding: utf-8
    from: ffj0721@163.com
```
一切就是这样的简单清晰。不同的邮件个别配置数据不同，请自行查阅，这里只用 163 做测试。

配置了之后我们开始操刀敲代码，其实只需要调用 JavaMailSender 接口即可，传参实现，已经给我们封装好了。

#### 常用邮件接口
这里是几个常用的邮件接口：
```
package com.example.springbootmail.service;
import javax.mail.MessagingException;

/**
 * 常用邮件接口
 */
public interface IMailService {
    /**
     * 发送文本邮件
     * @param to
     * @param subject
     * @param content
     */
    public void sendSimpleMail(String to, String subject, String content);

    public void sendSimpleMail(String to, String subject, String content, String... cc);

    /**
     * 发送HTML邮件
     * @param to
     * @param subject
     * @param content
     * @throws MessagingException
     */
    public void sendHtmlMail(String to, String subject, String content) throws MessagingException;

    public void sendHtmlMail(String to, String subject, String content, String... cc);

    /**
     * 发送带附件的邮件
     * @param to
     * @param subject
     * @param content
     * @param filePath
     * @throws MessagingException
     */
    public void sendAttachmentsMail(String to, String subject, String content, String filePath) throws MessagingException;

    public void sendAttachmentsMail(String to, String subject, String content, String filePath, String... cc);

    /**
     * 发送正文中有静态资源的邮件
     * @param to
     * @param subject
     * @param content
     * @param rscPath
     * @param rscId
     * @throws MessagingException
     */
    public void sendResourceMail(String to, String subject, String content, String rscPath, String rscId) throws MessagingException;

    public void sendResourceMail(String to, String subject, String content, String rscPath, String rscId, String... cc);

}
```

#### 接口实现类注入 JavaMailSender
实现类分别实现上述接口 注入 JavaMailSender
```
/**
 * 发送邮件实现类
 */
@Service
public class IMailServiceImpl implements IMailService {

    @Autowired
    private JavaMailSender mailSender;

    @Value("${spring.mail.from}")
    private String from;
```
其中 `from` 就是配置中我们配置的发送方，直接读取使用。

#### 实现发送文本邮件
```
   /**
     * 发送文本邮件
     *
     * @param to
     *          邮件接收方
     * @param subject
     *          邮件标题
     * @param content
     *          邮件内容
     */
    @Override
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        mailSender.send(message);
    }

    /**
     *
     * @param to
     *          邮件接收方
     * @param subject
     *          邮件标题
     * @param content
     *          邮件内容
     * @param cc
     *          抄送方
     */
    @Override
    public void sendSimpleMail(String to, String subject, String content, String... cc) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setCc(cc);
        message.setSubject(subject);
        message.setText(content);
        mailSender.send(message);
    }
```
如上所示，三个参数的，第一个是你要发邮件的对象，第二个是邮件标题，第三个是邮件发送的内容，第二个 cc 字符串数组就是需要抄送的对象。

#### 实现发送 HTML 邮件
```
/**
     * 发送 HTML 邮件
     *
     * @param to
     * @param subject
     * @param content
     */
    @Override
    public void sendHtmlMail(String to, String subject, String content) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();

        // true 表示需要创建一个multipart message
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        mailSender.send(message);
    }
```

官网 [MimeMessageHelper](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/mail/javamail/MimeMessageHelper.html) 使用介绍。

#### 实现发送带附件邮件
```
    /**
     * 发送带附件的邮件
     *
     * @param to
     * @param subject
     * @param content
     * @param filePath
     */
    public void sendAttachmentsMail(String to, String subject, String content, String filePath) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();

        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        FileSystemResource file = new FileSystemResource(new File(filePath));
        // 截取附件名
        String fileName = filePath.substring(filePath.lastIndexOf("/") + 1);
        helper.addAttachment(fileName, file);

        mailSender.send(message);
    }
```
这里附件名的截取要按照自己的实际需求来，有人喜欢用 `\\`，有人喜欢用 `/`，看具体情况具体分析了。

#### 实现发送正文中有静态资源邮件
```
    /**
     * 发送正文中有静态资源（图片）的邮件
     *
     * @param to
     * @param subject
     * @param content
     * @param rscPath
     * @param rscId
     */
    public void sendResourceMail(String to, String subject, String content, String rscPath, String rscId) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();

        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        FileSystemResource res = new FileSystemResource(new File(rscPath));
        helper.addInline(rscId, res);

        mailSender.send(message);
    }
```
其中 rscId 是资源的唯一 id，rscPath 就是对应资源的路径。具体待会我们看 Controller 调用方法。

#### 实现发送模板邮件
前面说了我们选择使用 `Thymeleaf` 作为邮件的模板，那就需要在 POM 文件中加入对应依赖。 
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
然后在 templates 文件夹下新建 mailTemplate.html 页面，我的内容如下：
```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>spring-boot-mail test</title>
    <style>
        body {
            text-align: center;
            margin-left: auto;
            margin-right: auto;
        }
        #welcome {
            text-align: center;
        }
    </style>
</head>
<body>
<div id="welcome">
    <h3>Welcome To My Friend!</h3>

    GitHub：
        <a href="#" th:href="@{${github_url}}" target="_bank">
            <strong>GitHub</strong>
        </a>
    <br />
    <br />
    个人博客：
        <a href="#" th:href="@{${blog_url}}" target="_bank">
            <strong>DoubleFJ の Blog</strong>
        </a>
    <br />
    <br />
    <img width="258px" height="258px"
         src="https://raw.githubusercontent.com/Folgerjun/materials/master/blog/img/WC-GZH.jpg">
    <br />微信公众号（诗词鉴赏）
</div>
</body>
</html>
```
这个模板自己 DIY 即可，不过对应填充参数不可弄错。例如我上面的是 `github_url` 和 `blog_url`。

[Thymeleaf 官网](https://www.thymeleaf.org/)

#### 调用实现

**MailController.java**
```
package com.example.springbootmail.controller;

import com.example.springbootmail.service.impl.IMailServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@RestController
@RequestMapping("/Mail")
public class MailController {

    private static final String SUCC_MAIL = "邮件发送成功！";
    private static final String FAIL_MAIL = "邮件发送失败！";

    // 图片路径
    private static final String IMG_PATH = "C:/Users/zjj/Desktop/github/materials/blog/img/WC-GZH.jpg";
    // 发送对象
    private static final String MAIL_TO = "folgerjun@gmail.com";

    @Autowired
    private IMailServiceImpl mailService;
    @Autowired
    private TemplateEngine templateEngine;

    @RequestMapping("/Email")
    public String index(){
        try {
            mailService.sendSimpleMail(MAIL_TO,"这是一封普通的邮件","这是一封普通的SpringBoot测试邮件");
        }catch (Exception ex){
            ex.printStackTrace();
            return FAIL_MAIL;
        }
        return SUCC_MAIL;
    }

    @RequestMapping("/htmlEmail")
    public String htmlEmail(){
        try {
            mailService.sendHtmlMail(MAIL_TO,"这是一HTML的邮件","<body>\n" +
                    "<div id=\"welcome\">\n" +
                    "    <h3>Welcome To My Friend!</h3>\n" +
                    "\n" +
                    "    GitHub：\n" +
                    "        <a href=\"#\" th:href=\"@{${github_url}}\" target=\"_bank\">\n" +
                    "            <strong>GitHub</strong>\n" +
                    "        </a>\n" +
                    "    <br />\n" +
                    "    <br />\n" +
                    "    个人博客：\n" +
                    "        <a href=\"#\" th:href=\"@{${blog_url}}\" target=\"_bank\">\n" +
                    "            <strong>DoubleFJ の Blog</strong>\n" +
                    "        </a>\n" +
                    "    <br />\n" +
                    "    <br />\n" +
                    "    <img width=\"258px\" height=\"258px\"\n" +
                    "         src=\"https://raw.githubusercontent.com/Folgerjun/materials/master/blog/img/WC-GZH.jpg\">\n" +
                    "    <br />微信公众号（诗词鉴赏）\n" +
                    "</div>\n" +
                    "</body>");
        }catch (Exception ex){
            ex.printStackTrace();
            return FAIL_MAIL;
        }
        return SUCC_MAIL;
    }

    @RequestMapping("/attachmentsMail")
    public String attachmentsMail(){
        try {
            mailService.sendAttachmentsMail(MAIL_TO, "这是一封带附件的邮件", "邮件中有附件，请注意查收！", IMG_PATH);
        }catch (Exception ex){
            ex.printStackTrace();
            return FAIL_MAIL;
        }
        return SUCC_MAIL;
    }

    @RequestMapping("/resourceMail")
    public String resourceMail(){
        try {
            String rscId = "DoubleFJ";
            String content = "<html><body>这是有图片的邮件<br/><img src=\'cid:" + rscId + "\' ></body></html>";
            mailService.sendResourceMail(MAIL_TO, "这邮件中含有图片", content, IMG_PATH, rscId);

        }catch (Exception ex){
            ex.printStackTrace();
            return FAIL_MAIL;
        }
        return SUCC_MAIL;
    }

    @RequestMapping("/templateMail")
    public String templateMail(){
        try {
            Context context = new Context();
            context.setVariable("github_url", "https://github.com/Folgerjun");
            context.setVariable("blog_url", "http://putop.top/");
            String emailContent = templateEngine.process("mailTemplate", context);

            mailService.sendHtmlMail(MAIL_TO, "这是模板邮件", emailContent);
        }catch (Exception ex){
            ex.printStackTrace();
            return FAIL_MAIL;
        }
        return SUCC_MAIL;
    }
}
```

程序运行后若访问 http://localhost:8887/Mail/Email 页面出现`邮件发送成功！`字样就说明邮件发送已经实现了。

### 相关链接

- [SpringBoot 升级到 2.1.5.RELEASE 后 pom.xml 报 Unknown错误](http://www.wangwenhui.com.cn/archives/232)
- [SpringBoot 2.x 集成QQ邮箱、网易系邮箱、Gmail邮箱发送邮件](https://blog.csdn.net/zyw_java/article/details/81635375)
- [Thymeleaf](https://www.thymeleaf.org/)
- [MimeMessageHelper](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/mail/javamail/MimeMessageHelper.html)