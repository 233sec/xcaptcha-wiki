# xCAPTCHA 技术白皮书

### 1. 验证码的发展

> ***1997年,***
> 雅虎个人邮箱深受机器注册机所害, 求助于卡梅隆大学Luis Ann. 后者设计了现今被广泛使用的图形验证码. 利用当时机器难以识别图形中的被加以干扰的文字, 而人眼容易识别来实现人机挑战.

---

> ***2003年,***
> 卡梅隆大学发展reCAPTCHA项目, 将CAPTCHA第一次做为公众服务提供; 而作为回报, 用户在完成验证使用服务的同时, 也帮助项目方识别了图书中机器难以辨认的文字, 帮助书籍的数字化.

---

> ***2005年,***
> 百度贴吧业务发展做大, 爆吧事件层出不穷. 被迫上线神兽验证码: “扭曲的汉字”, 在爆吧发生时, 用户需要完成图形所要求文字输入才能继续发帖.

---

> ***2009年,***
> Google收购reCAPTCHA, 2011年左右, 用户发现reCAPTCHA中出现了Google街景的路标、门牌号.

---

> ***2014年,***
> Google收购后的reCAPTCHA更新, 代号"noCAPTCHA", 用户完成一次简单的点击, 即可继续被识别为人类或者马上被回落至图形验证.

---


### 2. 历史出现过的验证码

> ***图形文字验证码, ***
>   最常见的, 输出的是什么你就输入什么, 不赘述
>
> ***语音验证码, ***
>   最早出现的地方不详, 但据了解, reCAPTCHA也有(过)AudioCaptcha
>
> ***计算验证码, ***
>   如一张图片输出 “5 - 3 = ?”, 你需要输入2才能通过挑战
>
> ***短信验证码,***
>   常用于需要绑定手机号的场景, 并不算严格意义上的图灵测试
>
> ***“找你妹”型验证码,***
>   如12306 “请找出下面的C罩杯”
>
> ***noCAPTCHA型验证码,***
>   如 阿里巴巴旗下的“数据风控验证码服务”、极验验证旗下的“GeeTest”、Google 的 “noCAPTCHA”、当然也包括我们的 xCAPTCHA

---


### 3. 我们采取的技术

> ***构建可信的前端环境***
>
> ***设备(浏览器)指纹***
>
> ***用户行为采集***
>
> ***POW工作量证明***
>
> ***后端高效架构***

---
> ***下面我们来分别介绍上述概念***

---

### 4. 技术解释

> **可信前端**
>
> 其实不难理解, 在第三方支付、网银场景就有可信环境, 如支付宝、财付通均采用过“安全控件”, 目的就是为了监控客户端环境, 一定程度上保证用户输入的密码不被钩子、插件截获；另外就是出于信任的考虑, 支付公司不可能信任浏览器的, 让我的用户在你的控件内输入密码, 假如你存储的方式易于窃取那就危及到我的用户了, 所以对不起, 这个输入框只能我自己定制.
> 对于xCAPTCHA 的挑战也是类似, 我们的用户要在浏览器完成验证, 我们就要保证用户的环境是没有“破解意向”的 (尝试模拟一个虚拟的可信环境对xCAPTCHA发起虚假请求). 如果存在, 即使不做惩罚, 我们也要能主动识别, 在出现其他异常时, 进行减分打击.
>
>
> xCAPTCHA 的可信前端主要做了以下工作:
> 1. 针对某些带调试功能的客户端, 进行特殊代码注入, 屏蔽断点.
> 2. 代码混淆.
> 3. 结合cookie-sticky加上设备指纹记录进行历史数据长驻.
> 4. 极端模式检测, 如devTool检测, 历史访问网站检测等等.

---

> **设备指纹**
> 
> 参照阿里云同业公共号“云誉”的文章, 我在这里做个简述:
> 设备指纹是代表你的机器、或环境、或客户端(浏览器)独一无二的标识.
>
> 常见的设备指纹算法(以下数据均基于假设):
> 你的电脑是 Windows, 那么你是68%之一
> 你的电脑是 Windows 8.1, 那么你是 5% * 68%之一
> 你的浏览器是 Chrome 51, 那么你是 12% * 5% * 68% 之一
> 你的 Chrome 能正确显示 “方正圆体” 字体, 那么你是 6% * 12% * 5% * 68% 之一
> 你的电脑安装了百度云管家, 那么你是3% * 6% * 12% * 5% * 68% 之一
> 你的 Canvas 显示的帆布指纹, 千里挑一, 那么你是 1% * 3% * 6% * 12% * 5% * 68% 之一
> 你在过去1个月内访问过乌云, Freebuf, 那么你是 0.5% * 1% * 3% * 6% * 12% * 5% * 68% 之一
> ...
>
> 按照上述各种特征, 已将将精度缩小到了亿分之一, 再配合别的一些特征分析, 足够匹配你当前使用浏览器的独一无二的指纹.

---

> **用户行为采集**
>
> 用户什么时候加载的JS? 
> 锚页面打开花了多久?
> 页面CSS加载了多久? 
> 是否开启了强制刷新?
> 鼠标移动轨迹是否固定在某个区域(按键精灵)?
> 用户验证分析, 是否接近固定的Interval ?
> ...
>
> 我们采集这些数据, 可以用来决策是真实用户还是脚本用户.
---

> **POW工作量证明**
>
> 假设一种情况
> 如果恶意客户端破解了可信前端, 而我们的后端服务器资源储备很有可能会被恶意流量所干扰. 这时候我们就需要引入POW技术进行垃圾流量清洗了.
>
> 算法细节:
> ```php
> <?php
> $a = mt_rand(100000, 9999999);
> $b = mt_rand(100000, 9999999);
> $c = time()
> $d = password_hash($a * $b * $c, 1);
> ```
>
> 然后告诉客户端 `$a`, `$c`, `$d`, 让客户端计算`$b`
> 由算法可知, 客户端需要循环计算最多`8999999`次, 这时间足够让大部分4核、8核CPU计算1~3秒钟. 普通用户不会在意, 而攻击用户却会崩溃, 因为请求成本一下增加了数百万倍.
>
> 这样, 在这场前后端争夺战中, 我们的`可信前端` + `POW`机制为我们的`challenge server`后端争取了足够的时间, 来分析绝对程度上真实的用户的行为.
---


> **后端高效架构**
>
>

---

