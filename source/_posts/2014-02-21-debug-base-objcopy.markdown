---
layout: post
title: "objcopy命令介绍"
date: 2014-02-21 14:12:00 +0800
comments: false
categories:
- 2014
- 2014~02
- debug
- debug~base
tags:
---
objcopy把一种目标文件中的内容复制到另一种类型的目标文件中.

#### (1)将图像编译到可执行文件内
Q: 如何将一个二进制文件，比如图片，词典一类的东西做为.o文件，直接链接到可执行文件内部呢？  
A:  
```
$ objcopy -I binary -O elf32-i386 -B i386 14_95_13.jpg image.o
$ gcc image.o tt.o -o tt
$ nm tt | grep 14_95
0805d6c7 D _binary_14_95_13_jpg_end
00014213 A _binary_14_95_13_jpg_size
080494b4 D _binary_14_95_13_jpg_start
```

#### (2)使用objcopy把不用的信息去掉：
```
$ objcopy -R .comment -R .note halo halo.min
```

#### (3)
```
$ objcopy -R .note -R .comment -S -O binary xyb xyb.bin
-R .note -R .comment 表示移掉 .note 与 .comment 段
-S 表示移出所有的标志及重定位信息
-O binary xyb xyb.bin 表示由xyb生成二进制文件xyb.bin
```

### objcopy工具使用指南
```
objcopy Utility
objcopy [ -F bfdname | --target=bfdname ]
[ -I bfdname | --input-target=bfdname ]
[ -O bfdname | --output-target= bfdname ]
[ -S | --strip-all ] [ -g | --strip-debug ]
[ -K symbolname | --keep-symbol= symbolname ]
[ -N symbolname | --strip-symbol= symbolname ]
[ -L symbolname | --localize-symbol= symbolname ]
[ -W symbolname | --weaken-symbol= symbolname ]
[ -x | --discard-all ] [ -X | --discard-locals ]
[ -b byte | --byte= byte ]
[ -i interleave | --interleave= interleave ]
[ -R sectionname | --remove-section= sectionname ]
[ -p | --preserve-dates ] [ --debugging ]
[ --gap-fill= val ] [ --pad-to= address ]
[ --set-start= val ] [ --adjust-start= incr ]
[ --change-address= incr ]
[ --change-section-address= section{=,+,-} val ]
[ --change-warnings ] [ --no-change-warnings ]
[ --set-section-flags= section= flags ]
[ --add-section= sectionname= filename ]
[ --change-leading char ] [--remove-leading-char ]
[ --weaken ]
[ -v | --verbose ] [ -V | --version ] [ --help ]
input-file [ outfile ]  
```
  GNU实用工具程序objcopy的作用是拷贝一个目标文件的内容到另一个目标文件中。Objcopy使用GNU BFD库去读或写目标文件。Objcopy可以使用不同于源目标文件的格式来写目的目标文件（也即是说可以将一种格式的目标文件转换成另一种格式的目标文 件）。通过以上命令行选项可以控制Objcopy的具体操作。

  Objcopy在进行目标文件的转换时，将生成一个临时文件，转换完成后就将这个临 时文件删掉。Objcopy使用BFD做转换工作。如果没有明确地格式要求，则Objcopy将访问所有在BFD库中已经描述了的并且它可以识别的格式， 请参见《GNUpro Decelopment Tools》中“using ld”一章中“BFD库”部分和“BFD库中规范的目标文件格式”部分。

  通过使用srec作为输出目标（使用命令行选项-o srec），Objcopy可以产生S记录格式文件。

  通过使用binary作为输出目标（使用命令行选项-o binary），Objcopy可以产生原始的二进制文件。使用Objcopy产生一个原始的二进制文件，实质上是进行了一回输入目标文件内容的内存转 储。所有的符号和重定位信息都将被丢弃。内存转储起始于输入目标文件中那些将要拷贝到输出目标文件去的部分的最小虚地址处。

  使用Objcopy生成S记录格式文件或者原始的二进制文件的过程中，-S选项和-R选项可能会比较有用。-S选项是用来删掉包含调试信息的部分，-R选项是用来删掉包含了二进制文件不需要的内容的那些部分。

input-file  
outfile  
  参数input-file和outfile分别表示输入目标文件（源目标文件）和输出目标 文件（目的目标文件）。如果在命令行中没有明确地指定outfile，那么Objcopy将创建一个临时文件来存放目标结果，然后使用input- file的名字来重命名这个临时文件（这时候，原来的input-file将被覆盖）。

-I bfdname  
--input-target=bfdname  
  明确告诉Objcopy，源文件的格式是什么，bfdname是BFD库中描述的标准格式名。这样做要比“让Objcopy自己去分析源文件的格式，然后去和BFD中描述的各种格式比较，通过而得知源文件的目标格式名”的方法要高效得多。

