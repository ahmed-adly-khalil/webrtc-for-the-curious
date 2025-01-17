---
title: سیگنالینگ
type: docs
weight: 3
---

<div dir="rtl">

# سیگنالینگ WebRTC چیست؟
هنگامی که یک عامل WebRTC ایجاد می کنید، چیزی در مورد همتای دیگر نمی داند. هیچ ایده ای ندارد که قرار است با چه کسی ارتباط برقرار کند یا قرار است چه چیزی بفرستد!
سیگنالینگ راه‌اندازی اولیه است که تماس را ممکن می‌سازد. پس از رد و بدل شدن این مقادیر، عوامل WebRTC می توانند مستقیماً با یکدیگر ارتباط برقرار کنند.

پیام های سیگنالینگ فقط متن هستند. عوامل WebRTC اهمیتی نمی دهند که چگونه پیام ها منتقل می شوند. آنها معمولاً از طریق سوکت های وب به اشتراک گذاشته می شوند، اما همیشه اینگونه نیست.

## سیگنالینگ WebRTC چگونه کار می کند؟

WebRTC از یک پروتکل موجود به نام Session Description Protocol استفاده می کند. از طریق این پروتکل، دو عامل WebRTC تمام وضعیت مورد نیاز برای ایجاد یک اتصال را به اشتراک خواهند گذاشت. خود پروتکل برای خواندن و درک ساده است.
پیچیدگی این عمل از فهمیدن مقادیری است که توسط WebRTC پر می شود.

این پروتکل مختص WebRTC نیست. ابتدا پروتکل توضیحات جلسه را بدون حتی صحبت در مورد WebRTC یاد خواهیم گرفت. WebRTC از زیرمجموعه ای از پروتکل بهره می برد، بنابراین ما فقط می خواهیم آنچه را که نیاز داریم پوشش دهیم.
پس از درک پروتکل، ما نمونه استفاده شده در WebRTC را بررسی  می کنیم.

