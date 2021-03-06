---
title: 네티 4 - 이벤트 핸들러
categories:
  - Netty
tags:
  - Netty
date: 2020-04-12 12:44:43
---
네티 알아보기 4일차 - 이벤트 핸들러

<h2>이벤트 루프</h2>

네티에서 빼놓을 수 없는 이벤트에 대해서 알아보고자 합니다.

통상적인 이벤트 기반 어플리케이션이 이벤트를 처리하는 방법은 크게 두 가지라고 합니다.

1. 이벤트 리스너와 이벤트 처리 스레드에 기반한 방법
2. 이벤트 큐에 이벤트를 등록하고 이벤트 루프가 큐에 접근하여 사용하는 방법

맨 위의 방식은 대부분의 UI처리 프레임워크가 사용하는 방법이라고 합니다. 로직을 리스너에 등록하고 처리 스레드가 등록된 로직을 수행하는 방식이죠.

생각나는 예를 들면
{% codeblock %}
JS : document.getElementById("myBtn").addEventListener("click", displayDate);
{% endcodeblock %}
두번쨰 방식인 이벤트 루프+큐의 방식은 객체에서 이벤트가 발생하면 이벤트 큐에 입력되고 이벤트 루프 스레드가 체크해서 이벤트를 가져와 처리하는 방식입니다.

계속해서 체크하며 돌기 때문에 Loop인것일까요?

여기서 스레드가 단일과 다중으로 나뉘고 이벤트의 결과를 돌려주는 방식에 따라 콜백 패턴과 퓨처 패턴으로 나뉩니다.
Netty는 둘 다 지원한다고 하니 잠시 후 알아보겠습니다.

<h2>단일/다중 스레드 이벤트 루프</h2>

단일 스레드 루프는 이벤트를 처리하는 스레드가 하나인 상태를 말합니다.
그래서 하나의 입력된 스레드가 이벤트 큐에 입력된 이벤트를 처리하므로 큐에 입력된 이벤트를 순차적으로 실행이 가능한 장점을 가지고 있지요.
단점은 당연히 기본 멀티코어인 요즘 시대에 코어를 1개밖에 못써먹는 치명적인 단점이 있습니다.

다중 스레드 루프는 이벤트를 처리하는 스레드가 여러개인 상태를 말합니다.
단점은 이벤트 루프의 갯수가 한정적이므로 접근하려는 경합이 일어나며 여러 스레드가 실행하므로 실행순서와 발생순서가 일치하지 않습니다.
하지만 장점은 남는 자원을 아낌없이 다 쓸 수 있다는 점이지요.

영상하나를 올리고자 합니다. 해당 영상은 CPU vs GPU 영상 입니다만 CPU를 단일스레드, GPU를 멀티스레드라고 생각해보면 그림을 그린다는 목적을 똑같이 가졌을 때 멀티스레드로 인한 효율을 간접적으로 볼 수 있습니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/-P28LKWTzrI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

nvidia youtube에서 퍼왔습니다.

하지만 다중 스레드 루프도 만능은 아닙니다. 스레드 경합과 컨텍스트 스위칭(스레드가 가진 스택 정보를 레지스터로 복사하는 작업)에 의한 비용이 늘어나서 코어의 활용으로 인한 성능향상보다 성능을 깎아먹게 되는 현상이 있으니 늘 적당한 갯수를 탄력적으로 모니터링하며 유지해야 하겠습니다.

<h2>네티의 이벤트 루프</h2>

Netty가 다중 스레드 루프를 사용했다면 너도 나도 가져가려는 이벤트 스레드로 인해 전송하다가 채널이 닫히는 등 엄청난 혼란이 일어났겠지만 Netty의 경우 다음과 같은 특징으로 인하여 다중 스레드 루프를 사용함에도 단점을 극복했다고 합니다.

