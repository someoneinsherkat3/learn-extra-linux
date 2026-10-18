# Go deeper 
در اینجا ۳ مورد را بررسی میکنیم:

1. استفاده از  Makefile برای خودکار سازی برخی روند ها جهت پیاده سازی ساختار درست بجای ساختار دستی
2. افزودن اجزای داخلی بسته (وابستگی، اسکریپت‌ها، فایل تنظیمات)
3. روش استاندارد دبیان با `debhelper`

## ‌‌part 1 : application with config file 

میخواهیم به برنامه conffile اضافه کنیم .
همانطور که در درس قبل گفته شد conffile یک فایل است شامل فهرست فایل‌های config که کاربر می‌تواند ویرایش کند
برنامه را طوری تغییر می‌دهیم که پیام را از `etc/hello-deb.conf/` بخواند.

`hello.c`:

```c
#include <stdio.h>
#include <string.h>

#define CONF "/etc/hello-deb.conf"

int main(void) {
    char line[256] = "Hello from my first deb package!";
    FILE *f = fopen(CONF, "r");
    if (f) {
        if (fgets(line, sizeof line, f))
            line[strcspn(line, "\n")] = '\0';
        fclose(f);
    }
    puts(line);
    return 0;
}
```

یک فایل متنی هم به اسم `hello-deb.conf` کنار سورس میسازیم که محتوای دلخواه زیر را داشته باشد:

```
Hello from the config file!
```

## part 2 : Makefile (DESTDIR)

در ساخت پکیج‌ هیچ‌وقت فایل‌ها را دستی `cp` نمکنیم. پروژه یک target به نام `install` دارد و متغیر **`DESTDIR`** مشخص می‌کند فایل‌ها به‌جای `/` در کدام **temp-root** نصب شوند.

`Makefile`

```make
CC      ?= gcc
CFLAGS  ?= -O2 -Wall -Wextra
PREFIX  ?= /usr
DESTDIR ?=

hello: hello.c
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@-deb $<

install: hello
	install -D -m 0755 hello-deb $(DESTDIR)$(PREFIX)/bin/hello-deb
	install -D -m 0644 hello-deb.conf $(DESTDIR)/etc/hello-deb.conf

clean:
	rm -f hello-deb

.PHONY: install clean
```

علامت `=?` یعنی اگر این متغییر تعریف شده بود ان مقدار را برایش در نظیر بگیر . اگر تعریف نشده بود مقدار پیش فرض که جلویش نوشته شده را بردارد. 

---
### دستور install

دستور install  فایل `hello-deb.conf` را به مسیر داده شده `pkgroot/etc/hello-deb.conf` کپی می‌کند. سه کار را همزمان انجام می‌دهد:

1. کپی کردن فایل
2. ساختن پوشه‌های میانی اگر وجود نداشته باشند
3. تنظیم دسترسی فایل مقصد


```
install  -D  -m 0644  hello-deb.conf  pkgroot/etc/hello-deb.conf
  │      │     │          │                     │
  │      │     │          │                     └─ مقصد
  │      │     │          └─ مبدأ
  │      │     └─ دسترسی فایل مقصد
  │      └─ ساخت پوشه‌های میانی
  └─ برنامه install (از بسته coreutils)
```

سویچ `D-` : اگر پوشه‌های `pkgroot` و `pkgroot/etc` وجود نداشته باشند، آن‌ها را می‌سازد. با `cp` معمولی این خطا را می‌گیری:

```bash
cp hello-deb.conf pkgroot/etc/hello-deb.conf
# cp: cannot create regular file 'pkgroot/etc/hello-deb.conf': No such file or directory
```

پس بدون `install -D` باید اول `mkdir -p pkgroot/etc` را بزنیم.

سویچ  `m 0644-` : دسترسی فایل مقصد را تعیین می‌کند. `0644` یعنی

|بخش|عدد|معنی|
|---|---|---|
|مالک|6|خواندن و نوشتن|
|گروه|4|فقط خواندن|
|بقیه|4|فقط خواندن|

عدد اول (`0`) مربوط به بیت‌های ویژه (setuid و مانند آن) است و اینجا صفر.

