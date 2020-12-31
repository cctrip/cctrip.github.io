# Nginx匹配机制总结


### 写在前面

Nginx是一个当前主流的HTTP服务器和反向代理服务器，很多做WEB相关的同学基本都会用到，很多云厂商的七层负载均衡器也基本都是基于nginx实现的，个人在工作过程也算是经常接触，这篇文章主要想总结一下nginx的匹配机制，主要分为两块，一块是server的匹配，一块是location的匹配。

***

### Server匹配机制

配置过nginx的都知道，在一个http模块中是可以配置多个server模块的，并且多个server模块是可以配置相同的监听端口的，下面是一个简单的server配置例子：

```nginx
server {
    listen      80;
    server_name example.org www.example.org;
    ...
}

server {
    listen      80;
    server_name example.net www.example.net;
    ...
}

server {
    listen      80;
    server_name example.com www.example.com;
    ...
}
```

当我们对nginx发起http请求后，nginx会拿到http请求中对应的 `"Host"` 头部跟server模块中的`server_name`进行匹配，根据匹配的server结果进入具体的server模块处理http请求。那么，它具体的匹配机制是怎样的呢？

首先，我们先简单了解下nginx内部server的相关结构，

其中listen和server_name在配置文件中的写法有：

* listen(可带default_server标识)
  * `ip:port`
  * `ip`(监听80端口)
  * `port`(监听所有地址)
* server_name
  * `www.example.com`(完整域名)
  * `*.example.com(`带通配符开头的域名)
  * `www.example.*`(带通配符结尾的域名)
  * `~^(www.)?(.+)$`(正则写法的域名)

代码中的具体结构：

```c
/*************************************************************************************
伪结构体示例
(port) --> address(ip:port) --> server(example.com)
                            --> server(example.net)
                           
一个server模块的唯一标识是由address(listen配置)和server(server_name配置)组成
*************************************************************************************/

/* address 结构体，具有相同的ip:port */
struct ngx_http_addr_conf_s {
  /* default_server
  	 存储的是listen配置里带default_server标识的server，
     若没有就为顺序中的第一个server */
  ngx_http_core_srv_conf_t  *default_server;
  ngx_http_virtual_names_t  *virtual_names;
  unsigned                   ssl:1;
  unsigned                   http2:1;
  unsigned                   proxy_protocol:1;
};

/* virtual_name结构体，存储hash_combined和正则写法的server_name */
typedef struct {
  ngx_hash_combined_t        names;
  ngx_uint_t                 nregex;
  ngx_http_server_name_t    *regex;
} ngx_http_virtual_names_t;

/* hash_combined结构体，存储完成域名、通配符开头、通配符结尾的server_name */
typedef struct {
  ngx_hash_t            hash;
  ngx_hash_wildcard_t  *wc_head;
  ngx_hash_wildcard_t  *wc_tail;
} ngx_hash_combined_t;

```



通过结构体，我们来说明下server的匹配规则：

1. host是否匹配virtual_names中的names中的完整域名(`hash`)，若是则返回
2. host是否匹配virtual_name中的names中的通配符开头的域名(`wc_head`)，若是则返回
3. host是否匹配virtual_name中的names中的通配符结尾的域名(`wc_tail`)，若是则返回
4. host是否匹配virtual_name中的正则写法的域名(`regex`)，若是则返回
5. 返回`default_server`



具体示例如下：

```nginx
#精确匹配，第一优先级
server {
  listen 80;
  server_name www.test.com;
}

#通配符开头匹配，第二优先级，
server {
  listen 80;
  server_name *.test.com;
}

#通配符结尾匹配，第三优先级
server {
  listen 80;
  server_name www.test.*;
}

#正则匹配，第三优先级
server {
  listen 80;
  server_name ~^(www.)?(.+)$;
}


#default，没找到对应host，则以此优先
server {
  listen 80 defalut_server;
  server_name _;
}

#若没有加defalut_server，则第一个server为defalut_server
server {
  listen 80;
  server_name _;
}
```

***

### Location匹配机制

一个server模块可以配置多个location，nginx根据`URI`来进行匹配，

lication的写法有以下几种：

1. `= /uri` 精确匹配

2. `^~ /uri` 非正则前缀匹配

3. `~ 或者 ~* /uri` 正则匹配

4. `/uri` 前缀匹配



整个location匹配机制如下：

1. 针对所有前缀字符串测试URI(包括精确匹配、非正则前缀匹配、前缀匹配中的字符串)
2. uri等于精确匹配中的字符串，停止搜索
3. 最长(最相似)前缀字符串如果为非正则前缀匹配(带`^~`)，则停止正则搜索
4. 保存最长(最相似)的前缀字符串
5. 按顺序进行uri和正则匹配测试，有一个匹配成功后就停止搜索
6. 如果都没有，就使用最长(最相似)的前缀匹配



###### 额外说明：

最长(最相似)前缀字符串的测试阶段，非正则前缀匹配匹配、前缀匹配的优先级是一致的，谁的长度长，谁优先。优先前缀匹配的前提必须是前缀字符串`非正则前缀匹配的长度`大于`前缀匹配的长度`，这个很多网站都是直接写成了非正则前缀匹配是第二优先级，没有说明前提条件。



具体示例如下：

```nginx
#精确匹配，第一优先级
location = /test {
  
}
#前缀匹配，最低优先级，长度优先
location /test/aa {
  
}

#非正则前缀匹配，最长前缀下为第二优先级(特殊条件下)
location ^~ /test {
  
}

#正则匹配，第三优先级,顺序优先
# ~ : 区分大小写
# ~* : 不区分大小写 
location ~* ^/test {
  
}
```

***

### 参考

[https://nginx.org/en/docs/http/request_processing.html](https://nginx.org/en/docs/http/request_processing.html)

[https://nginx.org/en/docs/http/ngx_http_core_module.html#location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)

[https://docs.nginx.com/nginx/admin-guide/web-server/web-server/](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/)

[https://www.codedump.info/post/20190212-nginx-http-config/](https://www.codedump.info/post/20190212-nginx-http-config/)


