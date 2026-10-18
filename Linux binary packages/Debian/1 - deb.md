# inside the deb packages

تا اینجا فایل package حاوی ۲ مقدار زیر بود
```
package name/
├── DEBIAN/    # metadata
└── usr/       # files
```
که بجر directory اول بقیه فقط یک مسیر نیستند بلکه هر چیزی که بخواهیم روی سیستم مقصد قرار بگیرد، با هر مسیری که لازم باشد را شامل میشود.
```
package name/
├── DEBIAN/    # metadata
├── usr/       # files
├── etc/..     # config files 
└── ...        # log files ....
```



![[deb_package_three_members.svg|1446]]


خود فایل پکیج یک فایل  آرشیو از نوع `ar` است که شامل سه عضو میباشد :

- ‏`debian-binary` : یک مقدار ثابت که ابزار ساخت پکیج ان را تولید میکند (یک متن ۴ بایتی با مقدار "2.0")
- ‏`control.tar.xz` : یک فایل فشرده از مسیر DEBIAN (نوع فشرده سازی میتواند هرچیزی باشد`.zst`/`.gz` )
- ‏`data.tar.xz` : یک فایل فشرده از باقی مسیر هایی که درنو پکیج قرار دارند (فایل‌هایی که روی سیستم مقصد نصب می‌شوند)

ترتیب این سه ثابت است و `debian-binary` همیشه اول می‌آید.

```bash
➜  ar t hello.1.0.deb 
debian-binary
control.tar.zst
data.tar.zst
```

سویچ `t-` کارش نمایش لیست جداول محتوایی فایل های فشرده است . Display a table listing the contents of archive

## control.tar (DEBIAN directory)

در مسیر DEBIAN درون پکیج یک فایل اجباری وجود دارد به اسم `control` که شناسنامه بسته (نام، نسخه، معماری، وابستگی‌ها، توضیح، نگهدارنده و بقیه فیلدها) است
باقی فایل های درون  مسیر DEBIAN اختیاری هستند که در زیر لیست شده اند:
- ‏ `md5sums`   : هش هر فایل، برای `dpkg --verify`
- ‏ `conffiles` :فهرست فایل‌های config که کاربر می‌تواند ویرایش کند
- ‏ `preinst`   : اسکریپت قبل از باز کردن فایل‌ها
- ‏ `postinst`  : اسکریپت بعد از باز کردن فایل‌ها (configure)
- ‏ `prerm`     : اسکریپت قبل از حذف
- ‏ `postrm`    : اسکریپت بعد از حذف
- ‏ `triggers`   : اعلام علاقه به رویدادها (مثلاً «هر وقت فایلی در `/usr/share/applications` عوض شد مرا خبر کن»)
- ‏ `shlibs`    : اگر بسته کتابخانه اشتراکی دارد: «کتابخانه X را باید با وابستگی Y نشان داد»
- ‏ `symbols`   : نسخه‌دهی دقیق‌تر نمادهای کتابخانه (جایگزین پیشرفته `shlibs`)
- ‏ `templates` :‌ سؤال‌های debconf (پرسش‌های متنی هنگام نصب)
- ‏ `config`    : اسکریپتی که قبل از نصب سؤال‌های debconf را می‌پرسد


> دو فایل مختلف هر دو اسمشان `control` است و این باعث سردرگمی می‌شود:
> یکی درون `deb.` در مسیر DEBIAN که تا الان بررسیش میکردیم.
> یکی دیگر در کنار سورس پروژه درون مسیر debian که در `deb.` نهایی نمیاید 
نکته: فیلد `control` داخل فایل نهایی فقط یک بخش دارد (بخش `Package`)، یعنی همان چیزی که با `dpkg-deb --info` می‌بینی. بخش `Source` فقط در سورس بسته (`debian/control`) هست و در `.deb` نمی‌آید.

روند ساخت اصولی پکیج ها اینطوریه که در کنار سورس یک مسیر debian قرار میگیرد که شامل دستورات ساخت `deb.` است  

```
hello-deb-1.0/
├── hello.c
├── Makefile
└── debian/           ← دستور ساخت
    ├── control
    ├── rules
    └── changelog
        │
        │  dpkg-buildpackage
        ▼
debian/hello-deb/     ← پوشه‌ی موقت (همان نقش pkgroot ما)
├── DEBIAN/control    ← ساخته‌شده از debian/control
└── usr/bin/hello
        │
        │  dpkg-deb --build
        ▼
../hello-deb_1.0_amd64.deb   ← فقط DEBIAN/ و usr/... داخلش هست
```
‍
## data.tar 

تعیین می‌کند چه چیزی روی `/` سیستم مقصد می‌نشیند. هر ورودی می‌تواند این‌ها باشد:

- فایل معمولی
- دایرکتوری (حتی خالی)
- لینک نمادین (symlink)
- لینک سخت (hard link)
- فایل دستگاه (نادر)

