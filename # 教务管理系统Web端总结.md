# 教务管理系统Web端总结

## 一、Login

###  关于request.getHead 的说明
参考文章       [https://blog.csdn.net/yyz_suhua/article/details/74722193](https://blog.csdn.net/yyz_suhua/article/details/74722193)

###  Cookie (java)
Cookie的使用
```java
Cookie[] cookies = request.getCookies();//这样便可以获取一个cookie数组  
		if(null==cookies) {
			System.out.println("没有cookie=========");  
		}else{
			for(Cookie cookie : cookies){ 
				System.out.println("name:"+cookie.getName()+",value:"+ cookie.getValue());
			} 
		}
```
Cookie的添加
```java
Cookie cookie = newCookie(name.trim(), value.trim());  
cookie.setMaxAge(30* 60);// 设置为30min  
cookie.setPath("/");  
System.out.println("已添加===============");  
response.addCookie(cookie);  
```
### 关于验证码图片的生成
1、验证吗的生成（4位）
```java
String s = "1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
for( int c = 0; c < 4; c++ ){
    word += s.charAt((int)(Math.random()*s.length()));
}
```
2、BufferedImage类

*BufferedImage*是Image的一个子类，Image和BufferedImage的主要作用就是将一副图片加载到内存中。BufferedImage生成的图片在内存里有一个图像缓冲区，利用这个缓冲区我们可以很方便的操作这个图片，通常用来做图片修改操作如大小变换、图片变灰、设置图片透明或不透明等。Java将一副图片加载到内存中的方法是：
```java
BufferedImage bufferedImage = ImageIO.read(new FileInputStream(filePath)); 
```

3、验证码图片的生成

1. 首先创建一个BufferedImage的实例，即创建一个指定宽高的图片，该图片在内存中有一个图像缓冲区，第三个参数的意义是创建的图像不带透明色，*TYPE_INT_ARGB* --带透明色，
2. 创建Graphics2D的对象进行图片绘制，使用Graphics2D g = image.createGraphics()的方式创建
3. 


5. 每次登录把session清空，新建一个session，保存验证码到session中，便于用户登录时进行验证码的校对。
6. 使用ServletOutputStream 抽象类和JPEGImageEncoder类将生成的验证码图片输出到客户端浏览器  JPEGImageEncoder coder = JPEGCodec.createJPEGEncoder(sos);  
JPEGCodec.createJPEGEncoder的作用是创建一个和指定输出流关联的JPEGImageEncoder对象。
oder.encode(image);   -- 对生成的图片进行编码，即设置图片的编码格式

参考文章：使用BufferedImage实现将几张图片合成一张图片[https://blog.csdn.net/u011768325/article/details/37961907](https://blog.csdn.net/u011768325/article/details/37961907)
```java
int width = 120;
int height = 30;
BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);  
Graphics2D g = image.createGraphics();
g.setColor(Color.white);
g.fillRect(0,0,width,height);
int x,y;
g.setColor(new Color((int)(Math.random()*255),(int)(Math.random()*255),(int)(Math.random()*255)));
for(int i=0; i < 100; ++i)
{
x = (int)(Math.random()*width);
y = (int)(Math.random()*height);
g.drawLine(x,y,x,y);
x = (int)(Math.random()*width);
y = (int)(Math.random()*height);
g.drawLine(x,y,x,y);
}
g.setColor(Color.red);
int fontSize = 23;
Font ft = new Font("System",Font.BOLD,fontSize);
g.setFont(ft);
int verifySize = word.length();
int fluent;
char[] chars = word.toCharArray();
for(int i = 0; i < verifySize; i++){
	AffineTransform affine = new AffineTransform();
	affine.setToRotation(Math.PI / 10 * generator.nextDouble() *      (generator.nextBoolean() ? 1 : -1), (width / verifySize) * i + fontSize/5, height/2);
    g.setTransform(affine);
    fluent=(int)(Math.random()*30+10);
    g.drawChars(chars, i, 1, ((width-fluent) / verifySize) * i + 5, (int)( Math.random()*(height - fontSize) + fontSize ));
	   }
	g.dispose();
//每次进入登陆页面，如果session存在，就把seesion设置过期，然后重新获取一个,这么做，就可以清空session已经存在的一些信息
		HttpSession session = req.getSession(false);
		if (session != null) {
			session.invalidate();
			//System.out.println(" the session is over");
			session=null;
		}
		session = req.getSession(true);
		session.setAttribute("yzm",word);
		ServletOutputStream sos = resp.getOutputStream();
		JPEGImageEncoder coder = JPEGCodec.createJPEGEncoder(sos);
		coder.encode(image);
	}
```

4、Session获取的两种方式
*  request.getSession(true)  == request.getSession(); 当Session存在时获取该Session，不存在时新建一个Session对象。

* request.getSession(false)  当Session不存在时，不创建Session对象

5、 Servlet--ServletInputStream类，ServletOutputStream类

**ServletInputStream：**
这个类定义了一个用来读取客户端的请求信息的输入流。这是一个 Servlet 引擎提供的抽象类。一个 Servlet 通过使用ServletRequest 接口获得了对一个 ServletInputStream 对象的说明。这个类的子类必须提供一个从 InputStream 接口读取有关信息的方法。

**ServletOutputStream：**             
参考文章：[https://www.yiibai.com/servlet/servletoutputstream-class.html](https://www.yiibai.com/servlet/servletoutputstream-class.html)
这是一个由 Servlet 引擎使用的抽象类。Servlet 通过使用 ServletResponse 接口的使用获得了对一个这种类型的对象的说明。利用这个输出流可以将数据返回到客户端。这个类的子类必须提供一个向 OutputStream 接口写入有关信息的方法。在这个接口中， 当一个刷新或关闭的方法被调用时。 所有数据缓冲区的信息将会被发送到客户端，也就是说响应被提交了。请注意，关闭这种类型的对象时不一定要关闭隐含的socket 流。

# Servlet ServletOutputStream类写入图片