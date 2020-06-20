---
title: 네티 2 - 프로토콜
categories:
  - Netty(복구중)
tags:
  - Netty
date: 2020-03-25 14:36:17
---

<h2>간단한 용어설명</h2>
아무래도 통신관련 프레임워크기 때문에 통신 프로토콜을 모를 수도 없어서 간단하게 알아보도록 하겠습니다.

TCP - TCP의 경우 연결을 확인하고 메시지를 전송하는 것을 보장하는 특성을 가지고 있습니다.메시지를 받을때마다 받았다는 신호를 보내 서로 전송되었고 받았는지 확인하죠.

UDP - UDP는 메시지는 전송하지만 받았는지 전혀 확인하지 않기 때문에 누락, 분실, 메시지 순서를 보장하지 않는 등의 특징을 가지고 있습니다. 그럼에도 왜 쓰냐하면 체크하는 과정이 없기 때문에 TCP에 비해 상대적으로 오버헤드가 적고 속도가 빠른 장점이 있습니다.

SCTP - IBM에서 좋은 정보를 제공하기에 가져왔습니다.

스트림 전송 제어 프로토콜(SCTP)은 TCP와 유사한 연결 지향 프로토콜이지만 UDP와 유사한 메시지 지향 데이터 전송을 제공합니다. 일반적으로 SCTP는 안정적이면서도 메시지 지향적인 데이터 전송을 필요로 하는 VoIP(Voice over IP)와 같은 특정 애플리케이션에 대해 더 많은 유연성을 제공합니다. 이런 애플리케이션 범주에 대해 SCTP는 대부분 TCP나 UDP보다 더 적합합니다.
	•	TCP는 신뢰 가능하며 엄격한 전송 순서의 데이터 전달을 제공합니다. 신뢰성이 필요로 하지만 순서화되지 않았거나 부분적으로만 순서화된 데이터 전달을 허용하는 애플리케이션의 경우, TCP는 HOL(head-of-line) 블로킹으로 인해 불필요한 지연을 초래할 수 있습니다. 단일 연결 내에 여러 스트림 개념을 사용하는 SCTP는 데이터를 다른 스트림으로부터 논리적으로 분리하는 동시에 하나의 스트림 내에서 엄격한 순서로 전달할 수 있습니다.
	•	SCTP는 바이트 지향인 TCP와는 달리 메시지 지향입니다. TCP의 바이트 지향 특성으로 인해 애플리케이션은 메시지 경계를 유지하려면 고유한 레코드 표시를 추가해야 합니다.
	•	SCTP는 멀티-홈 기능을 사용하여 어느 정도의 결함 허용치를 제공합니다. 호스트는 동일하거나 다른 네트워크에서 둘 이상의 네트워크 인터페이스가 접속된 경우 멀티-홈으로 간주됩니다. 두 개의 멀티-홈 호스트 사이에 SCTP 연관을 설정할 수 있습니다. 이 경우 두 엔드포인트의 모든 IP 주소가 연관 시작 시에 교환됩니다. 이를 통해 각 엔드포인트는 인터페이스 중 하나가 어떤 이유에서이건 작동 중지된 경우 대체 인터페이스를 통해 피어에 연결 가능한 한 남은 연결 기간 동안 이러한 주소를 사용할 수 있습니다.
	•	SCTP는 TCP 및 UDP는 제공하지 않는 추가 보안 기능을 제공합니다. SCTP에서 연결 설정 시 자원 할당은 쿠키 교환 메커니즘을 사용하여 클라이언트의 ID를 검증하기 전까지 지연되어 서비스 거부 공격 가능성을 줄입니다.

프로토콜간 차이점
<p><img src="https://user-images.githubusercontent.com/20100284/50411940-77f8b900-0847-11e9-89af-4f0dc106a21b.png" alt="차이점(어디서 가져온지 기억이 잘..)"></p>
￼
<h2>부트스트랩이란?</h2>

동명의 웹 프론트엔드 프레임워크가 아니라 Netty에서 가장 처음 수행되는 부분으로써 동작과 설정을 지정하여 가장 기본이 되는 부분이라 합니다. API Doc의 Bootstrap 클래스에서는 다음과 같이 얘기하고 있습니다 : 부트스트랩은 채널을 쉽게 사용할 수 있도록 해주는 것

논리적 구조는 다음과 같습니다.
	•	소켓 모드 및 I/O종류
	•	이벤트 루프
	•	채널 파이프라인 설정
	•	소켓 주소 및 포트 등의 옵션 설정

