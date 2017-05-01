---
layout: post
title: "Tomcat crash 현상"
date: 2017-05-01 13:29:51
image: '/assets/img/'
description:
tags: 
categories:
twitter_text:
---

# 현상

aws t2 micro에서 Spring Boot를 이용한 서버 2개를 돌리고 있었다.

처음 켜둔 Rest	서버는 멀쩡하게 잘 돌아가는데 Auth서버가 자꾸 죽는 현상이 발생했다.

처음한번은 그러려니 넘어갔는데 똑같은현상이 계속 발생해서 로그를 까봤다. 톰캣 로그에는 아무것도 나오지않았다.

검색해보니까 OOME라는 얘기가 있었다

<http://stackoverflow.com/questions/10668900/tomcat-7-0-27-died-without-trace>

/var/log 에서 Grep해보니 tomcat pid가 kill된것을 발견할수 있었다. 

> Apr 28 21:51:45 ip-172-31-30-89 kernel: [25115716.240270] Out of memory: Kill process 1642 (java) score 360 or sacrifice child
> Apr 28 21:51:45 ip-172-31-30-89 kernel: [25115716.246419] Killed process 1642 (java) total-vm:2301412kB, anon-rss:376300kB, file-rss:0kB

# 해결방법

아무래도 t2.micro로는 톰캣을 두개 돌릴만한 메모리가 안나오는듯 하다.

메모리를 늘리던가 해야겠다.