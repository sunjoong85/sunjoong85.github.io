---
title: Progressive Web App - HTTP2 서버 푸시(Server Push)
layout: post
categories: PWA, HTTP2
comments : true
---
# 서버 푸시(Server Push)
HTTP2의 서버 푸시를 활용하면 자원(JS, CSS, Image 등)을 하나의 TCP connection으로 클라이언트에 전달할 수 있습니다. 네트워크 활용도를 높여주고 빠른 로딩을 가능하게 합니다.

# 빠른 로딩이 필요한 이유
다음 그림은 웹 페이지를 요청할 때 브라우저에서 일어나는 일입니다.

![로딩 비용 - Alkamai O'Reilly fluent]({{ "/assets/server_push/bottleneck.png"}})
HTML, JavaScript, CSS를 빠르게 로딩할수록 브라우저는 더 빨리 렌더링 작업을 합니다.
> [참고] 이 외에도 로딩 성능을 빠르게 하기 위해 PRPL 패턴, 압축 등 어려 기술들이 있습니다.

자원을 빠르게 로딩할 수 있다는 것은 브라우저가 더 빠른 시간에 렌더링 프로세스를 시작할 수 있음을 의미합니다. 유저는 웹 페이지와 더 빠르게 인터렉션할 수 있게 됩니다.

> 더불어 개발도상국 국가에서는 소득 대비 네트워크 비용이 상대적으로 비싸고 속도가 느립니다. 이러한 국가에서는 더 적은 네트워크 트래픽을 사용하도록 해야합니다.

# 서버 푸시와 서비스 워커
서비스 워커와 서버 푸시가 함께하면 더 강력해집니다.

`서버는 클라이언트의 상태를 모릅니다.` URL요청마다 이미 전달한 자원을 서버는 재전송할 수 있습니다. (사실 그렇게 됩니다. 클라이언트 즉 브라우저가 어떤 데이터를 캐시하고 있는지 모르니까요.)
이 경우 당연히 불필요한 네트워크 트래픽을 증가시키게 되고 서버 푸시를 사용하는 의미가 사라집니다.  


그렇기 때문에 우리의 웹 어플리케이션은 서비스 워커를 통해 캐시된 자원은 리턴받아야 합니다. 그리하여 서버에 중복된 요청을 보내지 않도록 합니다.
사실 이 모든 작업은 브라우저가 수행하기 때문에 유저는 별다른 구현을 하지 않아도 됩니다. 서비스 워커만 잘 작성하면 됩니다.

# 구현
Node를 이용하여 샘플을 구현하였습니다. HTTP2를 사용하기 위해서는 SSL 인증서가 필요합니다.

로컬에서도 생성할 수 있지만 인증되지 않은 SSL인 경우 크롬 브라우저에서 서버 푸시를 확인할 수 없습니다.(크롬에서 서버 푸시된 자원을 거절 합니다.)
[Let's Encrypt](https://letsencrypt.org/getting-started/])를 이용해서 인증서를 무료로 발급받을 수 있습니다. (도메인 필요)
`파이어 폭스에서는 디버깅 창에서 서버 푸시 결과를 확인할 수 있습니다.`

## 코드
http2 모듈을 이용한 코드입니다. 인증서 부분만 상황에 맞게 적용하면 됩니다. 가급적 web framework없이 node api만을 사용하였습니다.

```JavaScript
const http2 = require('http2');
const fs = require('fs');
const mime = require('mime');  //npm install mime  
const path = require('path');

const options = {
  //Local에서 생성한 인증서
    key : fs.readFileSync(path.join(__dirname, '/server/server.key')),
    cert: fs.readFileSync(path.join(__dirname, '/server/server.crt')),


 // 신뢰할 수 있는 인증서. let's encrypt를 사용했습니다.
 /*
    key : fs.readFileSync(path.join(__dirname, '/certs/privkey.pem')),
    cert: fs.readFileSync(path.join(__dirname, '/certs/fullchain.pem')),
    ca: fs.readFileSync(path.join(__dirname, '/certs/chain.pem')),
 */
}

const h2server = http2.createSecureServer(options);
h2server.listen(3001);

// file push
function push(stream, reqPath) {
    const file = getFile(reqPath);

    stream.pushStream({ ':path' : reqPath }, (err, pushStream) => {
        pushStream.respondWithFD(file.fd, file.headers);

        //브라우저가 push된 자원을 거절하는 경우 TCP 에러가 납니다. (또는 pending 현상) 확실한 해결법은 찾지 못 했습니다.
        pushStream.once('error', (error) => {
            console.log('Push Error : ' + error.code);
        })
        pushStream.once('frameError', () => {
            console.log('Push Frame Error');
        })

        pushStream.once('close', () => {
            console.log("#pushStream close fd : " + file.fd + " " + file.path);
            fs.closeSync(file.fd)
        });
    });

}

h2server.on('stream', (stream, headers) => {
    const reqPath = headers[':path'] === '/' ? 'index.html' : headers[':path'];
    const file = getFile(reqPath);

    if(reqPath === 'index.html') {
        // index.html을 전달하면서 필요한 자원들을 push합니다. 서버는 무엇을 클라이언트에 전달할지 알고 있어야합니다.
        // push부분을 비동기로 처리하면 더 빠르게 응답을 보낼 수 있습니다. 현재는 sync로 구현
        push(stream, '/app/lib/jquery.js');
        push(stream, '/app/lib/lodash.js');
        push(stream, '/app/lib/bootstrap.min.css');
        push(stream, '/app/lib/bootstrap.min.js');
    } else {
        console.log("another request");
    }

    if(!file) {
        stream.respond({':status' : 404});
        stream.end();
        return;
    }

    stream.respondWithFD(file.fd, file.headers);

    stream.once('close', () => {
        console.log("#stream close fd : " + file.fd + " " + file.path);
        fs.closeSync(file.fd)
    });
});

function getFile(reqPath) {
    try{
        const filePath = path.join(__dirname,  reqPath);

        const fd = fs.openSync(filePath, fs.constants.O_RDONLY);
        const contentType = mime.getType(filePath);
        const stat = fs.statSync(filePath);
        return {
            fd : fd,
            path : reqPath,
            headers : {
                'content-type' : contentType,
                'content-length' : stat.size, //optional
                'last-modified' : stat.mtime.toUTCString(), //optional
            }
        }
    } catch(e) {
        console.log("error. cannot read file");
        return null;
    }
}
```

# 크롬에서 서버 푸시 확인
![server push 확인]({{ "/assets/server_push/server_push.png"}})

# HTTP2 프로토콜도 확인 가능
`chrome://net-internals`을 사용하면 HTTP2 프로토콜을 확인할 수 있습니다.
![HTTP2 프로토콜 확인]({{ "/assets/server_push/chrome_netinternal.png"}})

# 적용 예
서버는 클라이언트가 필요한 자원을 알고 있습니다. 특히 SPA(Single Page Application)에서 Layout등 인프라에 해당하는 JS, HTML, CSS를 미리 클라이언트에 전달할 수 있습니다. 그 외 특정 시점에 필요한 자원들 또는 변경이 많은 부분은 코드 분할을 이용하여 Lazy Loading 처리하여도 됩니다.

# 정리
서비스 워커와 서버 푸시를 활용하면 `즉시 모든 자원을 로딩(Instantly load everything)`할 수 있습니다. 우리의 웹 어플리케이션이 더욱 빨라집니다.

# Reference
O'Reilly Fluent 2017 Web Performance, Alkamai  
[http2 server push](https://www.smashingmagazine.com/2017/04/guide-http2-server-push/)

{% include disqus.html %}
