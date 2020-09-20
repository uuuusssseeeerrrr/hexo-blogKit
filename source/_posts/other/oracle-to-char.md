---
title: oracle to_date literal does not match format string
categories:
  - other
tags:
  - other
date: 2020-09-12 11:09:06
---

oracle에서 to_date 함수 사용시 literal does not match format string 오류에 대해

okky qna 게시판을 보다가 해당 오류가 나왔다길래 왜 그럴까 궁금해서 한번 찾아봤다.

크게 두가지로 나눠지는듯 한데 

1. 완전히 날짜 문자열이 아닌경우

2. 제대로 입력한거 같은데 오류가 발생하는 이유

1번의 경우는 아예 날짜 문자열과 어긋나는 'yyyy-mm-dd' 이런식이 아니라 '홍길동짱' 이런식을 들어오는 경우라 이런 경우는 문자열을 정상적으로 고쳐주면 된다.

2번의 경우가 문제인데 해당 쿼리는 다음과 같았다.

{% codeblock %}
select to_char(to_date('2020-09' || '-21'), 'YYYYMMDD') from date_table
{% endcodeblock %}

위와 같은 쿼리는 dbms툴에서는 문제가 없이 구동되지만 서버에 올려서 구동시 오류가 발생한다고 한다.

오류가 발생하는 부분은 to_date부분인데 해당 문서에 잘 나와있다.

<a href="https://docs.oracle.com/cd/B19306_01/server.102/b14200/functions183.htm">문서</a> 를 보면 두번째줄에 "The default date format is determined implicitly by the NLS_TERRITORY initialization parameter or can be set explicitly by the NLS_DATE_FORMAT parameter" 라고 쓰여있다.

기본값은 "NLS_DATE_FORMAT", "NLS_TERRITORY"에 정의되어 있다는 말인데 특별한 경우 아니면 NLS_DATE_FORMAT를 확인해보면 될 것이다.

NLS_TERRITORY를 따로 정의 한경우 NLS_DATE_FORMAT를 다시 설정해줘야한다. NLS_TERRITORY가 변경되면 값도 변경되는듯 하다. 먼저 바라본다고 해야하는가 싶다. (https://stackoverflow.com/questions/60110325/oracle-nls-territory-overrides-nls-date-format)

전체 확인방법은 다음과 같다.

{% codeblock %}
select * from nls_session_parameters;
{% endcodeblock %}

NLS_DATE_FORMAT의 설정방법은 다음과 같다.

{% codeblock %}
alter session set nls_date_format='yyyy-mm-dd'
{% endcodeblock %}

그래도 가장 좋은건 뒤에 날짜포맷을 지정하면 된다. 즉 해결방법중에 가장 빠른건 쿼리를 다음과 같이 바꾸면 될 것이다.

{% codeblock %}
select to_char(to_date('2020-09' || '-21', 'YYYY-MM-DD'), 'YYYYMMDD') from date_table
{% endcodeblock %}