برای مقایسه، فایل اجرایی با `0755` نصب می‌شود، یعنی همه می‌توانند اجرایش کنند.

اگر `-m` را ندهیم، `install` به‌طور پیش‌فرض `0755` می‌گذارد، که برای فایل تنظیمات نادرست است چون اجرایی نیست. `cp` برعکس، دسترسی را از فایل مبدأ و `umask` کاربر را می‌گیرد، یعنی نتیجه به محیط کاربر بستگی دارد. `install -m` همیشه یک نتیجه ثابت می‌دهد، و برای ساخت بسته همین ثبات لازم است.


---


```bash
➜  tree 
.
├── hello.c
├── hello-deb.conf
└── Makefile

```

```bash
➜  make
cc -O2 -Wall -Wextra  -o hello-deb hello.c
```
با این دستور اولین target اجرا میشود که `hello` نام دارد ( و وابسته به فایل `hello.c` است )

حالا سراغ target بعدی میرویم . یهش یک temp-root تحت عنوان pkgroot میدهیم و مشخص میکنیم در مسیر فعلی ساخته شود.
از انجایی که install  از نوع `PHONY.` است باید مستقیم فراخوانیش کنیم که...
```bash
➜  make install DESTDIR=$PWD/pkgroot
cc -O2 -Wall -Wextra  -o hello-deb hello.c
install -D -m 0755 hello-deb /home/ubuntu/CLionProjects/untitled/pkgroot/usr/bin/hello-deb
install -D -m 0644 hello-deb.conf /home/ubuntu/CLionProjects/untitled/pkgroot/etc/hello-deb.conf
```


```bash
➜  tree 
.
├── hello.c
├── hello-deb
├── hello-deb.conf
├── Makefile
└── pkgroot
    ├── etc
    │   └── hello-deb.conf
    └── usr
        └── bin
            └── hello-deb
```

## part 3 : manually add somethings
حالا که ساختار پکیج را داریم فایل DEBIAN را میتوانیم دستی (فعلا دستی) اضافه کنیم 

```bash
mkdir -p pkgroot/DEBIAN

```

### find dependencies


حالا که داریم دستی برخی قسمت هارا میسازیم یکی از ان قسمت های مهم مشخص کردن وابستگی ها برنامه است 

```bash
➜  ldd pkgroot/usr/bin/hello-deb 
        linux-vdso.so.1 (0x0000777aee3f7000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x0000777aee000000)
        /lib64/ld-linux-x86-64.so.2 (0x0000777aee3f9000)
```


در `control` اضافه میکنیم:

```
Depends: libc6 (>= 2.34)
```

در روش استاندارد ابزار `dpkg-shlibdeps` این کار را خودکار انجام می‌دهد.

###  control file
فایل `control` را دستی میسازیم با محتوای زیر 
`pkgroot/DEBIAN/control`:

```
Package: hello-deb
Version: 1.0
Section: utils
Priority: optional
Architecture: amd64
Depends: libc6 (>= 2.34)
Maintainer: Your Name <you@example.com>
Description: A simple hello world program
 Reads its greeting from /etc/hello-deb.conf.
```

### conffiles


فایل‌های داخل `/etc` که کاربر ممکن است ویرایش کند باید در `DEBIAN/conffiles` اعلام شوند هر مسیر در یک خط:

```
/etc/hello-deb.conf
```

اثر آن: هنگام ارتقا، اگر کاربر فایل را تغییر داده باشد، dpkg تغییراتش را بدون پرسیدن پاک نمی‌کند.

### maintainer scripts

چهار اسکریپت وجود دارد: `preinst`، `postinst`، `prerm`، `postrm`. آرگومان اول نوع عملیات را می‌گوید.
آرگمان هارا dpkg موقع اجرا به اسکریپت ها ارسال میکند.

`pkgroot/DEBIAN/postinst`:

```sh
#!/bin/sh
set -e
case "$1" in
    configure)
        echo "hello-deb configured (previous version: ${2:-none})"
        ;;
esac
exit 0
```

`pkgroot/DEBIAN/prerm`:

```sh
#!/bin/sh
set -e
case "$1" in
    remove|upgrade)
        echo "hello-deb: preparing to $1"
        ;;
esac
exit 0
```

