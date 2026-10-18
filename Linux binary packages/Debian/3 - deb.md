روش استاندارد ساخت پکیج دبیان با `debhelper`

## use debhelper (standard way)

بسته‌های واقعی از روش دیگری و با ساختار directory دیگری به اسم `debian/` و با ابزار `dh` ساخته می‌شوند.
ابزارهای زیر را نصب میکنیم:

```bash
sudo apt install debhelper devscripts
```

برای استفاده از این ابزار باید ساختار پروژه به این شکل باشد:

```
hello-deb-1.0/
├── hello.c
├── hello-deb.conf
├── Makefile
└── debian/
    ├── control
    ├── changelog
    ├── rules
    ├── copyright
    └── source/format
```
مسیر debian فقط نقش **ابزار ساخت** را دارد و در  پکیج نهایی نمی‌آید. کارش دستور ساخت بسته (`rules`، `control` دو بخشی، `changelog` و غیره) است 


پوشه `debian/` سه دسته فایل دارد:
- **پنج فایل ضروری**
	- ‏`control` : شناسنامه‌ی سورس و بسته‌های باینری، با یک بخش `Source` و یک یا چند بخش `Package`
	- ‏`changelog` : تاریخچه‌ی نسخه‌ها. **نسخه‌ی بسته از آخرین ورودی این فایل خوانده می‌شود**، نه از `control`
	- ‏`rules` : یک Makefile اجرایی که روش ساخت را مشخص می‌کند (فایل باید `chmod +x` باشد.)
	- ‏`copyright` : مجوز برنامه و فایل‌هایش، با قالب DEP-5
	- ‏`source/format` : قالب بسته‌ی سورس را تعیین می‌کند
- **فایل‌های اختیاری برای هر بسته** 
	- فایل‌ها و مسیرها
		- ‏ `install.`  : کدام فایل‌ها داخل بسته بروند
		- ‏ `dirs.` : پوشه‌هایی که باید ساخته شوند
		- ‏ `links.` : لینک نمادین
		- ‏ `docs.` : مستنداتی که در `<usr/share/doc/<package-name/` قرار میگیرند
		- ‏ `examples.` : مثال‌ها
		- ‏ `manpages.` : همان man page برنامه             
		- ‏ `bash-completion.` : تکمیل خودکار bash
		- ‏ `not-installed.` (بدون نام بسته) : فایل‌هایی که عمداً نصب نمی‌شوند
	-  سرویس و سیستم
		- ‏`service`، `.timer.` : واحد systemd؛ `dh_installsystemd` فعال‌سازی را خودکار اضافه می‌کند
		- ‏`default.` : در مسیر  `<etc/default/<packge/` قرار میگیرد
		- ‏`init.` : اسکریپت init قدیمی (SysV)
		- ‏`logrotate.` : در مسیر  `<etc/logrotate.d/<packge/` قرار میگیرد
		- ‏`cron.d`، `.cron.daily.` : کارهای زمان‌بندی‌شده
		- ‏`udev.` : قوانین udev
		- ‏`tmpfiles`، `.sysusers.` : پوشه‌های موقت و کاربران سیستمی با systemd
	- اسکریپت‌ها و تنظیمات
		- ‏ `preinst`، `.postinst`، `.prerm`، `.postrm.`: اسکریپت‌های نگهدار
		- ‏ `.conffiles.` : فهرست فایل‌های تنظیمات؛ فایل‌های `/etc` خودکار conffile می‌شوند و معمولاً لازم نیست
		- ‏ `.maintscript.` : حذف یا تغییر نام conffile هنگام ارتقا، بدون نوشتن اسکریپت
		- ‏ `.triggers.` : اعلام علاقه به رویدادها
		- ‏ `templates`، `.config.` :سؤال‌های debconf
	- کتابخانه‌ها
		- ‏`symbols.` : نسخه‌ی دقیق هر نماد کتابخانه (برای وابستگی دقیق)
		- ‏`shlibs.` : شناسایی کتابخانه برای `dpkg-shlibdeps` (روش قدیمی‌تر)
	-  اطلاع‌رسانی و کیفیت
		- ‏`NEWS.` : خبر مهم هنگام ارتقا (نشان‌داده‌شده با `apt-listchanges`)
		- ‏`README.Debian.` : توضیحات خاص دبیان درباره‌ی بسته
		- ‏ `lintian-overrides.`  : بی‌اثر کردن یک هشدار `lintian` که عمدی است
