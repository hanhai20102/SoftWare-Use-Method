# Java实现邮件和短信的发送

## 1、邮件发送

邮件功能需要的Jar包：javax.mail-1.5.6.jar 、javax.mail-api-1.5.6.jar
>具体过程：
>1、设置邮件服务器的属性使用**Properties**类 ，使用**smtp**协议，邮件服务器为163邮件服务器 
>2、创建一个程序与邮件服务器会话的对象即**Session**，使用Properties的实例新建一个Session对象 
>3、创建一封邮件使用**MimeMessage**类 
>4、创建发送邮件的对象**Transport**，使用sendMeaage发送邮件
```java
import java.io.UnsupportedEncodingException;
import java.util.Date;
import java.util.Properties;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
public class EmailSender {
	private String senderAccount = "whujwb2018@163.com"; //发件人账号
	private String senderPassword = "lianglab601";  //发件人的密码
	private String smtpHost = "smtp.163.com"; //使用的是163邮箱服务器
	public void sendMail(String receive, String title, String content)
			throws UnsupportedEncodingException, MessagingException {
		Properties properties = new Properties();
		properties.setProperty("mail.transport.protocol", "smtp");
		properties.setProperty("mail.smtp.host", smtpHost);
		properties.setProperty("mail.smtp.auth", "true");
		Session session = Session.getInstance(properties);
		session.setDebug(true);
		MimeMessage message = new MimeMessage(session);
		message.setFrom(new InternetAddress(senderAccount));
		message.setRecipient(Message.RecipientType.TO, new InternetAddress(
				receive));
		// 抄送给自己一份
		message.addRecipients(Message.RecipientType.CC, senderAccount);
		message.setSubject(title, "utf-8");
		message.setContent(content, "text/html;charset=UTF-8");
		message.setSentDate(new Date());
		message.saveChanges();
		Transport transport = session.getTransport();
		transport.connect(senderAccount, senderPassword);
		transport.sendMessage(message, message.getAllRecipients());
		transport.close();
	}
}
```

---

## 2、使用邮箱验证码验证信息
使用邮箱验证时，一般向服务器发送两个请求：
第一个是发送验证码的请求，发送之后把验证码以及需要验证的账号或者地址保存在Session对象中；
第二个是验证验证码，去除Session对象中保存的属性，与用户填写的进行对照，如果成功可以进行入库或者其他操作。

---

## 3、短信发送
使用手机发送验证码进行验证时，跟邮箱一样，也是向服务器端发送两个请求：
第一个是发送验证码的请求，发送之后把验证码以及需要验证的账号或者地址保存在Session对象中；发送短信验证码需要**短信服务商**接口，教务系统使用了共享打印的短信接口
第二个是验证验证码，去除Session对象中保存的属性，与用户填写的进行对照，如果成功可以进行入库或者其他操作。
发送短信函数调用如下：
```java
public void send(String phoneNums, String message, String timing)
			throws Exception {
		String url = "http://47.95.33.177:9002/df_httpserver/smsSend.do";// 应用地址
		Map<String, String> map = new HashMap<String, String>();
		map.put("username", "zdbj00080");// 用户名
		map.put("pwd", MD5Maker.getMD5("xk6vs4uasxba"));// 密码
		map.put("mobile", phoneNums);// 手机号，可以写多个,隔开
		map.put("content", URLEncoder.encode(message, "utf8"));// 短信内容
		map.put("ext", "");// 扩展 如果需要
		map.put("msgfmt", "");// 编码类型,如果不填默认为GBK，可以选填GBK或utf8
		map.put("attime", timing);// 定时时间，例：2008-06-09 12:00:00
		map.put("msgid", "");// 唯一标记，如果客户自行传入了msgid，则用客户的
		try {
			String returnString = HttpSendMessage.post(url, map);
			System.out.println(returnString);
			// TODO 处理返回值,参见绿信通协议文档
		} catch (Exception e) {
			// TODO 处理异常
			e.printStackTrace();
		}
	}
```
发送短信的接口如下：
```java
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Set
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

public class HttpSendMessage {
	/**
	 * 
	 * @param url
	 *            应用地址，类似于http://ip:port/df_httpserver/smsSend.do
	 * @param map
	 *            参数列表
	 * @return 返回值定义参见协议文档
	 * @throws Exception
	 */
	public static String post(String url, Map<String, String> map) {
		CloseableHttpClient httpclient = HttpClients.createDefault();
		HttpPost httppost = new HttpPost(url);
		List<NameValuePair> params = new ArrayList<NameValuePair>();
		Set<String> set = map.keySet();
		Iterator<String> it = set.iterator();
		while (it.hasNext()) {
			String name = it.next();
			params.add(new BasicNameValuePair(name, map.get(name)));
		}
		try {
			httppost.setEntity(new UrlEncodedFormEntity(params, Charset
					.forName("UTF8")));
			HttpResponse response = httpclient.execute(httppost);
			int code = response.getStatusLine().getStatusCode();
			if (code == 200) {
				HttpEntity entity = response.getEntity();
				String string = EntityUtils.toString(entity, "utf-8");
				return string;
			} else {
				System.out.println("调用http接口响应状态异常，状态码：" + code);
				return null;
			}
		} catch (UnsupportedEncodingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ClientProtocolException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			try {
				httpclient.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		return null;
	}
}


```

---

## 4、消息队列 rabbitMQ