-O bfdname  
--output-target= bfdname  
  使用指定的格式来写输出文件（即目标文件），bfdname是BFD库中描述的标准格式名。

-F bfdname  
--target= bfdname  
明确告诉Objcopy，源文件的格式是什么，同时也使用这个格式来写输出文件（即目标文件），也就是说将源目标文件中的内容拷贝到目的目标文件的过程中，只进行拷贝不做格式转换，源目标文件是什么格式，目的目标文件就是什么格式。

-R sectionname  
--remove-section= sectionname  
从输出文件中删掉所有名为sectionname的段。这个选项可以多次使用。  
注意：不恰当地使用这个选项可能会导致输出文件不可用。

-S  
--strip-all （strip 剥去、剥）  
不从源文件中拷贝重定位信息和符号信息到输出文件（目的文件）中去。

-g  
--strip-debug  
不从源文件中拷贝调试符号到输出文件（目的文件）中去。

--strip-undeeded  
剥去所有在重定位处理时所不需要的符号。

-K symbolname  
--keep-symbol= symbolname  
仅从源文件中拷贝名为symbolname的符号。这个选项可以多次使用。

-N symbolname  
--strip-symbol= symbolname  
不从源文件中拷贝名为symbolname的符号。这个选项可以多次使用。它可以和其他的strip选项联合起来使用（除了-K symbolname | --keep-symbol= symbolname外）。

-L symbolname  
--localize-symbol= symbolname  
使名为symbolname的符号在文件内局部化，以便该符号在该文件外部是不可见的。这个选项可以多次使用。

-W symbolname  
-weaken-symbol= symbolname  
弱化名为symbolname的符号。这个选项可以多次使用。

-x  
--discard-all （discard 丢弃、抛弃）  
不从源文件中拷贝非全局符号。

-X  
--discard-locals  
不从源文件中拷贝又编译器生成的局部符号（这些符号通常是L或 . 开头的）。

-b byte  
--byte= byte  
Keep only every byte of the input file (header data is not affected). byte can be  
in the range from 0 to interleave-1, where interleave is given by the -i or  
--interleave option, or the default of 4. This option is useful for creating files to  
program ROM . It is typically used with an srec output target.  

-i interleave  
--interleave= interleave （interleave 隔行、交叉）  
Only copy one out of every interleave bytes. Select which byte to copy with the  
-b or --byte option. The default is 4. objcopy ignores this option if you do not  
specify either -b or --byte.

-p  
--preserve-dates (preserve 保存、保持)  
设置输出文件的访问和修改日期和输入文件相同。

[ --debugging ]  
如果可能的话，转换调试信息。因为只有特定的调试格式被支持，以及这个转换过程要耗费一定的时间，所以这个选项不是默认的。

--gap-fill= val  
使用内容val填充段与段之间的空隙。通过增加段的大小，在地址较低的一段附加空间中填充内容val来完成这一选项的功能。

--pad-to= address  
填充输出文件到虚拟地址address。通过增加输出文件中最后一个段的大小，在输出文件中最后一 段的末尾和address之间的这段附加空间中，用--gap-fill= val选项中指定的内容val来填充（默认内容是0，即没有使用--gap-fill= val选项的情况下）。

--set-start= val  
设置新文件（应该是输出文件吧？）的起始地址为val。不是所有的目标文件格式都支持设置起始地址。

--change-start = incr  
--adjust-start= incr  
通过增加值incr来改变起始地址。不是所有的目标文件格式都支持设置起始地址。

--change-addresses incr  
--adjust-vma incr  
Change the VMA and LMA addresses of all sections, section., as well as the  
start address, by adding incr. Some object file formats do not permit section  
addresses to be changed arbitrarily.

通过加上一个值incr，改变所有段的VMA（Virtual Memory Address运行时地址）和LMA（Load Memory Address装载地址），以及起始地址。某些目标文件格式不允许随便更改段的地址。

--change-section-address section{=,+,-} val  
--adjust-section-vma section{=,+,-} val  
  设置或者改变名为section的段的VMA（Virtual Memory Address运行时地址）和LMA（Load Memory Address装载地址）。如果这个选项中使用的是“=”，那么名为section的段的VMA（Virtual Memory Address运行时地址）和LMA（Load Memory Address装载地址）将被设置成val；如果这个选项中使用的是“-”或者“+”，那么上述两个地址将被设置或者改变成这两个地址的当前值减去或加上 val后的值。如果在输入文件中名为section的段不存在，那么Objcopy将发出一个警告，除非--no-change-warnings选项被 使用。

