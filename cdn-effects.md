---
title: "CDN에 대해 알아보기 [항해 플러스 9주차]"
date: "2025-02-22"
lastmod: "2025-02-22"
tags: ["항해솔직후기", "항해 플러스 프론트엔드", "프론트엔드 회고", "CDN"]
draft: false
summary: "Vercel을 사용하면서 잊고 있었던 CDN... AWS CloudFront를 통해 알아보자"
---

# 서론

항해 플러스 9주 차에서는 인프라 차원에서 어플리케이션 성능을 최적화하는 방법에 대해 배웠습니다.
인프라 차원에서 깊이 들어가면 무수히 많겠지만, 우선 기본부터!! CDN을 활용한 성능 최적화에 대해 알아보았습니다.
이론적으로 알고 있는 줄 알았지만, 눈으로 확인해 보니 더욱 확실히 이해할 수 있었습니다.

사내에서는 담당하고 있는 프로젝트는 초기 프로젝트이면서 아직 작은 프로젝트이기에 우선 Vercel을 사용하기로 결정했고 현재 자동 배포를 진행하고 있습니다. 하지만 추후 AWS로 이관할 계획이 있기에 이번 기회를 통해 간단히 S3와 CloudFront를 이해해보는 시간을 가지려고 합니다.

> 해당 포스트에선 다음과 같은 내용을 공유하려고 합니다.

```md
1. CDN이 무엇인지
2. 사내 프로젝트를 AWS S3에 배포하고 CDN이 적용된 URL과 기존 S3 URL을 비교하여 성능 차이 비교
3. Vercel의 CDN에 대해 간단히 정리
```

# 본론

## CDN이 무엇일까?

CDN은 콘텐츠 전송 네트워크를 의미합니다. 콘텐츠는 웹 페이지, 이미지, 비디오, 오디오 등 모든 파일 형식을 포함합니다.

### CDN의 주요 기능

1. 지리적 분산
2. 로드 밸런싱
3. 캐싱
4. 보안

> 결과적으로 CDN을 사용하면 웹 사이트의 로딩 속도가 빨리 집니다. 또한 사용자 경험이 향상되고 서버의 부하가 줄어드는 이점이 있다고 합니다.

> 조금만 더 깊이 볼까요?

### CDN에서 로드 시간이 개선되는 방식은?

CDN은 3가지의 주된 방식으로 네트워크 지연 문제를 해결했습니다.

**1. 물리적으로 거리 단축하여 로드 시간 단축**

사진으로 보면 굉장히 직관적입니다.

**before** 뉴욕 -> 싱가폴

