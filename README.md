#SearchIt!
具有反向代理功能的仿制版百Google度(baigoogledu)  
此项目主要为了解决天朝用户同时使用两种搜索引擎的问题。  
*Across the Great Wall, we can reach every corner in the world.*
## 概述
###什么是百Google度
百Google度--在他们之间平均85%链接均不相同！  
百google度是一种元搜索引擎，它综合了谷歌、百度两大搜索引擎。它的首页界面基本和谷歌、百度无异。只要在搜索框里键入关键词，就可搜索目标信息。不过，搜索界面很有特色，浏览器一分为二，左边是谷歌，右边是百度，搜出的信息和使用谷歌百度没有差别。  
（来源：[百度百科](http://baike.baidu.com/item/baigoogledu)）
###本仿制项目发展历史
为了翻墙和同时使用两种搜索引擎，我曾有过较长时间的研究：  
  
- 有时百Google度的Google栏失效。百Google度后来被封，无法正常使用。期间我用PHP仿制了一个类似的网站。  
- Google响应头含有`x-frame-options:SAMEORIGIN`，无法放到非同源的iframe中，过去都是使用`http://www.google.com/custom?btnG=Search&newwindow=1&q=test`配合浏览器代理插件使用；或者使用没有此响应头的反向代理。但是反向代理网站被封的越来越多，越来越难找，Google的custom URL某日失效，自动跳转到普通的搜索。以上方法基本完全失效。  
- 后来想办法一次搜索弹两个窗口，但是窗口太多使用不便。  
- 后来使用curl，服务端搭建反向代理。（2016年7月）  
  
### 文件功能
默认配置下：  

- index.php  首页  
- search.php?q=test 搜索test	 
- proxy/proxy.php 反向代理
- proxy/proxy_conf.php 反向代理配置
- `file_get_contents_search.inc.php` 使用 `file_get_contents()`获取网页的被包含文件
- `curl_search.inc.php` 使用 `curl`获取网页的被包含文件

### 主要配置
####search.php中的两个iframe的src属性
设置search.php 中的`$googleurl`,`$baiduurl`。这两个URL由浏览器请求。如需使用此项目中的代理，则需设置为：

    $googleurl='./proxy/proxy.php?engine=google&q=';
    $baiduurl='./proxy/proxy.php?engine=baidu&q=';
此时两个iframe中的内容由服务器端，proxy.php向google.com等发出请求，然后将结果转发给浏览器。  
如需直接连接，设置为：

    $googleurl = 'https://www.google.com/search?site=webhp&source=hp&newwindow=1&hl=zh-Hans&num=20&q=';
    $baiduurl = 'https://www.baidu.com/s?ie=utf-8&rn=20&wd=';
此时两个iframe由浏览器直接发出请求。
#### 反向代理配置
修改`proxy_conf.php`。  
`url`指定访问的url（不含被搜索字符串）。`proxy`,`proxyHost`,`proxyPort`设置服务器的上层代理服务器。用curl方式时，如果使用SSL，需要设置`sslCert`设置证书。`beautyFunc`指定进一步处理响应HTML的函数名。`userAgent`设置服务端请求网页时使用的UserAgent字符串。
下例中，`googleBeauty`函数将服务端获取到的网页做进一步处理，适应iframe较窄的宽度。

	$googleConf = ['url'=>'https://www.google.com/search?site=webhp&source=hp&newwindow=1&hl=zh-Hans&num=20&q=',
		'proxy'=>true,
		'proxyHost'=>'127.0.0.1',
		'proxyPort'=>8080,
		'sslCert'=>getcwd() . "/cert/google.com.crt",
		'beautyFunc'=>'googleBeauty',
		'userAgent'=>'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36'];
	
	function googleBeauty($html){
		$html = str_replace('<form class="tsf" action="/search" style="overflow:visible" id="tsf" method="GET" name="f" onsubmit="return q.value!=\'\'" role="search">', 
			'<form class="tsf" action="" style="overflow:visible" id="tsf" method="GET" name="f" onsubmit="return q.value!=\'\'" role="search"><input type="hidden" name="engine" value="google">', 
			$html);
		$html = str_replace('<div class="sbibtd">', 
			'<div class="sbibtd" style="width:500px;">', 
			$html);
		return str_replace('<div id="center_col">',
			 '<div id="center_col" style="margin:0;">',
			 $html);
	}

#### 关于curl 和 file_get_contents()两种获取网页方式
使用curl可以较好地使用代理服务器，但是需要安装该extension，HTTPS如果出现验证证书错误需要手动提供证书（/proxy/cert目录）。  
`file_get_contents()`无需安装extension，可以在GAE上直接使用。理论上用其`$context`参数可以控制使用代理服务器，但是在本机测试无法正常使用，出现各种错误。  
目前`proxy/proxy.php`自动判断是否安装`curl`库，根据结果require:优先`curl_search.inc.php`，其次`file_get_contents_search.inc.php`。  
附：安装[php_curl库](http://php.net/manual/zh/book.curl.php)  

### 测试
- test
- `<script>alert(1);</script>`
- AT&T
- C#
- 1+1=?
- 汉字