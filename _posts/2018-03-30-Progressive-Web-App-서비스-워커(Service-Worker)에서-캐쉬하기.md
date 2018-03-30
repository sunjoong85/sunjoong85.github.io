---
title: Progressive Web App - 서비스 워커(Service Worker)에서 캐쉬하기
layout: post
categories: PWA
---

# 서비스 워커(Service Worker)
서비스 워커는 웹 어플리케이션 백그라운드에서 동작합니다. 즉 클라이언트와 서버 사이의 미들웨어라고 보면 됩니다. 자바스크립트로 작성되고 별도의 설치가 필요 없습니다. 주요 기능은 Caching, Push Notification, Background sync가 있습니다.

서비스워커의 캐쉬 기능을 사용하면 서버에 불필요한 요청을 보낼 필요가 없습니다.

HTTP2의 서버 푸시(Server Push)를 사용하면 index.html을 전달 받을 때(최초의 요청) 웹 어플리케이션에서 필요한 자원(JS, CSS, Image 등)을 함께 보낼 수 있습니다. Connection을 여러번 맺는 라운드 트립 비용이 발생하지 않기 때문에 브라우저는 상당히 빠르게 화면 렌더링을 시작할 수 있습니다. 단, 모든 요청에 대해서 서버 푸시를 한다면 서버 자원 낭비가 됩니다. 그래서 서버 푸시가 서비스 워커의 캐시 가능을 만나면 매우 강력해 집니다.

# 브라우저에도 자체 캐쉬가 가능하지 않나요?
캐쉬만 본다면 사실 브라우저가 제공하는 캐쉬와 별 차이가 없습니다. 불필요한 구현이 됩니다.

하지만 서비스 워커는 모든 요청(request)/응답(response)를 핸들링할 수 있습니다. 즉 요청과 응답에 대한 제어를 할 수 있습니다. 그럼 다음과 같은 장점이 있습니다.

1. 캐쉬가 되어야하는 자원과 그렇지 않은 자원을 분리할 수 있다.
2. JS/CSS 뿐 아니라 Image, HTML 도 캐쉬할 수 있다.

서비스워커를 사용하면 웹 어플리케이션이 마치 네이티브 앱 처럼 오프라인에서 동작할 수 있게 됩니다. 이 부분은 다음에 다시 다루겠습니다.