빌더 패턴을 사용하여 선택적인 옵션을 파악하기 쉽도록 추가할 수 있습니다.

가장 처음에 작성했던 에코 서버 코드를 가져와서 조금 수정 후 확인해봅시다.

{% codeblock %}
EventLoopGroup loopGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();

ServerBootstrap bootstrap = new ServerBootstrap();
bootstrap.group(loopGroup, workerGroup)
    .channel(NioServerSocketChannel.class)
    .childHandler(new ChannelInitializer<SocketChannel>() {
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        ChannelPipeline pipeline = socketChannel.pipeline();
                        pipeline.addLast(new EchoServerHandler());
                    }
                });

ChannelFuture f =bootstrap.bind(8888).sync();
f.channel().closeFuture().sync();
{% endcodeblock %}

먼저 new NioEventLoopGroup()를 보면 생성자의 인수가 없는데 이럴 경우 사용할 스레드 수를 하드웨어의 스레드 수로 한다고 합니다. 
스레드 수는 하드웨어가 가지고 있는 CPU 코어 수의 2배를 사용한다 합니다. 4C 8T의 경우 하드웨어(CPU)에서 지원하는 8T의 두 배인 16개죠. 
loopGroup 변수는 사용할 스레드 수를 1개로 지정하였습니다. ServerBootstrap.group() 메소드는 NioEventLoopGroup을 인수로 받는데 첫번째는 연결에 사용할 스레드이고 두 번째 인수는 연결된 소켓에 대한 I/O처리를 담당합니다. 

위 코드에서는 연결의 경우 한번만 하고 끝나니 하나만 사용했고, I/O의 경우 어떻게 몇개가 들어올지 모르니 여러개의 스레드를 사용했다고 생각했습니다.
channel() 메소드는 API명세에 따르면 Channel 인터페이스를 구현한 객체나 채널팩토리를 사용할 수 있고 이 예제에서는 NioServerSocketChannel 클래스를 설정했기 때문에 Nio모드를 사용합니다.
childHandler 메소드는 API에 채널에서 요청이 왔을 때 제공할 ChannelHandler를 설정합니다.
연결 방법을 바꾸고 싶으면 <code>.channel()</code> 부분에서 제공하는 클래스를 변경하면 입출력 모드를 쉽게 바꿀 수 있도록 Netty에서는 제공하고 있습니다. 
Netty에서 지원하는 클래스는 다음과 같습니다.(안보이면 우클릭 - 새탭에서 윈도우 열기)
<p><img src="https://user-images.githubusercontent.com/20100284/50411468-8ba22080-0843-11e9-920e-54fe248d67a7.png" alt="지원클래스 with Netty"></p>￼

이 중 Epoll은 *nix에서 동작한다고 합니다.

부트스트랩 내 간단한 API 소개
channelFactory - 소켓 입출력 모드 설정
channel 메소드와 동일한 기능을 수행합니다.
handler - 이벤트 핸들러 설정
소켓 채널에서 발생한 이벤트를 수신하여 처리합니다.
childHandler - 소켓 채널의 데이터 가공 핸들러 설정
소켓 채널로 송수신 되는 데이터를 가공하는 핸들러입니다. ChannelHandler 인터페이스를 구현한 클래스를 인수로 입력 가능합니다.
option - 소켓의 옵션을 설정합니다.
소켓의 옵션이란 소켓의 동작 방식을 지정하는 것을 말합니다. 소켓 옵션은 어플리케이션의 값을 바꾸는 것이 아니라 커널에서 사용되는 값을 변경한다는 의미입니다.
childOption - 소켓의 옵션을 설정합니다.
option 메소드는 서버의 옵션을, childOption 메소드는 클라이언트의 옵션을 설정합니다.
group - 이벤트 루프 설정
소켓 채널의 이벤트 처리를 위한 루프 객체를 생성합니다. 클라이언트는 단 하나의 이벤트 루프만 설정할 수 있습니다.
channel - 입출력모드 설정
서버 - 클라이언트의 입출력 채널을 설정합니다. 서버와 클라이언트는 설정할 수 있는 입출력 모드가 상이합니다.

결론
부트스트랩으로 서버와 클라이언트에서 각자 연결하고 처리할 수 있는 방법을 쉽게 제공합니다. 여러가지 준비해 놓았으니 상황에 맞춰 가장 최적의 연결방법을 찾아서 해야 겠지요…