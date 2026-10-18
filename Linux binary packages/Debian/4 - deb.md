
# پوشه `debian/.debhelper/`

این پوشه **حافظه و فضای کار موقت** ابزار `debhelper` است. چون اسمش با نقطه شروع می‌شود، با `ls` معمولی دیده نمی‌شود و باید `ls -A debian/` بزنی. در `.deb` نهایی هم نمی‌آید.

## چه چیزهایی داخلش است

محتوایش به نسخه‌ی debhelper بستگی دارد.
```bash
cd ~/hello-deb-1.0
ls -A debian/
find debian/.debhelper | head -30
```

معمولاً این‌ها :

| محتوا                                                | کاربرد                                                                                                    |
| ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| لاگ هر بسته (`<بسته>.log` یا `<بسته>.debhelper.log`) | ثبت اینکه کدام دستورهای `dh_*` برای آن بسته اجرا شده‌اند                                                  |
| `generated/<بسته>/`                                  | قطعه‌کدهای تولیدشده برای اسکریپت‌ها (مثلاً بخش‌های مربوط به systemd که بعداً جای `#DEBHELPER#` می‌نشینند) |
| `<بسته>/dbgsym-root/`                                | پوشه‌ی موقت نمادهای دیباگ که `dh_strip` از باینری جدا می‌کند                                              |

## چرا لاگ لازم است

دستور `dh` که در `debian/rules` نوشتیم یک **ردیف از مراحل** را به‌ترتیب اجرا می‌کند (`dh_auto_build`، `dh_auto_install`، `dh_strip` و بقیه). هر مرحله که تمام شود، در لاگ ثبت می‌شود. این ثبت دو کاربرد دارد:

1. **ادامه از وسط:** اگر ساخت در مرحله‌ای شکست بخورد و مشکل را رفع کنیم، `dh` مراحل انجام‌شده را رد می‌کند و از همان‌جا ادامه می‌دهد.
2. **اجرای مرحله‌ای دستی:** گزینه‌هایی مثل `dh binary --until dh_strip` یا `--after` به همین لاگ تکیه دارند تا بدانند از کجا شروع کنند.

فایل‌های `generated/` هم باید بین مراحل بمانند، چون یک مرحله آن‌ها را می‌سازد و مرحله‌ی بعد (مثلاً `dh_installdeb`) می‌خواند.

## آیا می‌شود پاکش کرد؟

بله، کاملاً امن است. این پوشه فقط **حالت** است و منبع حقیقت نیست. اگر پاکش کنی، بعدی که `dh` اجرا شود همه‌ی مراحل را از اول انجام می‌دهد و هیچ چیزی خراب نمی‌شود. فقط وسط یک ساخت نیمه‌کاره پاک نکن، چون پیشرفت آن ساخت را از دست می‌دهی.

## چرا خودکار پاک نمی‌شود

دو دلیل دارد:

**۱. `dpkg-buildpackage` پاکسازی را قبل از ساخت انجام می‌دهد، نه بعد از آن.** ترتیبش این است:

```
debian/rules clean   ← اول پاک می‌کند
debian/rules build
debian/rules binary
```

پس هر ساخت تازه از صفر شروع می‌شود، ولی نتیجه‌ی ساخت پیشین بعد از تمام شدن می‌ماند.

**۲. ماندن آن‌ها مفید است.** بعد از ساخت می‌توانی پوشه‌ی `debian/hello-deb/` (همان `pkgroot`) را بررسی کنی، فایل‌های تولیدشده را بخوانی و علت مشکل را پیدا کنی. اگر خودکار پاک می‌شدند، دیباگ ساخت‌های شکست‌خورده سخت‌تر می‌شد.

## چطور پاک کنیم

```bash
debian/rules clean           # پاکسازی استاندارد
dh_clean                     # معادل
dpkg-buildpackage -tc ...    # پاکسازی خودکار بعد از ساخت (-tc = target clean)
dpkg-buildpackage -nc ...    # پاک نکردن قبل از ساخت (برای ساخت سریع تکراری)
```

آزمایش برای دیدن اثر:

