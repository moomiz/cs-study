# prepare_frontend_interview

## Computer Science

## 목차

- [CORS 🔥](#CORS)

  - CORS가 뭔가요?
  - CORS를 겪고 직접 해결해 본 경험이 있으면 말해주세요

- [CDN 🔥](#CDN)

  - CDN이란 뭔가요?

- [쿠키 세션 🔥](#쿠키-세션)

  - 쿠키, 세션을 왜 쓰나요?
  - 쿠키가 뭔가요?
  - 세션이 뭔가요?
  - 쿠키와 세션의 차이는 어떤 점이 있을까요?
  - JWT을 아나요?
  - JWT를 웹 스토리지에 저장해야 한다면 어디다 저장하시겠나요? 이유는요?

## CORS

### CORS가 뭔가요?

CORS(Cross-Origin Resource Sharing)

웹에서 다른 도메인 혹은 포트 등에 대한 접근을 허용하는 정책.
보통 다른 도메인에서 리소스를 요청 할 때 발생한다.

정확히는 다른 오리진에서 리소스를 요청 할 때 발생한다.

> 오리진 vs 리소스?
>
> 오리진은 프로토콜(http, https), 호스트(www.example.com), 포트(443)를 포함한 전체 주소를 의미한다.
>
> 예: https://www.example.com:443
>
> 리소스는 오리진 이후의 경로와 파일을 의미한다.
> 예: /path/to/resource.html
>
> 전체 URL 예시: https://www.example.com:443/path/to/resource.html

브라우저에서만 나타나기 때문에 서버와 서버간의 통신에서는 발생하지 않는다. 따라서 외부 API를 사용할 때 프론트에서 직접 사용하는 것이 아닌 서버에서 처리하는 이유가 이것 때문일 수 있다.

앱에서는 CORS 정책이 직접적으로 적용되지 않지만, 앱 내 웹뷰를 사용하거나
네이티브 HTTP 요청을 할 때 서버 측에서 CORS 설정이 필요할 수 있다.

CORS 통신을 하게 되면 네트워크 탭에서 OPTIONS 요청이 추가로 발생한다.

이 요청은 실제 데이터를 전송하는 요청이 아니고 실제 요청 전에 서버에 안전하게 요청을 보낼 수 있는지 확인하는 **"프리플라이트"** 요청이다.

보통 프론트에서는 헤더에 `WithCredentials: true`와 함께 리퀘스트를 보내고, 백엔드에서는 `Access-Control-Request-Method`와 `Access-Control-Request-Headers`를 포함하여 리스폰스를 보낸다.

### CORS를 겪고 직접 해결해 본 경험이 있으면 말해주세요

개발환경과 실제 배포환경에서 해결하는 방식으로 나누어보자.

1. 개발환경

   package.json에서 프록시 설정을 변경하여 해결할 수 있다.

   ```json
   {
     "name": "your-project-name",
     "version": "1.0.0",
     "private": true,
     "dependencies": {
       // ... 다른 의존성들 ...
     },
     "scripts": {
       // ... 스크립트들 ...
     },
     "proxy": "http://api.example.com"
   }
   ```

   위와 같이 설정하면 api요청이 프록시를 통해서 전달된다. 하지만 위 방법은 개발 환경에서만 동작을 하기 때문에 실 배포에서는 사용할 수 없는 방법.

2. 실제 배포환경

   1. 서버에서 허용해주는 방법
   2. 프록시 서버를 사용하는 방법

   위 두가지 방법으로 나뉜다.

   1. 서버에서 허용해주는 방법

      서버에서 허용해주는 방법은 서버에서 허용할 오리진을 명시해주는 방법이다.
      Nest에서는 다음과 같이 서버에서 허용할 수 있다.

      ```typescript
      // main.ts
      import { NestFactory } from "@nestjs/core";
      import { AppModule } from "./app.module";

      app.enableCors({
        origin: ["http://localhost:3000"],
      });

      bootstrap();
      ```

   2. 프록시 서버를 사용하는 방법

      이 방법은 서버와 서버간에는 CORS가 발생하지 않기 때문에 중간에 프록시서버를 두고 이를 통해 요청을 넣는 방법이다.

      Nginx를 사용하여 프록시 서버를 구축할 수 있다. 또는 헤로쿠에서도 프록시 서버를 구축할 수 있다. Next.js에서도 서버 api를 사용할 수 있기 때문에 프록시 역할을 하는 서버를 구축할 수 있다.

## CDN

### `CDN이란 뭔가요?`

**CDN(Content Delivery Network): 콘텐츠 전송 네트워크**

서버의 물리적인 위치와 사용자가 접속하는 위치는 동일할 수 도, 다를 수도 있다. 당연히 멀 경우에는 네트워크 지연이 발생 할 수 있기 때문에 전송 속도가 늦을 수 있다.

> 왜 물리적 위치가 멀 경우에는 네트워크 지연이 발생할 수 있을까?
>
> 1. 데이터는 전기나 빛의 형태로 전송되기 때문에 물리적인 거리가 영향을 준다.
> 2. 데이터는 여러 라우터, 스위치, 중계기를 통과하기 때문에 장비마다 데이터를 처리하고 전달하는 시간이 필요하다.
> 3. 긴 거리의 네트워크는 더 많은 사용자와 트래픽을 처리한다.
> 4. 긴 거리의 통신에는 추가적인 프로토콜 레이어가 필요할 수 있다.

CDN은 이런 문제를 해결하기 위해 사용된다.

1. 페이지 로드 시간 단축

2. 대역폭 비용 절감

   캐싱 및 기타 최적화를 통해 CDN은 오리진 서버가 제공해야하는 데이터 양을 줄여 비용을 절감할 수 있다.

3. 콘텐츠 가용성 재고

   한번에 많은 방문자가 방문하는 경우 오리진 서버에 부하가 걸리기 때문에 중간에 CDN 서버를 두어 부하를 분산할 수 있다. 하나이상의 CDN 서버가 오프라인으로 전환되면 다른 운영서버가 해당 서버를 대체하여 서비스가 중단되지 않도록 할 수 있다.

4. 웹 사이트 보안 강화
   DDoS 공격을 중간 서버 간에 분산하여 오리진 서버에 가해지는 부담을 줄인다.

CDN이 동작하는 원리는 다음과 같다.

1. 사용자가 최초로 사이트에서 정적 웹 콘텐츠를 요청한다.
2. 요청이 웹 애플리케이션 서버로 도달한다. 서버는 사용자에게 응답을 보내는 동시에 지리적으로 가까운 **CDN POP(접속 지점, Points of Presence)**에 응답 복사본을 보낸다.
3. CDN POP 서버는 복사본을 캐싱된 파일로 저장한다.
4. 다음에 해당 방문자 또는 해당 위치에 있는 다른 방문자가 동일한 요청을 하면, 오리진 서버가 아닌 캐싱서버에서 응답을 보낸다.

사용할 수 있는 예시는 다음과 같다.

- 넷플릭스, 유튜브와 같은 고용량 고화질 동영상 플랫폼
- 게임의 업데이트 파일
- 슬랙의 메시지나 파일을 캐싱, 웹소켓연결을 위한 엣지 서버로 사용

AWS에서는 Amazon CloudFront라는 CND 서비스를 제공한다.

## 쿠키 세션

### 쿠키, 세션을 왜 쓰나요?

브라우저에 사용자의 **정보를 저장**하기 위해서

기본적으로 브라우저에 있는 데이터들은 서버에 전달되어 외부 DB에 백업 되어있지 않는 한, 브라우저를 닫으면 사라지게 된다. 따라서 브라우저 내에 정보를 저장하기 위해 사용하는 것이 쿠키이다.

세션의 경우에는 서버에 저장되는 것,

### 쿠키가 뭔가요?

쿠키는 서버가 사용자의 웹 브라우저에 전송하는 작은 데이터 조각을 의미한다.

브라우저는 그 데이터 조각을 저장해 놓았다가, 동일한 서버에 재 요청시 쿠키를 함께 전송한다.

물론 서버뿐만 아니라 클라이언트에서도 쿠키를 생성해 브라우저에 저장하는 것이 가능하다.

#### 쿠키 생성 방법

1. Set-Cookie
   서버에서 응답을 보낼 때 응답의 헤더에 Set-Cookie를 포함하여 응답을 보내면 클라이언트에서는 해당 요청을 받고 쿠키를 생성한다.

   이 때 쿠키는 서버에서 전송한 쿠키의 속성에 따라 다양하게 설정이 된다.
   origin, path, expires, secure, httpOnly 등이 있다.

   > origin: 쿠키를 생성한 도메인
   >
   > path: 쿠키가 저장될 경로
   >
   > expires: 쿠키의 만료 시간
   >
   > secure: 쿠키가 전송될 때 HTTPS 프로토콜만 사용
   >
   > httpOnly: 브라우저의 스크립트로 쿠키에 접근하는 것을 방지
   >
   > domain: 쿠키가 전송될 도메인

   현재 브라우저는 서드 파티 쿠키를 제한하는 정책을 가지고 있다.
   서드 파티 쿠키란 다른 도메인의 쿠키를 말하는데, 이를 제한하는 이유는 보안상의 이유 때문이다.

2. 브라우저의 쿠키 저장소에 직접 저장
   ```javascript
   document.cookie =
     "name=value; expires=Thu, 18 Dec 2023 12:00:00 UTC; path=/";
   ```
   다만 이때는 HttpOnly 속성이 적용되지 않는다.

#### 쿠키의 특징

- 이름, 값, 만료일, 경로(path) 로 구성되어 있음
- 클라이언트에 300개 제한
- 하나의 도메인 당 20개의 쿠키 제한
- 하나의 쿠키는 4KB까지 저장 가능

#### 쿠키의 동작 순서

1. 서버에서 Set-Cookie 헤더를 통해서 브라우저에 쿠키를 설정
2. 브라우저는 쿠키 저장소에 쿠키를 저장
3. 동일 사이트 방문 혹은 동일 도메인 요청 시 브라우저는 요청의 헤더에 쿠키를 함께 담아 전송

#### 쿠키 사용 예시

- JWT의 리프레시 토큰과 같이 서버에 주기적으로 보내야하는 요청
- I18n의 국제화 설정
- 팝업 창의 오늘 보지 않기 설정
- 자동 로그인 같은 설정

#### 쿠키의 약점

httpOnly를 설정하지 않은 경우 스크립트를 통해 쿠키를 탈취할 수 있음.
Secure를 설정하지 않으면 Http로도 흘러들어갈 수 있기 때문에 통신 과정에서 통신을 탈취당하면 헤더에 담긴 정보가 고스란히 노출될 수 있다.
여러모로 XSS, CSRF 공격에 취약하다.

### 세션이 뭔가요?

세션은 클라이언트가 웹 서버에 연결된 순간부터 웹 브라우저를 닫아 서버와의 HTTP 통신을 끝낼 때 까지의 기간을 의미한다.

하지만 보통 세션의 경우에는 서버에 세션에 대한 정보를 저장하고, 세션 쿠키를 클라이언트에게 주어 서버가 클라이언트를 식별할 수 있도록 하는 방식을 의미한다.

기본적으로 백엔드에서 세션을 저장할 때는 메모리에 저장하지만, 메모리에 저장하는 것은 서버의 메모리 용량에 따라 한계가 있기 때문에 데이터베이스에 저장하는 경우가 많다.

이 때 사용되는 것이 Redis와 같은 캐시 서버이다. 이 때는 브라우저 연결 종료시 자동으로 삭제되지는 않고, 설정된 만료 시간까지 레디스에 유지된다.

예를 들어 로그인 상태를 유지하는 경우 세션에 사용자의 정보를 저장하고, 세션 쿠키를 통해 서버에 전달하는 방식으로 사용된다. 이 때 로그아웃 요청을 하면 세션에 저장된 정보를 삭제하고 세션 쿠키를 삭제하는 방식으로 사용된다.

레디스를 사용한다면 로그아웃 요청이 들어올때 세션에 저장된 정보를 삭제하게 된다.

### 쿠키와 세션의 차이는 어떤 점이 있을까요?

쿠키는 클라이언트, 세션은 서버

쿠키는 보안상 취약점이 있을 수 있고 세션은 서버에서 처리하기 때문에 비교적 안전하다.

쿠키는 클라이언트에 저장되어 서버 요청시 속도가 빠르고 세션은 정보가 서버에 있기 때문에 느리다.


### JWT을 아나요?

### JWT를 웹 스토리지에 저장해야 한다면 어디다 저장하시겠나요? 이유는요?
