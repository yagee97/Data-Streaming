# Data Streaming

데이터 스트리밍의 개념과 처리 방식을 이해해본다.

<br>

### 1. 한정 데이터(Bounded data)

이미 저장된 데이터를 말한다. 데이터의 증가나 변경 없이 **유지**되는 데이터다.

ex) 1월 정산 데이터

<br>

### 2. 무제한 데이터(Unbounded data)

데이터의 수가 정해져 있지 않고, 계속해서 추가되는 데이터를 말한다.

ex) SNS 타임 피드, 증권거래 ...

<br>

### 3. Event time과 Processing time

* Event time: 데이터의 발생 시간
* Processing time: event가 시스템에서 처리되는 시간

> 이상적으로는 Event time과 Processing time이 동일하면 좋겠지만, 네트워크 상황에 따라 Processing time이 Event time보다 늦고, 또한 Processing time에서 소요되는 실제 처리 시간은 일정하지 않고 들쭉날쭉 하다. 네트워크 상황이나 서버의 CPU, I/O 상황이 그때마다 차이가 나기 때문이다.

<br>

   ![img](https://t1.daumcdn.net/cfile/tistory/21328C3E577A10D50A) 

```
위의 그래프에서 Skew는 Event time과 Processing time간의 간격을 의미한다. 
Processing time에서 3초에는 Event time 1초에서 발생한 데이터를 처리하고 있는데, 
Event time에서는 3초 시간의 데이터가 발생하고 있기 때문에 Skew는 2초가 된다. 
```

<br>

### 4. Bounded data의 처리

Bounded data는 이미 저장되어 있는 데이터를 처리하는 것이기 때문에 별다른 처리 패턴이 필요하지 않다. 그저 데이터를 읽어서 한번에 처리해서 저장하면 된다. 

> Batch-processing을 뜻한다. Batch-processing은 일괄처리방식이다. 컴퓨터 프로그램의 흐름에 따라 일정 기간 또는 한정된 (Bounded) 데이터를 모아두었다가 한 시점에 순차적으로 처리하는 방식을 뜻한다. 맵리듀스(MapReduce) 기법인 하둡 또한 Batch-processing을 사용한다.

   ![img](https://t1.daumcdn.net/cfile/tistory/223A743E577A10D703) 

######  <br>

### 5. Unbounded data의 처리

스트리밍 데이터로 **배치(batch)** 와 **스트리밍(streaming)** 의 두가지 패턴으로 처리한다.

> - Fixed Windows: 스트리밍 데이터를 일정 시간 단위로 모아 처리하는 방식(Batch). 구현이 간단하지만 데이터 수집 후 처리가 되기 때문에 실시간 처리성이 떨어진다.
>
>   ex) 10-11시 데이터 수집, 11-12시 데이터 수집
>
>    ![img](https://cdn-images-1.medium.com/max/1200/1*7xyafF15E5AWSbg2Nerb8g.png) 
>
>   <br>
>
> - Streaming: 스트리밍 처리에는 Time agnostic, Filtering, Inner join, Approximation algorithms, Windowing 방식 등이 있고, 배치에 비해 복잡하다. Skew가 환경에 따른 변화가 심해 데이터가 시스템에 도착하는 순서 역시 순차적이지 않기 때문이다.

<br>

### 6. Streaming 처리 방법

#### 6-1) Time agnostic

: 데이터가 시간 속성을 가지고 있지 않은 데이터 이다. <u>들어오는 순서대로 처리</u>를 하면 된다.

<br>

#### 6-2) Filtering

: 들어오는 데이터 중 **특정 데이터만 필터링**하여 저장하는 구조

 ![img](https://t1.daumcdn.net/cfile/tistory/227B743E577A10D838) 

ex) 웹 로깅 데이터를 수집해서 특정 IP나 국가 대역에서 들어오는 데이터만 필터링해서 저장하는 시나리오

<br>

#### 6-3) Inner joins (교집합)

: 두 개의 Unbounded data에서 들어오는 값을 서로 비교하여 매칭시켜서 값을 구하는 방식이다. 양쪽 스트림에서 데이터가 항상 같은 시간에 도착하는 것이 아니기 때문에 아래와 같은 매커니즘이 필요하다.

```
A 도착 => B 도착할 때까지 버퍼에 저장 => B 도착 => 조인하여 결과 저장 => 버퍼에서 삭제
```

만약 반대쪽의 데이터가 도착하지 않으면, 이 버퍼 영역에 데이터가 계속 쌓이기 때문에 일정 기간이 지나면 반대쪽 스트림에서 데이터가 도착하지 않은 데이터를 주기적으로 삭제해주는 **garbage collection** 정책이 필요하다.

 ![img](https://t1.daumcdn.net/cfile/tistory/2207813E577A10D92E) 

<br>

ex) 모바일 뉴스 앱이 있다고 가정할때, 뉴스 앱에서는 사용자가 어떤 컨텐츠를 보는지에 대한 데이타를 수집 전송하고, 지도 앱에서는 현재 사용자의 위치를 수집해서 전송한다고 하자. 

이 경우 사용자별 뉴스 뷰에 대한 Unbounded data 와, 사용자별 위치에 대한 Unbounded data 가 있게 되는데, 이 두개의 데이타 스트림을 사용자로 Inner Join을 하면 사용자가 어떤 위치에서 어떤 뉴스를 보는지에 대해서 분석을 할 수 있다. 

<br>

#### 6-4) Approximation algorithms 근사치 추정

: 시급한 분석이 필요한 경우, 전체 데이터를 분석하지 않고 <u>일부만 분석</u>하거나, 대략적인 데이터의 <u>근사값</u>만을 구하는 방법으로 대표적으로 K-means 나 Approximate Top-N등이 있다.

ex) VOD에서 최근 10분간 인기있는 비디오 목록, 12시간 동안 가장 많이 팔린 제품

<br>

#### 6-5) Windowing

: 스트리밍 데이터를 처리할 때 일정 시간 간격으로 처리하는 것을 정의

> * Fixed Windows: 정확하게 일정 시간 단위로 시간 윈도우를 쪼개는 개념이다. 예를 들어, 윈도우 사이즈가 10분일 때, 1시 10분은 1시 00분 ~ 1시 10분까지의 데이터를, 1시 20분은 1시 10분~ 1시 20분까지의 데이터를 처리한다.
>
> * Sliding Windows: 윈도우가 움직이는 개념이다. 현재 시간으로부터 +- N 시간 전후의 데이터를 매 M 시간마다 추출하는 것을 슬라이딩 윈도우라고 하고 이 윈도우들은 서로 겹치게 된다. 
>
>   예를 들면 현재 시간으로부터 10분 전에서 측정시간까지의 접속자를 1분 단위로 측정하는 시나리오가 될 수 있다. 매 1분 간격으로 데이터를 추출하고 매번 그 시간으로부터 10분전의 데이터를 추출하기 때문에 데이터가 중첩이 된다. 이렇게 추출하는 간격을 Period, 그리고 추출하는 기간을 Length 또는 size라고 한다.
>
>    ![img](https://t1.daumcdn.net/cfile/tistory/2402323E577A10DA32)<br>
>
> * Session Windows: 사용자가 일정 기간동안 반응이 없는 경우 (데이터가 올라오지 않는 경우)에 세션 시작에서부터 반응이 없어지는 시간까지를 한 세션으로 묶어서 처리한다.
>
>   ex) timeout을 두는 것

<br>

### 6. 시간대별 Window 처리 방식

스트리밍 데이터에서 윈도우를 사용할 때, **어느 시간을 기준 시간**으로 할 것인가를 정해야하는데, 데이터가 시스템에 도착하는 Processing time을 기준으로 할 수 도 있고, 데이터가 실제 발생한 시간인 Event time을 기준으로도 할 수 있다.

<br>

#### * Processing time based windowing

: processing time을 기준으로 데이터를 처리하는 것은 어렵지 않다. 데이터가 도착한 순서대로 처리해서 저장하면 된다.



#### * Event time based windowing

: Event time을 기준으로 데이터를 처리하는 경우이다. 데이터가 들어오는 것이 순서대로 들어오지 않는 경우가 많고, 데이터의 도착 시간도 일정하지 않다.

 ![img](https://t1.daumcdn.net/cfile/tistory/250FEE3E577A10DD27) 

이 그림은 Event time을 기준으로 데이터를 처리하는 개념인데, 좌측 하얀색 화살표 처럼 12:00~13:00에 도착한 데이터가 11:00~12:00에 발생한 데이터일 경우, 11:00~12:00 윈도우에 데이터를 반영해줘야한다.

이러한 Event time 기반의 스트리밍 처리는 **Skew**가 발생하는 경우 저장했다가 처리해줘야 하기 때문에 **Buffering**과 **Completeness**라는 개념이 필요하다.



```
1) Buffering: 늦게 도착한 데이터를 처리해야 하기 때문에 윈도우를 일정시간동안 유지해야 한다. 
이를 위해서 메모리나 별도의 디스크 공간을 사용한다.

2) Completeness: Buffering을 적용했으면 얼마 동안이나 버퍼를 유지해야 하는가? 라는 문제가 발생한다. 
즉 해당 시간에 발생한 모든 데이터가 모두 도착이 완료(Completeness)되는 시간을 결정하는 것이다. 
```

<br>

<br>

참고: https://bcho.tistory.com/1119
