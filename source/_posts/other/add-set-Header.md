---
title: add/set Header
categories:
  - other
tags:
  - other
date: 2020-09-20 15:32:39
---

HttpServletResponse의 addHeader와 setHeader의 차이점

addHeader메소드와 setHeader의 차이점에 대해 문득 궁금해졌다.

둘 다 공통점으로는 헤더를 세팅해 주는것에 있는데 큰 차이점은 덮어쓸수 있냐 없냐 차이가 가장 큰 것 같다.

즉 addHeader("sample", 1)을 입력하고 addHeader("sample", 2);를 입력 후 addHeader("sample"); 을 가져온다면 1, 2을 가져온다.

setHeader("sample", 1)을 입력하고 setHeader("sample", 2);를 입력한다면 2만 가져오게 된다.

사실 대부분의 경우는 setHeader면 다되긴 한다.

addHeader의 docs : https://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpServletResponse.html#addHeader(java.lang.String,%20java.lang.String)