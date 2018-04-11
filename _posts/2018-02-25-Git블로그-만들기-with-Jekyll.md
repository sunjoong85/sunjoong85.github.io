---
title: Jekyll로 Github 블로그 만들기
layout: post
categories: Github Page
comments : true
---
# Jekyll 로 Github 블로그 만들기
[Jekyll](https://jekyllrb.com/)은 마크다운을 이용하여 블로그를 생성해주는 도구입니다. Jekyll을 이용하면 Git Page와 연결하여 멋진 블로그를 만들 수 있습니다.

## Jekyll 시작하기
커맨드창에서 다음과 같이 실행합니다.
```
# jekyll 설치
$ gem install jekyll bundler

# my-awesome-site 프로젝트 만들기
$ jekyll new my-awesome-site

$ cd my-awesome-site

# 빌드 및 로컬 서버 실행

~/my-awesome-site $ bundle exec jekyll serve

# => 다음 url 에서 블로그의 모습을 확인할 수 있습니다. http://localhost:4000
```

[참고] ruby 버전이 낮은 경우 Jekyll설치가 안 됩니다. 다음 명령어로 최신 버전 업데이트를 합니다.
```
curl -sSL https://get.rvm.io | bash -s stable
rvm list known
rvm install ruby-2.4.1
rvm use ruby-2.4.1 --default
```

## Github repository 연결하기
Github에서 repository를 생성하고 my-awesome-site를 push 합니다.
```
git init
git add .
git commit -m "first commit"
git remote add origin https://github.com/my-git-id/my-awesome-site.git
git push -u origin master
```

## GitPage 설정
setting 에서 git page 설정을 추가하면 내 블로그 url이 생성됩니다.
(저는 jekyll-test 로 repository를 생성했습니다.)

![Git Setting에서 page 설정]({{ "/_image/git블로그만들기/gitpage_setting.png"}})

다시 my-awesome-site 폴더로 돌아가서 `_config.yml` 파일을 열어 봅니다.
`baseurl`과 `url` 속성을 수정해줍니다.

{% highlight yml %}
baseurl: "/jekyll-test"
url: "https://sunjoong85.github.io"
{% endhighlight %}

이제 나의 md파일을 멋진 블로그 포스트로 바꿀 수 있습니다.

## 댓글 기능 추가하기
[disqus](https://disqus.com/)를 내 블로그에 추가할 수 있습니다. 다음 블로그를 참고하면 도움을 받을 수 있습니다. [Jekyll 블로그에 Disqus 붙이기](https://cjh5414.github.io/Disqus/)

## 검색 엔진에 노출시키기
[지킬 블로그 구글 검색 가능하게 하는 방법](https://wayhome25.github.io/etc/2017/02/20/google-search-sitemap-jekyll/)을 참고합니다.
{% include disqus.html %}
