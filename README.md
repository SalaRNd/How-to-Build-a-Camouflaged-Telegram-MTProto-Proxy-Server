# چگونه یک سرور پروکسی تلگرام MTProto استتار شده بسازیم
<h4>پیش نیازها:</h4>

1. یک سرور مجازی (VPS) که اوبونتو 20.04 را اجرا می کند. می توانید از هر ارائه دهنده vps یک سرور مجازی اجاره کنید.
2. یک دامنه به نام خودتان، که ممکن است رایگان یا پولی باشد.
3. یک رکورد DNS نوع A در کلودفلر یا هر ارائه دهنده دیگر DNS دیگر بسازید، که به آدرس IP شما اشاره میکند. (شما باید ابتدا رکورد های DNS را در دامنه خود تنظیم کنید)

نرم افزار <a href="https://uploadb.me/direct/cjlbd3c6vuwm/CC_%208.0l.rar.html" target="_blank"> PUTTY </a> را دانلود کنید

<h2>قدم اول</h2>

اگر به عنوان کاربر root وارد نشده ایید نیاز هست که مراحل زیر را در سرور خود انجام دهید:

اگر از قبل رمز عبور root را نمی‌دانید، آن را با دستور زیر تنظیم کنید:

<pre>sudo passwd root</pre>

از شما خواسته می شود رمز عبور خود را وارد کنید(باید رمز عبور غیر root خود را وارد کنید):

<pre>[sudo] password for amy:</pre>

سپس از شما خواسته می شود که یک رمز عبور جدید برای root وارد کنید:

<pre>New password:</pre>

نیاز هست که این پسورد را مجددا تایید کنید

اگر همه چیز خوب باشد، یک پیام بازخورد دریافت خواهید کرد:

<pre>passwd: password updated successfully</pre>

برای تغییر دادن به root باید از دستور زیر استفاده کنید

<pre>su -</pre>

از شما خواسته می شود رمز عبوری را که چند لحظه پیش تنظیم کرده اید برای root وارد کنید

متوجه خواهید شد که خط فرمان شما از علامت دلار ($) به علامت هش (#) تغییر می کند

اکنون که root هستید، نیازی به پیشوند دستورات ممتاز با sudo ندارید.

سرور خود را با دستور زیر بروز و اپدیت کنید

<pre>apt update && apt upgrade -y</pre>

<h2>قدم دوم</h2>

فعال کردن فایروال برای افزایش امنیت سرور به خصوص پورت ssh

در بسیاری از سیستم های اوبونتو، ufw قبلاً نصب شده است. بررسی کنید که ufw روی سیستم شما نصب شده باشد:

<pre>apt list ufw</pre>

شما باید این پیام را ببینید:

<pre>ufw/focal-updates,focal-updates,now 0.36-6ubuntu1 all [installed]</pre>

اگر پیامی مبنی بر اینکه ufw قبلاً نصب شده است را نمی بینید، اکنون بسته ufw را نصب کنید:

<pre>apt install ufw</pre>

اکنون فایروال را راه اندازی خواهید کرد. اگر همیشه آدرس IP شما در یک رنج خاصی میباشد، دسترسی SSH را فقط به آن آدرس IP ها محدود کنید. به عنوان مثال، اگر آدرس IP شما 1.1.1.1/30 است، برای امنیت بیشتر سرور خود، دستور زیر را وارد کنید:

<pre>ufw allow from 12.12.12.0/24 to any port 22 proto tcp</pre>

در غیر اینصورت میتوانید از دستورات زیر برای باز کردن پورت های 22، 80 و 443 و سایر پورت ها استفاده کنید:

<pre>ufw allow ssh
ufw allow http
ufw allow https
ufw allow to any port 2083 proto tcp</pre>

فعال کردن UFW:
<pre>ufw enable</pre>

وضعیت فایروال خود را بررسی کنید:

<pre>ufw status</pre>

یک نمایشگر وضعیت فایروال را خواهید دید که به شکل زیر است:

<pre>Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
2083/tcp                   ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
2083/tcp (v6)              ALLOW       Anywhere (v6)</pre>

<h2>قدم سوم</h2>

ایجاد یک وب سایت استتار

ابتدا Nginx را نصب کرده و گواهی SSL دریافت می کنید. هدف Nginx این است که سرور پروکسی MTProto شما را مخفی کند و باعث افزایش امنیت پروکسی شود.

<pre>apt install nginx -y</pre>

اکنون فایل ایجاد شده در مسیر /etc/nginx/sites-available/default را با ویرایشگر nano تغییر دهید:

<pre>nano /etc/nginx/sites-available/default</pre>

خط حاوی server_name _; را پیدا کنید و مطابق دستور زیر نام دامنه خود را در ادامه آن قرار دهید:

<pre>server_name your_domain;</pre>

تغییرات را ذخیره کنید و خارج شوید

سپس Nginx را با پیکربندی اصلاح شده مجددا راه اندازی کنید:

<pre>systemctl restart nginx</pre>

شما از snap برای نصب certbot، استفاده خواهید کرد. ابتدا core snap را نصب کنید:

<pre>snap install core</pre>

اکنون از snap برای نصب classic certbot استفاده کنید:

<pre>snap install --classic certbot</pre>

یک پ.شه جدید ایجاد کنید و فایل های certbot را در دایرکتوری جدید انتقال قرار دهید برای اینکار از دستور زیر استفاده کنید:

<pre>ln -s /snap/bin/certbot /usr/bin/certbot</pre>

اکنون می توانید certbot را فراخوانی کنید تا گواهی SSL خود را دریافت کنید:

<pre>certbot --nginx </pre>

پس از گرفتن سرتیفیکیت برای وب سایت خود را تست کنید. در یک مرورگر، از نام دامنه کاملاً واجد شرایط خود (https://your_domain) بازدید کنید. شما باید خوش آمدید به nginx را ببینید!

به صورت اختیاری، اکنون می توانید مقداری محتوا (HTML، CSS، وردپرس و غیره) به وب سایت خود در /var/www/html اضافه کنید تا واقع گرایی آن را افزایش دهید.



 



