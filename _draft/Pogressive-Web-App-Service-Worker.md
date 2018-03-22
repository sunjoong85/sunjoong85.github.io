---
title: Progressive Web App - Service Worker 와 Server Push
layout: post
categories: PWA
---

# ServerPush
HTTP 에서 자원 (JS, HTML, CSS)를 요청하게 되면 매 요청 마다 하나의 Connection을 맺는다.
HTTP2의 Push 를 사용하면 서버에서 필요한 자원을 한번에 전달할 수 있다. 즉 Request마다 Connection을 맺을 필요가 없으므로라운드 트립 비용이 감소한다.
그 결과 빠른 초기 로딩이 가능하다. (장점)

# 구현 샘플

# 크롬 네트워크 텝 H2 확인

# Server는 클라이언트의 상태를 모른다
서버는 클라이언트의 상태를 모른다. 이미 방문한 페이지여서 브라우저 캐쉬가 되어 있을수도 있다. 하지만 서버는 URL요청에 대해서 매번 자원을 푸시하게 된다.

이론에 의하면 브라우저는 HTTP2 Server Push 요청을 cancel할 수 있는 기능이 있지만 아직 이를 구현한 브라우저는 없다. (찾아보자.)
또한 요청을 취소한 시점에 이미 서버로부터 response가 오는 중일 수도 있다. 문제다.

서버푸쉬는 서비스워커와 함께하면 더 강력해진다.


# 서비스워커(Service Worker)
H2의 서버푸시가 서비스워커를 만나면 매우 강력해 집니다.

서비스워커는 웹 어플리케이션 백그라운드에서 동작합니다. 즉 클라이언트와 서버 사이의 미들웨어라고 보면 됩니다. 자바스크립트로 작성되고 별도의 설치가 필요 없습니다. 주요 기능은 Caching, Push Notification, Background sync가 있습니다.

즉 서비스워커의 캐쉬 기능을 사용하면 서버에 불필요한 요청을 보낼 필요가 없습니다.

# 브라우저에도 자체 캐쉬가 가능하지 않나요?
캐쉬만 본다면 사실 브라우저가 제공하는 캐쉬와 별 차이가 없습니다. 불필요한 구현이 됩니다.

하지만 ServiceWorker는 모든 요청(request)/응답(response)를 핸들링할 수 있습니다. 즉 요청과 응답에 대한 제어를 할 수 있습니다. 그럼 다음과 같은 장점이 있습니다.

1. 캐쉬가 되어야하는 자원과 그렇지 않은 자원을 분리할 수 있다.
2. JS/CSS 뿐 아니라 Image, HTML 도 캐쉬할 수 있다.

ServiceWorker를 사용하면 웹 어플리케이션이 마치 네이티브 앱 처럼 오프라인에서 동작할 수 있게 됩니다. 이 부분은 다음에 다시 다루겠습니다.

# 서비스워커 구현
ServiceWorker는 HTTPs에서만 동작합니다. 로컬에서 HTTPs key를 발급받고 테스트하는 방법은 하단의 [이슈해결](#이슈해결)을 참조합니다.

## 등록
서비스워커를 등록하는 방법에 대해서 다뤄보겠습니다. 서비스워커에 해당하는 JS파일을 다음과 같이 등록합니다.
```
if ('serviceWorker' in navigator) {
    window.addEventListener('load', function() {

        navigator.serviceWorker.register('/sw.js').then(function(registration) {
            // Registration was successful

            console.log('ServiceWorker registration successful with scope: ', registration.scope);

        }).catch(function(err) {
            // registration failed :(
            console.log('ServiceWorker registration failed: ', err);
        });
    });
}
```

sw.js는 여러개 있을 수 있습니다. 중요한 것은 캐쉬하려는 자원이 있는 곳에 sw.js를 추가해주어야 합니다.
현재 샘플은 context 루트에 sw.js를 추가하였습니다. context 루트를 통해서 들어오는 모든 자원에 대해서 캐쉬할 수 있습니다.

##sw.js 구현
### install 단계 (캐쉬할 항목 지정)
그럼 sw.js를 살펴보겠습니다. 첫 단계는 install 이벤트를 받아서 캐쉬할 resource 의 경로를 추가해야 합니다.

```
var CACHE_NAME = 'my-site-cache-v1';

var urlsToCache = [
    '/',
    '/lib/jquery.js',
    '/lib/lodash.js',
    '/lib/my_infra.js',
    '/lib/my_infra.css',
];

//url에 대해서 하나라도 캐쉬에 실패하면 에러
self.addEventListener('install', function(event) {
    // Perform install steps
    event.waitUntil(
        caches.open(CACHE_NAME)
        .then(function(cache) {
            console.log('Opened cache', cache);
            return cache.addAll(urlsToCache);
        }).catch(function(){
            console.log('install error')
        })
    );
});
```

urlsToCache에 캐쉬할 경로들을 정의 하였습니다. 첫 번째 항목인 '/' 의 경우 '/' 요청으로 전달되는 index.html을 캐쉬합니다. 오프라인이더라도 이전에 캐쉬한 index.html 을 서비스워커로부터 로드하면 됩니다. 이어서 어플리케이션의 구조 즉 인프라(infractructure)에 해당하는 JS, CSS들을 캐쉬하도록 합니다.

뒤에서 App Shell Model을 다룰 때 인프라에 해당하는 모듈과 컨텐츠에 해당하는 모듈을 어떻게 분리하여 캐쉬하는지 다루겠습니다. 인프라와 컨텐츠는 다음과 같이 정의하겠습니다.
1. 인프라 영역 : 앱 실행 동안 바뀌지 않는 부분. 레이아웃, 코어 영역
2. 컨텐츠 영역 : 매번 서버에서 새로 로드되어야 하는 부분

### fetch 단계 (자원을 요청하는 단계)
fetch 이벤트는 자원을 요청할 때 호출됩니다. 서비스워커에 캐쉬된 자원을 리턴할지, 아니면 서버에 자원을 요청해야하는지 판단하는 로직을 구성합니다.

```
self.addEventListener('fetch', function(event) {

    console.log("fetch url " ,event.request);

    event.respondWith(
        caches.match(event.request)
        .then(function(response) {
                // Cache hit - return response
                console.log("response " , response);
                if (response) { //cache found don't hit to server
                    return response;
                }
                console.log("is not match ??? fetch again : " , event);
                return fetch(event.request);
            }
        ).catch(function(){
            console.log("fetch error");
        })
    );
});
```
# 서버의 최신상태 반영하기


# 이슈 해결
1. 로컬 테스트를 위한 SSL key 발급 방법

2. 크롬에서 동작시키기 위한 설정
로컬에서 발급받은 SSL을 사용하는 경우 서비스워커가 실행되지 않습니다.
[self signed certificate을 크롬에서 사용하기](https://deanhume.com/home/blogpost/testing-service-workers-locally-with-self-signed-certificates/10155) 를 참고합니다.


# Reference

[HTTP2 Server Push and Service Workers](https://24ways.org/2016/http2-server-push-and-service-workers/)  
[Service Workers](https://developers.google.com/web/fundamentals/primers/service-workers/?hl=ko)  
[Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
