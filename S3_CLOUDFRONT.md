# S3 + CloudFront



### (1) S3에 정적 컨텐츠 업로드

1. 속성 탭

   ![image](https://user-images.githubusercontent.com/42775225/145999252-e5da8454-1597-490e-a228-d8f3f2bb9978.png)

   403 forbidden 뜨면



2. 권한 탭

![image](https://user-images.githubusercontent.com/42775225/145998311-18becd93-36bb-45ed-900a-47c40be1669b.png)



<br />



### (2) CloudFront

![image](https://user-images.githubusercontent.com/42775225/145999822-14282161-8e7a-42de-93d9-467077e16e8f.png)







<br />

<br />

### 정적 파일 제공

스토리지는 `오라클 오브젝트 스토리지` 로 결정되었으므로, CDN 서비스만 비교하겠습니다.

**비용 산정 기준**은 다음과 같습니다.

\< Dev-Event 서비스 >

- 트래픽

  : 먼저 <a href="https://github.com/brave-people/Dev-Event/graphs/traffic">Dev-Event Github 저장소 visitor</a> 통계 근거로 하루 120~220views 이며, 웹으로 쉽게 사용할 수 있을 때 트래픽을 넉넉잡아 하루 300view로 산정하였습니다.

  => 9000views/Month

- 이미지 크기

  : `application.yml` 기준 이미지 파일 하나의 크기는 최대 3MB이며, 한 페이지에서 10개의 이미지를 보여주며, 최소 2페이지 이상 방문이라고 가정하였습니다.

  => 60MB/1view

<br />

\< 네트워크 서비스 >

- 네트워크 : **50GB**/월
- HTTPS요청 : 20000/월 *(크케 의미 있는 factor는 아닙니다.)*

<br /> 

<br />

__선택1. AWS Cloud Front__

> 비용, 장점

| 비용                                                         | 실제 테스트 | 장점             |
| ------------------------------------------------------------ | ----------- | ---------------- |
| 약 7100원<br />(<a href="https://calculator.aws/#/createCalculator/CloudFront">AWS 가격측정기</a>) | O (완료)    | 사용 편의성 우수 |

cf) 1년 사용에 대한 약정 시 CloudFront Savings Bundle을 사용하여 최대 30%를 절감할 수 있습니다.





<br />

__선택2. GCP Cloud CDN__

> 비용, 장점

| 비용                                                         | 실제 테스트 | 장점 |
| ------------------------------------------------------------ | ----------- | ---- |
| 약 5320원<br />(<a href="https://cloud.google.com/products/calculator#id=8e3dfe4e-bfb1-462d-9434-550f14c62465">GCP 가격측정기</a>) | X           |      |



<br />

__선택3. NCP CDN+__

> 비용, 장점

| 비용                                                         | 실제 테스트 | 장점 |
| ------------------------------------------------------------ | ----------- | ---- |
| 4500원<br />(<a href="https://www.ncloud.com/charge/calc/ko?category=networking#cdn">NCP 가격측정기</a>) | X           |      |



<br />

__선택4. NHN CDN__

> 비용, 장점

| 비용                                                         | 실제 테스트 | 장점 |
| ------------------------------------------------------------ | ----------- | ---- |
| 5000원<br />(<a href="https://www.toast.com/kr/service/content_delivery/cdn">NHN 가격</a>) | X           |      |



<br />

__선택5. 가비아 로컬 CDN__

> 비용, 장점

| 비용                                                         | 실제 테스트 | 장점 |
| ------------------------------------------------------------ | ----------- | ---- |
| 3500원<br />(<a href="https://cloud.gabia.com/cdn">gabia가격</a>) | X           |      |

cf) 로컬 CDN은 해외 트래픽을 지원하지 않습니다.



<br />

<br />

=> 직접 사용 전에 비교했을 시에는 비용적인 부분을 제외하고는 큰 차이가 없는 것 같습니다.