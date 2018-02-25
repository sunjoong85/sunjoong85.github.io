---
title: Jekyll로 Github 블로그 만들기
layout: post
categories: Github Page
comments : true
---
# Jekyll 로 Github 블로그 만들기
[Jekyll](https://jekyllrb.com/)은 마크다운을 이용하여 블로그를 생성해주는 도구입니다. Jekyll을 이용하면 Git Page와 연결하여 자신만의 블로그를 만들 수 있습니다.

## Jekyll 시작하기
커맨드창에서 다음과 같이 실행합니다.
```
$ gem install jekyll bundler

$ jekyll new my-awesome-site

$ cd my-awesome-site

$ ~/my-awesome-site $ bundle exec jekyll serve

# => 다음 url 에서 블로그의 모습을 확인할 수 있습니다. http://localhost:4000
```

[참고] ruby 버전이 낮은 경우 Jekyll설치가 안 됩니다. 다음 명령어로 최신 버전 업데이트를 합니다.
```
curl -sSL https://get.rvm.io | bash -s stable
rvm list known
rvm install ruby-2.4.1
vm use ruby-2.4.1 --default
```

## Github repository 연결하기
Github 에서 만든 repository에 위에서 만든 my-awesome-site를 push 합니다.
```
git init
git add .
git commit -m "first commit"
git remote add origin https://github.com/my-git-id/my-awesome-site.git
git push -u origin master
```

## GitPage 설정
setting 에서 git page 설정을 추가하면 완성!

![Git Setting에서 page 설정]({{ "/assets/gitpage_setting.png"}})


## 댓글 기능 추가하기
[disqus](https://disqus.com/)를 내 블로그에 추가할 수 있습니다. 다음 블로그를 참고하면 도움을 받을 수 있습니다. [Jekyll 블로그에 Disqus 붙이기](https://cjh5414.github.io/Disqus/)

{% include disqus.html %}
