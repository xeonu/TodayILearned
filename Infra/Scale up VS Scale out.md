여러분이 속한 개발팀에서 열심히 개발을 해서 성공적으로 서비스를 배포했다. 유저들이 유입되면서 서버는 한계에 도달했고 여러분은 이를 해결하기 위해 서버를 확장하고자 한다. 어떤 방법으로 서버를 확장할지 여러분이 결정해야한다면 뭐 부터 시작해야할지 걱정이 많을 것이다. 이 포스팅은 여러분이 서버를 확장하기 위한 고민을 해결하기 위한 첫 관문이 될 것이다.

# 서버 확장 전략
서버를 확장하기 위한 방법에는 크게 **Scale up**과 **Scale out**이 있다. Scale up은 서버 자체의 사양을 증가시키는 것이고, Scale out은 비슷한 사양의 서버를 여러대 두어 확장하는 방법이다. 클라우드 서비스를 통해 이 두 전략에 대해 알아보자.
## 🔎 Scale up
현재 서비스를 운영중인 서버가 1대라고 하자. 이 서버의 사양을 업그레이드 하는 것이 Scale up이다. 여러분이 사용하는 컴퓨터가 버벅거리면 RAM을 사서 하나 더 꽂을 수도 있고 배틀그라운드를 4K해상도에서 144hz로 아주 높은 그래픽으로 즐기고 싶다면 RTX3090과 같은 그래픽카드로 교체할 수 있다. 

서비스를 운영중인 서버도 여러분이 사용하는 컴퓨터와 큰 차이 없이 같은 방법으로 사양을 변경하면된다. 아래 사진은 네이버 클라우드 플랫폼의 서버 이용요금표이다.

<img src = "./img/네이버 클라우드 플랫폼 가격표.png" width = 80%/>


현재 우리가 맨 윗줄에 있는 월 88,000원 짜리 서버를 이용중일 때 Scale up 방식으로 서버를 확장한다고 하면 더 높은 사양의 서버를 선택하여 업그레이드만 해주면 된다. **정말 쉽지 않은가**?

또 Scale up으로 확장한 서비스는 단일 서버로 구성되어 있기 때문에 **소프트웨어 라이센스를 1개만 구매해도 서비스를 운영할 수 있다**.

하지만 단일 서버로 구성되어 있기 때문에 해당 서버에 **장애가 발생하면 해결될 때 까지 서비스가 중단될 수 있다**. 

또 다른 문제점으로는 Scale up 방식으로 **증가시킬 수 있는 사양에는 한계가 있다는 것이다**. 우리는 1000기가의 메모리가 필요한데 단일 서버로는 그 사양을 구현할 수 없다. 그리고 무엇보다 Scale up은 **가성비가 좋지 않다**. cpu 및 스토리지의 가격이 비싸질 수록 성능 향상은 미미해진다. 아래 그래프는 가격 대비 cpu 성능 그래프이다.

<img src = "./img/cpu 가격 성능 그래프.png" width = 80%>

그래프가 직관적이지는 않지만 고사양 cpu로 갈수록 성능에 대한 가격폭이 커짐을 알 수 있다. 이 처럼 같은 가격이면 성능 좋은 cpu 1대보다 적당한 cpu 2대를 병렬로 구성하는게 더 좋은 성능을 보여줄 수 있다. 좀 더 확실하게 알고싶다면 해당 논문을 찾아 보기 바란다._(Scale Up vs. Scale Out in Cloud Storage and Graph Processing Systems)_

## 🔎 Scale out
서버 사양을 업그레이드 하기보단 비슷한 사양의 서버를 여러대 두어 트래픽을 분산시키는 방식으로 확장하는 방식이다. **로드밸런서**는 클라이언트의 요청을 적절한 서버로 보내주는 클라이언트와 서버의 교환원 역할을 한다. AWS의 로드밸런서 역할을 하는 ELB의 구조를 한번 확인해보자. 

<img src = "./img/ELB 구조.png" width = 80%>


