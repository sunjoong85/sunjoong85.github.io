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

# Reference
https://www.smashingmagazine.com/2017/04/guide-http2-server-push/
