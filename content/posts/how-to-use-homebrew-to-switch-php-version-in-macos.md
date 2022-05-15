---
title: "如何使用 Homebrew 在 macOS 中切换 PHP 版本"
date: 2022-05-14T16:37:04+08:00
tags: [ "php", "homebrew", "tips" ]
draft: true
---

## TLDR

安装不同版本的 PHP

```bash
# php version in [7.2, 7.3, 7.4, 8.0, 8.1]
brew install php@7.4
# or use shivammathur/homebrew-php tap
brew install shivammathur/php/php@7.4

# php version in [5.6, 7.0, 7.1]
# only can use shivammathur/homebrew-php tap
brew install shivammathur/php/php@7.1
```

切换版本

```bash
brew unlink php@7.4
brew link php@7.1
```

## 起因

为了追查[ Laravel 5.8 的一个 RCE 漏洞](https://www.cnvd.org.cn/flaw/show/CNVD-2022-36040)，我需要在本机复现。当我安装完 Laravel 5.8 后，执行 `composer install` 安装依赖却没成功：

```bash
$ composer install
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Root composer.json requires php ^7.1.3 but your php version (8.1.5) does not satisfy that requirement.
  Problem 2
    - laravel/framework[v5.8.0, ..., 5.8.x-dev] require php ^7.1.3 -> your php version (8.1.5) does not satisfy that requirement.
    - Root composer.json requires laravel/framework 5.8.* -> satisfiable by laravel/framework[v5.8.0, ..., 5.8.x-dev].
```

Laravel 5.8 最高支持的 PHP 版本为 7.1。所以我得降版本。

> 在工作中，也会遇到类似的问题：追踪以前的代码时有些场景可能需要安装依赖，但版本并不匹配，之前都是跳过，寻找别的解决方案。正好借此机会总结一下。

## 解决

search 一下发现 [Homebrew core](https://github.com/Homebrew/homebrew-core) 只维护了 [php@7.2](https://github.com/Homebrew/homebrew-core/blob/5d0f563955ccb6a22577838df57242f2d62ca060/Formula/php%407.2.rb) 以上的版本：

```bash
$ brew search --formula php
==> Formulae
brew-php-switcher      php-cs-fixer           php@7.3                phpbrew                phpmyadmin             pcp
php ✔                  php-cs-fixer@2         php@7.4                phplint                phpstan                pup
php-code-sniffer       php@7.2                php@8.0 ✔              phpmd                  phpunit
```
从上面的结果中发现了 [phpbrew](https://github.com/phpbrew/phpbrew)。它安装完后还需要设置环境变量，有点麻烦，并且[最近一次提交](https://github.com/phpbrew/phpbrew/commit/0aac194e897469ce7448e37077b4db95b0a20dfb)在 2021/1/31 就觉得不太靠谱，放弃。

通过搜索找到了 [Valet](https://github.com/laravel/valet)，它是 Laravel 团队开发的在 Mac 上快速搭建 PHP 开发环境的工具。

在[代码中](https://github.com/laravel/valet/blob/master/cli/Valet/PhpFpm.php#L16-L19)可以看到，它通过 [shivammathur/homebrew-php](https://github.com/shivammathur/homebrew-php) tap 来安装 Homebrew core 没有维护的 PHP 版本。其中就包含了我想要的 php\@7.1：

```bash
brew install shivammathur/php/php@7.1
```

安装完成后，使用 Homebrew 中的 link/unlink ，将 PHP 版本设置为 7.1：

```
brew unlink php
brew link php@7.1
```

最后愉快的在 Laravel 5.8 中 `composer install`

## Ref
* https://learnku.com/laravel/t/67866