## پروتکل Session Description Protocol (SDP) چیست؟
پروتکل شرح جلسه در [RFC 8866](https://tools.ietf.org/html/rfc8866) معرفی شده است. این یک پروتکل کلید / مقدار با یک خط جدید پس از هر مقدار است. این همانند یک فایل INI خواهد بود.
شرح جلسه، حاوی هیچ یا بیشتر از یک توضیحات رسانه است. از نظر ذهنی می‌توانید آن را به‌عنوان شرح جلسه که حاوی مجموعه‌ای از توضیحات رسانه است، مدل‌سازی کنید.

توصیف رسانه معمولاً به یک جریان رسانه منفرد نگاشت می شود. بنابراین اگر می‌خواهید تماسی را با سه جریان ویدیویی و دو آهنگ صوتی توصیف کنید، باید پنج توضیح رسانه داشته باشید.

### نحوه خواندن SDP
هر خط در توضیحات جلسه با یک کاراکتر شروع می شود، این کلید شماست. سپس علامت مساوی به دنبال آن خواهد آمد. همه چیز بعد از آن علامت مساوی، مقدار است. پس از تکمیل مقدار، یک خط جدید خواهید داشت.

پروتکل Session Description تمام کلیدهای معتبر را تعریف می کند. شما فقط می توانید از حروف برای کلیدها همانطور که در پروتکل تعریف شده است استفاده کنید. این کلیدها همگی معنی قابل توجهی دارند که بعدا توضیح داده خواهد شد.

این گزیده توضیحات جلسه را در نظر بگیرید:

```
a=my-sdp-value
a=second-value
```

شما دو خط دارید. هر کدام با کلید `a`. خط اول دارای مقدار `my-sdp-value` است، خط دوم دارای مقدار `second-value` است.

### WebRTC فقط از برخی کلیدهای SDP استفاده می کند
تمام مقادیر کلیدی تعریف شده توسط پروتکل شرح جلسه توسط WebRTC استفاده نمی شود. فقط کلیدهای مورد استفاده در پروتکل ایجاد جلسه جاوا اسکریپت (JSEP) که در [RFC 8829](https://datatracker.ietf.org/doc/html/rfc8829) تعریف شده است، مهم هستند. هفت کلید زیر تنها کلیدهایی هستند که در حال حاضر باید بدانید:

* `v` - نسخه، باید برابر با 0 باشد.
* `o` - مبدا، حاوی شناسه منحصر به فردی است که برای مذاکره مجدد مفید است.
* `s` - نام جلسه، باید برابر با` -` باشد.
* `t` - زمان بندی، باید برابر با 0 0 باشد.
* `m` - توضیحات رسانه (` m = <media> <port> <proto> <fmt> ...`)، به تفصیل در زیر توضیح داده شده است.
* `a` - ویژگی ها، یک فیلد متن آزاد. این رایج ترین خط در WebRTC است.
* `c` - داده های اتصال، باید برابر با` IN IP4 0.0.0.0 باشد.

### توضیحات رسانه در شرح جلسه

شرح جلسه می تواند شامل تعداد نامحدودی از توضیحات رسانه باشد.

تعریف توصیف رسانه حاوی لیستی از قالب ها است. این قالب‌ها به انواع RTP Payload نگاشت می‌شوند. سپس کدک واقعی توسط یک ویژگی با مقدار `rtpmap` در توضیحات رسانه تعریف می شود.
اهمیت انواع RTP و RTP Payload بعداً در فصل رسانه مورد بحث قرار می گیرد. هر توصیف رسانه می تواند شامل تعداد نامحدودی از ویژگی ها باشد.

این گزیده توضیحات جلسه را به عنوان مثال در نظر بگیرید:

```
v=0
m=audio 4000 RTP/AVP 111
a=rtpmap:111 OPUS/48000/2
m=video 4000 RTP/AVP 96
a=rtpmap:96 VP8/90000
a=my-sdp-value
```

شما دو توصیف رسانه دارید، یکی از نوع صوتی با فرمت `111` و یکی از نوع ویدیویی با فرمت `96`. اولین توصیف رسانه ای تنها یک ویژگی دارد. این ویژگی نوع Payload '111' را به Opus ترسیم می کند.
شرح رسانه دوم دو ویژگی دارد. ویژگی اول، نوع Payload ا`96` را به عنوان VP8 ترسیم می کند، و ویژگی دوم فقط `my-sdp-value` است.

### مثال کامل

در ادامه تمام مفاهیمی که در مورد آنها صحبت کرده ایم را گرد هم آورده است. اینها همه ویژگی های پروتکل شرح جلسه است که WebRTC از آن استفاده می کند.
اگر می توانید این را بخوانید، می توانید هر شرح جلسه WebRTC را بخوانید!

```
v=0
o=- 0 0 IN IP4 127.0.0.1
s=-
c=IN IP4 127.0.0.1
t=0 0
m=audio 4000 RTP/AVP 111
a=rtpmap:111 OPUS/48000/2
m=video 4002 RTP/AVP 96
a=rtpmap:96 VP8/90000
```


* کلیدهای  `v`,` o`, `s`,` c`, `t` تعریف شده اند، اما بر جلسه WebRTC تأثیری ندارند.
* دو شرح رسانه ای دارید. یکی از نوع `صوتی` و یکی از نوع `ویدئو`.
* هر کدام از آن ها یک ویژگی دارند. این ویژگی جزئیات پایپ لاین RTP را که در فصل `ارتباطات رسانه ای` مورد بحث قرار گرفته است، پیکربندی می کند.

## چگونه پروتکل شرح جلسه و WebRTC با هم کار می کنند

بخش بعدی این پازل درک این که چگونه WebRTC از پروتکل شرح جلسه استفاده می کند می باشد.

### پیشنهادات`Offers` و پاسخ ها`Answers` چیست؟

WebRTC از مدل پیشنهاد/پاسخ استفاده می کند. همه اینها به این معنی است که یک نماینده WebRTC یک `پیشنهاد` برای شروع یک تماس ارائه می دهد، و سایر نمایندگان WebRTC در صورتی که مایل به پذیرش آنچه ارائه شده است `پاسخ می دهند`.

این به پاسخ دهنده فرصتی می دهد تا کدک های پشتیبانی نشده در توضیحات رسانه را رد کند. به این ترتیب دو همتا می توانند بفهمند که مایل به تبادل چه قالب هایی هستند.

### مفهوم Transceivers برای ارسال و دریافت است

واژه Transceivers یک مفهوم خاص WebRTC است که در API خواهید دید. کاری که انجام می دهد این است که `توضیحات رسانه` را در API جاوا اسکریپت قرار می دهد. هر توصیف رسانه ای تبدیل به یک Transceiver می شود.
هر بار که یک Transceiver ایجاد می کنید، یک توضیح رسانه جدید به توضیحات جلسه محلی اضافه می شود.

هر توضیح رسانه در WebRTC یک ویژگی direction دارد. این به یک نماینده WebRTC اجازه می دهد تا اعلام کند "من قصد دارم این کدک را برای شما ارسال کنم، اما حاضر نیستم چیزی را پس بگیرم". چهار مقدار معتبر وجود دارد:

* `send`
* `recv`
* `sendrecv`
* `inactive`


### مقادیر SDP استفاده شده توسط WebRTC

این لیستی از برخی ویژگی های رایج است که در توضیحات جلسه از یک عامل WebRTC مشاهده خواهید کرد. بسیاری از این مقادیر، زیرسیستم هایی را کنترل می کنند که ما هنوز در مورد آنها صحبت نکرده ایم.

##### `group:BUNDLE`
باندلینگ عملی است برای اجرای چندین نوع ترافیک روی یک اتصال. برخی از پیاده سازی های WebRTC از یک اتصال اختصاصی از طریق جریان رسانه استفاده می کنند که باندلینگ باید در اولویت باشد.

##### `fingerprint:sha-256`
این هش گواهی است که یک همتا برای DTLS استفاده می کند. پس از تکمیل دست دادن DTLS، آن را با گواهی واقعی مقایسه می کند تا تأیید کند که با کسی که انتظار دارید در ارتباط هستید.

##### `setup:`
این رفتار عامل DTLS را کنترل می کند. این مشخص می کند که آیا پس از اتصال، ICE به عنوان مشتری یا سرور اجرا می شود. مقادیر ممکن عبارتند از:

* `setup:active` - به عنوان سرویس گیرنده DTLS اجرا شود.
* `setup:passive` - به عنوان سرور DTLS اجرا شود.
* `setup: actpass` - از دیگر نماینده WebRTC بخواهد انتخاب کند.

##### `ice-ufrag`
این مقدار قطعه کاربر برای عامل ICE است. برای احراز هویت ترافیک ICE استفاده می شود.

##### `ice-pwd`
این رمز عبور عامل ICE است. برای احراز هویت ترافیک ICE استفاده می شود.

##### `rtpmap`
این مقدار برای نگاشت یک کدک خاص به نوع RTP Payload استفاده می شود. انواع بار(Payload) ثابت نیستند، بنابراین برای هر تماس پیشنهاد دهنده انواع بار(Payload) برای هر کدک را تعیین می کند.

##### `fmtp`
مقادیر اضافی را برای یک نوع محموله تعریف می کند. این برای برقراری ارتباط با یک نمایه ویدیویی خاص یا تنظیمات رمزگذار مفید است.

##### `candidate`
این یک نامزد ICE است که از عامل ICE می آید. این یکی از آدرس‌های احتمالی است که WebRTC Agent در آن موجود است. این موارد در فصل بعدی به طور کامل توضیح داده می شود.

##### `src`
منبع همگام سازی (SSRC) یک مسیر جریان رسانه واحد را تعریف می کند.

`label` شناسه این جریان فردی است. `mslabel` شناسه کانتینری است که می‌تواند چندین جریان درون آن داشته باشد.

### مثالی از توضیحات جلسه WebRTC
در زیر شرح کامل جلسه(SDP) ایجاد شده توسط یک کلاینت WebRTC است:

```
v=0
o=- 3546004397921447048 1596742744 IN IP4 0.0.0.0
s=-
t=0 0
a=fingerprint:sha-256 0F:74:31:25:CB:A2:13:EC:28:6F:6D:2C:61:FF:5D:C2:BC:B9:DB:3D:98:14:8D:1A:BB:EA:33:0C:A4:60:A8:8E
a=group:BUNDLE 0 1
m=audio 9 UDP/TLS/RTP/SAVPF 111
c=IN IP4 0.0.0.0
a=setup:active
a=mid:0
a=ice-ufrag:CsxzEWmoKpJyscFj
a=ice-pwd:mktpbhgREmjEwUFSIJyPINPUhgDqJlSd
a=rtcp-mux
a=rtcp-rsize
a=rtpmap:111 opus/48000/2
a=fmtp:111 minptime=10;useinbandfec=1
a=ssrc:350842737 cname:yvKPspsHcYcwGFTw
a=ssrc:350842737 msid:yvKPspsHcYcwGFTw DfQnKjQQuwceLFdV
a=ssrc:350842737 mslabel:yvKPspsHcYcwGFTw
a=ssrc:350842737 label:DfQnKjQQuwceLFdV
a=msid:yvKPspsHcYcwGFTw DfQnKjQQuwceLFdV
a=sendrecv
a=candidate:foundation 1 udp 2130706431 192.168.1.1 53165 typ host generation 0
a=candidate:foundation 2 udp 2130706431 192.168.1.1 53165 typ host generation 0
a=candidate:foundation 1 udp 1694498815 1.2.3.4 57336 typ srflx raddr 0.0.0.0 rport 57336 generation 0
a=candidate:foundation 2 udp 1694498815 1.2.3.4 57336 typ srflx raddr 0.0.0.0 rport 57336 generation 0
a=end-of-candidates
m=video 9 UDP/TLS/RTP/SAVPF 96
c=IN IP4 0.0.0.0
a=setup:active
a=mid:1
a=ice-ufrag:CsxzEWmoKpJyscFj
a=ice-pwd:mktpbhgREmjEwUFSIJyPINPUhgDqJlSd
a=rtcp-mux
a=rtcp-rsize
a=rtpmap:96 VP8/90000
a=ssrc:2180035812 cname:XHbOTNRFnLtesHwJ
a=ssrc:2180035812 msid:XHbOTNRFnLtesHwJ JgtwEhBWNEiOnhuW
a=ssrc:2180035812 mslabel:XHbOTNRFnLtesHwJ
a=ssrc:2180035812 label:JgtwEhBWNEiOnhuW
a=msid:XHbOTNRFnLtesHwJ JgtwEhBWNEiOnhuW
a=sendrecv
```

این چیزی است که ما از این پیام می دانیم:

* دو بخش رسانه داریم، یکی صوتی و دیگری تصویری.
* هر دوی آنها فرستنده گیرنده(Transceiver) `sendrecv` هستند. ما در حال دریافت دو جریان هستیم و می توانیم دو جریان را بفرستیم.
* ما جزئیات ICE Candidates و Authentication را داریم، بنابراین می‌توانیم برای اتصال تلاش کنیم.
* ما یک اثر انگشت گواهی داریم، بنابراین می توانیم تماس ایمن داشته باشیم.

### موضوعات بیشتر
در نسخه های بعدی این کتاب، به موضوعات زیر نیز پرداخته خواهد شد:

* مذاکره مجدد (Renegotiation)
* پخش همزمان (Simulcast)
</div>
