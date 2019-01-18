# nginx图片防盗链及缩略图的使用方式和常用脚本





## nginx防盗链配置 

### nginx参数列表

| 参数                  | 参数名称                                             |
| --------------------- | ---------------------------------------------------- |
| $remote_addr          | 客户端的ip地址(代理服务器，显示代理服务ip)           |
| $remote_user          | 用于记录远程客户端的用户名称（一般为“-”）            |
| $time_local           | 用于记录访问时间和时区                               |
| $request              | 用于记录请求的url以及请求方法                        |
| $status               | 响应状态码，例如：200成功、404页面找不到等           |
| $body_bytes_sent      | 给客户端发送的文件主体内容字节数                     |
| $http_user_agent      | 用户所使用的代理（一般为浏览器）                     |
| $http_x_forwarded_for | 可以记录客户端IP，通过代理服务器来记录客户端的ip地址 |
| $http_referer         | 可以记录用户是从哪个链接访问过来的                   |
| $host                 | 本机地址                                             |



### nginx 代码编写

* 编写脚本

   ~~~
  # 配置return返回及 设置值
  location /code {
        set $url = 'hello world'；
         	   return 200 "my $url $remote_addr $remote_user $time_local request: $request $body_bytes_sent referer: $http_referer";
   }
  
  
   ~~~

* 配置跳转方式

  ~~~
  ## 跳转代码的编写
  
  
  ## if条件的判断
  
  
  
  
  ~~~

* if指令的使用方式

  * 语法为`if(condition){...}`，对给定的条件condition进行判断。如果为真，大括号内的rewrite指令将被执行，if条件(conditon)可以是如下任何内容：
    * 当表达式只是一个变量时，如果值为空或任何以0开头的字符串都会当做false
    * 直接比较变量和内容时，使用`=`或`!=`
    * `~`正则表达式匹配，`~*`不区分大小写的匹配，`!~`区分大小写的不匹配
  * `-f`和`!-f`用来判断是否存在文件
    `-d`和`!-d`用来判断是否存在目录
    `-e`和`!-e`用来判断是否存在文件或目录
    `-x`和`!-x`用来判断文件是否可执行

  ~~~
  if ($http_user_agent ~ MSIE) {
      rewrite ^(.*)$ /msie/$1 break;
  } //如果UA包含"MSIE"，rewrite请求到/msid/目录下
  
  if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
      set $id $1;
   } //如果cookie匹配正则，设置变量$id等于正则引用部分
  
  if ($request_method = POST) {
      return 405;
  } //如果提交方法为POST，则返回状态405（Method not allowed）。return不能返回301,302
  
  if ($slow) {
      limit_rate 10k;
  } //限速，$slow可以通过 set 指令设置
  
  if (!-f $request_filename){
      break;
      proxy_pass  http://127.0.0.1; 
  } //如果请求的文件名不存在，则反向代理到localhost 。这里的break也是停止rewrite检查
  
  if ($args ~ post=140){
      rewrite ^ http://example.com/ permanent;
  } //如果query string中包含"post=140"，永久重定向到example.com
  
  location ~* \.(gif|jpg|png|swf|flv)$ {
      valid_referers none blocked www.jefflei.com www.leizhenfang.com;
      if ($invalid_referer) {
          return 404;
      } //防盗链
  }
  ~~~

  * if判断如果不是本域名的网站重定向

    ~~~
    ## if判断
    # http://pc.huwuxianglian.com/img/base/slogo.png
    
    
    
    
    ~~~

    * if语句中的判断条件(nginx)

      1、正则表达式匹配：

      ==:等值比较;

      ~：与指定正则表达式模式匹配时返回“真”，判断匹配与否时区分字符大小写；

      ~*：与指定正则表达式模式匹配时返回“真”，判断匹配与否时不区分字符大小写；

      !~：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时区分字符大小写；

      !~*：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时不区分字符大小写；

       

      2、文件及目录匹配判断：

      -f, !-f：判断指定的路径是否为存在且为文件；

      -d, !-d：判断指定的路径是否为存在且为目录；

      -e, !-e：判断指定的路径是否存在，文件或目录均可；

      -x, !-x：判断指定路径的文件是否存在且可执行；
      --------------------- 

    * if多条件判断

      ~~~
      ## if和（） 之间必须有个空格
      if ($http_referer ~* '?*\.huwuxianglian\.com.*' ){
          
      }
      if ($http_referer !~* ".*\.huwuxianglian.com.*"){
                       
                       rewrite ^/   http://pc.huwuxianglian.com/img/base/slogo.png                      redirect;
                      }
                     
                         root /data/file/img;
      
      
      
      ~~~

      

​    