اسکریپت‌ها باید اجرایی باشند:

```bash
chmod 755 pkgroot/DEBIAN/postinst pkgroot/DEBIAN/prerm
```

ترتیب اجرا هنگام **ارتقا** از نسخه قدیم به جدید:

1. `prerm upgrade` (نسخه قدیم)
2. `preinst upgrade` (نسخه جدید)
3. باز کردن فایل‌های جدید روی سیستم
4. `postrm upgrade` (نسخه قدیم)
5. `postinst configure <old-version>` (نسخه جدید)

### md5sums 
برای اطمینان از اصالت فایل ها:

```bash
(cd pkgroot && find usr etc -type f -exec md5sum {} + > DEBIAN/md5sums)
```

```
46d4f3c9b23d86d82732fc11f6ea0d87  usr/bin/hello-deb
f0e9c1d893a7e2d2dc766e5832a8cdfb  etc/hello-deb.conf

```
بعد از نصب، با `dpkg --verify hello-deb` می‌توان چک کرد فایلی تغییر کرده یا نه.

### create and install

```bash
➜  dpkg-deb --build --root-owner-group pkgroot hello-deb_1.0_amd64.deb
dpkg-deb: building package 'hello-deb' in 'hello-deb_1.0_amd64.deb'.

➜  sudo dpkg -i hello-deb_1.0_amd64.deb
[sudo] password for ubuntu: 
(Reading database… 244657 files and directories currently installed.)
Preparing to unpack hello-deb_1.0_amd64.deb…
hello-deb: preparing to upgrade
Unpacking hello-deb (1.0) over (1.0)…
Setting up hello-deb (1.0)…
hello-deb configured (previous version: 1.0)

```
اسم فایل به شکل `name_version_arcithecture.deb` است، قرارداد استاندارد دبیان.

```bash
➜  dpkg-deb --build --root-owner-group pkgroot hello-deb_1.0_amd64.deb
dpkg-deb: error: parsing file 'pkgroot/DEBIAN/control' near line 8 package 'hello-deb':
 end of file during value of field 'Description' (missing final newline)
```

> اگر به این خطا برخرودیم تنها باید یک خط جدید انتهای فایل `control`  اضافه کرد

اجرای فایل 
```bash
➜  hello-deb 
Hello from the config file!
➜  cat /etc/hello-deb.conf 
Hello from the config file!%     

```

## part 4 : inside our .deb packge

فایل deb یک آرشیو `ar` است:
```bash
➜  ar t hello-deb_1.0_amd64.deb 
debian-binary
control.tar.zst
data.tar.zst
```

```bash
➜   mkdir /tmp/x && cd /tmp/x
➜   ar x ~/hello-deb/hello-deb_1.0_amd64.deb
➜   ls
control.tar.zst  data.tar.zst  debian-binary
➜   cat debian-binary
2.0
➜   tar tf control.tar.*
./
./conffile
./control
./md5sums
./postinst
./prerm

➜   tar tf data.tar.*
./
./etc/
./etc/hello-deb.conf
./usr/
./usr/bin/
./usr/bin/hello-deb
```

شامل سه بخش است: `debian-binary` (نسخه فرمت، یعنی `2.0`)، `control.tar.*` (اطلاعات و اسکریپت‌ها) و `data.tar.*` (خود فایل‌ها).

حالا ببینیم dpkg بعد از نصب چه چیزی ثبت کرده:

```bash
➜  ls /var/lib/dpkg/info/ | grep hello-deb
hello-deb.conffile
hello-deb.list
hello-deb.md5sums
hello-deb.postinst
hello-deb.prerm

➜  grep -A8 '^Package: hello-deb' /var/lib/dpkg/status
Package: hello-deb
Status: install ok installed
Priority: optional
Section: utils
Maintainer: Your Name <you@example.com>
Architecture: amd64
Version: 1.0
Depends: libc6 (>= 2.34)
Description: A simple hello world program
```

پایگاه داده dpkg همین فایل‌های متنی است: `status` وضعیت همه بسته‌ها را دارد و پوشه `info/` برای هر بسته فایل‌های `.list` (فایل‌های نصب‌شده)، `.md5sums` و اسکریپت‌ها را نگه می‌دارد.

