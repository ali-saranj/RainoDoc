## معرفی
کلاس `CacheManager` در یک برنامه اندرویدی برای مدیریت کش منابع وب (مانند فایل‌های JavaScript، CSS، HTML و فونت‌ها) در WebView طراحی شده است. هدف اصلی این کلاس، کاهش درخواست‌های شبکه‌ای، بهبود سرعت بارگذاری منابع، و امکان دسترسی آفلاین به برخی منابع است. این کلاس از استراتژی‌های "اولویت کش" و "اولویت شبکه" استفاده می‌کند و قابلیت‌هایی مانند مدیریت فایل‌های موقت، حذف فایل‌های کش، و پشتیبانی از پاسخ‌های HTTP را فراهم می‌کند.

---

## تحلیل خط به خط کد

### **بخش ۱: تعریف پکیج و ایمپورت‌ها**
```kotlin
package com.today.connect

import android.annotation.SuppressLint
import android.content.Context
import android.util.Log
import android.webkit.MimeTypeMap
import android.webkit.URLUtil
import android.webkit.WebResourceRequest
import android.webkit.WebResourceResponse
import com.today.connect.ui.activity.main.MainActivity
import java.io.*
import java.net.HttpURLConnection
import java.net.URL
import java.security.KeyManagementException
import java.security.NoSuchAlgorithmException
import java.security.SecureRandom
import java.security.cert.X509Certificate
import java.util.*
import javax.net.ssl.HttpsURLConnection
import javax.net.ssl.SSLContext
import javax.net.ssl.TrustManager
import javax.net.ssl.X509TrustManager
```
- **توضیح**:
  - کد در پکیج `com.today.connect` تعریف شده است.
  - کتابخانه‌های استاندارد اندروید (مانند `Context`, `WebResourceRequest`, `WebResourceResponse`) برای کار با WebView و مدیریت فایل‌ها وارد شده‌اند.
  - کتابخانه‌های جاوا مانند `java.io.*` برای عملیات ورودی/خروجی و `java.net.*` برای اتصالات شبکه‌ای استفاده شده‌اند.
  - برای مدیریت SSL و HTTPS، کلاس‌های `javax.net.ssl.*` وارد شده‌اند.
  - کلاس `MainActivity` از پروژه وارد شده است که نشان می‌دهد این کلاس به نمونه‌ای از `MainActivity` برای دسترسی به فایل‌های داخلی برنامه نیاز دارد.
  - استفاده از `@SuppressLint` برای سرکوب هشدارهای مربوط به مسائل امنیتی (مانند غیرفعال کردن بررسی گواهینامه SSL).

---

### **بخش ۲: تعریف کلاس و متغیرها**
```kotlin
internal class CacheManager(activity: MainActivity) {
    private val mActivity: MainActivity
    init {
        mActivity = activity
        disableSSLCertificateChecking()
    }
```
- **توضیح**:
  - کلاس `CacheManager` به‌صورت `internal` تعریف شده است، یعنی فقط در ماژول فعلی قابل دسترسی است.
  - یک متغیر خصوصی `mActivity` از نوع `MainActivity` برای ذخیره نمونه‌ای از اکتیویتی اصلی تعریف شده است.
  - در بلوک `init`:
    - متغیر `mActivity` با نمونه ارسالی از `MainActivity` مقداردهی می‌شود.
    - متد `disableSSLCertificateChecking()` فراخوانی می‌شود تا بررسی گواهینامه‌های SSL غیرفعال شود (این بخش بعداً توضیح داده می‌شود).

---

