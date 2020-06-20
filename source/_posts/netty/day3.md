---
title: 네티 3 - 채널 파이프라인 + 코덱
categories:
  - Netty(복구중)
tags:
  - Netty
date: 2020-03-25 14:55:02
---

<h2>주요 용어들</h2>

채널 파이프라인 - 채널에서 발생한 이벤트가 이동하는 통로
이벤트 핸들러 - 채널 파이프라인을 따라 이동한 이벤트를 처리하는 클래스
코덱 - 이벤트 핸들러를 상속받아서 구현한 것
네티의 이벤트 실행

보통의 연결된 소켓에서 데이터를 받는다면 다음과 같은 로직으로 처리할 것입니다.

1. 소켓에 데이터가 있는지 확인
2. 데이터가 있으면 읽는 메서드를 호출 후 데이터 처리
3. 데이터가 올 때 까지 기달리다가 오면 2번으로 돌아감
4. 네트워크가 종료되면 관련된 부분 처리

Netty에서는 다음과 같이 처리합니다.

1. 데이터가 들어오면 Netty의 이벤트 루프가 채널 파이프라인에 등록된 첫 번째 이벤트 핸들러를 가져온다.
2. 이벤트 핸들러에 데이터 수신 후 작동할 이벤트가 있는지 확인
3. 가져온 이벤트 핸들러에 관련 메소드가 없으면 이벤트가 처리되거나 핸들러를 다 가져올떄까지 계속 반복
4. 네트워크가 종료되면 관련된 부분 처리

즉, 데이터를 입출력 후 처리하는 부분은 이벤트로 관리하고 있으니 이벤트 처리만 해주면 Netty를 쉽게 쓸 수 있습니다.

<h2>채널 파이프라인</h2>

<p><img src="https://user-images.githubusercontent.com/20100284/50535726-5887c600-0b90-11e9-9a1a-b81707c78ce5.png" alt="netty.io API의 설명"></p>

저렇게 채널에서 이벤트를 받으면 파이프라인으로 들어가서 이벤트 핸들러에 의해 맞는 이벤트를 처리하게 됩니다.
하나의 채널 파이프라인에 여러 이벤트 핸들러를 등록할 수 있습니다.

다시 에코서버안의 내용을 가져와서 보겠습니다.

{% codeblock %}
bootstrap.group(loopGroup, workerGroup)
         .channel(NioServerSocketChannel.class)
         .childHandler(new ChannelInitializer<SocketChannel>() {
           protected void initChannel(SocketChannel socketChannel) throws Exception {
             ChannelPipeline pipeline = socketChannel.pipeline();
             pipeline.addLast(new EchoServerHandler());
           }
         });
{% endcodeblock %}

해당 코드를 보면 ChannelPipeline pipeline = socketChannel.pipeline()에서 파라미터로 받은 채널안의 파이프라인을 가져오고 다음 pipeline.addLast(new EchoServerHandler()) 메소드로 파이프라인의 마지막에 핸들러를 등록하는 것을 볼 수 있습니다.

그리고 Netty는 소켓채널에 파이프라인을 등록하고 핸들러의 설정을 등록하기 위해서 세 단계 프로세스를 거칩니다.

1. 클라이언트 연결에 대응하는 채널 객체를 생성하고 빈 채널 파이프라인 객체를 생성하여 소켓 채널에 할당합니다.
2. 채널에 등록된 ChannelInitializer 인터페이스의 구현체를 가져와서 initChannel 메소드를 호출합니다.
3. 1에서 등록된 파이프라인 객체를 가져온 후 파이프라인에 입력된 이벤트 핸들러 객체를 등록합니다.

위의 세 단계가 완료 되면 채널이 등록됐다는 이벤트와 함께 데이터 송수신을 위한 이벤트 처리가 시작된다.

<h2>이벤트 핸들러</h2>

Netty는 비동기 처리를 위해 두가지를 제공합니다. 하나는 퓨처패턴이며 하나는 이벤트 핸들러입니다.
책에서는 이벤트 핸들러에 대해 자세히 알아보겠습니다.

대부분의 이벤트는 한 번만 수행하지만 다시 이벤트를 발생시켜 다른 등록된 핸들러에서 추가로 이벤트 처리를 시킬 수 있습니다.