Availability Zone이라는 영역에 서버들을 병렬로 연결해 놓았다. 사진 최상단에 있는 로드밸런서는 클라이언트로부터 요청을 받고 적절한 알고리즘(라운드 로빈)을 통해 서버로 요청을 전달한다. 위 사진은 10개의 서버로 구성되어 있는 서비스의 서버구성도를 보여주고 있으며 각 서버는 10이라는 트래픽을 균일하게 할당받고 있다. 이 덕분에 **Scale up에 비해 합리적인 비용으로 높은 사양의 서버의 성능을 발휘**할 수 있다. 또한 대규모 **서버 확장에도 큰 제한이 없다**.

여담으로 Availability Zone으로 서버 구획을 나눈 이유는 해당 Zone에 50대 50으로 트래픽을 분산시키고 Zone 내부에서 또 트래픽을 분산시킬 수 있게 하기 위해서다. 해당 개념에 대해서는 '교차 영역 로드밸런싱'에 대해 알아보기 바란다.

하지만 로그인을 위한 세션정보를 서버마다 다르게 가지고 있을 수 있는 **세션 불일치 문제**가 발생할 수 있다. 해당 문제가 발생하면 유저가 같은 서비스를 이용하지만 다른 서버에 요청을 보낼 때 마다 로그인을 해야하는 번거로움이 발생할 수 있다.

Scale out은 사진만 봐도 알겠지만 **Scale up에 비해 구축하기 어렵다**. 또 미리 서버를 어렵게 분산시켜 놓고 구축해 놓았는데 트래픽이 몰리지 않을 때 **노는 서버가 늘어나 불필요한 지출이 발생한다**.

Scale up이면 소프트웨어를 1개만 구입하면 됐었는데 10개로 서버가 늘어난 지금은 무려 10개나 구입해야해서 **라이센스 구입비용이 증가한다**.

## ❓우리 서비스에는 무엇을 선택해야할까?
우리는 웹서비스에 관해서 서버 확장 전략에 대해 고민하고 있었지만 서버 확장은 다음과 같이 다양한 목적으로 이루어진다.

 <img src = "./img/스케일업아웃 사용처.png" width = 70%>
 
 위 사진을 확인해 보면 알겠지만 우리가 구현하려는 web infrastrucre(웹서비스)는 Scale out을 선택한다고 해당 지표가 보여준다.

그럼 왜 Scale out을 선택해야할까?
>비용이 저렴하다

인건비를 제외한 웹서비스의 운영비용중 가장 큰 부분을 차지하는 것은 서버비용일 것이다. 이를 절약하는 것은 곧 기업의 영업이익 증가로 이어진다.

> 서버 확장이 용이하다.

웹서비스는 서비스가 유명해져서 이용자가 늘어날 수도 있고 어떤 이벤트로 인해 특정 시간에 트래픽이 몰릴 수 있다. 이런 상황을 대비해 항상 서버 확장에 열러있는 Scale Out 방식이 유리하다.

> 장애 대처가 Scale up에 비해 수월하다.

Scale out은 여러대의 서버로 구축되어 있기 때문에 특정 서버에 트래픽이 몰려 서버가 다운되더라도 다른 서버로 트래픽을 분산시키면 되어서 장애극복하는 과정에서 서비스가 중단될 가능성이 줄어든다.

> Scale out의 문제점은 어느정도 극복이 가능하다.

로그인 상황에서 발생할 수 있는 세션 불일치 문제는 Session Storage를 별도로 두거나 Session Clustering의 방법을 통해 극복할 수 있다. 대부분의 클라우드 서비스에서 상황에 따라 서버 대수를 조절할 수 있는 Auto Scaling 기능을 제공하므로 노는 서버로 지출되는 비용을 줄일 수 있다. 세션 불일치 문제에 대해서는 다음 포스팅에서 자세히 다루겠다.


<br>

필자가 개발하려는 서비스는 인플루언서와 같은 유명인이 자신만의 벼룩시장을 개설해 물품을 판매하는 서비스이다. 특정 시간에 판매게시를 예고하므로 판매시작 시점에 트래픽이 몰릴 것이다. 유연한 서버 확장과 수월한 장애 대처를 위해서 Scale out방식을 선택하는 것이 적절해 보인다.