## part 5: update conffile and re-build

برای تست بیشتر محتوای فایل  `hello-deb.conf` در temp-root را عوض میکنیم  (مثلاً `Hello version 1.1!`)، سپس در `control` فیلد version را به `1.1` تغییر میدیم . دوباره build میکنیم (`hello-deb_1.1_amd64.deb`).

قبل از نصب، فایل نصب‌شده را دستی تغییر میدیم:

```bash
sudo vim /etc/hello-deb.conf
```

```
➜  dpkg-deb --build --root-owner-group pkgroot hello-deb_1.1_amd64.deb
dpkg-deb: building package 'hello-deb' in 'hello-deb_1.1_amd64.deb'.
```

حالا نسخه ۱.۱ را نصب میکنیم:

```bash
➜  sudo dpkg -i hello-deb_1.1_amd64.deb
(Reading database… 244657 files and directories currently installed.)
Preparing to unpack hello-deb_1.1_amd64.deb…
hello-deb: preparing to upgrade
Unpacking hello-deb (1.1) over (1.0)…
Setting up hello-deb (1.1)…
hello-deb configured (previous version: 1.0)
```

‏dpkg می‌پرسد با فایل تنظیمات چه کند:

- ‏`Y` یا `I`: نسخه جدید بسته نصب شود
- ‏`N` یا `O`: نسخه خودت بماند
- ‏`D`: تفاوت‌ها را نشان بده
- ‏`Z`: شل موقت باز کن تا بررسی کنی

با `dpkg --compare-versions 1.0 lt 1.1 && echo yes` ترتیب نسخه‌ها را هم می‌توانیم تست کنیم. فرمت کامل نسخه `epoch:upstream-revision` است، مثلاً `1:2.3-4`. `epoch` بالاترین اولویت را دارد و `revision` شماره بسته‌بندی debian است، نه شماره نسخه برنامه.

## part 6: use lintian tool

```bash
sudo apt install lintian

➜   lintian hello-deb_1.0_amd64.deb
E: hello-deb: file-in-etc-not-marked-as-conffile [etc/hello-deb.conf]
E: hello-deb: md5sum-mismatch etc/hello-deb.conf [md5sums]
E: hello-deb: no-changelog usr/share/doc/hello-deb/changelog.gz (native package)
E: hello-deb: no-copyright-file
E: hello-deb: unstripped-binary-or-object [usr/bin/hello-deb]
W: hello-deb: description-synopsis-starts-with-article
W: hello-deb: maintainer-script-empty [postinst]
W: hello-deb: maintainer-script-empty [prerm]
W: hello-deb: no-manual-page [usr/bin/hello-deb]
W: hello-deb: unknown-control-file [conffile]
```

‏lintian هشدارهایی می‌دهد، مثل نبود فایل `copyright`، `changelog` یا man page. هر هشدار را بخوان و با `lintian --info` توضیحش را بررسی میکنیم. این بهترین راه یاد گرفتن قوانین  است.