### **بخش ۳: متد اصلی `tryHandle`**
```kotlin
fun tryHandle(request: WebResourceRequest): WebResourceResponse? {
    if (request.method != "GET") return null
    val url = request.url.toString()
    if (isNetworkOnly(url)) return null
    val mimetype = getMimeType(url)
    val hash = request.url.host + request.url.pathSegments.joinToString("/")
    val safeHash = hash.replace(Regex("[^a-zA-Z0-9_]"), "_")
    val fileName = "$safeHash-${URLUtil.guessFileName(url, null, mimetype)}"
    if (isNetworkStream(fileName)) return null
    return if (!isNetworkFirst(fileName)) {
        if (fileExists(fileName)) createWebResourceResponseFromFile(
            fileName,
            mimetype
        ) else createWebResourceResponse(request, fileName)
    } else {
        val response = createWebResourceResponse(request, fileName)
        if (response != null) return response
        if (fileExists(fileName)) createWebResourceResponseFromFile(
            fileName,
            mimetype
        ) else null
    }
}
```
- **توضیح**:
  - **هدف**: این متد نقطه ورود برای مدیریت درخواست‌های WebView است و تصمیم می‌گیرد که آیا پاسخ از کش محلی ارائه شود یا از شبکه دریافت و کش شود.
  - **مراحل**:
    1. **بررسی نوع درخواست**:
       - اگر درخواست از نوع GET نباشد (مثلاً POST)، متد `null` برمی‌گرداند و WebView مستقیماً درخواست را از شبکه پردازش می‌کند.
    2. **استخراج URL**:
       - URL درخواست به رشته تبدیل می‌شود (`request.url.toString()`).
    3. **بررسی مسیرهای بدون کش**:
       - متد `isNetworkOnly(url)` بررسی می‌کند که آیا URL شامل مسیرهایی مانند `/TokenGenerator` یا `/AuthService` است. اگر باشد، `null` برمی‌گرداند تا درخواست مستقیماً از شبکه پردازش شود.
    4. **تعیین نوع MIME**:
       - متد `getMimeType(url)` نوع MIME فایل (مانند `text/html` یا `application/javascript`) را بر اساس پسوند URL تعیین می‌کند.
    5. **ساخت نام فایل**:
       - یک هش از هاست و مسیر URL ساخته می‌شود (`host + pathSegments.joinToString("/")`).
       - با استفاده از `Regex("[^a-zA-Z0-9_]")`، کاراکترهای غیرمجاز به `_` تبدیل می‌شوند تا نام فایل امن باشد.
       - نام فایل نهایی با ترکیب `safeHash` و نام فایل حدس‌زده‌شده توسط `URLUtil.guessFileName` ساخته می‌شود.
    6. **بررسی فایل‌های استریم**:
       - متد `isNetworkStream(fileName)` بررسی می‌کند که آیا فایل از نوع استریم (مانند `.mp3` یا `.opus`) است. اگر باشد، `null` برمی‌گرداند تا کش نشود.
    7. **انتخاب استراتژی کش**:
       - اگر فایل از نوع "اولویت شبکه" نباشد (`!isNetworkFirst(fileName)`):
         - اگر فایل در کش موجود باشد (`fileExists(fileName)`), از کش خوانده می‌شود (`createWebResourceResponseFromFile`).
         - در غیر این صورت، از شبکه دریافت و کش می‌شود (`createWebResourceResponse`).
       - اگر فایل از نوع "اولویت شبکه" باشد (مانند `.ver` یا `.json`):
         - ابتدا سعی می‌کند از شبکه دریافت کند (`createWebResourceResponse`).
         - اگر دریافت موفق بود، پاسخ را برمی‌گرداند.
         - اگر دریافت ناموفق بود و فایل در کش موجود باشد، از کش خوانده می‌شود.
         - اگر هیچ‌کدام ممکن نبود، `null` برمی‌گرداند.

---