- **فایل‌های کمکی سورس**
	- ‏`patches/series` : فهرست ترتیب وصله‌ها (فقط در `3.0 (quilt)`)
	- ‏`patches/*.patch` : وصله‌هایی که روی سورس بالادستی اعمال می‌شوند
	- ‏`watch` : چک کردن نسخه‌ی جدید بالادست، مثلاً `version=4` و یک خط URL
	- ‏`clean` : فایل‌هایی که موقع `clean` پاک شوند
	- ‏`source/options` : گزینه‌های `dpkg-source` (مثل نادیده گرفتن فایل‌ها)
	- ‏`source/lintian-overrides` : بی‌اثر کردن هشدارهای مربوط به سورس
	- ‏`upstream/metadata` : اطلاعات پروژه‌ی بالادستی (مخزن، ردیاب باگ)
	- ‏`tests/control` : تست‌های autopkgtest (تست نصب‌شده بعد از ساخت)
	- ‏`gbp.conf` : تنظیم `git-buildpackage`
	- ‏`salsa-ci.yml` : 

در آخر هم فایل‌هایی که ابزار خودش تولید می‌کند میاید.

## part 1 

`debian/control`:

```
Source: hello-deb
Section: utils
Priority: optional
Maintainer: Your Name <you@example.com>
Build-Depends: debhelper-compat (= 13)
Standards-Version: 4.6.2

Package: hello-deb
Architecture: any
Depends: ${shlibs:Depends}, ${misc:Depends}
Description: A simple hello world program
 Reads its greeting from /etc/hello-deb.conf.
```

دو بخش دارد: بخش `Source` درباره سورس و بخش `Package` درباره بسته باینری. `${shlibs:Depends}` را `dpkg-shlibdeps` خودکار پر می‌کند.

`debian/rules` :
این فایل باید اجرایی باشد

```make
#!/usr/bin/make -f
%:
	dh $@
```

همین دو خط کار اصلی را انجام می‌دهد. `dh` به‌ترتیب `dh_auto_build` (اجرای `make`)، `dh_auto_install` (اجرای `make install DESTDIR=debian/hello-deb`)، `dh_strip`، `dh_shlibdeps`، `dh_installdeb` (که فایل‌های `/etc` را خودکار conffile می‌کند) و `dh_md5sums` و `dh_builddeb` را صدا می‌زند.

`debian/changelog` :
(فرمت دقیق به این صورت است. دو فاصله قبل از `*` و یک فاصله قبل از `--`):

```
hello-deb (1.0) unstable; urgency=medium

  * Initial release.

 -- Your Name <you@example.com>  Wed, 14 Oct 2026 10:50:00 +0330
```

`debian/source/format`:

```
3.0 (native)
```

`debian/copyright`:

```
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Upstream-Name: hello-deb

Files: *
Copyright: 2026 Your Name
License: GPL-3+
 This program is free software; see /usr/share/common-licenses/GPL-3
 for the full license text.
```

دستورها:

```bash
➜  chmod +x debian/rules
➜  cd hello-deb-1.0
➜  dpkg-buildpackage -us -uc -b

```
سویچ `us -uc-` یعنی امضا نکن (کلید GPG لازم نیست) و `b-` یعنی فقط بسته باینری بساز. 