채널 인바운드 이벤트 - 인바운드 이벤트는 소켓 채널에서 발생한 이벤트 중 상대방이 어떤 동작을 취했을 때 발생합니다.
클라이언트가 서버에 접속한 상태에서 서버에게 데이터를 보낼 경우 Netty의 소켓 채널은 데이터를 받았다고 채널 파이프라인에게 데이터 수신 이벤트를 보내고 채널 파이프라인은 등록한 인바운드 이벤트 핸들러를 호출합니다.

ChannelInboundHandler인터페이스를 가지고 인바운드 이벤트를 처리할 수 있습니다.
안의 메소드는 다음과 같습니다.
<p><img src="https://user-images.githubusercontent.com/20100284/50545348-05774700-0c54-11e9-8014-9de540244325.png" alt="Inbound"></p>

이벤트 호출 순서는 다음과 같습니다.

1. 이벤트 루프에 채널 등록(channelRegistered)
2. 채널 활성화(channelActive)
3. 데이터 수신(channelRead)
4. 데이터 수신 완료(channelReadComplete)
5. 채널 비활성화(channelInactive)
6. 이벤트 루프에서 채널 제거(channelUnregistered)

아마 대부분 메인로직은 데이터 수신~수신완료에 쓰일것이라 보입니다.

이벤트를 간단히 정리해보겠습니다.

1. channelRegistered의 이벤트 발생 시점은 서버와 클라이언트 모두 처음 소켓 채널이 생성할 때 발생하며, 서버의 경우 클라이언트 소켓 채널이 서버 소켓 채널에 등록될 때 다시 발생합니다.
2. channelActive의 이벤트는 서버 또는 클라이언트가 연결한 직후 한 번 수행할 작업을 처리하기에 적합합니다.(예를 들면 Hello Client 라던가…)
3. channelRead와 channelReadComplete 이벤트는 데이터의 수신시 발생하지만 Read는 채널에 데이터가 있을 때 발생하고 Complete는 데이터를 다 읽었을 때 발생합니다.
4. channelInactive가 발생하면 Read가 불가능합니다.
5. channelUnregistered가 발생하면 등록했던 소켓에 대한 이벤트 처리가 불가능합니다.

채널 아웃바운드 이벤트 - 대부분 클라이언트 객체에서 처리할 것으로 보이는 아웃바운드입니다. 요청을 보내거나 데이터 전송, 소켓 닫기 등의 이벤트를 가지고 있습니다.
<p><img src="https://user-images.githubusercontent.com/20100284/50545453-6ce2c600-0c57-11e9-8942-ee94220bf243.png" alt="Outbound"></p>

이벤트를 간단히 정리해보면..

1. bind 이벤트는 서버 소켓 채널에 클라이언트가 연결을 대기하는 IP와 포트가 설정되었을 때 발생합니다. 파라미터 중 SocketAddress로 서버에서 사용하는 IP/포트 정보를 확인 가능합니다.
2. connect 이벤트는 서버에 연결되었을 때 발생합니다.
3. disconnect 이벤트는 연결이 끊어졌을 때 발생합니다.
4. close 이벤트는 소켓 채널의 연결이 닫혔을 때 발생합니다.
5. write 이벤트는 소켓 채널에 데이터가 기록되면 발생합니다.
6. flush 이벤트는 소켓 채널에 대한 flush 메소드가 호출되면 발생한다고 합니다.

<h2>ChannelHandlerContext</h2>

대부분의 이벤트가 ChannelHandlerContext를 파라미터로 제공하는 것을 볼 수 있습니다. 이 녀석의 경우 채널 파이프라인이나 다른 핸들러의 상호작용을 도와주는 인터페이스입니다.

즉, writeAndFlush등의 메서드로 채널에 데이터를 기록하며 close 메소드로 연결종료나 자신이 속한 파이프라인의 동적 수정, 다음 Handler에 통지하는 등 다양한 기능을 수행합니다.

<h2>코덱</h2>

우리가 동영상 플레이어를 설치 시 말하는 그 코덱과 의미가 같습니다. 즉, 전송을 보내는/받은 파일을 특정 루틴에 의해 압축,변환 시키고 해제를 시키는 라이브러리 같은 개념으로 생각하면 되겠죠.
Netty 에서는 코덱을 통해 데이터-패킷으로 상호 변환을 합니다. 그리고 Netty의 코덱은 템플릿 메소드 패턴을 사용했다고 합니다.(실행순서만 위에서 설정하고 실제 구현은 하위에서)

사용자가 직접 만들 수 있지만 대부분의 프로토콜은 Netty에서 기본제공하므로 가져다가 쓰시는 것을 추천합니다.
기본 예제는 io.netty.example 패키지에 포함되어있다고 합니다.