네티의 이벤트는 채널에서 발생 -> 이벤트 루프 객체는 큐를 가지고 있다 -> 네티의 채널은 하나의 이벤트 루프에 등록된다.
즉, 1채널당 1개의 이벤트 루프에만 등록되기 때문에 한곳에서만 처리가 가능하다는 것이지요.
그리고 루프 객체당 큐를 가지고 있기 때문에 큐를 공유하지 않아 다른 객체에서 해당 큐의 접근이 불가능하여 이벤트를 빼앗기는 일이 없다고 하겠습니다.

조금 비유하면 컨퍼런스를 갔는데 여러 홀에서 동시에 발표를 한다 가정하면 우리의 몸이 만화속 닌자마냥 분신술이 불가능하기 때문에 하나의 발표밖에 못듣게 되죠.
그래서 여러 발표(이벤트)를 함에도 홀(이벤트 큐)이 공유되지 않아 하나의 발표밖에 못듣는거죠.(이벤트 처리)

<h2>네티의 비동기 I/O 처리</h2>

앞서 살펴보았던 이벤트 처리에 비해 유용하게 쓸 수 있는 퓨처패턴을 사용해 보겠다고 합니다.(책에서)
퓨처패턴은 당장 완료되지는 않지만 언젠가는 완료될 것으로 생각하고 미리 짜 놓으면 퓨처 객체에서 메소드의 처리 결과에 따라 진행하는 패턴입니다.

간단한 코드로 표현하면 다음과 같은 것입니다.(실제로 실행은 안되니 의사코드로써 봐주시기 바랍니다.)

{% codeblock %}
public class NettyFuture {
    public static void main(String[] args) {
        Work work = new Work();
        Future<Work> nettyFuture = new Future<Work>() {
            if (future.isDone) {
                //일이 끝났다 다음 해야할 일을...
            } else {
                //끝나지 않았으니 빨리 빨리 정신으로 갈구자
            }
        }
    }
}
{% endcodeblock %}
실제로 여러분이 처음 작성한 코드에서는 퓨처패턴이 사용중이었습니다.
{% codeblock %}
ChannelFuture f =bootstrap.bind(8888).sync();
f.channel().closeFuture().sync();
{% endcodeblock %}
위의 채널퓨처 객체를 사용하고 있었죠. sync() 메소드가 bind의 결과가 올 때 까지 블로킹하고 bind의 처리가 완료되면 같이 sync메소드도 진행됩니다.
위의 채널이 닫힐 경우 sync로 같이 닫아주는 형식으로 진행되고 있죠.

즉 퓨처패턴은 일단 프로그래머는 로직을 짜면 추후에 미래의 결과에 따라 진행하는 방식입니다.

퓨처패턴의 진행결과를 가져와서 처리해야 하는데 while로 계속해서 가져오는 방식도 있으나 복잡성 증가라던지 조금 그래서… 이벤트 방식의 리스너에 담아서 사용도 가능합니다.

<code>ChannelFuture.addListener(ChannelFutureListener.CLOSE)</code> <-- 이런식으로 말이죠
다음은 해당리스너 인터페이스를 구현하면 될 것이고요.

일단 ChannelFutureListener 인터페이스만 살펴보면 해당 인터페이스를 통해서 제공하는 것은 다음과 같습니다.

{% codeblock %}
ChannelFutureListener.CLOSE
ChannelFutureListener.CLOSE_ON_FAILURE
ChannelFutureListener.FIRE_EXCEPTION_ON_FAILURE
{% endcodeblock %}
그 외에 여러가지 이벤트 리스너를 사용해서 비동기의 제어도 가능합니다.

<code>ChannelFuture.addListener API</code>

보시는것과 같이 꼭 하나의 리스너만 받고 있지는 않으니 여러가지로 응용하시면 되겠습니다.

결론

네티의 이벤트 루프 스레드는 단일과 다중 스레드에서 이벤트 수행 순서의 차이가 없는 것이 장점이며 루프 스레드의 갯수를 쉽게 조정할 수 있으니 편하신대로 개발하시면 되겠습니다.
더 적은 고민으로 좋은 품질의 제품을 만들 수 있도록 해주었으니 잘 써먹어야겠죠.