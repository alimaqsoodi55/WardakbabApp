# Wardak Baba WebView (راه‌حل بنیادی و مطمئن)

این پروژه اپ اندروید شماست که وب‌سایت را در **WebView** باز می‌کند. هدف: حل اساسی مشکل «App not installed» با ساخت صحیح و CI.

## 1) تنظیم‌های اولیه
- `app/src/main/res/values/strings.xml` → `base_url` را به لینک سایت تغییر دهید.
- `app/build.gradle` → `applicationId` را به پکیج‌نیم یکتا (مثل `com.wardakbaba.app`) تغییر دهید.

## 2) بیلد خودکار بدون Android Studio (GitHub Actions)
دو ورک‌فلو داخل `.github/workflows/` قرار دارد:
- **Android Debug APK** → خروجی `app-debug.apk` برای تست و توزیع دستی
- **Android Release APK (Signed)** → خروجی `app-release.apk` امضاشده

### اجرای Debug
1. پروژه را در یک ریپوی گیت‌هاب قرار دهید.
2. به تب **Actions** بروید و **Android Debug APK** را Run کنید.
3. فایل `app-debug.apk` را از بخش **Artifacts** دانلود کنید.

### اجرای Release (امضا شده)
Secrets لازم (Settings → Secrets → Actions):
- `ANDROID_KEYSTORE_BASE64` : فایل keystore به Base64
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_KEY_ALIAS`
- `ANDROID_KEY_PASSWORD`

ساخت keystore (یک‌بار):
```bash
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias wardak
base64 my-release-key.jks > keystore.b64
# محتویات keystore.b64 را در Secret بنام ANDROID_KEYSTORE_BASE64 قرار دهید.
```

سپس ورک‌فلو **Android Release APK (Signed)** را اجرا کنید.

## 3) نکات نصب
- اگر نسخه‌ای با امضای دیگر نصب بوده، ابتدا Uninstall کنید.
- «Install unknown apps» را برای مرورگر دانلودکننده فعال کنید.
- برای میزبانی APK:
**Nginx**
```nginx
types { application/vnd.android.package-archive apk; }
add_header X-Content-Type-Options nosniff;
```
**Apache**
```apache
AddType application/vnd.android.package-archive .apk
Header set X-Content-Type-Options "nosniff"
```

## 4) Play Store (اختیاری)
- `./gradlew bundleRelease` → فایل `.aab`
- صفحهٔ Privacy Policy روی سایت

## 5) سفارشی‌سازی
- آیکن: `mipmap-*/ic_launcher*`
- اسپلش: `res/drawable/splash_screen.xml`
- هندل لینک‌ها (tel/mailto/whatsapp) آماده است.

با این ساختار، بیلد/امضا دست خودتان است، خروجی **یونیورسال** و سازگار با arm/arm64 می‌گیرید و مشکل نصب حل می‌شود.