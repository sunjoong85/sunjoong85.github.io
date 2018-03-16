---
title: Progressive Web App - 시작하기
layout: post
categories: PWA
comments : true
---

## PWA(Progressive Web App) 에 대한 소개
PWA는 보다 나은 웹 사용 경험을 제공합니다. 단어 그대로 점진적으로 적용하면 되고 무엇보다 별도의 설치가 없습니다. PWA는 빠르고, 연결 독립적이고 (오프라인 동작), 네이티브 스러우며, 안전합니다.  

구글에서는 PWA를 다음 세가지 특징으로 정의하고 있습니다.  

1. 신뢰할 수 있다 (Reliable)  
네트워크 환경이 느린 곳 심지어 오프라인 환경에서도 동작해야합니다. 중요한 자원(JS, HTML, CSS, Image 등)은 즉시 로드되어야 합니다. 이를 위해서 Server Push, Service Worker 를 사용합니다.

2. 빠르다 (Fast)  
화면 로딩에 3초 이상의 시간이 걸리면 53%의 사용자는 해당 사이트를 재방문 하지 않습니다. 버벅이는 스크롤링이 없어야 하고 애니매이션은 부드러워야 합니다. 유저 인터렉션은 즉각적이어야 합니다.

3. 네이티브 친화적이다 (Engaging)  
모바일 디바이스의 홈 스크린에 추가 되어야 합니다. 또한 몰입도 높은 풀 스크린 UI를 제공해야 합니다. 더불어 Push Notification 을 비롯하여 디바이스의 네이티브 API도 활용 가능해야합니다.

PWA 는 보다 나은 웹을 개발하기 위한 컨셉으로 이해하면 됩니다. 모든 기능들을 적용할 필요는 없습니다. 현재 브라우저 또는 요구사항에 맞게 점진적으로 적용하면 됩니다. 중요한 점은 브라우저 벤더들이 더 나은 기능들을 제공할 것이고 우리는 그걸 이용하면 됩니다.

## PWA 사례
[Twitter PWA 적용 사례](https://developers.google.com/web/showcase/2017/twitter)에 소개된 Lite Twitter의 PWA 적용 사례입니다.
1. 낮은 데이터 소모량
 - 캐쉬 기능을 최대한 활용. Image Optimization을 활용하여 스크롤링 시 데이터 소모량 감소.
 - 앱 용량 600KB (안드로이드 앱의 경우 23.5MB, ios 는 더 큼)
 - 네트워크 속도가 느린 이머징 마켓에서 보다 빠르게 타임라인을 읽을 수 있도록 지원
2. Service Worker 를 활용하여 거의 즉시 모든 자원 로딩
 - 3G 환경에서 5초안에 최초 로드 완료. 이어서 이어지는 로드 시간은 거의 즉시(instant)
 - 최초 보여지는 화면에서 필요한 자원만 로딩(Code Splitting), 나중에 필요한 자원은 Service Worker 에 캐쉬. 화면 전환이 즉시 일어나는 효과.
   -> Twitter Lite를 사용하는 약 80%의 유저가 2G 또는 3G 네트워크를 사용 중. 유저가 빠르게 Twit을 작성하게 하는 것이 목표.
 - PRPL(Push, Render, Pre-cache and Lazy-load) 패턴 활용 -> 로그인 후 로드 시간 30% 빨라짐, 반응시간이 50% 빨라짐을 경험
 - 순차적으로 Service Worker를 확대 적용하고 있으며 오프라인에서도 동작하도록 개발. CSS, Image 같은 정적 자원(static resource)는 캐시하여 재활용.
 - 최근에는 [Application Shell](https://developers.google.com/web/fundamentals/architecture/app-shell)을 적용하였고 그 결과 네트워크 환경에 상관없이 3초 안에 앱이 부팅될 수 있도록 함.

## PWA 체크 리스트

PWA 체크 리스트 [Google PWA Checklist](https://developers.google.com/web/progressive-web-apps/checklist) 에서 확인 가능합니다.

### 측정
측정이 가능해야 개선할 수 있습니다.[Chrome Lighthouse](https://developers.google.com/web/tools/lighthouse/)는 PWA 수준을 측정해 줍니다.

### 기본 원칙
1. 사이트는 HTTPs를 통해 제공되어야 합니다.  
2. 테블릿 또는 모바일 다비이스에서 반응형(responsive) 디자인을 제공해야 합니다.  
3. 모든 페이지(혹은 URL)는 오프라인에서 동작해야 합니다.  
4. 홈 스크린에 추가되어야 합니다.  
5. 3G 환경에서도 빠르게 로드되어야 합니다.  
6. 크로스 브라우저(cross-browser)에서 동작해야 합니다.
7. 화면 전환은 즉각적이며 네트워크 지연을 느낄 수 없어야 합니다.  
8. 각각의 화면은 URL로 표현되어야 합니다.  

## 앞으로 다룰 내용
- App Shell Architecture 를 이용해서 App 과 유사하게 동작하도록 합니다.
- HTTPs 를 통해 보안을 강화하고 H2(HTTP/2) 를 통해 네트워크 활용도를 높입니다.
- Service Worker & Server Push 를 이용하면 로딩 퍼포먼스를 높일 수 있습니다. 오프라인에도 동작할 수 있습니다.
- 홈 스크린에 앱을 추가하고 풀 스크린을 제공합니다.
- 네이티브 디바이스 API 를 활용합니다.
- Push Notification 기능을 사용합니다.
- 웹 접근성을 제공하고 Responsive 디자인을 적용합니다.

## 참고 자료
[Google Progressive Web App](https://developers.google.com/web/progressive-web-apps/)  
[Getting started with Progressive Web Apps - Addy Osmani](https://addyosmani.com/blog/getting-started-with-progressive-web-apps/)


{% include disqus.html %}
