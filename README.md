# نمونه B4A پوش‌پنل

پروژه نمونه B4A برای اتصال کتابخانه پوش PushPanel.

کتابخانه: `ir.push-panel:push-sdk:1.8.3` از MavenCentral

> این راهنما بر اساس API نیتیو SDK نوشته شده و در این محیط بیلد نشده است.

## ۱. اضافه کردن کتابخانه

1. فایل `push-sdk-1.8.3.aar` را از MavenCentral دانلود کن.
2. در B4A جدید، aar را در پوشه libs اضافه (AdditionalLibs) بگذار و در پروژه:

```basic
#AdditionalJar: push-sdk-1.8.3.aar
```

اگر نسخه B4A تو فقط `jar` قبول می‌کند، `classes.jar` داخل aar را بیرون بکش و به جایش معرفی کن و پوشه `res` آن را با `#AdditionalRes` اضافه کن.

وابستگی‌های داخلی (messaging ،okhttp ،coroutines) را هم از Maven بگیر و با `#AdditionalJar` اضافه کن، وگرنه در ران‌تایم `ClassNotFoundException` می‌گیری.

## ۲. دسترسی‌ها و منیفست (Manifest Editor)

```basic
AddPermission(android.permission.INTERNET)
AddPermission(android.permission.POST_NOTIFICATIONS)
```

## ۳. مقداردهی

در `Starter` (فقط یک بار):

```basic
Sub Service_Create
    Dim sdk As JavaObject
    sdk.InitializeStatic("ir.pushpanel.sdk.PushPanel")
    sdk.RunMethod("init", Array(GetContext))
End Sub
```

در `Main` (اکتیویتی لانچر):

```basic
Sub Activity_Create(FirstTime As Boolean)
    Activity.LoadLayout("Main")
    RequestNotificationPermission
End Sub

Sub RequestNotificationPermission
    Dim p As RuntimePermissions
    If p.Check(p.PERMISSION_POST_NOTIFICATIONS) = False Then
        p.Request(p.PERMISSION_POST_NOTIFICATIONS)
    End If
End Sub
```

`GetContext` همان `Context` اکتیویتی است که `init` می‌خواهد.

> در نسخه 1.8.3 نقطه ورود به `PushPanel` تغییر نام داده (قبلاً `PushSdk`) و به `handleIntent` و کد جداگانه برای اکتیویتی اسپلش نیازی نیست؛ فقط `init` کافی است.

## ۴. فایربیس (برای دریافت واقعی پوش — اجباری)

بدون این مرحله توکن FCM ساخته نمی‌شود و پوشی دریافت نمی‌کنی (کامپایل موفق می‌شود ولی خبری از پوش نیست).

۱. فایل `google-services.json` را از کنسول فایربیس بگیر و در مسیر پروژه بگذار؛ `package_name` داخل آن باید با پکیج برنامه یکی باشد.

۲. یکپارچه‌سازی Firebase را از منوی Firebase در B4A کامل کن تا `google-services.json` واقعاً داخل APK پردازش شود (معادل مرحله پلاگین `google-services` در پروژه‌های گریدلی). اگر این اتصال ناقص بماند، json فقط یک فایل کنار پروژه است و هیچ اثری ندارد.

۳. برای اطمینان بعد از نصب روی دیوایس، لاگ را با فیلتر `PushSDK` ببین — باید ثبت توکن را نشان بدهد. اگر ثبت توکن دیده نشد، برگرد به قدم ۱ و ۲.

## ۵. سرویس فایربیس اختصاصی (اختیاری — فقط اگر کتابخانه پوش دیگری هم داری)

FCM در هر اپ فقط به **یک** سرویس پیام تحویل می‌دهد. اگر کتابخانه دیگری هم سرویس فایربیس خودش را دارد، باید فقط یکی از سرویس‌ها پیام بگیرد و پیام‌ها بین کتابخانه‌ها تقسیم شوند — وگرنه پیام‌ها به دست یکی از آن‌ها نمی‌رسد.

اصل کار (مثل نیتیو): سرویس داخلی SDK (`ir.pushpanel.sdk.PushMessagingService`) با `tools:node="remove"` از مانیفست حذف می‌شود و سرویس واحد، همه پیام‌ها را با این دو متد به SDK پاس می‌دهد — پیام‌هایی که مال پنل نیستند نادیده گرفته می‌شوند (مارکر `pushpanel=pushpanel`):

```basic
' معادل نیتیو:
' PushPanel.forwardMessage(context, remoteMessage)
' PushPanel.forwardToken(context, token)
Dim sdk As JavaObject
sdk.InitializeStatic("ir.pushpanel.sdk.PushPanel")
sdk.RunMethod("forwardMessage", Array(GetContext, msg))
sdk.RunMethod("forwardToken", Array(GetContext, token))
```

نکته: `init` (بخش ۳) همچنان لازم است؛ فوروارد قبل از init نادیده گرفته می‌شود. اگر کتابخانه دیگری نداری، این بخش را رد کن — سرویس داخلی SDK به‌صورت پیش‌فرض کار می‌کند.

## ۶. بیلد

`Project > Compile & Run` — اگر خطای Dex/Multidex گرفتی، Multidex را فعال کن و نسخه وابستگی‌ها را با هم هماهنگ نگه دار.