* 全局变量参数
  - `$args` ： #这个变量等于请求行中的参数，同`$query_string`
  - `$content_length` ： 请求头中的Content-length字段。
  - `$content_type` ： 请求头中的Content-Type字段。
  - `$document_root` ： 当前请求在root指令中指定的值。
  - `$host` ： 请求主机头字段，否则为服务器名称。
  - `$http_user_agent` ： 客户端agent信息
  - `$http_cookie` ： 客户端cookie信息
  - `$limit_rate` ： 这个变量可以限制连接速率。
  - `$request_method` ： 客户端请求的动作，通常为GET或POST。
  - `$remote_addr` ： 客户端的IP地址。
  - `$remote_port` ： 客户端的端口。
  - `$remote_user` ： 已经经过Auth Basic Module验证的用户名。
  - `$request_filename` ： 当前请求的文件路径，由root或alias指令与URI请求生成。
  - `$scheme` ： HTTP方法（如http，https）。
  - `$server_protocol` ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
  - `$server_addr` ： 服务器地址，在完成一次系统调用后可以确定这个值。
  - `$server_name` ： 服务器名称。
  - `$server_port` ： 请求到达服务器的端口号。
  - `$request_uri` ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
  - `$uri` ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
  - `$document_uri` ： 与$uri相同。
  - $http_x_forwarded_for ： 可以记录客户端IP，通过代理服务器来记录客户端的ip地址

​        



### nginx 实现valid_referer全面解析

可以看出两者的主要区别在于referer，

了解了背景知识后我们来解决问题

现公司要求实现通过搜索引擎打开这个网址的化进行跳转到别的网址

直接输入网址打开的就打开这个网址

valid_referers 指令详解

该指令后面可以接 none blocked serevr_names string或者是正则[表达式](http://www.07net01.com/tags-%E8%A1%A8%E8%BE%BE%E5%BC%8F-0.html)

none 代表没有referer

blocked 代表有referer但是被[防火墙](http://www.07net01.com/tags-%E9%98%B2%E7%81%AB%E5%A2%99-0.html)或者是代理给去除了

string或者正在表达式 用来匹配referer

[nginx](http://www.07net01.com/linux/)会通过查看referer字段和valid_referers后面的referer列表进行匹配，如果匹配到了就invalid_referer字段值为0 否则设置该值为1



~~~
## 标准
#valid_referers none blocked server_names; 
#if ($invalid_referer) { 
# rewrite ^/ http://********.com/ redirect; 
#}
location /code {
           valid_referers none blocked *.huwuxianglian.com *.yiwangqingshui.com;
           # 如果匹配 $valid_referer 返回0
	   if($invalid_referer){
             rewrite ^/ http://pc.huwuxianglian.com/ywqs redirect;
           }	
           set $url 'hello world';
       	   return 200 "args=$args  my $url uri=$uri   $remote_addr $remote_user $time_local request: $request $body_bytes_sent referer: $http_referer";
        }
        
        
location /img/download{
	   expires 10d;
           valid_referers none blocked *.huwuxianglian.com *.yiwangqingshui.com;
           if ($invalid_referer){
              rewrite ^/ http://pc.huwuxianglian.com/img/base/slogo.png redirect;
           }
           alias /data/ywqs/img/upload;	
}	        
~~~

## nginx缩略图的配置

### lua脚本的引入









## nginx中加脚本代码

### nginx.conf文件的常用配置修改

* 增加日志的写入方式

  ~~~
  # 自定义日志输出 在http模块中，在location当中也行
  http {
       log_format mylog '请求地址:$uri, 来源网址:$http_referer ,浏览器类型:$http_user_agent';
     
    access_log logs/mylog.log mylog;
  }
   
  ~~~

* rewrite的编写

  ~~~
  location /test/ {
      return 504;
  }  
  location /break/ {  
      rewrite ^/break/(.*) /test/$1 break;
      return 402;
  }
    location /per/ {
      rewrite ^/per/(.*) /test/$1 permanent;
      return 402;
  }
  location /last {  
      rewrite ^/last/(.*) /param permanent;
      return 403;
  }
  ~~~

  * 语法标准

    ~~~
    rewrite 拦击url 跳转的url flag
    
    
    #### flag 标识值
    last : 相当于Apache的[L]标记，表示完成rewrite
    break : 停止执行当前虚拟主机的后续rewrite指令集
    redirect : 返回302临时重定向，地址栏会显示跳转后的地址
    permanent : 返回301永久重定向，地址栏会显示跳转后的地址
    
    
    ~~~

* 反向代理跳转

  ~~~
  # 反向代理
  location /front{
  	proxy_pass http://localhost:8000;			
  }
  ~~~

  



### nginx正则表达式: 

* 正则：
  * ^： 匹配字符串的开始位置；
  *  $：匹配字符串的结束位置
  * .*:   .匹配任意字符，*匹配数量0到正无穷
  * \. 斜杠用来转义，\.匹配 .    特殊使用方法，记住记性了
  * （值1|值2|值3|值4）：或匹配模式，例：（jpg|gif|png|bmp）匹配jpg或gif或png或bmp
  * i不区分大小写

  

























































