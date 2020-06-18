## The Web In Depth

### Cookie安全性

子域名可以读取父域名的Cookie

子域名的Cookie仅可在该子域名以及其之下的子域名读取

子域名可以设置父域名及其子域名的Cookie，但不可设置兄弟域名的Cookie



Cookie Flags

- Secure：只有HTTPS才可访问
- HTTPOnly：JavaScript不可读



### Parsing

HTML的解析不仅仅是在浏览器，还有可能在防火墙或是其他Filter。如果这两者之间的解析规则存在差异，则有可能存在被攻击的风险。例如script/xxx这类HTML tag，Firefox会把/视为空格，则执行script，如果Filter忽略这类tag则可能被攻击。



HTML还有很多关于兼容性解析规则（Legacy Parsing）

- `<script>`在页面结束会自动close
- 未close的tag会在下一个tag出现时自动close



### Content Sniffing

MIME Sniffing

浏览器通常不仅仅根据Content-Type来解析内容，还会根据实际内容进行判决。如果内容像是HTML，就按HTML解析。在IE 6/7，如果图片或者文本包含HTML Tag就会按HTML解析。这也是使用独立域名访问用户上传内容的其中一个原因。



Encoding Sniffing

如果没有指定encoding，浏览器会自行推导。

例如：+ADw-SCRIPT+AD4-alert('XSS');+ADw-/SCRIPT+AD4-，在UTF8下都是合法字符，IE8-会判定为UTF7进行解析



### Same-Orign Policy（SOP）

限制XMLHttpRequest可以通信的域名

控制DOM的访问（frame / window）



同源匹配规则

- 协议要匹配（HTTP / HTTPS）
- 端口要匹配
- 域名要准确匹配



跨域

- 可以设置document.domain（只可设置为父域，且不能设置端口）
- window.postMessage解决iframe之间跨域
- CORS



### CSRF

Cross-Site Request Forgery

用户登陆恶意网站，网站以用户的身份提交数据到目标网站

常用防止攻击方法是，在form内加入CSRF Token（随机字符串且与用户Session绑定）

只有POST才有效，因为使用GET时不应该执行状态修改操作