```bash
debian/rules clean
ls -A debian/          # فقط فایل‌های خودت مانده‌اند
dpkg-buildpackage -us -uc -b
ls -A debian/          # دوباره .debhelper، substvars، پوشه‌ی بسته و غیره
cat debian/files
```

## یک نکته برای گیت

همه‌ی این‌ها فایل تولیدشده‌اند و نباید در مخزن گیت بروند. در `.gitignore` بنویس:

```
debian/.debhelper/
debian/*.debhelper
debian/*.debhelper.log
debian/*.substvars
debian/debhelper-build-stamp
debian/files
debian/hello-deb/
```

فایل‌هایی که خودت نوشته‌ای (`control`، `rules`، `changelog`، `copyright`، `source/format`، `*.install` و بقیه) را حتماً در گیت نگه دار.


# فایل های `changes.` و  `buildinfo.`

## فایل `changes.`  :  «بسته‌ی تحویلی»

یک فایل متنی است که **فهرست و مشخصات همه‌ی فایل‌های یک آپلود** را می‌دهد. مخزن (مثل آرشیو دبیان) اول این فایل را می‌خواند و بر اساس آن بقیه را دریافت و بررسی می‌کند.


```bash
➜  cat hello-deb_1.0_amd64.changes 
Format: 1.8
Date: Wed, 14 Oct 2026 10:50:00 +0330
Source: hello-deb
Binary: hello-deb
Built-For-Profiles: noudeb
Architecture: amd64
Version: 1.0
Distribution: unstable
Urgency: medium
Maintainer: Your Name <you@example.com>
Changed-By: Your Name <you@example.com>
Description:
 hello-deb  - A simple hello world program
Changes:
 hello-deb (1.0) unstable; urgency=medium
 .
   * Initial release.
Checksums-Sha1:
 fec4669fc9452164fe834a3758b75b1f85fe8a60 4424 hello-deb-dbgsym_1.0_amd64.ddeb
 47469a66b95999fe31a177483bfb1456d271009f 6186 hello-deb_1.0_amd64.buildinfo
 34680bd22a1d83c07917509c0aeb4c97cda2b59c 3130 hello-deb_1.0_amd64.deb
Checksums-Sha256:
 0febf252d59bb361f93d8d61cbd700e62ad42835c4291bfdbae751a2dc8a3588 4424 hello-deb-dbgsym_1.0_amd64.ddeb
 5d9ec7155fd13c485689712e2545fb43b1b29c62522fd2d77e11d09a4a2ff9e3 6186 hello-deb_1.0_amd64.buildinfo
 55cd9500bfdbd63a1942d251d0831efc3f742ae847812131303632b2e5788ee5 3130 hello-deb_1.0_amd64.deb
Files:
 4295aadbf4f38ac02046c27d611df23e 4424 debug optional hello-deb-dbgsym_1.0_amd64.ddeb
 4187412dfed4ad83e4dd7851172c2ae4 6186 utils optional hello-deb_1.0_amd64.buildinfo
 31bc469a5d65edb39941f2a39ff2a580 3130 utils optional hello-deb_1.0_amd64.deb

```

فیلدهای مهم:

|فیلد|معنی|
|---|---|
|`Source`، `Binary`، `Version`، `Architecture`|چه چیزی ساخته شده|
|`Distribution`|به کدام شاخه بروید (`unstable`، `jammy` و غیره). از خط اول `changelog` می‌آید|
|`Urgency`|فوریت (از `changelog`)|
|`Maintainer`، `Changed-By`|نگهدارنده و کسی که این نسخه را ساخته|
|`Description`|فهرست بسته‌های باینری با توضیح یک‌خطی|
|`Changes`|متن آخرین ورودی `changelog`|
|`Checksums-Sha1`، `Checksums-Sha256`، `Files`|اندازه و هش **هر فایل** آپلود (`.deb`، `.dsc`، `.buildinfo` و غیره)|

