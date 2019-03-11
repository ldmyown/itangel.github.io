---
layout: post
title: springboot(五)：Spring boot中邮件服务使用
category: springboot
tags: [springboot]
---

Springboot中集成了邮件的发送，这里介绍一下如何实现邮件发送。

## 说明

​	在目前互联网的世界中，邮件的使用基本成为一个网站必备的功能，譬如基本的用户注册验证，忘记密码等，在springboot之前，JavaMail相关的API提供了邮件发送，发送的流程相对复杂。

​	在Spring中，推出了**JavaMailSender** ，对邮件的发送流程做了简化了，Springboot 出来后对此进行了封装，出现了现在的**spring-boot-starter-mail** ，使得邮件的发送流程更加的简单，我们只需要做简单的配置，就能实现邮件的发送，非常方便。

​	下面将介绍一下如何使用Springboot实现邮件的发送。



## 项目搭建

### 创建一个简单的springboot项目

在pom文件中引入邮件**spring-boot-starter-mail**的依赖

~~~xml
<dependencies>
	<dependency> 
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-mail</artifactId>
	</dependency> 
</dependencies>
~~~

### 配置email参数

在application.properties中添加邮件的相关配置

~~~properties
spring.mail.host=smtp.qiye.163.com //邮箱服务器地址
spring.mail.username=xxx@oo.com //用户名
spring.mail.password=xxyyooo    //密码
spring.mail.default-encoding=UTF-8

mail.fromMail.addr=xxx@oo.com  //以谁来发送邮件
~~~

### 简单邮件发送

编写service服务类

~~~java
@Service
public class MailServiceImpl implements MailService{

    private final Logger logger = LoggerFactory.getLogger(MailServiceImpl.class);

    @Autowired
    private JavaMailSender mailSender;

    @Value("${mail.fromMail.addr}")
    private String from;

    @Override
    public void sendSimpleMail(String toUser, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(toUser);
        message.setSubject(subject);
        message.setText(content);
        try {
            mailSender.send(message);
            logger.info("用户{} 发送给{} 的简单邮件发送成功", from, toUser);
        } catch (MailException e) {
            logger.error("用户{} 发送给{} 的简单邮件发送失败", from, toUser, e);
        }

    }


}
~~~



### 发送html格式的邮件

发送html格式的邮件，需要使用****MimeMessageHelper**，在**MailService**中添加sendHtmlMail

~~~java
@Override
    public void sendHtmlMail(String toUser, String subject, String content) {
        MimeMessage message = mailSender.createMimeMessage();
        try {
            //true表示需要创建一个multipart message
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            helper.setTo(toUser);
            helper.setSubject(subject);
          	// 此处参数为true表示邮件为html格式的，为false表示普通的文本
            helper.setText(content, true);
            mailSender.send(message);
            logger.info("用户{} 发送给{} 的html邮件发送成功", from, toUser);
        } catch (MailException e) {
            logger.error("用户{} 发送给{} 的html邮件发送失败", from, toUser, e);
        } catch (MessagingException e) {
            logger.error("用户{} 发送给{} 的html邮件发送失败", from, toUser, e);
        }

    }
~~~



### 发送带附件的邮件

~~~java
@Override
    public void sendAttachmentsMail(String toUser, String subject, String content, String filePath) {
        MimeMessage message = mailSender.createMimeMessage();
        try {
            //true表示需要创建一个multipart message
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            helper.setTo(toUser);
            helper.setSubject(subject);
            // 此处参数为true表示邮件为html格式的，为false表示普通的文本
            helper.setText(content, true);

            // 添加附件
            FileSystemResource file = new FileSystemResource(new File(filePath));
            String fileName = file.getFile().getName();
            helper.addAttachment(fileName, file);

            mailSender.send(message);
            logger.info("用户{} 发送给{} 的带附件邮件发送成功", from, toUser);
        } catch (MailException e) {
            logger.error("用户{} 发送给{} 的带附件邮件发送失败", from, toUser, e);
        } catch (MessagingException e) {
            logger.error("用户{} 发送给{} 的带附件邮件发送失败", from, toUser, e);
        }
    }
~~~

> 添加多个附件，可以使用多条 ```helper.addAttachment(fileName, file)```来实现。

### 发送带静态资源的邮件

这里的静态资源一般指的是图片资源

~~~java
 @Override
    public void sendInlineResourceMail(String toUser, String subject, String content, String rscPath, String rscId) {
        MimeMessage message = mailSender.createMimeMessage();

        try {
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            helper.setTo(toUser);
            helper.setSubject(subject);
            helper.setText(content, true);

            FileSystemResource res = new FileSystemResource(new File(rscPath));
            helper.addInline(rscId, res);

            mailSender.send(message);
            logger.info("用户{} 发送给{} 的嵌入静态资源的邮件已经发送。");
        } catch (MessagingException e) {
            logger.error("用户{} 发送给{} 的发送嵌入静态资源的邮件时发生异常！", e);
        }
    }
~~~

> 如果需要添加多个静态资源，可以多次调用```helper.addInline(rscId, res)```方法来实现



### 编写测试用例测试邮件发送

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MailServiceImplTest {

    @Autowired
    private MailService mailService;

    @Test
    public void sendSimpleMail() throws Exception {
        mailService.sendSimpleMail("lid@koal.com", "测试邮件", "this is just a test");
    }

    @Test
    public void sendHtmlMail() throws Exception {
        String content = "<html>\n" +
                "<body>\n" +
                "    <h3>测试html邮件， 这是一封Html邮件!</h3>\n" +
                "</body>\n" +
                "</html>";

        mailService.sendHtmlMail("lid@koal.com", "测试邮件", content);
    }

    @Test
    public void sendAttachmentsMail() throws Exception {
        String content = "<html>\n" +
                "<body>\n" +
                "    <h3>测试html邮件， 这是一封Html邮件!</h3>\n" +
                "</body>\n" +
                "</html>";

        String file = "D:\\ssr.md";
        mailService.sendAttachmentsMail("lid@koal.com", "测试邮件", content, file);
    }
}
~~~



### 发送失败

在实际使用中，有很多原因会导致发送失败，譬如：发送过于频繁，网络异常等等。这里我们可以考虑将邮件入库，根据发送返回的信息来记录右键是否发送成功，然后启动定时任务扫描时间段内的邮件，将失败的邮件重新发送，可以设置重试次数，以免出现无限制发送。



### 异步发送

有些邮件，不需要实时发送，比如通知类或者提醒类的邮件，这时候，我们可以采用异步发送的形式来实现邮件发送，加快主业务的执行速度，在实际项目中，我们可以采用MQ发送邮件，监听到消息队列后启动邮件发送。




**[示例代码-github](**https://github.com/ldmyown/springboot-learning**)**

-------------

**作者：telAngel**  
**出处：[https://ldmyown.github.io](https://ldmyown.github.io)**      
**版权所有，欢迎保留原文链接进行转载：)**