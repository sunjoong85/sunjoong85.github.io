title: Progressive Web App 시작하기
layout: post
categories: PWA
---

## PWA(Progressive Web App) 에 대한 소개
PWA는 보다 나은 웹 사용 경험을 제공합니다. 다음 세 가지 주요 특징이 있습니다.

1. 신뢰할 수 있다 (Reliable)  
네트워크 환경이 느린 곳 심지어 오프라인 환경에서도 동작해야합니다. 중요한 자원(JS, HTML, CSS, Image 등)은 즉시 로드되어야 합니다. 이를 위해서 Server Push, Service Worker 를 사용합니다.

2. 빠르다 (Fast)  
화면 로딩에 3초 이상의 시간이 걸리면 53%의 사용자는 해당 사이트를 재방문 하지 않습니다. 버벅이는 스크롤링이 없어야 하고 애니매이션은 부드러워야 합니다. 유저 인터렉션은 즉각적이어야 합니다.

3. 네이티브 친화적이다 (Engaging)  
모바일 디바이스의 홈 스크린에 추가 되어야 합니다. 또한 몰입도 높은 풀 스크린 UI를 제공해야 합니다. 더불어 Push Notification 을 비롯하여 디바이스의 네이티브 API도 활용 가능해야합니다.

PWA 는 보다 나은 웹을 개발하기 위한 컨셉으로 이해하면 됩니다. 모든 기능들을 적용할 필요는 없습니다. 현재 브라우저 또는 요구사항에 맞게 점진적으로 적용하면 됩니다.

## PWA 체크 리스트

PWA 체크 리스트 [Google PWA Checklist](https://developers.google.com/web/progressive-web-apps/checklist) 에서 확인 가능합니다.

### 기본 원칙
1. 사이트는 HTTPs를 통해 제공되어야 합니다.
2. 테블릿 또는 모바일 다비이스에서 반응형(responsive) 디자인을 제공해야 합니다.  
3. 오프라인에서 동작해야 합니다.
4. 홈 스크린에 추가되어야 합니다.
5. 3G 환경에서도 빠르게 로드되어야 합니다.
6. 크로스 브라우저(cross-browser)에서 동작해야 합니다.
7. 화면 전환은 즉각적이며 네트워크 지연을 느낄 수 없어야 합니다.
8. 각각의 화면은 URL로 표현되어야 합니다.


웹 사이트가 progressive 한지 여부는 [Chrome Light House](https://developers.google.com/web/tools/lighthouse/) 를 통해서 측정 가능합니다. 측정이 가능해야 개선할 수 있습니다.

## 참고 자료
[Google Progressive Web App](https://developers.google.com/web/progressive-web-apps/)