بخش هش‌ها مهم‌ترین قسمت است: مخزن با آن مطمئن می‌شود هیچ فایلی در راه خراب یا عوض نشده. فایل معمولاً با GPG امضا می‌شود (در ابتدای فایل `-----BEGIN PGP SIGNED MESSAGE-----` می‌بینی). این امضا نشان می‌دهد چه کسی بسته را تحویل داده است.

کاربرد اصلی:

```bash
debsign hello-deb_1.0_amd64.changes          # امضا (اگر با -us -uc ساخته‌ای)
dput ppa:youruser/yourppa hello-deb_1.0_amd64.changes   # آپلود
```

ابزار `dput` (یا `dupload`) اول `.changes` را می‌خواند، همه‌ی فایل‌هایی که فهرست کرده را با هم می‌فرستد و سرور آن‌ها را اعتبارسنجی می‌کند.

برای ساخت و نصب محلی، که فقط `.deb` را با `dpkg -i` نصب می‌کنی، `.changes` هیچ کاری نمی‌کند.

## فایل `buildinfo.` : «رسید محیط ساخت»

یک فایل متنی دیگر که ثبت می‌کند این بسته **دقیقاً در چه شرایطی** ساخته شده، تا کسی دیگر بتواند همان محیط را بازسازی کند.

