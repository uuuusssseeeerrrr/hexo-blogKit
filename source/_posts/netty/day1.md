---
title: 네티 1 - 기본 환경설정
date: 2020-03-23 16:28:53
categories: Netty(복구중)
tags:
  - Netty
---

그동안 사놓고 안읽어봤던 자바 네트워크 소녀 Netty를 읽고 정리하면서 쓰는 글입니다.github blog + hexo로 만들어보는 첫번쨰 블로그이기도 하죠.

네티란? Netty는 유지 보수 가능한 높은 성능 프로토콜 서버 &amp; 클라이언트를 빠르게 개발할 수 있는 비동기적인 이벤트 기반 네트워크 어플리케이션 프레임워크라고 합니다.
<h2 id="시작해보기">시작해보기</h2>
1탄으로 Java 1.8 + maven 환경에서 작업해 보겠습니다. (추후에 생각나면 스프링도…) 라이브러리를 다운받아서 직접 lib 폴더에 넣으셔도 되지만 저는 그냥 maven으로 하겠습니다.

가장 먼저 프로젝트를 만들고 pom.xml에 netty 레포지토리 태그를 집어넣습니다.

개발환경은 5.0 alpha가 있지만 4.1 final버전으로 하겠습니다.

pom.xml에 다음과 같이 넣어주세요.

그리고 maven install을 하셔서 lib에 netty.jar가 정상적으로 들어있는지 확인해주세요.

intellij IDEA를 예로 들어서 정상적으로 라이브러리가 들어왔는지 확인 하려면 External Libraries에 들어있는지 확인하시면 되겠습니다.

<img src="https://user-images.githubusercontent.com/20100284/50393342-fd716000-0798-11e9-9b1d-3aa65d041566.png" alt="라이브러리 확인">

그럼 이제 Netty 프레임워크로 책내용을 따라서 만들어보겠습니다.

<h2 id="에코서버-만들기">에코서버 만들기</h2>

서버를 만들어 봅시다.

{% codeblock %}
public class EchoServer {
    public void echo() throws InterruptedException {
        EventLoopGroup loopGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {            
                ServerBootstrap bootstrap = new ServerBootstrap();            
                bootstrap.group(loopGroup, workerGroup).channel(NioServerSocketChannel.class)
                         .childHandler(new ChannelInitializer<SocketChannel>() {                        
                            protected void initChannel(SocketChannel socketChannel) throws Exception {                                       ChannelPipeline pipeline = socketChannel.pipeline();                            
                                pipeline.addLast(new EchoServerHandler());                        
                            }});            
                ChannelFuture f =bootstrap.bind(8888).sync();           
                f.channel().closeFuture().sync();       
            } Catch(Exception ex) {
                ex.printStackTrace();
            }finally {
                loopGroup.shutdownGracefully();            
                workerGroup.shutdownGracefully();
            }
    }
}
{% endcodeblock %}
따라치다보면 같은 클래스 명을 가진 라이브러리를 만날 수도 있습니다만…(안만날수도있고요) 무조건 io.netty.이하로 선택합니다.

살펴보니 EventLoopGroup이라는 것을 만들어서 ServerBootStrap이라는 것에 집어넣습니다.채널에 클래스를 뭔가 하나 담고 밑에서 일할 핸들러를 하나 만들고 파이프라인이라는 것을 만들어서 마지막에 핸들러를 하나 담았습니다.그리고 8888 포트를 만들어서 sync메소드로 동기화같은 작업을 하나 봅니다.

당연히 실행은 new EchoServer.echo()로 하겠지요.

이번에는 파이프 라인에 들어갈 핸들러를 만들어 보겠습니다.

{% codeblock %}
public class EchoServerHandler extends ChannelInboundHandlerAdapter {    
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String readMsg = ((ByteBuf)msg).toString(Charset.defaultCharset());
        System.out.println("수신한 문자열 [" + readMsg + "]");
        ctx.write(msg);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        super.exceptionCaught(ctx, cause);
        ctx.close();
    }
}
{% endcodeblock %}