### **بخش ۴: ایجاد پاسخ از فایل کش‌شده (`createWebResourceResponseFromFile`)**
```kotlin
private fun createWebResourceResponseFromFile(
    fileName: String,
    mimeType: String?
): WebResourceResponse? {
    val inputStream: InputStream = try {
        mActivity.openFileInput(fileName)
    } catch (e: FileNotFoundException) {
        e.printStackTrace()
        return null
    }
    val encoding = "UTF-8"
    val statusCode = 200
    val reasonPhase = "OK"
    val responseHeaders: MutableMap<String, String> = HashMap()
    responseHeaders["Access-Control-Allow-Origin"] = "*"
    val metaFile = "$fileName.meta"
    if (fileExists(metaFile)) {
        responseHeaders["Content-Disposition"] = readFromFile(metaFile)
    }
    return WebResourceResponse(
        mimeType,
        encoding,
        statusCode,
        reasonPhase,
        responseHeaders,
        inputStream
    )
}
```
- **توضیح**:
  - **هدف**: این متد یک پاسخ `WebResourceResponse` از فایل کش‌شده در حافظه داخلی دستگاه تولید می‌کند.
  - **مراحل**:
    1. **خواندن فایل**:
       - با استفاده از `mActivity.openFileInput(fileName)`، فایل کش‌شده به‌صورت `InputStream` باز می‌شود.
       - اگر فایل یافت نشد، خطا ثبت شده و `null` برگردانده می‌شود.
    2. **تنظیم متادیتا**:
       - کدگذاری پاسخ به `UTF-8` تنظیم می‌شود.
       - کد وضعیت HTTP به 200 ("OK") تنظیم می‌شود.
       - یک `HashMap` برای ذخیره هدرهای پاسخ ایجاد می‌شود.
       - هدر `Access-Control-Allow-Origin: *` برای اجازه دسترسی از همه دامنه‌ها اضافه می‌شود.
    3. **مدیریت متادیتا**:
       - اگر فایل متادیتا (`fileName.meta`) وجود داشته باشد، محتوای آن (معمولاً `Content-Disposition`) با متد `readFromFile` خوانده شده و به هدرها اضافه می‌شود.
    4. **ایجاد پاسخ**:
       - یک شیء `WebResourceResponse` با نوع MIME، کدگذاری، کد وضعیت، دلیل وضعیت، هدرها، و جریان ورودی فایل ساخته شده و برگردانده می‌شود.

---

### **بخش ۵: دریافت و کش کردن از شبکه (`createWebResourceResponse`)**
```kotlin
private fun createWebResourceResponse(
    request: WebResourceRequest,
    fileName: String
): WebResourceResponse? {
    return try {
        val url = URL(request.url.toString())
        val conn = url.openConnection() as HttpURLConnection
        val requestHeaders = request.requestHeaders
        for ((key, value) in requestHeaders) {
            conn.setRequestProperty(key, value)
        }
        val mimeType = conn.contentType
        val contentDisposition = conn.getHeaderField("Content-Disposition")
        val input: InputStream = BufferedInputStream(url.openStream())
        val output: FileOutputStream =
            mActivity.openFileOutput(fileName + TEMP_FILE_EXTENSION, Context.MODE_PRIVATE)
        if (contentDisposition != null) {
            writeToFile("$fileName.meta$TEMP_FILE_EXTENSION", contentDisposition)
        }
        val data = ByteArray(1024)
        var count: Int
        while (input.read(data).also { count = it } != -1) {
            output.write(data, 0, count)
        }
        output.flush()
        output.close()
        input.close()
        renameFile(fileName + TEMP_FILE_EXTENSION, fileName)
        renameFile(
            "$fileName.meta$TEMP_FILE_EXTENSION",
            "$fileName.meta"
        )
        createWebResourceResponseFromFile(fileName, mimeType)
    } catch (e: Exception) {
        e.printStackTrace()
        null
    }
}
```
- **توضیح**:
  - **هدف**: این متد منابع را از شبکه دریافت کرده، در حافظه داخلی کش می‌کند، و سپس پاسخ را ارائه می‌دهد.
  - **مراحل**:
    1. **ایجاد اتصال شبکه**:
       - URL درخواست به شیء `URL` تبدیل می‌شود.
       - یک اتصال HTTP با `url.openConnection()` ایجاد می‌شود و به `HttpURLConnection` تبدیل می‌شود.
    2. **انتقال هدرهای درخواست**:
       - هدرهای درخواست WebView (`request.requestHeaders`) به اتصال شبکه اعمال می‌شوند.
    3. **دریافت متادیتا**:
       - نوع MIME پاسخ از `conn.contentType` استخراج می‌شود.
       - هدر `Content-Disposition` (اگر وجود داشته باشد) برای ذخیره متادیتا دریافت می‌شود.
    4. **دانلود و کش کردن**:
       - جریان ورودی شبکه با `BufferedInputStream` باز می‌شود.
       - یک فایل موقت با پسوند `.tmp00` در حافظه داخلی با `mActivity.openFileOutput` ایجاد می‌شود.
       - اگر هدر `Content-Disposition` وجود داشته باشد، در فایل متادیتای موقت ذخیره می‌شود.
       - داده‌ها در بافرهای 1024 بایتی خوانده شده و در فایل موقت نوشته می‌شوند.
    5. **بستن و تغییر نام**:
       - پس از اتمام دانلود، جریان‌های ورودی و خروجی بسته می‌شوند.
       - فایل موقت به نام اصلی تغییر نام داده می‌شود (`renameFile`).
       - فایل متادیتای موقت (اگر وجود داشته باشد) نیز به نام اصلی تغییر نام داده می‌شود.
    6. **ایجاد پاسخ**:
       - متد `createWebResourceResponseFromFile` فراخوانی می‌شود تا فایل کش‌شده ارائه شود.
    7. **مدیریت خطا**:
       - هرگونه خطا (مانند مشکلات شبکه) ثبت شده و `null` برگردانده می‌شود.