```bash
➜  cat hello-deb_1.0_amd64.buildinfo
Format: 1.0
Source: hello-deb
Binary: hello-deb hello-deb-dbgsym
Architecture: amd64
Version: 1.0
Checksums-Md5:
 4295aadbf4f38ac02046c27d611df23e 4424 hello-deb-dbgsym_1.0_amd64.ddeb
 31bc469a5d65edb39941f2a39ff2a580 3130 hello-deb_1.0_amd64.deb
Checksums-Sha1:
 fec4669fc9452164fe834a3758b75b1f85fe8a60 4424 hello-deb-dbgsym_1.0_amd64.ddeb
 34680bd22a1d83c07917509c0aeb4c97cda2b59c 3130 hello-deb_1.0_amd64.deb
Checksums-Sha256:
 0febf252d59bb361f93d8d61cbd700e62ad42835c4291bfdbae751a2dc8a3588 4424 hello-deb-dbgsym_1.0_amd64.ddeb
 55cd9500bfdbd63a1942d251d0831efc3f742ae847812131303632b2e5788ee5 3130 hello-deb_1.0_amd64.deb
Build-Origin: Ubuntu
Build-Architecture: amd64
Build-Date: Sun, 18 Oct 2026 13:38:45 +0330
Build-Tainted-By:
 merged-usr-via-aliased-dirs
Installed-Build-Depends:
 autoconf (= 2.71-3),
 automake (= 1:1.16.5-1.3ubuntu1),
 autopoint (= 0.21-14ubuntu2),
 autotools-dev (= 20220109.1),
 base-files (= 13ubuntu10.5),
 base-passwd (= 3.6.3build1),
 bash (= 5.2.21-2ubuntu4),
 binutils (= 2.42-4ubuntu2.10),
 binutils-common (= 2.42-4ubuntu2.10),
 binutils-x86-64-linux-gnu (= 2.42-4ubuntu2.10),
 bsdextrautils (= 2.39.3-9ubuntu6.6),
 bsdutils (= 1:2.39.3-9ubuntu6.6),
 build-essential (= 12.10ubuntu1),
 bzip2 (= 1.0.8-5.1ubuntu0.1),
 coreutils (= 9.4-3ubuntu6.3),
 cpp (= 4:13.2.0-7ubuntu1),
 cpp-13 (= 13.3.0-6ubuntu2~24.04.1),
 cpp-13-x86-64-linux-gnu (= 13.3.0-6ubuntu2~24.04.1),
 cpp-x86-64-linux-gnu (= 4:13.2.0-7ubuntu1),
 dash (= 0.5.12-6ubuntu5),
 debconf (= 1.5.86ubuntu1),
 debhelper (= 13.14.1ubuntu5),
 debianutils (= 5.17build1),
 debugedit (= 1:5.0-5build2),
 dh-autoreconf (= 20),
 dh-strip-nondeterminism (= 1.13.1-1),
 diffutils (= 1:3.10-1ubuntu0.1),
 dpkg (= 1.22.6ubuntu6.6),
 dpkg-dev (= 1.22.6ubuntu6.6),
 dwz (= 0.15-1build6),
 file (= 1:5.45-3build1),
 findutils (= 4.9.0-5build1),
 g++ (= 4:13.2.0-7ubuntu1),
 g++-13 (= 13.3.0-6ubuntu2~24.04.1),
 g++-13-x86-64-linux-gnu (= 13.3.0-6ubuntu2~24.04.1),
 g++-x86-64-linux-gnu (= 4:13.2.0-7ubuntu1),
 gcc (= 4:13.2.0-7ubuntu1),
 gcc-13 (= 13.3.0-6ubuntu2~24.04.1),
 gcc-13-base (= 13.3.0-6ubuntu2~24.04.1),
 gcc-13-x86-64-linux-gnu (= 13.3.0-6ubuntu2~24.04.1),
 gcc-14-base (= 14.2.0-4ubuntu2~24.04.1),
 gcc-x86-64-linux-gnu (= 4:13.2.0-7ubuntu1),
 gettext (= 0.21-14ubuntu2),
 gettext-base (= 0.21-14ubuntu2),
 grep (= 3.11-4build1),
 groff-base (= 1.23.0-3build2),
 gzip (= 1.12-1ubuntu3.2),
 hostname (= 3.23+nmu2ubuntu2),
 init-system-helpers (= 1.66ubuntu1),
 install-info (= 7.1-3build2),
 intltool-debian (= 0.35.0+20060710.6),
 libacl1 (= 2.3.2-1build1.1),
 libarchive-zip-perl (= 1.68-1),
 libasan8 (= 14.2.0-4ubuntu2~24.04.1),
 libatomic1 (= 14.2.0-4ubuntu2~24.04.1),
 libattr1 (= 1:2.5.2-1ubuntu0.1),
 libaudit-common (= 1:3.1.2-2.1ubuntu0.1),
 libaudit1 (= 1:3.1.2-2.1ubuntu0.1),
 libbinutils (= 2.42-4ubuntu2.10),
 libblkid1 (= 2.39.3-9ubuntu6.6),
 libbz2-1.0 (= 1.0.8-5.1ubuntu0.1),
 libc-bin (= 2.39-0ubuntu8.9),
 libc-dev-bin (= 2.39-0ubuntu8.9),
 libc6 (= 2.39-0ubuntu8.9),
 libc6-dev (= 2.39-0ubuntu8.9),
 libcap-ng0 (= 0.8.4-2build2),
 libcap2 (= 1:2.66-5ubuntu2.4),
 libcc1-0 (= 14.2.0-4ubuntu2~24.04.1),
 libcrypt-dev (= 1:4.4.36-4build1),
 libcrypt1 (= 1:4.4.36-4build1),
 libctf-nobfd0 (= 2.42-4ubuntu2.10),
 libctf0 (= 2.42-4ubuntu2.10),
 libdb5.3t64 (= 5.3.28+dfsg2-7),
 libdebconfclient0 (= 0.271ubuntu3),
 libdebhelper-perl (= 13.14.1ubuntu5),
 libdpkg-perl (= 1.22.6ubuntu6.6),
 libdw1t64 (= 0.190-1.1ubuntu0.1),
 libelf1t64 (= 0.190-1.1ubuntu0.1),
 libfile-stripnondeterminism-perl (= 1.13.1-1),
 libgcc-13-dev (= 13.3.0-6ubuntu2~24.04.1),
 libgcc-s1 (= 14.2.0-4ubuntu2~24.04.1),
 libgcrypt20 (= 1.10.3-2ubuntu0.2),
 libgdbm-compat4t64 (= 1.23-5.1build1),
 libgdbm6t64 (= 1.23-5.1build1),
 libgmp10 (= 2:6.3.0+dfsg-2ubuntu6.1),
 libgomp1 (= 14.2.0-4ubuntu2~24.04.1),
 libgpg-error0 (= 1.47-3build2.1),
 libgprofng0 (= 2.42-4ubuntu2.10),
 libhwasan0 (= 14.2.0-4ubuntu2~24.04.1),
 libicu74 (= 74.2-1ubuntu3.1),
 libisl23 (= 0.26-3build1.1),
 libitm1 (= 14.2.0-4ubuntu2~24.04.1),
 libjansson4 (= 2.14-2build2),
 liblsan0 (= 14.2.0-4ubuntu2~24.04.1),
 liblz4-1 (= 1.9.4-1build1.1),
 liblzma5 (= 5.6.1+really5.4.5-1ubuntu0.3),
 libmagic-mgc (= 1:5.45-3build1),
 libmagic1t64 (= 1:5.45-3build1),
 libmd0 (= 1.1.0-2build1.1),
 libmount1 (= 2.39.3-9ubuntu6.6),
 libmpc3 (= 1.3.1-1build1.1),
 libmpfr6 (= 4.2.1-1build1.1),
 libpam-modules (= 1.5.3-5ubuntu5.7),
 libpam-modules-bin (= 1.5.3-5ubuntu5.7),
 libpam-runtime (= 1.5.3-5ubuntu5.7),
 libpam0g (= 1.5.3-5ubuntu5.7),
 libpcre2-8-0 (= 10.42-4ubuntu2.1),
 libperl5.38t64 (= 5.38.2-3.2ubuntu0.6),
 libpipeline1 (= 1.5.7-2),
 libquadmath0 (= 14.2.0-4ubuntu2~24.04.1),
 libseccomp2 (= 2.5.5-1ubuntu3.1),
 libselinux1 (= 3.5-2ubuntu2.1),
 libsframe1 (= 2.42-4ubuntu2.10),
 libsmartcols1 (= 2.39.3-9ubuntu6.6),
 libssl3t64 (= 3.0.13-0ubuntu3.16),
 libstdc++-13-dev (= 13.3.0-6ubuntu2~24.04.1),
 libstdc++6 (= 14.2.0-4ubuntu2~24.04.1),
 libsub-override-perl (= 0.10-1),
 libsystemd0 (= 255.4-1ubuntu8.18),
 libtinfo6 (= 6.4+20240113-1ubuntu2.2),
 libtool (= 2.4.7-7build1),
 libtsan2 (= 14.2.0-4ubuntu2~24.04.1),
 libubsan1 (= 14.2.0-4ubuntu2~24.04.1),
 libuchardet0 (= 0.0.8-1build1),
 libudev1 (= 255.4-1ubuntu8.18),
 libunistring5 (= 1.1-2build1.1),
 libuuid1 (= 2.39.3-9ubuntu6.6),
 libxml2 (= 2.9.14+dfsg-1.3ubuntu3.10),
 libzstd1 (= 1.5.5+dfsg2-2build1.1),
 linux-libc-dev (= 6.8.0-147.147),
 login (= 1:4.13+dfsg1-4ubuntu3.2),
 lto-disabled-list (= 47),
 m4 (= 1.4.19-4build1),
 make (= 4.3-4.1build2),
 man-db (= 2.12.0-4build2),
 mawk (= 1.3.4.20240123-1build1),
 ncurses-base (= 6.4+20240113-1ubuntu2.2),
 ncurses-bin (= 6.4+20240113-1ubuntu2.2),
 patch (= 2.7.6-7build3),
 perl (= 5.38.2-3.2ubuntu0.6),
 perl-base (= 5.38.2-3.2ubuntu0.6),
 perl-modules-5.38 (= 5.38.2-3.2ubuntu0.6),
 po-debconf (= 1.0.21+nmu1),
 rpcsvc-proto (= 1.4.2-0ubuntu7),
 sed (= 4.9-2ubuntu0.24.04.1),
 sensible-utils (= 0.0.22),
 sysvinit-utils (= 3.08-6ubuntu3),
 tar (= 1.35+dfsg-3ubuntu0.4),
 util-linux (= 2.39.3-9ubuntu6.6),
 xz-utils (= 5.6.1+really5.4.5-1ubuntu0.3),
 zlib1g (= 1:1.3.dfsg-3.1ubuntu2.2)
Environment:
 DEB_BUILD_OPTIONS="parallel=12"
 DEB_BUILD_PROFILES="noudeb"
 LANG="en_US.UTF-8"
 SOURCE_DATE_EPOCH="1791962400"
```

