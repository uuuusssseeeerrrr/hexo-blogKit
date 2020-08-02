---
title: No CacheManager Eventlistener ....
categories:
  - error
tags:
  - error
date: 2020-08-02 13:25:07
---

No CacheManagerEventListenerFactory class specified. Skipping... 이렇게 스프링에서 오류는 아니고 메시지가 뜨는 경우가 있다.

스프링 캐시를 사용할 때 정상적으로 설정되지 않아서 그런것인데 설정에 오류가 있어서 발생한다.

개인적으로 해결했던 경우는 ehcache.xml에서 xml문법이 발생해서 정상적으로 설정되지 않았었다.

그래서 설정해서 해결했다.