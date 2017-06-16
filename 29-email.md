# 29. 电子邮件

## 29.1 介绍

依赖库：使用Spring框架的邮件功能需要将[JavaMail](https://java.net/projects/javamail/pages/Home)的Jar包添加到依赖中。这个库可以Maven中心找到：[com.sun.mail:javax.mail](https://mvnrepository.com/artifact/com.sun.mail/javax.mail)。

Spring提供了一个实用的发送电子邮件库，它为使用者屏蔽了邮件系统的底层细节和客户端的底层资源处理。

Spring邮件相关功能在`org.springframework.mail`包下,其中`MailSender`是发送邮件的核心接口；`SimpleMailMessage`类是对邮件属性（发件人、收件人以等）进行简单的封装。这个包中也包含一系列的检查异常，它们是对邮件系统低级别的异常进行抽象,且均继承自`MailException`。有关异常的更多信息,请参阅相关javadoc。

`org.springframework.mail.javamail.JavaMailSender`接口继承自`MailSender`接口，并增加了一些特有的`JavaMail`功能，如_MIME_邮件的支持。`JavaMailSender`还提供了一个用于编写_MIME_消息的回调`org.springframework.mail.javamail.MimeMessagePreparator`接口。



## 29.2 使用

我们假设有一个`OrderManager`业务接口:

```
public interface OrderManager {

    void placeOrder(Order order);

}
```

假设我们需要生成一个有订单号的邮件，并发送给相关的客户。

### 29.2.1`MailSender`和`SimpleMailMessage`的基本用法

```
import org.springframework.mail.MailException;
import org.springframework.mail.MailSender;
import org.springframework.mail.SimpleMailMessage;

public class SimpleOrderManager implements OrderManager {

    private MailSender mailSender;
    private SimpleMailMessage templateMessage;

    public void setMailSender(MailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void setTemplateMessage(SimpleMailMessage templateMessage) {
        this.templateMessage = templateMessage;
    }

    public void placeOrder(Order order) {

        // Do the business calculations...

        // Call the collaborators to persist the order...

        // Create a thread safe &amp;amp;quot;copy&amp;amp;quot; of the template message and customize it
        SimpleMailMessage msg = new SimpleMailMessage(this.templateMessage);
        msg.setTo(order.getCustomer().getEmailAddress());
        msg.setText(
            &amp;amp;quot;Dear &amp;amp;quot; + order.getCustomer().getFirstName()
                + order.getCustomer().getLastName()
                + &amp;amp;quot;, thank you for placing order. Your order number is &amp;amp;quot;
                + order.getOrderNumber());
        try{
            this.mailSender.send(msg);
        }
        catch (MailException ex) {
            // simply log it and go on...
            System.err.println(ex.getMessage());
        }
    }

}
```

在xml中添加相关的Bean定义：

```
<bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
	<property name="host" value="mail.mycompany.com"/>
</bean>

<!-- this is a template message that we can pre-load with default state -->
<bean id="templateMessage" class="org.springframework.mail.SimpleMailMessage">
	<property name="from" value="customerservice@mycompany.com"/>
	<property name="subject" value="Your order"/>
</bean>

<bean id="orderManager" class="com.mycompany.businessapp.support.SimpleOrderManager">
	<property name="mailSender" ref="mailSender"/>
	<property name="templateMessage" ref="templateMessage"/>
</bean>
```

### 29.2.2 使用`JavaMailSender`和`MimeMessagePreparator`

下面的例子是`OrderManager`接口的另一种实现，其中使用了`MimeMessagePreparator`类。 在这里，`mailSender`是`JavaMailSender`类型，因此我们可以使用`JavaMail MimeMessage`类：

```
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

import javax.mail.internet.MimeMessage;
import org.springframework.mail.MailException;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessagePreparator;

public class SimpleOrderManager implements OrderManager {

    private JavaMailSender mailSender;

    public void setMailSender(JavaMailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void placeOrder(final Order order) {

        // Do the business calculations...

        // Call the collaborators to persist the order...

        MimeMessagePreparator preparator = new MimeMessagePreparator() {

            public void prepare(MimeMessage mimeMessage) throws Exception {

                mimeMessage.setRecipient(Message.RecipientType.TO,
                        new InternetAddress(order.getCustomer().getEmailAddress()));
                mimeMessage.setFrom(new InternetAddress(&amp;amp;quot;mail@mycompany.com&amp;amp;quot;));
                mimeMessage.setText(
                        &amp;amp;quot;Dear &amp;amp;quot; + order.getCustomer().getFirstName() + &amp;amp;quot; &amp;amp;quot;
                            + order.getCustomer().getLastName()
                            + &amp;amp;quot;, thank you for placing order. Your order number is &amp;amp;quot;
                            + order.getOrderNumber());
            }
        };

        try {
            this.mailSender.send(preparator);
        }
        catch (MailException ex) {
            // simply log it and go on...
            System.err.println(ex.getMessage());
        }
    }

}
```

> 上面中邮件代码只是作为示例，最好方式是将邮件发送代码重构到其它Bean中，并在OrderManager合适的地方调用它。

Spring框架邮件也支持标准的JavaMail实现。 了解更多信息，请参阅相关的javadocs。

## 29.2 使用MimeMessageHelper

`org.springframework.mail.javamail.MimeMessageHelper`是一个处理JavaMail消息的好工具，它屏蔽了很多JavaMail API的细节，所以使用`MimeMessageHelper`可以很简便的创建一个`MimeMessage`。

```
// of course you would use DI in any real-world cases
JavaMailSenderImpl sender = new JavaMailSenderImpl();
sender.setHost(&amp;amp;quot;mail.host.com&amp;amp;quot;);

MimeMessage message = sender.createMimeMessage();
MimeMessageHelper helper = new MimeMessageHelper(message);
helper.setTo(&amp;amp;quot;test@host.com&amp;amp;quot;);
helper.setText(&amp;amp;quot;Thank you for ordering!&amp;amp;quot;);

sender.send(message);
```

### 29.3.1 附件和嵌入资源

邮件允许添加附件和内联资源。嵌入资源是你嵌入到邮件中的图片或样式，但又不希望显示为附件。

**附件**

下面的例子将展示如何使用`MimeMessageHelper`发送一个带JPEG图片附件的邮件：

```
JavaMailSenderImpl sender = new JavaMailSenderImpl();
sender.setHost(&amp;amp;quot;mail.host.com&amp;amp;quot;);

MimeMessage message = sender.createMimeMessage();

// use the true flag to indicate you need a multipart message
MimeMessageHelper helper = new MimeMessageHelper(message, true);
helper.setTo(&amp;amp;quot;test@host.com&amp;amp;quot;);

helper.setText(&amp;amp;quot;Check out this image!&amp;amp;quot;);

// let's attach the infamous windows Sample file (this time copied to c:/)
FileSystemResource file = new FileSystemResource(new File(&amp;amp;quot;c:/Sample.jpg&amp;amp;quot;));
helper.addAttachment(&amp;amp;quot;CoolImage.jpg&amp;amp;quot;, file);

sender.send(message);
```

**嵌入资源**

下面的例子将展示如何使用`MimeMessageHelper`发送一个嵌入图片的邮件：

```
JavaMailSenderImpl sender = new JavaMailSenderImpl();
sender.setHost(&amp;amp;quot;mail.host.com&amp;amp;quot;);

MimeMessage message = sender.createMimeMessage();

// use the true flag to indicate you need a multipart message
MimeMessageHelper helper = new MimeMessageHelper(message, true);
helper.setTo(&amp;amp;quot;test@host.com&amp;amp;quot;);

// use the true flag to indicate the text included is HTML
helper.setText(&amp;amp;quot;&amp;amp;lt;html&amp;amp;gt;&amp;amp;lt;body&amp;amp;gt;&amp;amp;lt;img src='cid:identifier1234'&amp;amp;gt;&amp;amp;lt;/body&amp;amp;gt;&amp;amp;lt;/html&amp;amp;gt;&amp;amp;quot;, true);

// let's include the infamous windows Sample file (this time copied to c:/)
FileSystemResource res = new FileSystemResource(new File(&amp;amp;quot;c:/Sample.jpg&amp;amp;quot;));
helper.addInline(&amp;amp;quot;identifier1234&amp;amp;quot;, res);

sender.send(message);
```

> 嵌入资源需要使用Content-ID\(上面的例子identifier1234\)添加到MIME消息中。文本和嵌入资源添加是有顺序的，需要按照先添加文本，再添加嵌入资源的顺序。否则，它将不会工作!

### 29.3.2 使用模板库创建电子邮件内容

在前面的例子中，我们通常使用`message.setText(..)`等方法创建邮件内容。在简单的情况下，像前面例子那样使用API就可以满足我们的需要了。

在典型的企业应用程序中,下面的原因让你不定会使用上面的方法创建你的邮件内容。

* 在Java代码中创建HTML的电子邮件内容冗长，且容易出错
* 呈现逻辑和业务逻辑混杂
* 更改电子邮件内容的展示结构需要编写Java代码，重新编译，重新部署…

通常解决方法是使用模板框架定义电子邮件的呈现逻辑,如[FreeMarker](http://freemarker.org/)。分离呈现逻辑和业务逻辑使得你的代码更清晰。当你的邮件的内容变的复杂时，这绝对是一个最佳实践，而且Spring框架对[FreeMarker](http://freemarker.org/)有很好的支持。