فیلدهای مهم:

|فیلد|معنی|
|---|---|
|`Checksums-Sha256`|هش فایل‌های **خروجی** (`.deb`). مرجع مقایسه است|
|`Build-Architecture` و `Build-Date`|ماشین و زمان ساخت|
|`Build-Origin`|توزیع و محیط ساخت|
|`Build-Tainted-By`|هشدار اگر محیط «آلوده» بود، مثلاً مسیرهای `/usr/local` فعال یا `usr-is-merged`|
|`Installed-Build-Depends`|**نسخه‌ی دقیق همه‌ی بسته‌هایی** که موقع ساخت نصب بودند (`gcc`، `libc6`، `debhelper` و غیره)|
|`Environment`|متغیرهای محیطی تأثیرگذار (`DEB_BUILD_OPTIONS`، `LANG`، `SOURCE_DATE_EPOCH`)|

## «بازتولیدپذیری» یعنی چه

ایده این است: **اگر همان سورس را در همان محیط بسازی، باید همان `.deb` بایت‌به‌بایت یکسان درآید.** چرا مهم است؟ چون وقتی ببینی بسته‌ی منتشرشده با آنچه خودت از روی سورس ساختی یکسان است، مطمئن می‌شوی کسی در مرحله‌ی ساخت، کد مخربی وارد نکرده است.