هر ورودی درون این مسیر >== مالک، گروه، دسترسی و زمان تغییر دارد (همان چیزی که `--root-owner-group` اصلاح می کرد).

### مسیرهای متداول و کاربردشان

| مسیر                                                   | محتوا                                             |
| ------------------------------------------------------ | ------------------------------------------------- |
| `/usr/bin/`                                            | برنامه‌های کاربر عادی                             |
| `/usr/sbin/`                                           | برنامه‌های مدیریتی (فقط root)                     |
| `/usr/lib/<بسته>/`                                     | برنامه‌ها و فایل‌های کمکی که مستقیم اجرا نمی‌شوند |
| `/usr/lib/x86_64-linux-gnu/`                           | کتابخانه‌های اشتراکی (`.so`) و ایستا (`.a`)       |
| `/usr/include/`                                        | هدرهای C برای توسعه‌دهندگان (در بسته `-dev`)      |
| `/usr/share/<بسته>/`                                   | داده‌های مستقل از معماری (تصویر، قالب، فونت)      |
| `/usr/share/doc/<بسته>/`                               | مستندات                                           |
| `/usr/share/man/man1/`                                 | صفحه man (`hello.1.gz`)                           |
| `/usr/share/applications/`                             | فایل `.desktop` (ورود به منوی برنامه‌ها)          |
| `/usr/share/icons/` و `/usr/share/pixmaps/`            | آیکون‌ها                                          |
| `/usr/share/locale/`                                   | ترجمه‌ها (`.mo`)                                  |
| `/usr/share/bash-completion/completions/`              | تکمیل خودکار دستورات                              |
| `/usr/share/lintian/overrides/`                        | استثناهای lintian                                 |
| `/etc/`                                                | فایل‌های تنظیمات سیستمی (بسته conffile)           |
| `/etc/default/`                                        | مقادیر پیش‌فرض سرویس‌ها                           |
| `/etc/cron.d/`، `/etc/logrotate.d/`                    | زمان‌بندی و چرخش لاگ                              |
| `/lib/systemd/system/` (یا `/usr/lib/systemd/system/`) | فایل‌های `.service` و `.timer`                    |
| `/usr/lib/tmpfiles.d/`، `/usr/lib/sysusers.d/`         | ساخت پوشه‌های موقت و کاربران سیستمی               |
| `/lib/udev/rules.d/`                                   | قوانین udev برای سخت‌افزار                        |
| `/usr/share/dbus-1/`، `/usr/share/polkit-1/`           | تنظیمات D-Bus و دسترسی                            |

### مهم: هر بسته‌ای در این مسیرها فایل نمی‌گذارد

- ‏ **`/home`**: هرگز. فایل کاربر است.
- ‏ **`/usr/local`**: هرگز. محل نصب دستی ادمین است.
- **`/var/lib/<بسته>`، `/var/log/<بسته>`، `/var/cache`**:
   معمولاً در `data.tar` نیستند. داده‌های زمان اجرا را `postinst` با `mkdir` می‌سازد.
- **`/tmp`**: هرگز.

## what `dpkg` do after installation

بعد از نصب در `/var/lib/dpkg/info/` برای بسته چند فایل ساخته می‌شود:

```
hello-deb.list         ← فهرست همه فایل‌های نصب‌شده
hello-deb.md5sums      ← همان md5sums
hello-deb.conffiles    ← همان conffiles
hello-deb.postinst     ← اسکریپت‌ها با همین نام
hello-deb.prerm
hello-deb.triggers
```

همچنین در فایل  `/var/lib/dpkg/status` محتوای فایل `control` اضافه میشود . 
```bash
➜ pwd
/var/lib/dpkg

➜ cat status | grep -A 10 -m 1 hello-deb
Package: hello-deb
Status: install ok installed
Priority: optional
Section: utils
Maintainer: Your Name <you@example.com>
Architecture: amd64
Version: 1.0
Depends: libc6 (>= 2.34)
Description: A simple hello world program
 Reads its greeting from /etc/hello-deb.conf.
```

> یعنی پوشه `DEBIAN` موقع نصب **پراکنده می‌شود** و محتوای `data.tar` روی `/` می‌نشیند.





```bash
➜  pwd 
/var/lib/dpkg/info
➜  ls | grep hello
hello-deb.list
hello-deb.md5sums
```

## manually check 2 `.deb` packages
۲ مورد نمونه در اینجا بررسی میشود:

###  `chatgpt_amd64.deb`

```bash
➜  file chatgpt_amd64.deb 
chatgpt_amd64.deb: Debian binary package (format 2.0), with control.tar.xz , data compression xz

```

```bash
➜ ar tv chatgpt_amd64.deb
rw-r--r-- 0/0      4 Oct 14 02:42 2026 debian-binary
rw-r--r-- 0/0   3296 Oct 14 02:42 2026 control.tar.xz
rw-r--r-- 0/0 497173804 Oct 14 02:42 2026 data.tar.xz
rw-r--r-- 0/0    566 Jan  1 03:30 1970 _gpgorigin
```