---

### **بخش ۶: مدیریت فایل‌های کش**
```kotlin
fun deleteAppFiles() {
    for (file in mActivity.filesDir.list()!!) {
        try {
            if (isAppFile(file) || isTempFile(file)) mActivity.deleteFile(file)
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}

fun deleteAssetFiles() {
    for (file in mActivity.filesDir.list()!!) {
        try {
            if (!isAppFile(file)) mActivity.deleteFile(file)
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}

private fun fileExists(fileName: String): Boolean {
    return File(mActivity.filesDir.toString() + "/" + fileName).exists()
}

val isAppCached: Boolean
    get() = fileExists("bundle.js")

private fun renameFile(oldName: String, newName: String): Boolean {
    val root: String = mActivity.filesDir.toString() + "/"
    val source = File(root + oldName)
    val dest = File(root + newName)
    return source.renameTo(dest)
}

private fun writeToFile(fileName: String, data: String) {
    try {
        val outputStreamWriter = OutputStreamWriter(
            mActivity.openFileOutput(fileName, Context.MODE_PRIVATE)
        )
        outputStreamWriter.write(data)
        outputStreamWriter.close()
    } catch (e: IOException) {
        e.printStackTrace()
    }
}

private fun readFromFile(fileName: String): String {
    var ret = ""
    try {
        val inputStream: InputStream = mActivity.openFileInput(fileName)
        val inputStreamReader = InputStreamReader(inputStream)
        val bufferedReader = BufferedReader(inputStreamReader)
        var receiveString: String?
        val stringBuilder = StringBuilder()
        while (bufferedReader.readLine().also { receiveString = it } != null) {
            stringBuilder.append(receiveString)
        }
        inputStream.close()
        ret = stringBuilder.toString()
    } catch (e: Exception) {
        e.printStackTrace()
    }
    return ret
}
```
- **توضیح**:
  - **حذف فایل‌های برنامه (`deleteAppFiles`)**:
    - تمام فایل‌های موجود در دایرکتوری داخلی (`mActivity.filesDir`) بررسی می‌شوند.
    - فایل‌هایی که یا از نوع برنامه‌ای (`isAppFile`) باشند (مانند `.js`, `.css`) یا موقت (`isTempFile`) باشند، حذف می‌شوند.
    - خطاها ثبت می‌شوند اما فرآیند ادامه می‌یابد.
  - **حذف فایل‌های غیربرنامه‌ای (`deleteAssetFiles`)**:
    - فایل‌هایی که جزو فایل‌های برنامه‌ای نیستند حذف می‌شوند.
  - **بررسی وجود فایل (`fileExists`)**:
    - بررسی می‌کند که آیا فایل در دایرکتوری داخلی وجود دارد یا خیر.
  - **وضعیت کش برنامه (`isAppCached`)**:
    - بررسی می‌کند که آیا فایل `bundle.js` (احتمالاً فایل اصلی برنامه) در کش وجود دارد یا خیر.
  - **تغییر نام فایل (`renameFile`)**:
    - فایل را از نام قدیمی به نام جدید تغییر می‌دهد.
  - **نوشتن در فایل (`writeToFile`)**:
    - داده‌های متنی (مانند متادیتا) را در فایل ذخیره می‌کند.
  - **خواندن از فایل (`readFromFile`)**:
    - محتوای فایل (مانند متادیتا) را به‌صورت رشته می‌خواند.