这里的段地址设置和改变都是输出文件中的段相对于输入文件中的段而言的。例如：  
（1）--change-section-address .text = 10000  
这里是指将输入文件（即源文件）中名为.text的段拷贝到输出文件中后，输出文件中的.text段的VMA（Virtual Memory Address运行时地址）和LMA（Load Memory Address装载地址）将都被设置成10000。  
（2）--change-section-address .text + 100  
这 里是指将输入文件（即源文件）中名为.text的段拷贝到输出文件中后，输出文件中的.text段的VMA（Virtual Memory Address运行时地址）和LMA（Load Memory Address装载地址）将都被设置成以前输入文件中.text段的地址（当前地址）加上100后的值。

--change-section-lma section{=,+,-} val  
  仅设置或者改变名为section的段的 LMA（Load Memory Address装载地址）。一个段的LMA是程序被加载时，该段将被加载到的一段内存空间的首地址。通常LMA和VMA（Virtual Memory Address运行时地址）是相同的，但是在某些系统中，特别是在那些程序放在ROM的系统中，LMA和VMA是不相同的。如果这个选项中使用的是 “=”，那么名为section的段的LMA（Load Memory Address装载地址）将被设置成val；如果这个选项中使用的是“-”或者“+”，那么LMA将被设置或者改变成这两个地址的当前值减去或加上val 后的值。如果在输入文件中名为section的段不存在，那么Objcopy将发出一个警告，除非--no-change-warnings选项被使用。

--change-section-vma section{=,+,-} val  
  仅设置或者改变名为section的段的 VMA（Load Memory Address装载地址）。一个段的VMA是程序运行时，该段的定位地址。通常VMA和LMA（Virtual Memory Address运行时地址）是相同的，但是在某些系统中，特别是在那些程序放在ROM的系统中，LMA和VMA是不相同的。如果这个选项中使用的是 “=”，那么名为section的段的LMA（Load Memory Address装载地址）将被设置成val；如果这个选项中使用的是“-”或者“+”，那么LMA将被设置或者改变成这两个地址的当前值减去或加上val 后的值。如果在输入文件中名为section的段不存在，那么Objcopy将发出一个警告，除非--no-change-warnings选项被使用。

--change-warnings  
--adjust-warnings  
  如果命令行中使用了--change- section-address section{=,+,-} val或者--adjust-section-vma section{=,+,-} val，又或者--change-section-lma section{=,+,-} val，又或者--change-section-vma section{=,+,-} val，并且输入文件中名为section的段不存在，则Objcopy发出警告。这是默认的选项。  

--no-chagne-warnings  
--no-adjust-warnings  
  如果命令行中使用了--change- section-address section{=,+,-} val或者--adjust-section-vma section{=,+,-} val，又或者--change-section-lma section{=,+,-} val，又或者--change-section-vma section{=,+,-} val，即使输入文件中名为section的段不存在， Objcopy也不会发出警告。

--set-section-flags section=flags  
  为section的段设置一个标识。这个flags变量的可以取逗号分隔的多个标识名字符串（这些标识名字符串是能够被Objcopy程序所识别的），合法的标识名有alloc，load，readonly，code，data和rom。
You can set the contents flag for a section which does not havecontents, but it is not meaningful to clear the contents flag of a section which does have contents; just remove the section instead. Not all flags are meaningful for all object file formats.

--add-section sectionname=filename  
  进行目标文件拷贝的过程中，在输出文件中增加一个名为 sectionname的新段。这个新增加的段的内容从文件filename得到。这个新增加的段的大小就是这个文件filename的大小。只要输出文 件的格式允许该文件的段可以有任意的段名（段名不是标准的，固定的），这个选项才能使用。

--change-leading-char  
Some object file formats use special characters at the start of symbols. The most  
common such character is underscore, which compilers often add before every  
symbol. This option tells objcopy to change the leading character of every  
symbol when it converts between object file formats. If the object file formats use  
the same leading character, this option has no effect. Otherwise, it will add a  
character, or remove a character, or change a character, as appropriate.  

--remove-leading-char  
If the first character of a global symbol is a special symbol leading character used  
by the object file format, remove the character. The most common symbol leading  
character is underscore. This option will remove a leading underscore from all  
global symbols. This can be useful if you want to link together objects of different  
file formats with different conventions for symbol names.

--weaken  
Change all global symbols in the file to be weak. This can be useful when building  
an object that will be linked against other objects using the -R option to the linker.  
This option is only effective when using an object file format that supports weak  
symbols.

-V  
--version  
Show the version number of objcopy.

-v  
--verbose  
Verbose output: list all object files modified. In the case of archives, objcopy -V  
lists all members of the archive.

--help  
Show a summary of the options to objcopy.

