## HTTP缓存
由服务端设置响应头来区分：
- Cache-control 用于控制HTTP缓存（在 HTTP/1.0 中可能部分没实现，仅仅实现了 Pragma: no-cache）；  
- Expires 表示存在时间，允许客户端在这个时间之前不去检查（发请求），等同 max-age 的效果。但是如果同时存在，则被 Cache-Control 的 max-age 覆盖。


### 强缓存和协商缓存
F5 会跳过强缓存规则，直接走协商缓存；Ctrl+F5 ，跳过所有缓存规则，和第一次请求一样，重新获取资源，其他情况根据响应头 cache-control 区分。
- 服务器响应 cache-control 含有 max-age，未过期时不发起请求，强缓存 200（from cache）
- 服务器响应 cache-control 未设置 max-age，或缓存时间过期，发起http请求时，协商缓存 304(no modified)
- 服务器响应 cache-control 设置 no-store，两者均不缓存

### cache-control解析：
must-revalidate 指令是用来表示在一个缓存过期之后，不能直接使用这个过期的缓存，必须校验之后才能使用。

#### 1、cache-control: max-age=xxxx，public
- 客户端和代理服务器都可以缓存该资源；
- 客户端在xxx秒的有效期内，如果有请求该资源的需求的话就直接读取缓存, statu code:200 ，如果用户做了刷新操作F5，就向服务器发起 http 请求

#### 2、cache-control: max-age=xxxx，private
与第一点的区别：只让客户端可以缓存该资源；代理服务器不缓存

#### 3、cache-control: max-age=xxxx，immutable
客户端在xxx秒的有效期内，如果有请求该资源的需求的话就直接读取缓存, statu code:200 ，即使用户做了刷新操作，也不向服务器发起 http 请求

#### 4、cache-control: no-cache
不设置强缓存，但是不妨碍设置协商缓存；一般如果你做了强缓存，只有在强缓存失效了才走协商缓存的，设置了 no-cache 就不会走强缓存了，每次请求都回询问服务端。

#### 5、cache-control: no-store
不缓存，这个会让客户端、服务器都不缓存，也就没有所谓的强缓存、协商缓存了。

### 协商缓存判定：
#### 响应返回：
Last-Modified 更改时间  
Etag 文件内容唯一标识

#### 请求头携带：
If-Modified-Since 上一次返回的Last-Modified（比较时间）  
If-None-Match 上一次返回的Etag（比较标记）
