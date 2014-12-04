---
layout: post
title: "rpmbuild SPEC文件说明"
date: 2014-02-10 16:19:00 +0800
comments: false
categories:
- 2014
- 2014~02
- system
- system~rpm
tags:
---
在rpmbuild -ba时，遇到如下错误：  
ERROR: No build ID note found in /home/wuyang/rpmbuild/BUILDROOT/******  
error: Bad exit status from /var/tmp/rpm-tmp.BPd1OI (%install)
解决方法是在.spec文件中任意位置添加如下参数：  
%define __debug_install_post %{_rpmconfigdir}/find-debuginfo.sh %{?_find_debuginfo_opts} "%{_builddir}/%{?buildsubdir}" %{nil}  

----------

##### rpmbuild源代码打包

  将源代码打包，如 stardict-2.0.tar.gz，并将文件放到spec文件Source段所描述的路径下，通常为/usr/src/redhat /SOURCES/目录下（不同的Linux发布版本略有不同，如OpenSUSE为 /usr/src/packages/SOURCES/）

`rpmbuild -ba ‘spec文件路径’`  
（rpmbuild常用参数： -bb 只编译二进制rpm包 -bs 只编译源码rpm包 -ba 同时编译二进制和源码rpm包）  
build完后，可以在/usr/src/redhat/RPMS/下找到二进制rpm包，rpm包按照其对应的cpu体系结构分类，通常在/usr/src/redhat/RPMS/i386目录下  
/usr/src/redhat/SRPMS/下找到源码rpm包，此时由于是源代码，所以无须按体系结构分类。

----------

#### 一、关键字
spec脚本包括很多关键字，主要有：
```
Name: 软件包的名称，后面可使用%{name}的方式引用
Summary: 软件包的内容概要
Version: 软件的实际版本号，例如：1.0.1等，后面可使用%{version}引用
Release: 发布序列号，例如：1linuxing等，标明第几次打包，后面可使用%{release}引用
Group: 软件分组，建议使用标准分组
License: 软件授权方式，通常就是GPL
Source: 源代码包，可以带多个用Source1、Source2等源，后面也可以用%{source1}、%{source2}引用
BuildRoot: 这个是安装或编译时使用的“虚拟目录”，考虑到多用户的环境，一般定义为：
%{_tmppath}/%{name}-%{version}-%{release}-root
或
%{_tmppath}/%{name}-%{version}-%{release}-buildroot-%(%{__id_u} -n}
该参数非常重要，因为在生成rpm的过程中，执行make install时就会把软件安装到上述的路径中，在打包的时候，同样依赖“虚拟目录”为“根目录”进行操作。
后面可使用$RPM_BUILD_ROOT方式引用。
URL: 软件的主页
Vendor: 发行商或打包组织的信息，例如RedFlag Co,Ltd
Disstribution: 发行版标识
Patch: 补丁源码，可使用Patch1、Patch2等标识多个补丁，使用%patch0或%{patch0}引用
Prefix: %{_prefix} 这个主要是为了解决今后安装rpm包时，并不一定把软件安装到rpm中打包的目录的情况。这样，必须在这里定义该标识，并在编写%install脚本的时候引用，才能实现rpm安装时重新指定位置的功能
Prefix: %{_sysconfdir} 这个原因和上面的一样，但由于%{_prefix}指/usr，而对于其他的文件，例如/etc下的配置文件，则需要用%{_sysconfdir}标识
Build Arch: 指编译的目标处理器架构，noarch标识不指定，但通常都是以/usr/lib/rpm/marcros中的内容为默认值
Requires: 该rpm包所依赖的软件包名称，可以用>=或<=表示大于或小于某一特定版本，例如：
libpng-devel >= 1.0.20 zlib
※“>=”号两边需用空格隔开，而不同软件名称也用空格分开
还有例如PreReq、Requires(pre)、Requires(post)、Requires(preun)、Requires(postun)、BuildRequires等都是针对不同阶段的依赖指定
Provides: 指明本软件一些特定的功能，以便其他rpm识别
Packager: 打包者的信息
%description 软件的详细说明
```

#### 二、spec脚本主体
spec脚本的主体中也包括了很多关键字和描述。  
##### %prep 预处理脚本  
##### %setup -n %{name}-%{version} 把源码包解压并放好  
通常是从/usr/src/asianux/SOURCES里的包解压到/usr/src/asianux/BUILD/%{name}-%{version}中。  
一般用%setup -c就可以了，但有两种情况：一就是同时编译多个源码包，二就是源码的tar包的名称与解压出来的目录不一致，此时，就需要使用-n参数指定一下了。  

```
%setup 不加任何选项，仅将软件包打开。
%setup -n newdir 将软件包解压在newdir目录。
%setup -c 解压缩之前先产生目录。
%setup -b num 将第num个source文件解压缩。
%setup -T 不使用default的解压缩操作。
%setup -T -b 0 将第0个源代码文件解压缩。
%setup -c -n newdir 指定目录名称newdir，并在此目录产生rpm套件。
%patch 最简单的补丁方式，自动指定patch level。
%patch 0 使用第0个补丁文件，相当于%patch ?p 0。
%patch -s 不显示打补丁时的信息。
%patch -T 将所有打补丁时产生的输出文件删除。
```
##### %patch 打补丁  
通常补丁都会一起在源码tar.gz包中，或放到SOURCES目录下。一般参数为：  
%patch -p1 使用前面定义的Patch补丁进行，-p1是忽略patch的第一层目录  
%Patch2 -p1 -b xxx.patch 打上指定的补丁，-b是指生成备份文件  

##### %configure 这个不是关键字，而是rpm定义的标准宏命令。意思是执行源代码的configure配置
在/usr/src/asianux/BUILD/%{name}-%{version}目录中进行，使用标准写法，会引用/usr/lib/rpm/marcros中定义的参数。
另一种不标准的写法是，可参考源码中的参数自定义，例如：  
CFLAGS="$RPM_OPT_FLAGS" CXXFLAGS="$RPM_OPT_FLAGS" ./configure --prefix=%{_prefix}

##### %build 开始构建包
在/usr/src/asianux/BUILD/%{name}-%{version}目录中进行make的工作，常见写法：  
make %{?_smp_mflags} OPTIMIZE="%{optflags}"

都是一些优化参数，定义在/usr/lib/rpm/marcros中  
##### %install 开始把软件安装到虚拟的根目录中  
在/usr/src/asianux/BUILD/%{name}-%{version}目录中进行make install的操作。这个很重要，因为如果这里的路径不对的话，则下面%file中寻找文件的时候就会失败。常见内容有：  
%makeinstall 这不是关键字，而是rpm定义的标准宏命令。也可以使用非标准写法：  
make DESTDIR=$RPM_BUILD_ROOT install  
或  
make prefix=$RPM_BUILD_ROOT install  
需要说明的是，这里的%install主要就是为了后面的%file服务的。所以，还可以使用常规的系统命令：  
install -d $RPM_BUILD_ROOT/
cp -a * $RPM_BUILD_ROOT/

##### %clean 清理临时文件
通常内容为：
[ "$RPM_BUILD_ROOT" != "/" ] && rm -rf "$RPM_BUILD_ROOT"
rm -rf $RPM_BUILD_DIR/%{name}-%{version}


##### ※注意区分$RPM_BUILD_ROOT和$RPM_BUILD_DIR：
$RPM_BUILD_ROOT是指开头定义的BuildRoot，而$RPM_BUILD_DIR通常就是指/usr/src/asianux/BUILD，其中，前面的才是%file需要的。  
%pre rpm安装前执行的脚本  
%post rpm安装后执行的脚本  
%preun rpm卸载前执行的脚本  
%postun rpm卸载后执行的脚本  

##### %preun %postun 的区别是什么呢？

前者在升级的时候会执行，后者在升级rpm包的时候不会执行  
##### %files 定义那些文件或目录会放入rpm中  
这里会在虚拟根目录下进行，千万不要写绝对路径，而应用宏或变量表示相对路径。如果描述为目录，表示目录中除%exclude外的所有文件。  
%defattr (-,root,root) 指定包装文件的属性，分别是(mode,owner,group)，-表示默认值，对文本文件是0644，可执行文件是0755  
%exclude 列出不想打包到rpm中的文件  
※小心，如果%exclude指定的文件不存在，也会出错的。

##### %changelog 变更日志

※特别需要注意的是：%install部分使用的是绝对路径，而%file部分使用则是相对路径，虽然其描述的是同一个地方。千万不要写错。

#### 三、其他
##### 1. 扩展  
虽然上面的范例很简陋，而且缺少%build部分，但实际上只要记住两点：  
a）就是%build和%install的过程中，都必须把编译和安装的文件定义到“虚拟根目录”中。  
b）就是%file中必须明白，用的是相对目录  

##### 2. 一些rpm相关信息  
rpm软件包系统的标准分组：/usr/share/doc/rpm-4.3.3/GROUPS  
各种宏定义： /usr/lib/rpm/macros  
已经安装的rpm包数据库： /var/lib/rpm  
如果要避免生成debuginfo包：这个是默认会生成的rpm包。则可以使用下面的命令：  
echo '%debug_package %{nil}' >> ~/.rpmmacros  
如果rpm包已经做好，但在安装的时候想修改默认路径，则可以：  
rpm -ivh --prefix=/opt/usr xxx.rpm  
又或者同时修改多个路径：  
rpm xxx.rpm --relocate=/usr=/opt/usr --relocate=/etc=/usr/etc  

##### 3. 制作补丁  
详细看参考：[原]使用diff同patch工具  

##### 4. 关于rpm中的执行脚本  
如果正在制作的rpm包是准备作为放到系统安装光盘中的话，则需要考虑rpm中定义的脚本是否有问题。由于系统在安装的时候只是依赖于一个小环境进行，而该环境与实际安装完的环境有很大的区别，所以，大部分的脚本在该安装环境中都是无法生效，甚至会带来麻烦的。  
所以，对于这样的，需要放到安装光盘中的套件，不加入执行脚本是较佳的方法。  
另外，为提供操作中可参考的信息，rpm还提供了一种信号机制：不同的操作会返回不同的信息，并放到默认变量$1中。  
<span style="color:red">0代表卸载、1代表安装、2代表升级</span>  
可这样使用： 
``` 
%postun
if [ "$1" = "0" ]; then
/sbin/ldconfig
fi
```

