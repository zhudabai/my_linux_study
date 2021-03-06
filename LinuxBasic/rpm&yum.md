程序包管理 rpm/yum/源码编译
======================================

​	在Windows中装软件是特别轻松的事情,基本上下载一个exe的程序,然后双击,下一步就可以成功了,那么面对纯命令行接口的Linux,我们想装一个程序的话就需要使用相应的命令了,利用vim编辑文件都那么容易,装个软件包肯定不在话下,这篇文章主要总结一下Linux下程序包管理相关工具使用,以及相关的原理

​	对于开发人员或者是IT技术从业人员来说,喜欢Linux与OSX的原因我认为一部分的原因为,这两个家伙,你想去使用它必须了解一些其相关的工作原理,才能很好的使用它.这是一个多么好的学以致用的机会呀.那么首先就来了解一个算是常识性的东西吧,就是exe程序为什么在Linux下不能运行,首先需要介绍两个软件运行环境一个叫做ABI(Application Binary Interface)另一个叫做API(Application Programming Interface),一个叫做应用二进制接口,另一个叫做应用程序接口,这里不做详细的解释,因为我目前理解的状态还不够我来完全解释,最主要的就是在Windows和Linux中这两个接口的实现方式不相同,导致了程序无法跨平台来运行

​	在Linux中一个程序主要包含了,二进制文件,库文件,配置文件,帮助文档,当然也不是所有的程序都包含这些内容,程序包其实就是将这些文件进行打包,那安装程序包,其实就是将这个软件包解压,然后移动到相应的目录中,当然,安装前需要探测一下当前的环境是否满足软件包的需要,当然不只是去检测一下这个软件包是否安装,而是这个软件包中的二进制程序在开发时需要用到很多的库,如果当前系统没有有这个库,那么软件安装就会报错,这里说到了库,库其实就是一个个的功能模块,比如连接网络等等可能会觉得程序包不是自带了吗,然而并不是这样,程序自带的其实是程序自己使用的库,还有一些库是需要在执行时动态调用系统的库,这些库都封装的是公共的功能模块,也就是说可能很多程序会用到,这样就不同每个程序都自带一个这样的库了,那样就太累了,

​	说了这么一堆,也不知道说明白没有,其实想说的就是,在安装很多软件包的时候会发生一个特别恶心的事情就是,软件有依赖关系,最常见的就是依赖某些库,这就会导致软件的安装失败,当然,这个问题早已有了解决方案,

### RPM包管理器

​	在Redhat系列的Linux平台上软件包的后缀名为rpm,当然还有一个用于安装rpm包的工具叫做rpm,这个工具没有解决依赖关系的功能,废话说了一箩筐,下面就来介绍一下怎样使用这个工具来,安装,卸载,更新软件包

**安装:**rpm {-i|--install} [install-options] PACKAGE_FILE ...

​	首先来个小demo,也是最常用的就是安装,rpm -ivh PACKAGE_FILE,-i是最主要的,就是安装,v就是显示安装的详细信息,h的作用是用#显示进度,PACKAGE_FILE就是软件包文件的文件名,用rpm安装必须要先把rpm包下载到本地,当然其实大部分常用的软件包都已经放在了CentOS的安装光盘中了

下面就拉详细介绍一下一些常用参数:

​	-vv:显示更加详细的信息

​	--test:这个选项好,可以在安装前测试一下,并不会真正的将软件包安装到系统中

​	--nodeps:忽略依赖关系,为什么要有这个选项呢,最主要的是有时候软件包依赖会出现循环依赖,就是包1依赖包2包2依赖包3而包3又依赖包1

​	--replacepkgs:重新安装,当不小心删除了这个软件中的某个文件后,可以使用这个选项再重新安装

​	--replacefiles:当安装某个程序包时,程序包中的某个文件已经存在系统上的时候,直接安装软件包就会报错,可以使用这个选项,强制覆盖已有的文件

​	--nosignature:不检查软件包的来源合法性,在centos安装光盘中会有一个RPM-GPG-KEY-CentOS-6文件,可以将这个文件使用--import导入到系统中,然后在每个rpm包中都附加了加密信息,可以使用导入的那个文件中的密文和软件包中的密文进行比对,

​	--noscript:不执行软件包自带的脚本

​		%pre:		安装前的脚本		--nopre

​		%post:		安装后的脚本		--nopost

​		%preun:		卸载前的脚本		--nopreun

​		%postun:	卸载后的脚本		--postun

​	--oldpackage:降级操作

​	--force:强制安装,只能用于安装卸载

**升级:**
	rpm {-U|--upgrade} [install-options] PACKAGE_FILE ...
	rpm {-F|--freshen} [install-options] PACKAGE_FILE ...

