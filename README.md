# نمونه B4A پوش‌پنل

پروژه نمونه B4A برای اتصال کتابخانه پوش PushPanel.

کتابخانه: `ir.push-panel:push-sdk:1.7.2` از MavenCentral

> این راهنما بر اساس API نیتیو SDK نوشته شده و در این محیط بیلد نشده است.

## ۱. اضافه کردن کتابخانه

1. فایل `push-sdk-1.7.2.aar` را از MavenCentral دانلود کن.
2. در B4A جدید، aar را در پوشه libs اضافه (AdditionalLibs) بگذار و در پروژه:

```basic
#AdditionalJar: push-sdk-1.7.2.aar
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
    sdk.InitializeStatic("ir.pushpanel.sdk.PushSdk")
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

> در نسخه 1.7.2 به `handleIntent` و کد جداگانه برای اکتیویتی اسپلش نیازی نیست؛ فقط `init` کافی است.

## ۴. فایربیس (برای دریافت واقعی پوش — اجباری)

بدون این مرحله توکن FCM ساخته نمی‌شود و پوشی دریافت نمی‌کنی (کامپایل موفق می‌شود ولی خبری از پوش نیست).

۱. فایل `google-services.json` را از کنسول فایربیس بگیر و در مسیر پروژه بگذار؛ `package_name` داخل آن باید با پکیج برنامه یکی باشد.

۲. یکپارچه‌سازی Firebase را از منوی Firebase در B4A کامل کن تا `google-services.json` واقعاً داخل APK پردازش شود (معادل مرحله پلاگین `google-services` در پروژه‌های گریدلی). اگر این اتصال ناقص بماند، json فقط یک فایل کنار پروژه است و هیچ اثری ندارد.

۳. برای اطمینان بعد از نصب روی دیوایس، لاگ را با فیلتر `PushSDK` ببین — باید ثبت توکن را نشان بدهد. اگر ثبت توکن دیده نشد، برگرد به قدم ۱ و ۲.

## ۵. بیلد

`Project > Compile & Run` — اگر خطای Dex/Multidex گرفتی، Multidex را فعال کن و نسخه وابستگی‌ها را با هم هماهنگ نگه دار.
