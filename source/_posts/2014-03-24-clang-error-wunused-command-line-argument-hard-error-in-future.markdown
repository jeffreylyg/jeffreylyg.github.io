---
layout: post
title: "Clang Error: -Wunused-command-line-argument-hard-error-in-future"
date: 2014-03-24 02:29:32 +0800
comments: true
categories: [Life Of Code]
---

在公司的电脑上用**Octopress**搭好个人Blog后进行了一些初步设置后因为工作太忙然后搁那儿就没动了，等周末时间空出来后准备用自己的Mac Air重新搭建和配置一下Octopress以便以后都用自己的电脑在闲暇的时间来写写博客。在搭建时执行 `bundle install` 命令时总是报如下的错误：
<!--more-->

	Gem::Installer::ExtensionBuildError: ERROR: Failed to 	build gem native extension.
				
	/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/bin/ruby extconf.rb --with-cflags=-w
	checking for main() in -lc... yes
	creating Makefile

	make "DESTDIR="
	compiling redcloth_attributes.c
	compiling redcloth_inline.c
	compiling redcloth_scan.c
	linking shared-object redcloth_scan.bundle
	clang: error: unknown argument: '-multiply_definedsuppress' [-Wunused-command-line-argument-hard-error-in-future]
	clang: note: this will be a hard error (cannot be downgraded to a warning) in the future
	make: *** [redcloth_scan.bundle] Error 1
	

	Gem files will remain installed in /Library/Ruby/Gems/2.0.0/gems/RedCloth-4.2.9 for inspection.
	Results logged to /Library/Ruby/Gems/2.0.0/gems/RedCloth-4.2.9/ext/redcloth_scan/gem_make.out
	An error occurred while installing RedCloth (4.2.9), and Bundler cannot continue.
	Make sure that `gem install RedCloth -v '4.2.9'` succeeds before bundling.

在各种Google和StackOverflow后问题出在`clang: error: unknown argument: '-multiply_definedsuppress' [-Wunused-command-line-argument-hard-error-in-future]`这句上，原因是更新了最新的Xocde 5.1后的Apple LLVM compiler把所有无法识别的命令行选项当成了错误，下面是[Xcode 5.1 Release Notes](https://developer.apple.com/library/ios/releasenotes/DeveloperTools/RN-Xcode/Introduction/Introduction.html)中的解释：
>The Apple LLVM compiler in Xcode 5.1 treats unrecognized command-line options as errors. This issue has been seen when building both Python native extensions and Ruby Gems, where some invalid compiler options are currently specified.

很感谢StackOverflow里这篇文章的帮助<http://stackoverflow.com/questions/22352838/ruby-gem-install-json-fails-on-mavericks-and-xcode-5-1-unknown-argument-mul>. 

这个问题有两种解决办法：

* 在Terminal里执行如下命令

```
sudo ARCHFLAGS=-Wno-error=unused-command-line-argument-hard-error-in-future bundle install 
sudo ARCHFLAGS=-Wno-error=unused-command-line-argument-hard-error-in-future gem install Gemmodule 
```
然而这种方法是一种临时性的解决办法，想要永久性的解决这个问题就要用第二种方法。

* 升级ruby的版本，升级到解决了这个问题的版本就行，貌似是2.0.0-p451，大于等于这个版本的都行，我下面的操作就是升级到了2.1.1版本。 
	
ruby的安装有两种方法，分别是**rvm**和**rbenv**，而在执行`rvm install ruby`命令时又出现了如下蛋疼的错误：
	
	Searching for binary rubies, this might take some time.
	No binary rubies available for: osx/10.9/x86_64/ruby-2.1.1.
	Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
	Checking requirements for osx.
	Installing macports...............................................
	Error running 'requirements_osx_port_install_port',
	showing last 15 lines of /Users/apple/.rvm/log/1395596183_ruby-2.1.1/port_install.log
	checking for lockf... yes
	checking for flock... yes
	checking for setmode... yes
	checking for strcasecmp... yes
	checking for strncasecmp... yes
	checking for strlcpy... yes
	checking for copyfile... yes
	checking for clearenv... no
	checking for sysctlbyname... yes
	checking if readlink conforms to POSIX 1003.1a... yes
	checking CommonCrypto/CommonDigest.h usability... yes
	checking CommonCrypto/CommonDigest.h presence... yes
	checking for CommonCrypto/CommonDigest.h... yes
	checking for Tcl configuration... configure: error: Can't find Tcl configuration definitions
	++ return 1
	Requirements installation failed with status: 1.
	
于是又用**rbenv**去升级**ruby**。

首先用如下命令安装**Homebrew**：

```
ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
```
然后安装**rbenv**：

```
brew update
brew install rbenv
brew install ruby-build
```
最后安装**ruby**：

```
rbenv install 2.1.1
rbenv local 2.1.1
rbenv rehash
```
按照第二种方法升级ruby后在Octopress目录下执行`bundle install`命令，问题解决，然后就可以用**rake**命令进行发布，预览等一系列操作了。