​		-U:也就是upgrade:如果这个软件包存在,就升级这个软件包,如果不存在就将这个软包包安装到系统中

​		-F:也就freshen:这个就是只升级,如果软件包不存在,也不进行任何操作

查询:rpm {-q|--query} \[select-options\]\[query-options\]

​	-q package_name:查看软件包是否被安装,以下的查询选项都需要加-q,这个也是比较重要的一些内容

​			-l:列出安装包中的文件

​			-a:列出所有已安装的包

​			-f:查看某一个文件属于哪个软件包

​			-d:查询软件包的文档文件

​			-p:指定对安装包文件进行查询

​			-c:查询安装包都有哪些配置文件

​			-i:查看安装包的元数据信息

​			-R:查询软件包的依赖关系

​			--changelog:查看软件包的更新日志,注意,是该软件包的,并不是源代码

​			--scripts:查询软件包自带的脚本

​	**注意**:每使用rpm安装,升级,卸载一个软件包的时候,rpm都会记录在自己的数据中,数据库的位置在/var/lib/rpm/中,这样以上的这些查询都是通过这个数据库查询得到的,当然如果破坏了这个数据库的话,不光是查询,安装卸载等操作也无法进行,因为在安装前,rpm肯定是要去查一下这个软件包是否被安装,以及他的依赖包是否被安装,这些信息都是通过这个数据库进行查询的,如果破坏了,系统就会乱掉

**卸载**:rpm {-e|--erase} \[--allmatches]\[--justdb]\[--nodeps]\[--noscripts]\[--notriggers]\[--test] PACKAGE_NAME ...
			--allmatches:卸载所有版本

**包校验**: rpm {-V|--verify} \[select-options][verify-options]
			-V:与数据库比对,查看哪些文件的属性发生了改变
				各段代表的含义:
					S file Size differs
					M Mode differs (includes permissions and file type)
					5 digest (formerly MD5 sum) differs
					D Device major/minor number mismatch
					L readLink(2) path mismatch
					U User ownership differs
					G Group ownership differs
					T mTime differs
					P caPabilities differ

**数据库重建:**注意这个重建可能只是部分数据破坏了可以重建,当所有数据库文件都被删除后,这实际上是没有任何意义的
			rpm {--initdb|--rebuilddb}
				initdb:初始化
					如果事不存在数据库,则新建之,否则不执行任何操作
				rebuilddb:重建
					无论当前存在与否,直接重新创建数据库

### YUM包管理器

当你使用以上的命令去安装了一些软件包的时候,你可能已经开始吐槽了,因为依赖关系实在是太让人头疼了,接下来我们就来介绍一下可以解决依赖关系的包管理器yum,全称yellow dog

​	yum是用Python实现的一个rpm的前端程序,也就是说其底层仍然调用的是rpm来安装软件,yum是一个c/s架构的软件也就是说需要一个服务端也称为软件仓库,本地的yum命令是yum的客户端,主要的实现方式是,在yum服务器上的repodata目录中存放了所有软件包的元数据信息,包括各个软件包之间的依赖关系,在客户端连接服务器的时候,发现本地缓存没有软件包的元数据信息,首先会将元数据信息同步到本地,然后会分析出需要安装的软件包的依赖关系,然后从服务器将这些软件包下载到本地的缓存目录中并安装之,安装成功后再将下载的软件包删除

接下来就介绍一下怎样去配置yum,yum有两个配置文件,一个主配置文件,另一个是软件仓库配置文件

​	主配置文件:/etc/yum.conf:以下是主配置文件的一些选项

​		[main]	这个是配置段名,main是必须的

​		cachedir=/path/to/cahcedir:缓存文件的目录

​		keepcache={0|1}:安装软件完成后,软件包是否删除,

​		debuglevel=2:设置debug等级

​		logfile=/path/to/log:日志目录的位置

​		installonly_limit:一次yum命令可以同时安装几个软件包,注意这个指的是单个yum进程的,也就是说,如果现在运行着一个yum命令,是不允许再运行第二个的,必须等第一个执行完成后才可以执行第二个

