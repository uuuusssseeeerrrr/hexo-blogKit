---
title: expected ~~~ 1 bean ~~~ candidate
categories:
  - error
tags:
  - error
date: 2020-07-26 17:03:43
---

expected at least 1 bean which qualifies as autowire candidate 의 해결했던 경험

이 메시지를 첨봤는데 어노테이션으로 스프링 구성된 특징상 직접 실행해야 알려준다.

내가 발생했을때는 정상적으로 주입이 되지 않았을때 생기는 오류였는데 오류메시지를 보면 어떠한 파일에 오류가 발생했는지 뜬다.

그 파일에 들어가서 @Service, @Repository 등의 어노테이션이 정상적으로 써졌는지 확인하면 된다.

필자의 경우는 @Service가 빠져있어서 등록하고 정상적으로 수정되었다.