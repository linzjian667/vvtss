
## 注意事项

0）**最新** Cloudflare Page域名还没被SNI阻断，利用[此项目](https://github.com/xyTom/cf-page-func-proxy)可利用CF Pages反代。（无需自定义域名）

1）2022年5月8日晚，CloudFlare Workers 的业务域名 Workers.dev 被防火长城 DNS 污染、SNI阻断。

2）CloudFlare Workers，可自定义workers域名；经过添加自定义域名，更换Host和SNI后已可正常使用。

3）接入点可以直接使用自选IP域名`uicdn.cf`，每30分钟更新一次。

## 一、操作步骤：

1、点击下面按钮开始部署

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy) 


2、之前没有登录记录的话，会先提示注册并或登录Heroku界面，大家自己注册或者登录下



3、Heroku app名称与国家随意，最后设置图如下



4、输入UUID，建议使用V2rayN等工具生成，点击Deploy app，几秒种后就完成安装。

-------------------------------------------------------------------------------------------


## 二、Cloudflare http 80端口自选IP域名，绕过SNI阻断，Cloudflare workers直连复活，无须自定义域名路由

某大佬总结：80端口无TLS协议，没有TLS就没有SNI。这样就不会被 SNI阻断。适合vmess，ss自带加密的协议，vless与trojan也可以，但就是明文流量，不安全。没有TLS还有个好处就是没有TLS握手，就能降低延迟。
![dy8werhf89rei4r8w30o9ikurfdoyyg](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiQ4vfNGfWV-HWJr48Lr_YZA-9h7dRP7UUJYjMh_fA6UCNbLNKS3U_bew7cijWXL6rthsPLaY98UtyoF_Kp95Y2IOI70TxMRB9fTzEs4H8V85tSe6FZU2Nijs6_8al5SQI7dXkDzz6wov9BJfAIG6R9QQRztWFAM0CsaB6jpH_9n12EOQf-5CNxViHGew/w634-h356/%E5%B0%81%E9%9D%A2%E5%9B%BE-002.JPG)

## 三、客户端配置如下

IOS端小火箭就可以通吃，安卓端推荐V2rayNG或搭配Kitsunebi

协议：(vless/vmess/trojan)-ws

地址：app.heroku.com（自选IP/域名）

端口：443

用户ID/密码：自定义UUID

传输协议：ws

伪装host：app.heroku.com（workers或pages反代/自定义域）

路径path：/自定义UUID-协议开头两小写字母

传输安全：tls

SNI：app.heroku.com（workers或pages反代/自定义域）

其他设置保持默认即可！



shadowsocks-ws与socks5-ws推荐用Kitsunebi，配置简单，不需要plugin插件

协议：(shadowsocks/socks5)-ws

地址：app.heroku.com（自选IP/域名）

端口：443

shadowsocks密码：自定义UUID

shadowsocks加密方式：chacha20-ietf-poly1305(默认)

socks5用户名：空

socks5密码：空

传输协议：ws

伪装host：app.heroku.com（workers或pages反代/自定义域）

路径path：/自定义UUID-协议开头两小写字母

传输安全：tls

SNI(证书域名)：app.heroku.com（workers或pages反代/自定义域）

其他设置保持默认即可！


### workers反代与pages反代及自定义域，配置文件信息等相关操作拓展教程，请关注：[博客视频教程](https://ygkkk.blogspot.com/2022/05/heroku-cloudflare-workers-pages.html)


### CloudFlare Workers反代代码（可分别用两个账号的应用程序名（`path路径`、`协议`、`UUID`保持一致），单双号天分别执行，那一个月就有550+550小时（每个账号一个月免费使用550小时））
<details>
<summary>CloudFlare Workers单账户反代代码</summary>

```js
addEventListener(
    "fetch",event => {
        let url=new URL(event.request.url);
        url.hostname="appname.herokuapp.com";
        let request=new Request(url,event.request);
        event. respondWith(
            fetch(request)
        )
    }
)
```
</details>

<details>
<summary>CloudFlare Workers单双日轮换反代代码</summary>

```js
const SingleDay = 'app0.herokuapp.com'
const DoubleDay = 'app1.herokuapp.com'
addEventListener(
    "fetch",event => {
    
        let nd = new Date();
        if (nd.getDate()%2) {
            host = SingleDay
        } else {
            host = DoubleDay
        }
        
        let url=new URL(event.request.url);
        url.hostname=host;
        let request=new Request(url,event.request);
        event. respondWith(
            fetch(request)
        )
    }
)
```
</details>

<details>
<summary>CloudFlare Workers每五天轮换一遍式反代代码</summary>

```js
const Day0 = 'app0.herokuapp.com'
const Day1 = 'app1.herokuapp.com'
const Day2 = 'app2.herokuapp.com'
const Day3 = 'app3.herokuapp.com'
const Day4 = 'app4.herokuapp.com'
addEventListener(
    "fetch",event => {
    
        let nd = new Date();
        let day = nd.getDate() % 5;
        if (day === 0) {
            host = Day0
        } else if (day === 1) {
            host = Day1
        } else if (day === 2) {
            host = Day2
        } else if (day === 3){
            host = Day3
        } else if (day === 4){
            host = Day4
        } else {
            host = Day1
        }
        
        let url=new URL(event.request.url);
        url.hostname=host;
        let request=new Request(url,event.request);
        event. respondWith(
            fetch(request)
        )
    }
)
```
</details>

<details>
<summary>CloudFlare Workers一周轮换反代代码</summary>

```js
const Day0 = 'app0.herokuapp.com'
const Day1 = 'app1.herokuapp.com'
const Day2 = 'app2.herokuapp.com'
const Day3 = 'app3.herokuapp.com'
const Day4 = 'app4.herokuapp.com'
const Day5 = 'app5.herokuapp.com'
const Day6 = 'app6.herokuapp.com'
addEventListener(
    "fetch",event => {
    
        let nd = new Date();
        let day = nd.getDay();
        if (day === 0) {
            host = Day0
        } else if (day === 1) {
            host = Day1
        } else if (day === 2) {
            host = Day2
        } else if (day === 3){
            host = Day3
        } else if (day === 4) {
            host = Day4
        } else if (day === 5) {
            host = Day5
        } else if (day === 6) {
            host = Day6
        } else {
            host = Day1
        }
        
        let url=new URL(event.request.url);
        url.hostname=host;
        let request=new Request(url,event.request);
        event. respondWith(
            fetch(request)
        )
    }
)
```
</details>
