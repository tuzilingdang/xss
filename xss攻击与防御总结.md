
# XSS攻击与防御总结

### 1. 最近碰到的问题

最近工作中接到漏洞管理平台报出的安全漏洞，还原一下XSS攻击的思路：

在input中输入\<img src=x onerror=alert(document.cookie)>,点击保存后，发现居然弹出了cookie信息了啊。 这有点严重，具体看图：

1. 在input框输入下图中的img标签，类似的JavaScript标签像\<script>,\<img>,\<svg>	等都可能会引发此问题。

   ![43](1.png)
   
2. 点击保存，结果页面开始弹啦啦啦……Cookie信息全部暴露。

   ![43](2.png)

3. 打开谷歌开发者工具，在console中看看是不是直接可以看到document.cookie,也可以截取ajax中的xhr对象，看看xhr中的setRequestHeader()这个接口是不是可以设置cookie信息。

	在ajax请求中插入这段代码，准备设置cookie：

	```
            beforeSend: function(xhr) {
                 xhr.setRequestHeader("Cookie", 'SESS83c26757d9118c6bafca4e38b2233563=G9oaLEcjP1BPlZQ5E4eSSgu0mfk2WqWrggxLNo5DsLk');  
             },
	```

	结果发现不行：

  	 ![43](3.png)

	报错显示在jquery中，所以究竟是jquery的ajax方法限制了cookie的设置还是浏览器直接在发送XMLHttpRequest时限制了呢？错误在这里：
 	 ![43](4.png)

	看起来是浏览器在setRequestHeader()时做了限制，下面这个链接里也有说明，翻译一下：尝试发送XMLHttpRequest请求时，设置'Cookie'header会在谷歌浏览器中会产生报错。 W3C spec将Cookie列为XMLHttpRequest不允许发送的headers之一。

	[Use of Cookie header for authentication means that remote jQuery clients can not authenticate](https://www.drupal.org/node/1133084)


	W3C的说明在这里: [the-setrequestheader-method](https://xhr.spec.whatwg.org/#the-setrequestheader()-method).这里才是最标准的答案┗|｀O′|┛ 嗷~~
	
	 ![43](5.png)

	以下都是不能设置的header name：
	![43](6.png)


	第3步尝试到此失败，回归正题。
	
4. 如果通过上述方法不能设置cookie，那唯一的设置Request Header的方式就是用document.cookie强行设置了。

	结果真的就设置成功了.
	![43](7.png)
	
5. 所以，现在看起来篡改cookie可以做到，接下来就是需要劫持其他用户的cookie了。 有一个方法是自己建一个无密码的WiFi，其他用户肯定会连的。接下来用Wireshark工具抓包就可以劫持到别人登录某网站的cookie。 现在正常的网站用户信息都不会直接存储在cookie中，cookie中带有sessionID，那么拿到cookie中的sessionID去请求后台接口，应该就可以直接登录了。 这里我没有试过，不知道是不是会有其他的问题需要绕过去才能成功。 目前看起来这个方案应该可以成功拿到别人的信息登入网站。

以上就是整个攻击的实现思路，接下来开始复习一下XSS攻击的相关知识点。	


### 2. XSS跨站点攻击

#### （1）反射型跨站脚本攻击
【描述】 攻击者会通过社会工程学手段，发送一个URL连接给用户打开，在用户打开页面的同时，浏览器会执行页面中嵌入的恶意脚本。


#### （2）存储型跨站脚本攻击
【描述】 攻击者利用web应用程序提供的录入或修改数据功能，将数据存储到服务器或用户cookie中，当其他用户浏览展示该数据的页面时，浏览器会执行页面中嵌入的恶意脚本。所有浏览者都会受到攻击。

#### （3）DOM跨站攻击

【描述】 由于html页面中，定义了一段JS，根据用户的输入，显示一段html代码，攻击者可以在输入时，插入一段恶意脚本，最终展示时，会执行恶意脚本。

 DOM跨站和以上两个跨站攻击的差别是，DOM跨站是纯页面脚本的输出，只有规范使用JAVASCRIPT，才可以防御。
 
 
### 3. 对应的XSS攻击案例
#### （1） 反射型跨站脚本攻击案例

待补充

#### （2）存储型跨站脚本攻击

待补充

#### （3） DOM跨站攻击碰到的问题

见第1小节问题描述。









