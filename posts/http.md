# HTTP 备忘录

## HTTP Status code

- Informational `1XX`
- Successful `2XX`
- Redirection `3XX`
- Client Error `4XX`
- Server Error `5XX`

状态码|Name|名称|说明
:-:|:-----------|:------|:---------------------------
100|Continue|继续|请求者应当继续提出请求。 服务器返回此代码表示已收到请求的第一部分，正在等待其余部分。
101|Switching Protocols|切换协议|请求者已要求服务器切换协议，服务器已确认并准备切换。
102|Processing (WebDAV)||
103|Early Hints||
200|OK|成功|
201|Created|已创建|请求成功并且服务器创建了新的资源。 
202|Accepted|已接受|服务器已接受请求，但尚未处理。 
203|Non-Authoritative Information|非授权信息|服务器已成功处理了请求，但返回的信息可能来自另一来源。 
204|No Content|无内容|服务器成功处理了请求，但没有返回任何内容。 
205|Reset Content|重置内容|服务器成功处理了请求，但没有返回任何内容。 
206|Partial Content|部分内容|服务器成功处理了部分 GET 请求。
207|Multi-Status (WebDAV)||代表之后的消息体将是一个XML消息，并且可能依照之前子请求数量的不同，包含一系列独立的响应代码。
208|Already Reported (WebDAV)||DAV绑定的成员已经在（多状态）响应之前的部分被列举，且未被再次包含。
226|IM Used||服务器已经满足了对资源的请求，对实体请求的一个或多个实体操作的结果表示。
300|Multiple Choices|多种选择|针对请求，服务器可执行多种操作。 服务器可根据请求者 (user agent) 选择一项操作，或提供操作列表供请求者选择。 
301|Moved Permanently|永久移动|请求的网页已永久移动到新位置。 服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置。
302|Found (Previously "Moved temporarily")|临时移动|服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。 
303|See Other|查看其他位置|请求者应当对不同的位置使用单独的 GET 请求来检索响应时，服务器返回此代码。 
304|Not Modified|未修改|自从上次请求后，请求的网页未修改过。 服务器返回此响应时，不会返回网页内容。 
305|Use Proxy|使用代理|请求者只能使用代理访问请求的网页。 如果服务器返回此响应，还表示请求者应使用代理。 
306|Switch Proxy|临时重定向|服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。
307|Temporary Redirect||在这种情况下，请求应该与另一个URI重复，但后续的请求应仍使用原始的URI。 与302相反，当重新发出原始请求时，不允许更改请求方法。 例如，应该使用另一个POST请求来重复POST请求。
308|Permanent Redirect||请求和所有将来的请求应该使用另一个URI重复。 307和308重复302和301的行为，但不允许HTTP方法更改。 例如，将表单提交给永久重定向的资源可能会顺利进行。
400|Bad Request|请求错误|这些状态代码表示请求可能出错，妨碍了服务器的处理。
401|Unauthorized|未授权|请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。 
402|Payment Required||
403|Forbidden|禁止|服务器拒绝请求。 
404|Not Found|未找到|服务器找不到请求的网页。 
405|Method Not Allowed|方法禁用|禁用请求中指定的方法。 
406|Not Acceptable|不接受|无法使用请求的内容特性响应请求的网页。 
407|Proxy Authentication Required|需要代理授权|此状态代码与 401（未授权）类似，但指定请求者应当授权使用代理。 
408|Request Timeout|请求超时|服务器等候请求时发生超时。 
409|Conflict|冲突|服务器在完成请求时发生冲突。 服务器必须在响应中包含有关冲突的信息。 
410|Gone|已删除|如果请求的资源已永久删除，服务器就会返回此响应。 
411|Length Required|需要有效长度|服务器不接受不含有效内容长度标头字段的请求。 
412|Precondition Failed|未满足前提条件|服务器未满足请求者在请求中设置的其中一个前提条件。 
413|Payload Too Large|请求实体过大|服务器无法处理请求，因为请求实体过大，超出服务器的处理能力。 
414|URI Too Long|请求的 URI 过长|请求的 URI（通常为网址）过长，服务器无法处理。 
415|Unsupported Media Type|不支持的媒体类型|请求的格式不受请求页面的支持。 
416|Range Not Satisfiable|请求范围不符合要求|如果页面无法提供请求的范围，则服务器会返回此状态代码。 
417|Expectation Failed|未满足期望值|服务器未满足”期望”请求标头字段的要求。
418|I'm a teapot||
421|Misdirected Request||
422|Unprocessable Entity||
423|Locked (WebDAV)||
424|Failed Dependency (WebDAV)||
426|Upgrade Required||
428|Precondition Required||
429|Too Many Requests||
431|Request Header Fields Too Large||
451|Unavailable For Legal Reasons||
500|Internal Server Error|服务器内部错误|服务器遇到错误，无法完成请求。 
501|Not Implemented|尚未实施|服务器不具备完成请求的功能。 例如，服务器无法识别请求方法时可能会返回此代码。 
502|Bad Gateway|错误网关|服务器作为网关或代理，从上游服务器收到无效响应。 
503|Service Unavailable|服务不可用|服务器目前无法使用（由于超载或停机维护）。 通常，这只是暂时状态。 
504|Gateway Timeout|网关超时|服务器作为网关或代理，但是没有及时从上游服务器收到请求。 
505|HTTP Version Not Supported|HTTP 版本不受支持|服务器不支持请求中所用的 HTTP 协议版本。
506|Variant Also Negotiates||
507|Insufficient Storage (WebDAV)||
508|Loop Detected||
510|Not Extended||
511|Network Authentication Required||