```bash
➜  dpkg-buildpackage -us -uc -b
dpkg-buildpackage: info: source package hello-deb
dpkg-buildpackage: info: source version 1.0
dpkg-buildpackage: info: source distribution unstable
dpkg-buildpackage: info: source changed by Your Name <you@example.com>
dpkg-buildpackage: info: host architecture amd64
 dpkg-source --before-build .
 fakeroot debian/rules clean
dh clean                                                                                                                                                                                                                            
   dh_auto_clean
        make -j12 clean
make[1]: Entering directory '/home/ubuntu/CLionProjects/untitled/hello-deb-1.0'
rm -f hello
make[1]: Leaving directory '/home/ubuntu/CLionProjects/untitled/hello-deb-1.0'
   dh_clean
 debian/rules build
dh build                                                                                                                                                                                                                            
   dh_update_autotools_config
   dh_autoreconf
   dh_auto_configure
   dh_auto_build
        make -j12 "INSTALL=install --strip-program=true"
make[1]: Entering directory '/home/ubuntu/CLionProjects/untitled/hello-deb-1.0'
cc -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/home/ubuntu/CLionProjects/untitled/hello-deb-1.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/home/ubuntu/CLionProjects/untitled/hello-deb-1.0=/usr/src/hello-deb-1.0 -Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -o hello-deb hello.c
make[1]: Leaving directory '/home/ubuntu/CLionProjects/untitled/hello-deb-1.0'
   dh_auto_test
   create-stamp debian/debhelper-build-stamp
 fakeroot debian/rules binary
dh binary                                                                                                                                                                                                                           
   dh_testroot
   dh_prep
   dh_auto_install --destdir=debian/hello-deb/
        make -j12 install DESTDIR=/home/ubuntu/CLionProjects/untitled/hello-deb-1.0/debian/hello-deb AM_UPDATE_INFO_DIR=no "INSTALL=install --strip-program=true"
make[1]: Entering directory '/home/ubuntu/CLionProjects/untitled/hello-deb-1.0'
cc -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/home/ubuntu/CLionProjects/untitled/hello-deb-1.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/home/ubuntu/CLionProjects/untitled/hello-deb-1.0=/usr/src/hello-deb-1.0 -Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -o hello-deb hello.c
install -D -m 0755 hello-deb /home/ubuntu/CLionProjects/untitled/hello-deb-1.0/debian/hello-deb/usr/bin/hello-deb
install -D -m 0644 hello-deb.conf /home/ubuntu/CLionProjects/untitled/hello-deb-1.0/debian/hello-deb/etc/hello-deb.conf
make[1]: Leaving directory '/home/ubuntu/CLionProjects/untitled/hello-deb-1.0'
   dh_installdocs
   dh_installchangelogs
   dh_perl
   dh_link
   dh_strip_nondeterminism
   dh_compress
   dh_fixperms
   dh_missing
   dh_dwz -a
   dh_strip -a
e7616e4517f360803698b856151e0d2e4d7e74db
   dh_makeshlibs -a
   dh_shlibdeps -a
   dh_installdeb
   dh_gencontrol
   dh_md5sums
   dh_builddeb
dpkg-deb: building package 'hello-deb' in '../hello-deb_1.0_amd64.deb'.
dpkg-deb: building package 'hello-deb-dbgsym' in 'debian/.debhelper/scratch-space/build-hello-deb/hello-deb-dbgsym_1.0_amd64.deb'.
        Renaming hello-deb-dbgsym_1.0_amd64.deb to hello-deb-dbgsym_1.0_amd64.ddeb
 dpkg-genbuildinfo --build=binary -O../hello-deb_1.0_amd64.buildinfo
 dpkg-genchanges --build=binary -O../hello-deb_1.0_amd64.changes                                                                                                                                                                    
dpkg-genchanges: info: binary-only upload (no source code included)                                                                                                                                                                 
 dpkg-source --after-build .
dpkg-buildpackage: info: binary-only upload (no source included)       
```

دستور `dpkg-buildpackage` پوشه‌های `debian/hello-deb/` و فایل‌های `debhelper.` را داخل سورس (همان مسیر `/debian`) تولید می‌کند.
این پوشه **حافظه و فضای کار موقت** ابزار `debhelper` است در `.deb` نهایی هم نمی‌آید. با `dh_clean` یا `debian/rules clean` میتوان پاکشان کرد.

### چه چیزهایی داخل  `debhelper.`   است
```bash
➜  tree 
.
├── generated
│   ├── hello-deb
│   │   ├── dh_installchangelogs.dch.trimmed
│   │   └── installed-by-dh_installdocs
│   └── _source
│       └── home
├── hello-deb
│   ├── dbgsym-build-ids
│   └── dbgsym-root
│       ├── DEBIAN
│       │   ├── control
│       │   └── md5sums
│       └── usr
│           ├── lib
│           │   └── debug
│           └── share
│               └── doc
│                   └── hello-deb-dbgsym -> hello-deb
└── scratch-space
    └── build-hello-deb

15 directories, 6 files
```



خروجی در پوشه **بالاتر** ساخته می‌شود:

```bash
cd ..
➜  ls *.deb *.buildinfo *.changes
hello-deb_1.0_amd64.buildinfo  hello-deb_1.0_amd64.changes  hello-deb_1.0_amd64.deb
lintian hello-deb_1.0_amd64.deb
sudo apt install ./hello-deb_1.0_amd64.deb
```

فایل‌های`changes.` و `buildinfo.` هر دو را `dpkg-buildpackage` بعد از ساخت، **کنار** بسته‌ی `.deb` تولید می‌کند یعنی در پوشه‌ی بالاتر از سورس. هیچ‌کدام داخل `deb.` نمی‌آیند.