이벤트 기반의 프레임워크라고 소개했듯이 핸들러를 통해 이벤트에 맞춰 어떻게 작업할 것인지 선택합니다.

여기서는 ChannelInboundHandlerAdapter를 상속받아서 이벤트 콜백을 설정합니다.channelRead와 모두 완료됐을때 호출될 channelReadComplete를 오버라이드 해서 완료했을때의 이벤트를 설정합니다.

그 다음 자바를 실행하고 telnet으로 접속해서 날려보면 자바 콘솔창에 확인이 가능합니다.

<h2 id="에코-클라이언트-만들기">에코 클라이언트 만들기</h2>

{% codeblock %}
public class EchoClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ByteBuf msgBuffer = Unpooled.buffer();
        byte[] sendMsgByte = "반가워요".getBytes();
        msgBuffer.writeBytes(sendMsgByte);
        ctx.writeAndFlush(msgBuffer);
    }
    
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String readMsg = ((ByteBuf)msg).toString(Charset.defaultCharset());
        System.out.println("수신한 문자열 ["+readMsg + "]");
        ctx.write(msg);
    }    
    
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }
}
{% endcodeblock %}

책에 나온 차이점이라면 처음에 채널이 연결되었을때 이벤트를 추가해준것 밖에 없습니다.실행해봅시다.

<h2>간단한 용어 설명</h2>

<h3 id="인바운드-아웃바운드">인바운드, 아웃바운드</h3>

인바운드란? 당사자 입장에서 들어오는 것

아웃바운드란? 당사자 입장에서 나가는 것

내가 친구에게 <code>안녕</code>이라는 메시지를 보냈다면, 친구입장에서는 <code>안녕</code>이라는 메시지가 인바운드 된 것이고 나에게는 <code>안녕</code>메시지가 아웃바운드 된 것이다.

<h3 id="동기-비동기">동기, 비동기</h3>

이 것은 굳이 Netty가 아니더라도 수 많은 곳에서 사용합니다. 사전에 정의된 해설은 다음과 같습니다.

동기란? 데이터 전송에 있어서 일정 클록 신호에 맞추어 데이터의 송수신을 하는 방법으로 통신을 시작할 때, 데이터의 최초에 통신 속도와 통신 방법을 식별하기 위한 동기 캐릭터(synchronous character)를 첨부하여 송신하는 방법

비동기란? 통신을 하는 양쪽 장치가 데이터를 주고받을 때 일정한 속도를 유지하는 것이 아니라, 약정된 신호에 기준하여 동기를 맞추는 통신 방법

<h3 id="블로킹-논블로킹">블로킹, 논블로킹</h3>

소켓통신에 주로 쓰이는 용어입니다. 물론 다른곳에도 쓰일수도 있지요.

블로킹이란? 요청한 작업이 완료되거나 에러가 발생해서 끝날 때 까지 응답을 돌려주지 않는 방법
논 블로킹이란? 작업 완료 여부와 상관없이 결과를 돌려주는 방법

<h3 id="이벤트기반-프로그래밍">이벤트기반 프로그래밍</h3>

내가 설정한 이벤트가 발생하면 이벤트에 직접 등록한 루틴이 실행되는 방식입니다.즉, 이벤트 발생 -&gt; 이벤트를 받아서 처리할 것이 있는지 검색 -&gt; 실행 순이겠죠.

Netty에서는 ChannelInboundHandlerAdapter로 여러 이벤트 기반 프로그래밍을 지원합니다.여기서 채널은 소켓 채널입니다.

<h3 id="Future-java">Future(java)</h3>
간단하게 알아보면 어떠한 로직을 처리했을때 처리한 값을 받아 작업하는 방식입니다. 비동기에서 쓰이며 결과값을 얻을 때 까지 필요에 의해 블록처리가 될 수 있습니다.
작업이 정상적으로 완료/취소 되었는지 확인을 위해 방법을 제공하며 성공적으로 수행시 <code>get</code> 취소했을경우 <code>cancel</code> 메소드로 수행됩니다.(인터페이스 기준)
