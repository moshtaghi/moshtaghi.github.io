---
layout: post
title:  "حل اساسی مشکل CORS در Apache"
date:   2017-07-20 12:10:29 +0430
comments: true
tags:
    - apache
    - cors
---
خوب در ابتدا اگر نمی‌دونید CORS چیست، باید بگم که [cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) .

حالا مشکل شخص بنده اونجایی پیش اومد که برای پیاده‌سازی web ui پروژه‌ای و ارتباط با API اون از AngularJS استفاده کردم و با پیام خطای cors مواجه شدم. کمی جستجو به [این](https://enable-cors.org/server_apache.html) ختم شد تا متوجه شم که باید مقدار Access-Control-Allow-Origin رو در آپاچی (فایل کانفیگ vhost یا حتی فایل htaccess موجود در webroot پروژه) تغییر بدم. یعنی کدی مشابه زیر:

```Apache
Header set Access-Control-Allow-Origin "*"
```

این خط به این معناست که وب‌سرور یا vhost شما درخواست‌ها رو از هر مبدایی پذیرا باشه.

قاعدتا می‌بایست بعد از این تغییر، آپاچی را reload یا restart کنیم اما این کافی نیست! چرا که AngularJS یا لایبری‌های مشابه قبل از درخواست اصلی که معمولا با متد‌های مرسوم همچون GET یا POST یا … هست، یک درخواست با متد OPTION می‌زنند تا از معتبر بودن درخواست اصلی اطمینان حاصل کنن. [اطلاعات بیشتر در این خصوص](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS).

پس ما بعد از ارسال درخواست با متد OPTION پیام خطای Method Not Allowed رو دریافت می‌کنیم. که می‌شه این مشکل هم با خط زیر حل کرد!

```Apache
Header set Access-Control-Allow-Methods "*"
```

همونطور که می‌دونید علامت «*» به معنی «همه» است، اما اینجا از این خبرا نیست! سر کار نرین مثل من! اینجا یعنی همه بجز OPTION. در واقع متد OPTION حکم «ناصر» رو داره در بین برادرانش!

پس نیاز هست تا تمامی متد‌های مورد نیازمون رو دستی اضافه کنیم. به این شکل

```Apache
Header set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"
```

چند خط دیگر را اضافه می‌کنیم به این صورت

```Apache
Header set Access-Control-Allow-Origin "*"
Header set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"
Header set Access-Control-Max-Age "1000"
Header set Access-Control-Allow-Headers "x-requested-with, Content-Type, origin, authorization, accept, client-security-token"

RewriteEngine On
RewriteCond %{REQUEST_METHOD} OPTIONS
RewriteRule ^(.*)$ $1 [R=200,L]
```

نکات لازم در مورد خطوط بالا اینکه ۳ خط پایانی برای پاسخ «200 SUCCESS» دادن به تمامی درخواست‌های OPTION است.

اگر مجدد آپاچی را reload یا restart کنیم و درخواست جدیدی ارسال کنیم باز هم با خطای همیشگی روبرو می‌شویم! یعنی این:

```Apache
XMLHttpRequest cannot load http://my.example.dev/setting/1316. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://mydevsite.dev' is therefore not allowed access. 
```

این خطا یعنی ترتیب اجرای دستورهای ما بروی درخواست‌ها درست نیست، یا به عبارتی HEADERهای ست شده پس از ریدایرکت از بین رفتن! برای رفع این مشکل می‌بایست فلَگ «always» را برای تمامی خطوط بالایی اضافه کنیم. و در نهایت کانفیگی مشابه زیر خواهیم داشت:

```Apache
Header always set Access-Control-Allow-Origin "*"
Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"
Header always set Access-Control-Max-Age "1000"
Header always set Access-Control-Allow-Headers "x-requested-with, Content-Type, origin, authorization, accept, client-security-token"

# Added a rewrite to respond with a 200 SUCCESS on every OPTIONS request
RewriteEngine On
RewriteCond %{REQUEST_METHOD} OPTIONS
RewriteRule ^(.*)$ $1 [R=200,L]
```

الان همه چیز به درستی کار می‌کند و می‌توانید با خیال راحت به کار اصلی خود برسید. اما راه حل بهتری هم وجود داشت و آن استفاده از nginx بجای apache است.