```bash
➜   lintian --info hello-deb_1.0_amd64.deb
N:
E: hello-deb: file-in-etc-not-marked-as-conffile [etc/hello-deb.conf]
N: 
N:   Files in /etc must be marked conffiles if they are included in a package. Otherwise they should be created by maintainer scripts.
N: 
N:   Please refer to Configuration files (Section 10.7) in the Debian Policy Manual for details.
N: 
N:   Visibility: error
N:   Show-Always: no
N:   Check: conffiles
N: 
N:
E: hello-deb: md5sum-mismatch etc/hello-deb.conf [md5sums]
N: 
N:   The md5sum listed for the file does not match the actual file contents.
N:   
N:   Usually, this error occurs during the package build process if the debian/tmp/ directory is touched after dh_md5sums is run.
N:   
N:   Font files regenerated at post-install time by t1c2pfb should be overridden.
N: 
N:   Visibility: error
N:   Show-Always: no
N:   Check: md5sums
N: 
N:
E: hello-deb: no-changelog usr/share/doc/hello-deb/changelog.gz (native package)
N: 
N:   A Debian package that provides a /usr/share/doc/*pkg* directory must install a changelog file.
N:   
N:   For native packages the best name is /usr/share/doc/*pkg*/changelog.gz.
N:   
N:   For non-native packages the best name is /usr/share/doc/*pkg*/changelog.Debian.gz.
N:   
N:   This tag may also be emitted when the changelog exists but does not otherwise resemble a Debian changelog.
N: 
N:   Please refer to Changelog files and release notes (Section 12.7) in the Debian Policy Manual for details.
N: 
N:   Visibility: error
N:   Show-Always: no
N:   Check: debian/changelog
N:   Renamed from: changelog-file-missing-in-native-package debian-changelog-file-missing
N: 
N:
E: hello-deb: no-copyright-file
N: 
N:   Each binary package has to include a plain file /usr/share/doc/*pkg*/copyright
N: 
N:   Please refer to Copyright information (Section 12.5) in the Debian Policy Manual for details.
N: 
N:   Visibility: error
N:   Show-Always: no
N:   Check: debian/copyright
N: 
N:
E: hello-deb: unstripped-binary-or-object [usr/bin/hello-deb]
N: 
N:   The package installs an unstripped binary or object file.
N:   
N:   Please note, that shared libraries have to be stripped with the --strip-unneeded option.
N: 
N:   Please refer to Binaries (Section 10.1) in the Debian Policy Manual and Libraries (Section 10.2) in the Debian Policy Manual for details.
N: 
N:   Visibility: error
N:   Show-Always: no
N:   Check: binaries/debug-symbols
N: 
N:
W: hello-deb: description-synopsis-starts-with-article
N: 
N:   The first line of the "Description:" should omit any initial indefinite or definite article: "a", "an", or "the". A good heuristic is that it should be possible to substitute the package name and synopsis into this
N:   formula:
N:   
N:   The package name provides {a,an,the,some} synopsis.
N: 
N:   Please refer to The package synopsis, or short description (Section 6.2.2) in the Debian Developer's Reference for details.
N: 
N:   Visibility: warning
N:   Show-Always: no
N:   Check: fields/description
N: 
N:
W: hello-deb: maintainer-script-empty [postinst]
N: 
N:   The named maintainer script does not appear to contain code other than comments or boilerplate such as set -e, exit statements, or a case statement to parse options.
N:   
N:   While harmless in most cases, it is not needed. The package may also leave files behind until purged, and can contribute to rare problems when dpkg fails because no maintainer scripts are present.
N:   
N:   Please do not ship the maintainer script unless it does something useful.
N: 
N:   Visibility: warning
N:   Show-Always: no
N:   Check: maintainer-scripts/empty
N: 
N:
W: hello-deb: maintainer-script-empty [prerm]
N:
W: hello-deb: no-manual-page [usr/bin/hello-deb]
N: 
N:   Each binary in /usr/bin, /usr/sbin, /bin, /sbin or /usr/games should have a manual page
N:   
N:   Note that though the man program has the capability to check for several program names in the NAMES section, each of these programs should have its own manual page (a symbolic link to the appropriate manual page is
N:   sufficient) because other manual page viewers such as xman or tkman don't support this.
N:   
N:   If the name of the manual page differs from the binary by case, man may be able to find it anyway; however, it is still best practice to match the exact capitalization of the executable in the manual page.
N:   
N:   If the manual pages are provided by another package on which this package depends, Lintian may not be able to determine that manual pages are available. In this case, after confirming that all binaries do have manual pages
N:   after this package and its dependencies are installed, please add a Lintian override.
N: 
N:   Please refer to Manual pages (Section 12.1) in the Debian Policy Manual for details.
N: 
N:   Visibility: warning
N:   Show-Always: no
N:   Check: documentation/manual
N:   Renamed from: binary-without-manpage
N: 
N:
W: hello-deb: unknown-control-file [conffile]
N: 
N:   The package contains an unknown control file. Policy says that putting additional files in the package control area is generally not a good idea.
N: 
N:   Please refer to Package control information files (Appendix appendix-2.2) in the Debian Policy Manual for details.
N: 
N:   Visibility: warning
N:   Show-Always: no
N:   Check: control-files
N: 
```