## HTTP header fields

参考：<https://zh.wikipedia.org/wiki/HTTP%E5%A4%B4%E5%AD%97%E6%AE%B5>

- 一般来说 header 以 `X-` 开头的为非标准头
- HTTP 头理论上没有大小和数量限制，但是大部分的服务器都有限制，如 apache 2.3 限制每个 field 8190 bytes，每个请求允许 100 个头

### Request fields

字段|说明|示例
:--|:-----------|:-----
Accept|能够接受的回应内容类型（Content-Types）。参见内容协商。|Accept: text/plain
Cache-Control|用来指定在这次的请求/响应链中的所有缓存机制 都必须 遵守的指令|Cache-Control: no-cache
Connection|该浏览器想要优先使用的连接类型|Connection: keep-alive<br>Connection: Upgrade
Cookie|之前由服务器通过 Set-Cookie（下文详述）发送的一个超文本传输协议 Cookie|Cookie: $Version=1; Skin=new;
Content-Length|以 八位字节数组 （8位的字节）表示的请求体的长度|Content-Length: 348
Content-Type|请求体的 多媒体类型 （用于POST和PUT请求中）|Content-Type: application/x-www-form-urlencoded
Date|发送该消息的日期和时间(按照 RFC 7231 中定义的"超文本传输协议日期"格式来发送)|Date: Tue, 15 Nov 1994 08:12:31 GMT
Host|服务器的域名(用于虚拟主机 )，以及服务器所监听的传输控制协议端口号。如果所请求的端口是对应的服务的标准端口，则端口号可被省略。|Host: en.wikipedia.org:80
Origin|发起一个针对 跨来源资源共享 的请求（要求服务器在回应中加入一个‘访问控制-允许来源’（'Access-Control-Allow-Origin'）字段）。|Origin: http://www.example-social-network.com
Referer|表示浏览器所访问的前一个页面，正是那个页面上的某个链接将浏览器带到了当前所请求的这个页面。|Referer: http://en.wikipedia.org/wiki/Main_Page
User-Agent|浏览器的浏览器身份标识字符串|

常见的非标准请求字段

字段|说明|示例
:--|:-----------|:-----
X-Requested-With|主要用于标识 Ajax 及可扩展标记语言 请求。大部分的JavaScript框架会发送这个字段，且将其值设置为 XMLHttpRequest|X-Requested-With: XMLHttpRequest
X-Forwarded-For|一个事实标准 ，用于标识某个通过超文本传输协议代理或负载均衡连接到某个网页服务器的客户端的原始互联网地址|X-Forwarded-For: client1, proxy1, proxy2
X-Forwarded-Host|一个事实标准 ，用于识别客户端原本发出的 Host 请求头部。|X-Forwarded-Host: en.wikipedia.org:80
X-Forwarded-Proto|一个事实标准，用于标识某个超文本传输协议请求最初所使用的协议。|X-Forwarded-Proto: https

### Response fields

字段|说明|示例
:--|:-----------|:-----
Access-Control-Allow-Origin|指定哪些网站可参与到跨来源资源共享过程中|Access-Control-Allow-Origin: *
Age|这个对象在代理缓存中存在的时间，以秒为单位|Age: 12
Allow|对于特定资源有效的动作。针对HTTP/405这一错误代码而使用|Allow: GET, HEAD
Cache-Control|向从服务器直到客户端在内的所有缓存机制告知，它们是否可以缓存这个对象。其单位为秒|Cache-Control: max-age=3600
Connection|针对该连接所预期的选项|Connection: close
Content-Disposition|一个可以让客户端下载文件并建议文件名的头部。文件名需要用双引号包裹。|Content-Disposition: attachment; filename="fname.ext"
Expires|指定一个日期/时间，超过该时间则认为此回应已经过期|Expires: Thu, 01 Dec 1994 16:00:00 GMT
Last-Modified|所请求的对象的最后修改日期(按照 RFC 7231 中定义的“超文本传输协议日期”格式来表示)|Last-Modified: Tue, 15 Nov 1994 12:45:26 GMT
Server|服务器的名字|Server: Apache/2.4.1 (Unix)
Set-Cookie|HTTP cookie|Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1
Strict-Transport-Security|HTTP 严格传输安全这一头部告知客户端缓存这一强制 HTTPS 策略的时间，以及这一策略是否适用于其子域名。|Strict-Transport-Security: max-age=16070400; includeSubDomains