# 서비스워커 구현
서비스 워커 HTTPs에서만 동작합니다. 로컬에서 HTTPs key를 발급받고 테스트하는 방법은 하단의 [이슈해결](#이슈해결)을 참조합니다.

## 등록
브라우저에서 서비스 워커 객체를 활성화 하려면 먼저 serviceWorker.js파일을 등록해야합니다. serviceWorker.js에는 서비스 워커가 무엇을 캐쉬해야하는지 알려줍니다. serviceWorker.js를 작성하는 방법은 뒤에서 다루겠습니다. 우선 다음과 같이 index.html에 script를 추가합니다.
```
//index.html

if ('serviceWorker' in navigator) {
    window.addEventListener('load', function() {

        navigator.serviceWorker.register('/serviceWorker.js').then(function(registration) {
            // Registration was successful

            console.log('ServiceWorker registration successful with scope: ', registration.scope);

        }).catch(function(err) {
            // registration failed :(
            console.log('ServiceWorker registration failed: ', err);
        });
    });
}
```

serviceWorker.js는 여러개 있을 수 있습니다. 중요한 것은 캐쉬하려는 자원이 있는 곳에 serviceWorker.js를 추가해주어야 합니다. 현재 샘플은 context 루트(https://locahost:3000/)에 serviceWorker.js를 추가하였습니다. context 루트를 통해서 들어오는 모든 자원에 대해서 캐쉬할 수 있습니다.

```
폴더 구조

ProgressiveWebDemo
                  - app
                  - server.js
                  - serviceWorker.js
                  - index.html
```

## serviceWorker.js 구현
그럼 serviceWorker.js를 살펴보겠습니다. serviceWorker.js에서는 'install'과 'fetch'이벤트를 핸들링합니다.
### install 단계 (캐쉬할 항목 지정)
install 이벤트 핸들러에서는 캐쉬할 resource들의 경로를 추가해야 합니다.

```
var CACHE_NAME = 'my-site-cache-v1';

var urlsToCache = [
    '/',     //최초 로드되는 html을 캐시
    '/lib/jquery.js',  //캐시해야할 js, css등을 명시함
    '/lib/lodash.js',
    '/lib/myscript.js',
    '/lib/mystyle.css',
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

urlsToCache에 캐쉬할 경로들을 정의 하였습니다. 첫 번째 항목인 '/' 의 경우 '/' 최초 전달되는 index.html을 캐쉬합니다. 만약 네트워크 사용이 불가능한 오프라인 상태라도 이전에 캐쉬한 index.html 을 서비스워커로부터 전달 받으면 됩니다.  
이어서 어플리케이션의 구조 즉 인프라에 해당하는 JS, CSS들을 캐쉬하도록 합니다. 인프라에 해당하는 자원을 캐쉬하여 오프라인 모드에서도 앱이 동작되도록 합니다.

뒤에서 App Shell Model을 다룰 때 인프라에 해당하는 모듈과 컨텐츠에 해당하는 모듈을 어떻게 분리하여 캐쉬하는지 다루겠습니다. 인프라와 컨텐츠는 다음과 같이 정의하겠습니다.
1. 인프라 영역 : 앱 실행 동안 바뀌지 않는 부분. 레이아웃, 코어 영역
2. 컨텐츠 영역 : 매번 서버에서 새로 로드되어야 하는 부분

### fetch 단계 (자원을 요청하는 단계)
fetch 이벤트 핸들러에서 자원을 요청할 때 호출됩니다. 서비스 워커에 캐쉬된 자원을 리턴할지, 아니면 서버에 자원을 요청해야하는지 판단하는 로직을 구성합니다.

```
self.addEventListener('fetch', function(event) {

    console.log("fetch url " ,event.request);

    event.respondWith(
        caches.match(event.request)
        .then(function(response) {

                if (response) { //cache found don't hit to server
                    return response;
                }

                //request to server
                return fetch(event.request);
            }
        ).catch(function(){
            console.log("fetch error");
        })
    );
});
```

# 크롬 디버깅에서 확인
//작업중

# 서버의 최신상태 반영하기
서비스 워커 파일을 업데이트 합니다. 브라우저는 serviceWorker.js를 로드할 것이고 1바이트라도 차이가 난다면 자원들을 로드할 것입니다.

새로운 자원이 로드되면 다음과 같이 동작합니다.

1. 기존 서비스 워커를 실행하여 현재 페이지가 동작되도록 한다.
기존 서비스 워커가 실행되어야 오프라인 모드에서도 웹 어플리케이션이 정상 동작합니다.
2. 변경된 서비스 워커는 새로운 자원들을 로드하여 캐쉬한다.
브라우저를 다시 방문하게되면 새롭게 업데이트된 자원들을 사용하게 됩니다

그래서 serviceWorker.js가 변경되더라도 처음 화면 리프레시가 발생하더라도 변경사항이 반영되지 않습니다. 이후 재방문시에 변경된 화면을 확인할 수 있습니다.

## 이전 캐시 지우기
//작업중

# 이슈 해결
1. 로컬 SSL을 사용하는 경우 (Self Signed SSL)
인증되지 않은 SSL의 경우 서비스 워커가 생성되지 않습니다. (크롬)
맥 사용자는 [Bypass Chrome SSL/certificate blockades](http://hints.macworld.com/article.php?story=20140510112547107)를 참고하여 크롬을 실행하시면 됩니다.

2. SSL
로컬 테스트를 위해서 Self Signed SSL 키를 생성할 수도 있지만 가급적 인증된 SSL키를 발급받도록 합니다.
특정 브라우저에서는 미인증된 키의 경우 서비스 워커와 서버 푸시를 사용할 수 없습니다.   
https://ssl.comodo.com/free-ssl-certificate.php

# Reference

[HTTP/2 Server Push and Service Workers: The Perfect Partnership](https://24ways.org/2016/http2-server-push-and-service-workers/)  
[Service Workers](https://developers.google.com/web/fundamentals/primers/service-workers/?hl=ko)  
[Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)

{% include disqus.html %}