سویچ `v` همان `verbose` است.

```bash
➜ pwd
/var/lib/dpkg/info
➜ ls | grep gpt  
chatgpt.conffiles
chatgpt.list
chatgpt.md5sums
chatgpt.postinst
chatgpt.postrm
chatgpt.prerm
```

```bash
➜ pwd
/var/lib/dpkg

➜  cat status | grep -A 10 -m 1 gpt  
Package: chatgpt
Status: install ok installed
Priority: optional
Section: utils
Installed-Size: 1653570
Maintainer: OpenAI <support@openai.com>
Architecture: amd64
Version: 26.1013.40934
Depends: libgtk-3-0, libnotify4, libnss3, xdg-utils, libatspi2.0-0, libdrm2, libgbm1, libxcb-dri3-0, libglib2.0-bin | kde-cli-tools | kde-runtime | trash-cli | gvfs-bin, libasound2 (>= 1.0.17), libatk-bridge2.0-0 (>= 2.5.3), libatk1.0-0 (>= 2.32.0), libatspi2.0-0 (>= 2.9.90), libc6 (>= 2.30), libcairo2 (>= 1.14.0), libcups2 (>= 1.7.0), libdbus-1-3 (>= 1.9.14), libexpat1 (>= 2.1~beta3), libgbm1 (>= 17.1.0~rc2), libgcc-s1 (>= 4.2), libgdk-pixbuf-2.0-0 (>= 2.36.9), libgl1, libglib2.0-0 (>= 2.49.3), libgtk-3-0 (>= 3.9.10), libnspr4 (>= 2:4.9-2~), libnss3 (>= 2:3.30), libpango-1.0-0 (>= 1.14.0), libssl3 (>= 3.0.0), libstdc++6 (>= 9), libtss2-esys-3.0.2-0 (>= 2.3.1), libtss2-mu0 (>= 3.0.1) | libtss2-mu-4.0.1-0t64 (>= 3.0.1), libtss2-tcti-device0 (>= 3.0.1), libudev1 (>= 183), libusb-1.0-0 (>= 2:1.0.16), libx11-6 (>= 2:1.4.99.1), libx11-xcb1, libxcb1 (>= 1.9.2), libxcomposite1 (>= 1:0.4.4-1), libxdamage1 (>= 1:1.1), libxext6, libxfixes3, libxkbcommon0 (>= 0.5.0), libxrandr2, mesa-vulkan-drivers | vulkan-icd, xz-utils
Recommends: pulseaudio | libasound2, git
Suggests: gir1.2-gnomekeyring-1.0, libgnome-keyring0, lsb-release
```

###  `v2rayN-linux-64.deb`

```bash
➜  file v2rayN-linux-64.deb
v2rayN-linux-64.deb: Debian binary package (format 2.0), with control.tar.xz , data compression xz
```


```bash
➜ ar tv v2rayN-linux-64.deb
rw-r--r-- 0/0      4 Sep 30 10:10 2026 debian-binary
rw-r--r-- 0/0    776 Sep 30 10:10 2026 control.tar.xz
rw-r--r-- 0/0 85993752 Sep 30 10:10 2026 data.tar.xz
```

```bash
➜ pwd
/var/lib/dpkg/info
➜ ls | grep v2ray
v2rayn.list
v2rayn.md5sums
v2rayn.postinst
v2rayn.postrm
v2rayn.substvars
```


```bash
➜ pwd
/var/lib/dpkg

➜  cat status | grep -A 10 -m 1 v2ray    
Package: v2rayn
Status: install ok installed
Priority: optional
Section: net
Maintainer: 2dust <noreply@github.com>
Architecture: amd64
Version: 7.25.4
Depends: libc6, libfontconfig1, libgcc-s1, libstdc++6, libc6 (>= 2.39), fontconfig (>= 2.15.0), desktop-file-utils (>= 0.26), xdg-utils (>= 1.1.3), coreutils (>= 9.4), bash (>= 5.2.21), libfreetype6 (>= 2.13)
Description: v2rayN (Avalonia) GUI client for Linux
 Support vless / vmess / Trojan / http / socks / Anytls / Hysteria2 /
 Shadowsocks / tuic / WireGuard.
```




```
.deb
├── debian-binary
├── control.tar     ← «بسته درباره خودش چه می‌گوید و چه اسکریپتی دارد»
│   ├── control (اجباری)
│   ├── md5sums, conffiles
│   ├── preinst, postinst, prerm, postrm
│   └── triggers, shlibs, symbols, templates, config
└── data.tar        ← «چه چیزی روی سیستم بنشیند»
    └── usr/bin, usr/lib, usr/share/{doc,man,applications,icons},
        etc, lib/systemd/system, ...
```
