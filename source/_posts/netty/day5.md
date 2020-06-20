---
title: 네티 5 - 바이트 버퍼
categories:
  - Netty
tags:
  - Netty
date: 2020-04-12 12:52:33
---

<h2>바이트</h2>

말그대로 1바이트를 저장하는 변수입니다.

원래는 1바이트는 -128에서 127의 범위를 가지지만 통신 관련 작업할때는 부호가 없는 0~255까지의 범위에서 씁니다.

저 같은 경우는 통신할 때 써봤고 16진수로 변환하여 통신을 했습니다.

그 외에는 쓰일곳이 저 같은 초보에게는 많을 것 같지는 않습니다. 나중이라면 많이 쓸 수 있겠지만요.

<h2>java.nio.ByteBuffer</h2>

자바에서 바이트 데이터를 저장하고 읽을 수 있도록 바이트버퍼를 제공합니다.

실제 버퍼를 구현하기 위해 배열을 이용하고 있으며 배열의 인덱스에 접근하지 않아도 쉽게 작업할 수 있도록 여러 메소드를 제공합니다.

책에 따르면 버퍼의 상태를 관리하는 속성 세 가지를 제공하는데 다음과 같다고 합니다.

capacity : 버퍼에 저장할 수 있는 최대 크기로 한번 정하면 변경이 불가능하며 생성자의 인수로 입력한 값이다.
position : 읽기 또는 쓰기가 작업중인 위치를 나타내며 처음 생성시 0으로 초기화되고 position <= limit <= capacity의 범위를 가진다.
limit : 읽고 쓸 수 있는 버퍼 공간의 최대치를 나타내며 capacity 보다 크게 설정할 수 없다.
저 세개의 값으로 버퍼에서 읽고 쓰는데 도움을 주며 각각 속성의 사용법은 아래의 사진을 보고 참조하시면 좋을 것 같습니다.

<img src="https://user-images.githubusercontent.com/20100284/51082041-f8715200-1741-11e9-844f-0b51e7b6f109.png"/> 

간단히 살펴보면 position과 limit로 어디까지 작업할지 선택하며 capacity로 오버플로우가 발생하지 않도록 체크하는 느낌입니다.

<h2>버퍼 만들기</h2>

자바에서 보통 객체 생성을 할 때는 new 생성자를 이용하지만 자바의 바이트 버퍼는 데이터 형에 따른 추상클래스 팩토리 메소도를 통해 생성합니다.

생성하는 메소드는 세 가지가 존재하며 다음과 같습니다.

1. allocate : JVM의 힙 영역에 바이트 버퍼를 생성합니다.
2. allocateDirect : JVM이 아닌 운영체제의 커널 영역에 바이트 버퍼를 생성합니다.
3. wrap : 입력된 바이트 배열을 사용하여 버퍼를 생성합니다. 말그대로 일반 바이트 배열을 버퍼로 wrap해주는 용도라 보시면 될 것 같습니다.
보통 다이렉트 버퍼가 힙에 생성된 버퍼에 비해 생성 시간은 길지만 빠른 성능을 제공합니다.

기본적인 생성 방법은 다음과 같습니다.
{% codeblock %}
public class javaBufSample {

    public void createTest() {
        ByteBuffer heapBuffer = ByteBuffer.allocate(3);
        ByteBuffer directBuffer = ByteBuffer.allocateDirect(3);
        byte[] testArray = {0x01, 0x02, 0x03};
        ByteBuffer wrapBuffer = ByteBuffer.wrap(testArray);

        System.out.println(heapBuffer.isDirect());      //false
        System.out.println(directBuffer.isDirect());    //true
        System.out.println(heapBuffer.isDirect());      //false
    }
}
{% endcodeblock %}
실행해 보시면 다이렉트 버퍼만 true고 나머지는 false로 되어있는 것을 볼 수 있습니다.
즉, wrap메소드의 처리결과로 봐서 직접적으로 명시하지 않는 이상 JVM의 영역을 기본적으로 사용한다는 것을 확인할 수 있습니다. 사실 당연한거죠..

보통 문자열이나 숫자등의 처리를 위해 byte형으로 변경할 수 있도록 메소드를 제공하고 있습니다.
그리고 선언한 버퍼의 크기보다 많으면 <code>java.nio.BufferOverflowException</code>이 발생합니다.

-flip 메소드-