![CDN 거리 단축](https://www.cloudflare.com/img/learning/cdn/performance/cdn-distance-1.png)

**after** 뉴욕 -> 엣지 서버

![CDN 거리 단축](https://www.cloudflare.com/img/learning/cdn/performance/cdn-distance-3.png)

클라이언트 요청시 해당 지역을 식별하여 주변 엣지 서버로 라우팅하여 더 빠른 응답을 제공하는 방식입니다. (예시: 미국 사용자가 미국 엣지 서버로 요청하면 미국 엣지 서버로 라우팅)

**2. 최초 접속 이후, CDN 서버에 정적 컨텐츠가 캐싱돼, 이후 방문 시 로드 시간 단축**

![최초 방문시](https://cf-assets.www.cloudflare.com/slt3lc6tev37/9AB550JoAAVALSmHqwE2h/b6b8669bdef989d3b8afcfc3dd75ec86/cdn-caching-first-request_koKR.svg)
최초 방문시에는 캐싱된 정적 콘텐츠가 존재하지 않아 원본 서버로 요청이 갑니다. 이후 재 요청하면 아래와 같이 요청이 진행됩니다.

![두 번째 방문시](https://cf-assets.www.cloudflare.com/slt3lc6tev37/5fte3bPFqkQKiwp29p3uNs/4a44bdd3d4bbf4ea60c41ba7203bf33a/cdn-caching-second-request_koKR.svg)
이렇게 되면 정적 콘텐츠를 다운로드 받는데 시간은 필요하지 않게 돼, 엣지 서버와 클라이언트 간 통신 시간만 게산 하면 된다.

**3. 파일 크기 자체를 줄여 속도를 높여 시간 단축**

CDN은 로드 시간을 개선하기 위해 서버에서 클라이언트로 전송하는 데이터 양을 줄입니다. 그로 인해 대역폭이 감소하여 대기 시간이 줄어들고 속도가 빨라집니다.

두 가지 핵심이 있습니다.

> 첫 번째, 최소화입니다.

마치 빌드 시, 코드의 양이 줄어들는 방식과 비슷합니다. 컴퓨터만 이해 가능하도록 변경하여 양을 줄입니다.

**최소화 전, 8줄**

```JavaScript
// This Function takes in name as a parameter

function greet(name) {
  return "Hello, " + name + "!";
}

// This Function takes in name as a parameter
greet("Juyoung");
```

**최소화 후, 1줄**

```JavaScript
function greet(name){return "Hello"+O+"!"}greet("Juyoung");
```

> 두 번째, 파일 압축입니다.

gzip이 널리 사용된다고 합니다. 하지만 vercel이나 AWS CloudFront는 그보다 압축률이 좋은 br (brotli)를 통해 압축을 진행하고 있습니다.

## CDN 실습 결과

### 인프라 최적화를 통해 기대하는 결과

위에서 CDN의 이점 기반으로 아래의 2가지 목표를 달성하고자 합니다.

1. 라이트하우스 성능 지표 개선 (FCP, LCP 로드 시간 개선)
2. 번들 사이즈 최적화 (네트워크 속도 개선)

### 비교 환경

- AS-IS: S3 bucket 사내 프로젝트 URL

- TO-BE: CloudFront와 S3 bucket 사내 프로젝트 연결한 URL

### S3와 CloudFront CDN 성능 비교

#### LightHouse 측면

- **S3**

<img src="https://github.com/CodyMan0/bucket/blob/main/s3-lighthouse1.png?raw=true" />

- **CloudFront**

<img src="https://github.com/CodyMan0/bucket/blob/main/cdn-lighthouse1.png?raw=true" />

#### LightHouse 점수 비교

| 항목            | CloudFront | S3    | 차이  |
| --------------- | ---------- | ----- | ----- |
| 성능            | 100점      | 63점  | +37점 |
| 접근성          | 100점      | 100점 | -     |
| 권장사항        | 100점      | 75점  | +25점 |
| 검색엔진 최적화 | 91점       | 91점  | -     |

#### 성능 지표 비교

| 항목                          | CloudFront | S3    | 차이       |
| ----------------------------- | ---------- | ----- | ---------- |
| FCP(First Contentful Paint)   | 0.7초      | 3.3초 | 2.6초 감소 |
| LCP(Largest Contentful Paint) | 0.7초      | 3.5초 | 2.6초 감소 |
| Total Blocking Time           | 0ms        | 0ms   | -          |
| CLS(Cumulative Layout Shift)  | 0          | 0     | -          |
| Speed Index                   | 0.7초      | 3.3초 | 2.6초 감소 |

#### 주요 개선사항

1. 전반적 성능 점수

- CloudFront 사용 시 99점으로 37점 향상
- 특히 권장사항 부분에서 25점 큰 폭 개선

### 네트워크 속도 및 파일 크기

- S3

<img src="https://github.com/CodyMan0/bucket/blob/main/s3-network.png?raw=true" />

- CloudFront

<img src="https://github.com/CodyMan0/bucket/blob/main/cdn-network.png?raw=true" />

| 항목         | CloudFront | S3     | 차이     |
| ------------ | ---------- | ------ | -------- |
| 총 로드 시간 | 269ms      | 2.58초 | -2.311s  |
| DOM 로드     | 198ms      | 2.16ms | -2.144s  |
| 리소스 크기  | 827KB      | 2.2MB  | -1.373MB |

#### 주요 성능 차이점

1. 전체 로딩 속도

- CloudFront: 269ms로 로딩 속도 보여짐
- S3 직접 호스팅: 2.58초로 상대적으로 느린 속도
- CloudFront 사용 시 약 90% 성능 향상

2. 개별 리소스 로딩 시간

- Font 파일:

<img src="https://github.com/CodyMan0/bucket/blob/main/s3-font.png?raw=true" />

<img src="https://github.com/CodyMan0/bucket/blob/main/vercel-font.png?raw=true" />

- CloudFront: 64ms
- S3: 1311ms

3. 압축 및 최적화

- CloudFront는 더 효율적인 파일 압축을 보여줌
- 전송된 데이터 크기가 약 `1.373MB` 감소 (CDN을 사용했을 뿐인데...)

### 실습 결론

AWS의 CloudFront를 사용해보니 얻을 수 있는 이점은 다음과 같았습니다.

- 전체 페이지 로드 시간 단축
- DOM 컨텐츠 로드 시간 단축
- 더 효율적인 리소스 압축과 전송: 특히 큰 파일(폰트, 스크립트)에서 현저한 성능 향상
- LightHouse 성능 점수 향상
- 검색엔진 최적화 유지

## Vercel의 CDN을 살펴보겠습니다.

[Vercel Edge Network](https://vercel.com/docs/edge-network/overview)를 참고하였습니다.

지금까지 버셀을 사용하여 배포했다면, 자동적으로 엣지 네트워크를 사용하고 있었던 것입니다. 엣지 네트워크는 위에서 살펴본 CDN을 말하는 것이죠.

![Vercel Edge Network](https://vercel.com/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1724702247%2Ffront%2Fdocs%2Fedge-network%2Flight-pops.png&w=1080&q=75)

### Compression 부분을 살펴보겠습니다.

Vercel에서 압축은 자동으로 해주고 있었습니다. 클라이언트에서 요청시 요청 헤더로 Acception Encoding을 포함하여 요청하면 압축을 해줍니다.

위의 그림과 같이 전 세계적으로 네트워크 구조가 존재합니다. Vercel은 위의 구조에 기반해서 만들어졌습니다.

- Points of Presence (POPs) : Vercel 네트워크는 100개의 Pops를 포함하고 있습니다. 요청에 대해 가장 가까운 POP에 연결하여 더 빠른 응답을 제공합니다.
- Edge Regions: Pops 이면에 데이터를 빠르게 주기 위해 18개의 컴퓨팅 영역을 유지합니다.
- Private Network : 데이터 전송 속도를 높이기 위해 사용되는 네트워크 구조입니다.

### Vercel의 캐싱 전략

효율과 성능을 극대화한 전략이라고 생각하면 좋을 것 같습니다. 단지 높은 캐시 적중률을 위해서 전략적으로 구현된 것으로 보입니다.

| Region Code | Region Name    | Reference Location      |
| ----------- | -------------- | ----------------------- |
| arn1        | eu-north-1     | Stockholm, Sweden       |
| bom1        | ap-south-1     | Mumbai, India           |
| cdg1        | eu-west-3      | Paris, France           |
| cle1        | us-east-2      | Cleveland, USA          |
| cpt1        | af-south-1     | Cape Town, South Africa |
| dub1        | eu-west-1      | Dublin, Ireland         |
| fra1        | eu-central-1   | Frankfurt, Germany      |
| gru1        | sa-east-1      | São Paulo, Brazil       |
| hkg1        | ap-east-1      | Hong Kong               |
| hnd1        | ap-northeast-1 | Tokyo, Japan            |
| iad1        | us-east-1      | Washington, D.C., USA   |
| icn1        | ap-northeast-2 | Seoul, South Korea      |
| kix1        | ap-northeast-3 | Osaka, Japan            |
| lhr1        | eu-west-2      | London, United Kingdom  |
| pdx1        | us-west-2      | Portland, USA           |
| sfo1        | us-west-1      | San Francisco, USA      |
| sin1        | ap-southeast-1 | Singapore               |
| syd1        | ap-southeast-2 | Sydney, Australia       |

> 대한민국도 존재하군요!!

### Points of Presence (Pops)란?

Pops는 18개의 컴퓨팅 지원 영역이에도 100개 이상 존재 Pops의 기능은 다음과 같습니다.

1. **Request Routing** : 가까운 엣지 및 서버로 라우팅하는 역할
2. **DDos 보호** : Pops는 DDos에 대한 최초 방어선 역할
3. **SSL 종료**:

- SSL/TLS 암호와 및 복호화를 Pop에서 처리한다.
- 원본 서버 부담 낮춤

# 결론

이번 9주차는 여러 사건 사고가 많았네요. 집중하게 하지 못하는 일들도 있었고 마음이 힘든 일도 있었지만 그럼에도 불구하고 목표만 생각하고 버틴 한 주였습니다. 현재 진행하고 있는 코드 레벨에서의 최적화 과제도 무사히 마무리 할 수 있으면 좋겠습니다.

> 모두 파이팅

# 출처

1. [AWS cloudfront 공식 문서](https://aws.amazon.com/ko/what-is/cdn/)
2. [cloudflare 공식 문서](https://www.cloudflare.com/ko-kr/learning/cdn/what-is-a-cdn/)
