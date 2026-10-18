ساده ترین نوع package ساختار زیر را دارد 
```
package name/
├── DEBIAN/
└── usr/
    └── bin/
        └── binary
```
دو directory زیر باید حداقل باشند:
-  مسیر `DEBIAN` : برای نوشتن اطلاعات پکیج
- مسیری که میخواهیم روی سیستم مقصد فایل باینری در ان قرار بگیرد (در اینجا مثلا `usr/bin/`) 

یک برنامه ساده به زبان C مینویسم و ان را package میکنیم
```c
#include <stdio.h>

int main(void){
	printf("Hello from my first deb package!\n");

	return 0;
}
```

برنامه را کامپایل میکنیم 
```bash
➜  gcc hello.c -o hello
```

با سویچ `Wall-` میتوان تمام هشدارهای مهم زمان کامپایل را دید.
```bash
➜  gcc -Wall hello.c -o hello
```

حالا که باینری را داریم یک directory با اسم دلخواه میسازیم. (اسمی که به عنوان اسم پکیج شناخته میشود)
```
➜  mkdir hello.1.0
```

در درون این directory باید حداقل ۲ تا directory دیگه هم باشد 
```
➜  mkdir -p hello.1.0/DEBIAN
➜  mkdir -p hello.1.0/usr/bin/
```

فایل باینری را در مسیر `hello-app/usr/bin/` قرار میدهیم .
> معمولا مسیر `/usr/bin/` محل استاندارد برنامه‌های نصب‌شده توسط پکیج منیجر است و مسیر `/usr/local/bin/` محل برنامه‌هایی که خودمون نصب می‌کنیم

در مسیر `/hello-app/DEBIAN` یک فایل به اسم **control** میسازیم که metadata برنامه نهایی را در ان قرار میدهیم
```
Package: hello-deb
Version: 1.0
Section: utils
Priority: optional
Architecture: amd64
Maintainer: Your Name <you@example.com>
Description: A simple hello world program
 This is my first Debian package, built to learn the packaging workflow.
```

در نهایت دستور زیر را برای ساخت deb package میزنیم
```bash
➜  dpkg --build hello.1.0
dpkg-deb: building package 'hello-deb' in 'hello.1.0.deb'.
```

میتوان از سویچ `root-owner-group--` هم استفاده کرد که یعنی مالک نهایی باینری بعد از قرار گیری در سیستم مقصد root user باشد. بدون این سویچ `userID` کاربر فعلی برایش میماند

در نهایت یک فایل فشرده و ارشیو شده تحویل گرفته میشود
```bash
➜  file hello.1.0.deb 
hello.1.0.deb: Debian binary package (format 2.0), with control.tar.zst, data compression zst
```

## بررسی درون پکیج ساخته شده
```bash
➜  dpkg --contents hello.1.0.deb
drwxrwxr-x ubuntu/ubuntu     0 2026-10-14 13:38 ./
drwxrwxr-x ubuntu/ubuntu     0 2026-10-14 13:38 ./usr/
drwxrwxr-x ubuntu/ubuntu     0 2026-10-14 13:38 ./usr/bin/
-rwxrwxr-x ubuntu/ubuntu 15960 2026-10-14 13:38 ./usr/bin/hello
```

اگر از سویچ  `root-owner-group--` هم استفاده میکردیم مالکیت فایل های درون package به این شکل میشود(برای این سویچ حتما باید از دستور `dpkg-deb` استفاده شود)
```bash
➜  dpkg-deb --build --root-owner-group hello.1.0
dpkg-deb: building package 'hello-deb' in 'hello.1.0.deb'.

➜  dpkg --contents hello.1.0.deb 
drwxrwxr-x root/root         0 2026-10-14 13:38 ./
drwxrwxr-x root/root         0 2026-10-14 13:38 ./usr/
drwxrwxr-x root/root         0 2026-10-14 13:38 ./usr/bin/
-rwxrwxr-x root/root     15960 2026-10-14 13:38 ./usr/bin/hello
```

سویچ `info--` برای دیدن اطلاعات درون package 

```bash
➜  dpkg --info hello.1.0.deb 
 new Debian package, version 2.0.
 size 2596 bytes: control archive=281 bytes.
     241 bytes,     8 lines      control
 Package: hello-deb
 Version: 1.0
 Section: utils
 Priority: optional
 Architecture: amd64
 Maintainer: Your Name <you@example.com>
 Description: A simple hello world program
  This is my first Debian package, built to learn the packaging workflow.
```

در نهایت نصب میکنیم
```bash
➜  sudo  dpkg -i ./hello.1.0.deb 
(Reading database… 237377 files and directories currently installed.)
Preparing to unpack ./hello.1.0.deb…
Unpacking hello-deb (1.0) over (1.0)…
Setting up hello-deb (1.0)…

➜  dpkg -l | grep 'hello'
ii  hello-deb                                        1.0                                              amd64        A simple hello world program
```

سویچ `l-` لسیت برنامه های نصب شده را نشان میدهد
برای دیدن لیست فایل های نصب شده توسط یک برنامه خاص سویچ `L-` استفاده میشود
```bash
# dpkg -L <package-name>

➜  dpkg -L hello-deb 
/.
/usr
/usr/bin
/usr/bin/hello
```


برای این که ببینیم یک فایل متعلق به کدام package است سویچ `S-` استفاد میشود
```bash
➜  dpkg -S /usr/bin/hello 
hello-deb: /usr/bin/hello
```


برای حذف هم از سویچ های `r-` (برای پاک کردن باینری ) ویا `P-` (برای پاک کردن کل داده ها حتی فایل تنظیمات ) استفاده میشود
```bash
# dpkg -r <package-name>
➜  sudo dpkg -r hello-deb 
(Reading database… 237377 files and directories currently installed.)
Removing hello-deb (1.0)…


# dpkg -P <package-name>
➜  sudo dpkg -P hello-deb
(Reading database… 237377 files and directories currently installed.)
Removing hello-deb (1.0)…

```

سویچ `p-` هم داریم که اطلاعات اضافی package را چاپ میکند (اطلاعاتی نظیر مواردی که در مسیر ` /var/lib/dpkg/available/` باشند)


## تکمیلی
- دستور `dpkg -i`  تنها فایل هارا کپی میکند . اگر برنامه نیاز به dependency داشته باشد خطا میدهد. برای حل آن بعد از خطا دستور `sudo apt -f install` را میزنیم . سویچ `f-` کوتاه شده `fix-broken--` است
- میشه بجای این راه مستقیم با `apt` نصب را انجام دهیم.
```bash
sudo apt install ./hello-1.0.deb
```