---

### **بخش ۷: Companion Object (متدهای استاتیک و تنظیمات)**
```kotlin
companion object {
    private val NETWORK_ONLY_PATHS = arrayOf(
        "/TokenGenerator",
        "/PezhvaMessageBroker",
        "/NativeFileUploadMainActivity",
        "/RFU",
        "/AuthService",
        "/FileUpload",
        "/Admin",
        "/DynaRest",
        "hamayand",
        "call",
        "tile",
        "8081"
    )
    private val NETWORK_FIRST_EXTENSIONS = arrayOf(".ver", ".json")
    private val NETWORK_STREAM_EXTENSIONS = arrayOf(".mp3", ".opus")
    private val APP_FILE_EXTENSIONS =
        arrayOf(".js", ".css", ".html", ".ttf", ".eot", "woff", "woff2")
    private const val TEMP_FILE_EXTENSION = ".tmp00"

    private fun getMimeType(url: String): String? {
        var type: String? = null
        val extension = MimeTypeMap.getFileExtensionFromUrl(url)
        if (extension != null) {
            type = MimeTypeMap.getSingleton().getMimeTypeFromExtension(extension)
        }
        return type
    }

    private fun disableSSLCertificateChecking() {
        val trustAllCerts = arrayOf<TrustManager>(
            @SuppressLint("CustomX509TrustManager")
            object : X509TrustManager {
                override fun getAcceptedIssuers(): Array<X509Certificate>? {
                    return null
                }

                @SuppressLint("TrustAllX509TrustManager")
                override fun checkClientTrusted(arg0: Array<X509Certificate>, arg1: String) {
                    // Not implemented
                }

                @SuppressLint("TrustAllX509TrustManager")
                override fun checkServerTrusted(arg0: Array<X509Certificate>, arg1: String) {
                    // Not implemented
                }
            })
        try {
            val sc = SSLContext.getInstance("TLS")
            sc.init(null, trustAllCerts, SecureRandom())
            HttpsURLConnection.setDefaultSSLSocketFactory(sc.socketFactory)
        } catch (e: KeyManagementException) {
            e.printStackTrace()
        } catch (e: NoSuchAlgorithmException) {
            e.printStackTrace()
        }
    }

    private fun isNetworkFirst(fileName: String): Boolean {
        for (ext in NETWORK_FIRST_EXTENSIONS) {
            if (fileName.lowercase(Locale.getDefault()).endsWith(ext)) {
                return true
            }
        }
        return false
    }

    private fun isNetworkStream(fileName: String): Boolean {
        for (ext in NETWORK_STREAM_EXTENSIONS) {
            if (fileName.lowercase(Locale.getDefault()).endsWith(ext)) {
                return true
            }
        }
        return false
    }

    private fun isNetworkOnly(url: String): Boolean {
        for (path in NETWORK_ONLY_PATHS) {
            if (url.lowercase(Locale.getDefault())
                    .contains(path.lowercase(Locale.getDefault()))
            ) return true
        }
        return false
    }

    private fun isAppFile(fileName: String): Boolean {
        for (ext in APP_FILE_EXTENSIONS) {
            if (fileName.lowercase(Locale.getDefault()).endsWith(ext)) {
                return true
            }
        }
        return false
    }

    private fun isTempFile(fileName: String): Boolean {
        return fileName.endsWith(TEMP_FILE_EXTENSION)
    }
}
```
- **توضیح**:
  - **ثابت‌ها**:
    - `NETWORK_ONLY_PATHS`: مسیرهایی که همیشه از شبکه دریافت می‌شوند و کش نمی‌شوند.
    - `NETWORK_FIRST_EXTENSIONS`: فایل‌هایی که ابتدا از شبکه دریافت می‌شوند (`.ver`, `.json`).
    - `NETWORK_STREAM_EXTENSIONS`: فایل‌های استریم که کش نمی‌شوند (`.mp3`, `.opus`).
    - `APP_FILE_EXTENSIONS`: فایل‌های اصلی برنامه (مانند `.js`, `.css`, `.html`, فونت‌ها).
    - `TEMP_FILE_EXTENSION`: پسوند فایل‌های موقت (`.tmp00`).
  - **متد `getMimeType`**:
    - پسوند فایل را از URL استخراج کرده و نوع MIME را با استفاده از `MimeTypeMap` تعیین می‌کند.
  - **متد `disableSSLCertificateChecking`**:
    - بررسی گواهینامه‌های SSL را غیرفعال می‌کند.
    - یک `X509TrustManager` سفارشی ایجاد می‌کند که همه گواهینامه‌ها را می‌پذیرد.
    - این روش ناامن است و فقط برای توسعه یا تست مناسب است.
  - **متدهای کمکی**:
    - `isNetworkFirst`: بررسی می‌کند که آیا فایل نیاز به استراتژی اولویت شبکه دارد.
    - `isNetworkStream`: بررسی می‌کند که آیا فایل از نوع استریم است.
    - `isNetworkOnly`: بررسی می‌کند که آیا URL شامل مسیرهای بدون کش است.
    - `isAppFile`: بررسی می‌کند که آیا فایل جزو فایل‌های برنامه است.
    - `isTempFile`: بررسی می‌کند که آیا فایل موقت است.