​	仓库配置文件/etc/yum.repos.d/*.repo:在yum.repos.d目录下只要是以.repo结尾的就认为是一个配置文件

​			[name] :中括号括起来的名字,可以随意写

​			name=REPO_NAME:这个是你个人为这个yum仓库起的名字,在使用是会显示

​			baseurl=url:软件仓库的url:支持的协议有http,https,ftp,file

​			gpgcheck={0|1}:是否检查软件包签名,完整性

​			gpgkey=/key/file/url:指定检查包完整性的key的位置

​			enable={1|0}是否启用这个仓库

​			mirrorlist=/path/to/url_list:将多个源的url写入一个文件,这里指向这个文件的位置

​			failovermethod={roundrobin|priority}:多个源地址时的使用顺序

​				roundroid:随机挑选,默认

​				priority:按顺序访问

​			cost=n:这个软件仓库的使用优先级

​	下面还有一些变量可以在url中使用,方便一个配置文件可以在多个系统中使用

​			$basharch:CPU基础平台比如说x86_64

​			$arch:平台

​			$releasever:服务器操作系统主版本号

​			\$YUM0-\$YUM9:为使用者分配的自定义变量,可以对这些变量进行赋值,然后使用

**yum命令**

```bash
		repolist [all|enabled|disabled]:查看当前的软件仓库列表列表,默认为已启用的可以使用
		repolist all显示所有,repolist disabled查看已禁用的
		clean [all|pacages|metadata|expr]:清理缓存
		install:
		reinstall:重新安装
		list [all|glob|package|installed]:最后一列带@的表示已经安装的@后面的内容表示从哪个仓库安装的,anaconda表示装系统时安装的
		update:升级
		downgrade:降级
		check-update:检查升级
		remove:卸载包
		info:显示软件包信息
		provides:查看特性属于哪个包
		makecahce:生成缓存
		search:搜索
		deplist:显示软件包依赖关系
		history:
			list:列出历史
			undo id:撤销某一次操作
			redo id:重新执行一次某一个操作
包组管理:
	yum groupinstall
其他选项:
	-y:自动回答为y
	-q:静默模式,不能和-y合并,必须分开写
	--enablerepo="repoid":临时启用
	--disablerepo="repoid":临时禁用
yum-config-manager:生成repo文件
	--add-repo=/url/path
	--disable '仓库名'
	--enable '仓库名'
```

以上就是yum的安装方式,它可以解决软件包的依赖关系,当然以上两种操作方式有一个共同的缺点,就是软件安装的可定制性太差,甚至我们连安装路径的选择权利都没有,这不太符合Linux的风格,众所周知,Linux最大的亮点就是来源,所以我们可以下载到软件的源码包,这样我们就可以自己定制安装了,下面就开始介绍一下

### 编译安装

​	如果一个文件一个文件的去编译的话,那肯定得死,因此,虽说是源码安装,其实也提供了很多工具,将代码组织好按用户给定的选项进行安装,需要编译安装,首先肯定需要编译器,大多数Linux下的软件都是用c/c++编写的,因此在编译前我们得先把gcc安装好,当然了源代码也是要下载的,可以到软件的官方站点下载,下载下来一般都是压缩包,解压之后进入到源代码的目录,开始操作,下面就说一下编译的步骤,

​	1.在源码目录的根目录下执行./configure,这是一个脚本,是通过一些其他工具生成的,主要是检查系统当前环境,是否满足该软件的编译安装,这个脚本还可以通过选项来指定要启用或者禁用的特性,以及各种文件的安装路径,常用的有--prefix=/path/to/install_path,指定安装路径,--sysconfdir=/path/to/conf_dir,指定配置文件目录,另外可以使用--with-something=/path/to/some_dir,指定软件依赖其他软件的安装位置,这个依赖软件的位置一般是也是编译安装的如果不指定,它会从系统的PATH变量中找需要依赖的软件,当然如果找不到肯定会报错的,还有--enable,--disable来启用或者禁用软件特性,--help当然就是获取帮助啦,在help中显示的--enable对应的特性是软件编译时默认没有启用的,--disable对应的是软件编译时默认会启用的,这configure脚本会通过源码中的一个模板文件Makefile.in最终生成一个Makefile文件,当然这个文件就记录了软件编译时的一些信息,比如启用禁用的特性,文件路径等等

​	2.这一步就比较简单了,直接执行make命令就好了,make命令会通过Makefile文件对源码进行编译,这里有一个选项-j这个可以指定编译时使用的CPU数量来提高编译速度,当然如果编译出错或者需要重新编译时需要将之前编译的文件清理,可以使用make clean命令

​	3.这一步也是比较简单的,直接执行make install命令就可以了,这一步主要的作用就是将编译好的二进制文件,配置文件拷贝到用户指定的目录中去

​	以上就是软件包的编译大概过程,当然如果第一次编译可能会遇到各种各样的报错,大部分应该都会在执行./configure脚本时,认真看返回的报错信息,一般情况下都是缺少某些依赖包,可以根据报错中的软件包的名称安装其devel包来进行安装,也就是pack_name-devel