하나의 바이트 버퍼가 쓰고 있다고 가정해봅시다. 쓰기 모드에서 버퍼에 데이터를 기록하면 position은 데이터의 가장 마지막을, limit는 버퍼의 가장 마지막을 가리키고 있을 것입니다.
이 버퍼에서 지금까지 작성한 데이터를 읽고 싶다면 예제야 0~position까지 읽으면 되겠지만 실제에서는 어떻게 되있을지 모르며 0부터 읽을 수 있을지 없을지도 모릅니다.
그래서 자바에서는 읽기에서 쓰기모드를 변환시켜주는 filp 메소드를 제공합니다.
이 메소드를 사용하면 버퍼에 존재한 데이터를 바탕으로 속성들의 위치를 변경시켜줍니다.

즉, 위의 그림에서 왼쪽에서 오른쪽으로 변경해 데이터를 안전하게 읽으려면 filp 메소드가 필요한 것이죠.
이러한 특성때문에 다중 스레드환경이나 읽기/쓰기를 분리해야 하는 등 불편한 사항이 많다고 합니다.

<h2>io.netty.buffer.ByteBuf</h2>

Netty 바이트 버퍼는 자바 바이트 버퍼보다 더 빠른 성능을 제공하며 버퍼 풀 등 여러가지 기능을 많이 가지고 있습니다. 이 바이트 버퍼가 가진 특징은 다음과 같습니다.

1. 별도의 읽기/쓰기 인덱스 존재
2. filp 메소드 없이 읽기/쓰기 가능
3. 가변 바이트 버퍼
4. 바이트 버퍼 풀
5. 복합 버퍼
6. 자바의 바이트 버퍼와 상호 변환 가능
이러한 바이트 버퍼를 따로 사용할 수 있도록 버퍼만 따로 클래스로 제공하기도 하니 필요하시면 가져다가 쓰시면 되겠습니다.
그리고 자바의 경우 intBuffer 등의 자료형마다 따로 버퍼클래스를 제공하지만 Netty 에서는 readInt/writeInt등의 자료형별로 메소드를 따로 제공하여 사용합니다.

사용해보기

먼저 Netty 바이트 버퍼를 생성할 때 풀링 여부 / 다이렉트 여부 두 가지를 선택해야 합니다.
그래서 이 기능에 따라 생성 가능한 버퍼 종류는 네가지입니다.

1. PooledHeapByteBuf
2. PooledDirectByteBuf
3. UnPooledHeapByteBuf
4. UnPooledDirectByteBuf
자바에서 추상 클래스의 팩토리 메소드를 사용해 생성했듯이 Netty도 클래스의 생성자가 아닌 메소드를 사용하여 생성합니다.
한 번 만들어 보겠습니다.

{% codeblock %}
public class NettyBufSmaple {
    public void createTest(){
        ByteBuf PooledHeapByteBuf = PooledByteBufAllocator.DEFAULT.heapBuffer();
        ByteBuf PooledDirectByteBuf = PooledByteBufAllocator.DEFAULT.directBuffer();
        ByteBuf UnPooledHeapByteBuf = Unpooled.buffer();
        ByteBuf UnPooledDirectByteBuf = Unpooled.directBuffer();
    }
}
{% endcodeblock %}
변수명을 보시면 어떻게 만들 수 있는지 보실 수 있습니다. 크기의 경우 메소드에 인자로 넣어주시면 크기가 지정 가능합니다.

그 외 특징으로는 capacity로 쉽게 크기 변경이 가능하며 readIndex/writeIndex의 분리로 인해 쉽게 읽고 쓰는 작업이 가능합니다.

<h2>바이트 버퍼 풀링</h2>

Netty의 가장 중요한 특징은 바이트 버퍼 풀링입니다.
객체 생성 등의 비용을 최대한으로 줄이기 위해 풀링 방법을 사용하며 다이렉트/힙 버퍼에 대해 모두 사용할 수 있습니다.
이러한 풀링으로 인해 GC에 부담을 적게 주며 메모리를 다른 곳에 더 사용할 수 있도록 합니다.

ReferenceCountUtil 클래스에 정의된 retain/release 메소드를 사용하여 풀링의 크기를 관리할 수 있습니다.

C언어와 같이 변수에 부호가 없는 경우 Netty는 한단계 더 큰 데이터형에 저장합니다.
예를 들면, getUnsignedInt 메소드를 호출 할 경우 int->long으로 업그레이드 해서 반환하는 식입니다.

자바 -> Netty 간 바이트버퍼 상호 변환을 위해 nioByteBuffer 메소드를 제공합니다.
Netty -> 자바에서는 wrappedBuffer 메소드를 사용하시면 됩니다.