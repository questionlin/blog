---
title: PHP event 和 libevent 扩展的关系
date: 2018-07-18 20:46:55
tags: php
id: 1531918053
---
PHP event 和 libevent 扩展都是对 libevent 的封装，前者有 stable 版本且一直在更新而后者只有 beta 版本。本来肯定要选 event 扩展，可是它的 pecl 页面上赫然写着依赖 libevent 扩展。陷入混乱的我给作者写了封邮件，**结论是：大胆选 event 扩展**。以下是邮件原文：

> Hi,
> 
> event extension is an alternative to libevent extension.
> 
> You can see "libevent" package listed as a dependency on
> https://pecl.php.net/package/event, but in reality "libevent"
> extension is declared as **conflicting** package in package.xml:
> 
>       <package>
>         <name>libevent</name>
>         <channel>pecl.php.net</channel>
>         <min>0.0.2</min>
>         <conflicts/>
>         <providesextension>libevent</providesextension>
>       </package>
> 
> I can't recall why. But you definitely don't need libevent extension
> in order to use "event".
> 
> On Wed, 18 Jul 2018 11:28:21 +0800 (CST)
> "Simon Lin" <boyquestion@163.com> wrote:
> 
> Hi,
> 
> I'm just confused that what is the relation between event package and
> libevent package? The pecl page shows event package has stable
> versions while libevent package does not. If event package depends on
> libevent package, is event package still stable?
> 
> 
> 
> 
> Sincerely,
> Simon Lin
> 
> -- 
> Ruslan Osmanov