---

## **جریان کلی عملکرد**
1. **دریافت درخواست WebView**:
   - متد `tryHandle` درخواست را بررسی می‌کند.
   - اگر درخواست غیرقابل کش باشد (غیر GET، مسیر خاص، یا استریم)، مستقیماً به شبکه ارجاع داده می‌شود.
2. **انتخاب استراتژی**:
   - برای فایل‌های معمولی (مانند `.js`, `.css`): ابتدا کش بررسی می‌شود، سپس شبکه.
   - برای فایل‌های `.ver` و `.json`: ابتدا شبکه، سپس کش.
3. **مدیریت کش**:
   - فایل‌ها در حافظه داخلی ذخیره می‌شوند.
   - فایل‌های موقت در حین دانلود استفاده می‌شوند و پس از اتمام به نام اصلی تغییر می‌کنند.
   - متادیتا (مانند `Content-Disposition`) در فایل‌های جداگانه ذخیره می‌شود.
4. **ارائه پاسخ**:
   - پاسخ‌های WebResourceResponse با داده‌های کش‌شده یا تازه دریافت‌شده به WebView ارائه می‌شوند.
5. **مدیریت فایل‌ها**:
   - امکان حذف فایل‌های برنامه‌ای یا غیربرنامه‌ای فراهم است.
   - بررسی وضعیت کش (مثلاً وجود `bundle.js`) ممکن است.

---

## **نکات و هشدارها**
- **امنیت**: غیرفعال کردن بررسی SSL (با `disableSSLCertificateChecking`) خطرناک است و می‌تواند برنامه را در معرض حملات Man-in-the-Middle قرار دهد. در محیط تولیدی باید از TrustManager مناسب استفاده شود.
- **مدیریت خطا**: خطاها با `printStackTrace` ثبت می‌شوند که برای دیباگ کافی است، اما در تولید بهتر است از سیستم لاگ‌گیری قوی‌تری استفاده شود.
- **کارایی**: دانلود فایل‌ها در بافرهای 1024 بایتی انجام می‌شود که برای فایل‌های کوچک مناسب است، اما برای فایل‌های بزرگ ممکن است نیاز به بهینه‌سازی داشته باشد.
- **محدودیت‌ها**: این کد برای سناریوهای خاص طراحی شده و ممکن است برای برنامه‌های پیچیده‌تر نیاز به تنظیمات بیشتری داشته باشد (مثلاً مدیریت حجم کش یا انقضای فایل‌ها).

---

این مستند با جزئیات کامل نحوه عملکرد کد را توضیح می‌دهد. اگر نیاز به توضیحات بیشتر در مورد بخش خاصی یا سناریوهای خاص دارید، اطلاع دهید!