مشکل این است که ساخت معمولاً نتیجه‌ی متفاوت می‌دهد، چون چیزهای زیر وارد بسته می‌شوند:

- زمان ساخت فایل‌ها داخل `tar`
- ترتیب فایل‌ها در فهرست
- مسیر پوشه‌ی ساخت و نام کاربر
- نسخه‌ی دقیق کامپایلر و کتابخانه‌ها

برای حل آن، `.buildinfo` دو چیز را ثبت می‌کند: **محیط** (نسخه‌ی همه‌ی ابزارها) و **هش نتیجه**. بعد کسی که می‌خواهد بررسی کند:

1. همان نسخه‌ی بسته‌ها را در یک محیط تمیز نصب می‌کند (`debrebuild` این کار را خودکار انجام می‌دهد)
2. سورس را دوباره می‌سازد
3. هش `.deb` جدید را با `Checksums-Sha256` فایل `.buildinfo` مقایسه می‌کند

اگر یکی بودند، ساخت بازتولیدپذیر است. اگر نبودند، `diffoscope` تفاوت دو فایل را به‌صورت ساختاریافته نشان می‌دهد (حتی داخل آرشیو و فشرده‌سازی).

## آزمایش

هش فایل `.deb` خودت را با هش ثبت‌شده مقایسه کن:

```bash
sha256sum hello-deb_1.0_amd64.deb
grep -A3 Checksums-Sha256 hello-deb_1.0_amd64.buildinfo
```

باید یکی باشند. حالا بسته را دوباره بساز و دوباره مقایسه کن:

```bash
cd ~/hello-deb-1.0
dpkg-buildpackage -us -uc -b
cd ..
sha256sum hello-deb_1.0_amd64.deb
```

در روش `debhelper` معمولاً هش یکسان درمی‌آید، چون `dpkg-buildpackage` متغیر `SOURCE_DATE_EPOCH` را از تاریخ `changelog` می‌گیرد و زمان فایل‌ها را به آن محدود می‌کند. برای مقایسه، روش دستی (`make deb` خودمان) را دو بار با فاصله‌ی چند ثانیه اجرا کن و هش‌ها را ببین. احتمالاً **متفاوت** خواهند بود، چون زمان فایل‌ها داخل `data.tar` عوض شده. برای رفع، در `Makefile` پیش از ساخت فایل‌ها را با یک زمان ثابت `touch` کن و `tar` را با `--mtime` اجرا کن، یا `SOURCE_DATE_EPOCH` را ست کن. این همان کاری است که `debhelper` برایت خودکار می‌کند.