使用消息队列发送短信或者邮箱时需要引入的Jar包：rabbitmq的jar包（可以再maven repository）中下载。
使用rabbitMQ分为**生产者**，**消息队列**，与**消费者**   在*发送短信和邮件*时我们只需要作为一个生产者把需要发送的消息的内容通过生产者客户端发送给消息队列服务器端，让服务器端代理处理需要发送的消息，**消费者就是从消息队列中获取消息的一方**，消费者端的代码需要我们自己编写，当然也就是接受消息队列服务器端的信息，然后再消费者端进行邮件和短信的发送。
> [参考文章](https://blog.csdn.net/lom9357bye/article/details/70133744)
> 
>使用rabbitMQ的过程：
>1、使用Connection类建立一个TCP链接 **消费者**和**生产者**通过**TCP连接**到**RabbitMQ server**  而Connection事例是通过ConnectionFactory类创建的 *Connection conn = factory.newConnection()*
>2、Channel类 channel建立在Conection之上，因为频繁建立关闭TCP连接会影响到性能，因此使用Channel发送、接收消息  *channel = conn.createChannel()*
>3、通过channel的queueDeclare()方法可以指定要发送消息/接收消息的队列（函数的参数代码实例中有）
>4、通过channel的basicPublish()方法可以向队列中发送消息  
>basicPublish（）的参数意义:exchange：转发器，可以传入空""，""是默认的**转发器**;routingKey:路由键，根据路由键转发器决定将消息发送到哪个消费者队列;body:消息的字节形式
>5、Exchange 转发器  生产者发送的消息一般不会直接交给消费者队列，而是用转发器作为中转，生产者首先将消息发送到转发器，转发器根据路由规则决定将消息发送到哪个队列。转发器类型有direct、topic、fanout和headers.

PS：生产者端的代码示例：
```java
import java.io.IOException;
import java.util.concurrent.TimeoutException;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;
public class MQSender {
	private static final String QUE_NAME = "hello";
	private static final String HOST = "10.4.101.168";
	private static final String USERNAME = "mqadmin";
	private static final String PASSWORD = "123456";
	/**
	 * 通过消息队列发送消息
	 * @param message(Json字符串的格式)
	 */
	public static void sendMessage(String message) {
		ConnectionFactory factory = new ConnectionFactory();
		factory.setHost(HOST);
		factory.setUsername(USERNAME);
		factory.setPassword(PASSWORD);
		Connection conn = null;
		Channel channel = null;
		try {
			conn = factory.newConnection();
			channel = conn.createChannel();
			/**（为queueDeclare的四个参数的意义）
			 * queueName, durable:持久化
			 * exclusive:排他队列，如果一个队列被声明为排他队列，该队列仅对首次申明它的连接可见，并在连接断开时自动删除。
			 * authodelte:自动删除，如果该队列没有任何订阅的消费者的话，该队列会被自动删除。这种队列适用于临时队列。
			 */
			channel.queueDeclare(QUE_NAME, true, false, false, null);
			channel.basicPublish("", QUE_NAME,
					MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (TimeoutException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			try {
				channel.close();
				conn.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (TimeoutException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}
```
PS：消费者端的代码示例：
```java
public class Receiver {  
    //队列名称    
    private final static String QUEUE_NAME = "hello";  
    public static void main(String[] argv) throws java.io.IOException, java.lang.InterruptedException {  
        //1.创建一个ConnectionFactory连接工厂connectionFactory  
        ConnectionFactory connectionFactory = new ConnectionFactory();  
        //2.通过connectionFactory设置RabbitMQ所在IP等信息  
        connectionFactory.setHost("localhost");  
        //3.通过connectionFactory创建一个连接connection  
        Connection connection = connectionFactory.newConnection();  
        //4.通过connection创建一个频道channel  
        Channel channel = connection.createChannel();  
        //5.通过channel指定队列  
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);  
        //与发送消息不同的地方  
        //6.创建一个消费者队列consumer,并指定channel    
        QueueingConsumer consumer = new QueueingConsumer(channel);  
        //7.为channel指定消费者  
        channel.basicConsume(QUEUE_NAME, true, consumer);  
        while (true) {  
            //从consumer中获取队列中的消息,nextDelivery是一个阻塞方法,如果队列中无内容,则等待  
            Delivery delivery = consumer.nextDelivery();  
            String message = new String(delivery.getBody());  
            System.out.println("接收到了" + QUEUE_NAME + "中的消息:" + message);  
        }    
    }  
}  

```

---

## 5、验证码的生成
### （1）邮箱验证码（一般包含字母和数字6位）
```java
public static String generateRandomCodeWithChar() {
		String baseString = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
		String result = "";
		for (int i = 0; i < 6; i++) {
			result += baseString.charAt((int) (Math.random() * baseString
					.length()));
		}
		return result;
}
```
### （2）手机验证码 （一般是数字六位）
```java
public static String getRandomCode() {
		String result = "";
		for (int i = 0; i < 6; i++) {
			result += (int) (Math.random() * 10);
		}
		return result;
}
```
---

## 6、验证码超时机制

   在发送完验证码是，使用*System.currentTimeMillis()*获取系统当前时间的毫秒数保存在*Session*对象中，在验证阶段再次使用*System.currentTimeMillis()*减去*Session*对象中的保存的时间来确认邮件的验证码是否超过五分钟或者短信的验证码是否超过60s。

![Muyuxi](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1525164414579&di=cab4be5b3297037a91f84f4ac74b6a29&imgtype=0&src=http%3A%2F%2Fs22.mogucdn.com%2Fp1%2F160402%2F250zou_ifqtcndchbrwiylbg4zdambqgyyde_640x516.jpg))