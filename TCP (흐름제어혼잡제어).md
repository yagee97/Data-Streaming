# TCP (흐름제어/혼잡제어)

### TCP 통신

* 네트워크 통신에서 신뢰적인 연결방식(3 way handshake)
* TCP는 기본적으로 unreliable network에서 reliable network를 보장할 수 있도록 하는 프로토콜
* TCP는 network congestion avoidance algorithm을 사용한다.

<br>

### Reliable network를 보장한다는 것의 문제점

> 1) Loss: Packet이 손실될 수 있는 문제
>
> 2) 순서 바뀜: Packet의 순서가 바뀌는 문제
>
> 3) Congestion: 네트워크가 혼잡한 문제
>
> 4) Overload: receiver가 overload 되는 문제

<br>

### 흐름제어/혼잡제어란?

* **흐름제어**

  - 송신측과 수신측의 데이터 처리 속도 차이를 해결하기 위한 방법
  - Flow control은 receiver가 packet을 지나치게 받지 않도록 조절한다.
  - 기본 개념은 **receiver가 sender에게 현재 자신의 상태를 feedback** 한다는 점이다.

* **혼잡제어**

  : 송신측의 데이터 전달과 네트워크의 데이터 처리 속도 차이를 해결하기 위한 기법

<br>

<br>

## 전송의 전체 과정

 ![osi 7계층에 대한 이미지 검색결과](data:image/jpeg;base64,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) 

> Application layer: sender application layer가 socket에 data를 쓴다.
>
> Transport layer: data를 segment에 감싼다. 그리고 network layer에 넘겨준다.
>
> 그 후 아래 계층에서 receiving node로 전송이 된다. 이 때 sender의 send buffer에 data를 저장하고, receiver는 receive buffer에 data를 저장한다.
>
> application에서 준비가 되면 이 buffer에 있는 것을 읽기 시작한다.
>
> 따라서 **flow control의 핵심은 이 receiver buffer가 넘치지 않게 하는 것**이다.
>
> receiver는 RWND(Receive WiNDow): receive buffer의 남은 공간을 sender에게 알린다.

<br>

<br>

## 1. 흐름제어 (Flow Control)

* 수신측이 송신측보다 데이터 처리 속도가 빠르면 문제없지만, 송신측의 속도가 빠를 경우 문제가 생긴다. (buffer에 저장하는 속도보다 <u>보내는 속도가 더 빠르면</u>)

* 수신측에서 제한된 저장 용량을 초과한 이후에 도착하는 데이터는 손실 될 수 있으며, 만약 손실된다면 불필요하게 응답과 데이터 전송이 송/수신 측 간에 빈번히 발생한다.

  (receiver의 **buffer가 overflow**나면 그 이후에 들어오는 **packet은 loss**된다.)

* 이러한 risk를 줄이기 위해 <u>송신 측의 데이터 전송량을 수신측에 따라 조절해야한다.</u>

<br>

#### 해결방법

1) **Stop and Wait**: 매번 전송한 패킷에 대해 확인 응답(Ack)을 받아야만 그 다음 패킷을 전송하는 방법이다.

* **rdt(reliable data transfer)**

![image-20191115194635667](C:\Users\yeji\AppData\Roaming\Typora\typora-user-images\image-20191115194635667.png)

> * rdt v1.0: reliable transfer over a reliable channel
>
>   => 전송과정에서 에러가 없다고 가정
>
>   * no bit errors
>   * no loss of packets
>
> * rdt v2.0: channels with bit errors
>
>   * checksum을 통해 error가 발생했는지 체크
>   * no loss of packets
>   * receiver가 error를 감지하면 NAK을 보내서 sender에게 데이터를 재요청한다.
>   * NAK을 받은 sender는 retransmission을 해준다.
>
> * rdt v3.0: channel with errors and losses
>
>   * v2.0과 달리 NAK은 사용하지 않고, ACK만 사용한다.
>   * TIMEOUT을 둬서 그 시간내에 ACK이 오지 않으면 재전송. 
>   * 패킷은 잘 받았지만 보낸 ACK이 loss가 나면 같은 데이터를 두번 받게 되는 duplication이 발생하게 될 수도 있다.
>   * Timeout이후에 ACK이 도착하는 경우엔 ACK이 겹쳐서 온다. 이 경우 duplication ack이 발생할 수도 있는데 sender의 입장에선 그냥 무시한다.
>   * 그래서 ACK에도 sequence number를 붙여야한다.

![image-20191115195704667](C:\Users\yeji\AppData\Roaming\Typora\typora-user-images\image-20191115195704667.png)

하지만, Stop and wait 방식은 sender가 데이터를 보내고 receiver로부터 ACK을 받기 전까지 아무일도 하지 않고 있어야 하기 때문에 <u>효율성이 떨어진다.</u>

<br>

**2) Pipelined protocols**

* **go-Back-N**: N(=window size)만큼 한번에 packet을 전송한다. packet 여러개를 한번에 보내고 기다리는 것이기 때문에 어디서부터 어디까지 보냈는지 sender가 기억을 해야한다. <u>base</u>는 가장 오랫동안 ACK을 받지 못한 패킷을 말한다.

  ![image-20191115200203598](C:\Users\yeji\AppData\Roaming\Typora\typora-user-images\image-20191115200203598.png)

  중간에 loss가 나면 그 뒤에 도착한 packet들을 모두 무시하고, 가장 최근에 잘 받은 packet에 대한 ACK만 계속 전송한다. 일정시간이 지나면, 받지 못한 packet부터 다시 다 전송한다.

  이 때의 ACK 번호의 의미는 여기까지 잘 받았다. 하는 것 (= Cumulative ACK)

*  **selective repeat**: GBN과 sender의 역할은 같지만, receiver에서 차이가 있다. cumulative ack을 사용하지 않고 individual ack을 사용한다. 전송마다 타임아웃을 따로 둬서 타임아웃이 발생한 데이터에 대해서만 재전송을 한다.