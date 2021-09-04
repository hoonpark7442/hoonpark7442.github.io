---
layout: post
title: (번역) Performance profiling for your Rails App with rack-mini-profiler
author: sisipark
tags: [Ruby, Ruby on Rails, Rails Developer, 레일즈 개발자, 루비, 루비온레일즈, 성능개선, performance]
excerpt_separator: <!--more-->
---
> 해당 포스트는 [Performance profiling for your Rails App with rack-mini-profiler](https://reinteractive.com/posts/378-performance-profiling-for-your-rails-app-with-rack-mini-profiler)를 읽고 해석(+의역)한 글입니다.

Rack Mini Profiler는 데이터베이스 이슈, 메모리 이슈, gem/lib에 의한 시간소요 등의 문제를 쉽게 탐지해줍니다.

<!--more-->

이 툴은 프로덕션 환경에서 돌아가도록 설계되어있지만 개발환경에서 사용해도 무방합니다. 하지만 그러기 위해선 몇가지 tweak이 필요합니다. 

이 블로그에서 일반적인 Rails 성능 이슈를 탐지하기 위해 어떻게 이 툴을 사용할지에 대해 설명하겠습니다.

<br>
### rack-mini-profiler 설치

아래의 gem을 설치합니다.

```ruby
gem 'rack-mini-profiler'
gem 'flamegraph'
gem 'stackprof'
gem 'memory_profiler'
```

해당 젬들을 설치 후 서버를 키고 브라우저로 가보면 왼쪽 상단에 스피드배지가 보일겁니다. 
다음으로는 어플리케이션이 프로덕션 환경에 있는 것처럼 행동하도록 세팅하는 것입니다. 

먼저 Rack-mini-profiler storage를 파일시스템에서 메모리로 변경합니다.

```ruby
# config/initializers/rackminiprofiler.rb

Rack::MiniProfiler.config.storage = Rack::MiniProfiler::MemoryStore
```

그 다음으로는 로컬환경에서 SSL을 disable시킵니다.

```ruby
# config/environments/production.rb

config.force_ssl = false
```

그 다음으로는 데이터베이스 셋업, asset complie, secret key base를 세팅합니다.

```bash
rails db:reset RAILS_ENV=production 
rails assets:precompile RAILS_ENV=production 
rails s RAILS_ENV=production SECRET_KEY_BASE=secret 
```

<br>
### Speed Badge

루트 url로 접속하면 스피드배지가 보일것입니다. 이는 어플리케이션이 렌더되는데 얼마나 걸린지를 나타내줍니다.

**duration (ms)** 총 리퀘스트 타임을 나타냅니다.
**query** 얼마나 많은 쿼리를 생성했는지 나타냅니다.
**% in sql** sql에서 소요된 시간의 백분율을 보여줍니다.

<br>
### Garbage Collection Profiling

**profile-gc**

'?pp=profile-gc' 을 url에 붙이면 엄청나게 긴 결과물이 나올것입니다. 시간이 좀 걸릴테니 커피라도 한 잔 마시며 기다려봅니다.

결과물이 나왔다면 먼저, New bytes allocated outside of Ruby heaps 섹션으로 가봅니다.
만약 이 부분이 8MB를 넘는다면, 이유를 조사해보아야합니다. 무언가가 많은 메모리를 소비하고 있기 때문입니다.

그 다음으로, ObjectSpace delta caused by request 섹션으로 갑니다.
만약 String : 7300 라는 문구가 있다고 한다면, request 이후에 7300개의 스트링이 더 있다는 뜻입니다.

마지막으로 String stats 섹션으로 가봅니다.
이는 스트링이 할당된 횟수입니다. 만약 비정상적으로 많이 할당되는 경우, 스트링을 frozen 상수로 옮김으로써 이 횟수를 줄일 수 있습니다. 
만약 request에 "hello"라는 스트링이 3000번 할당 되었다고 가정해봅시다. 이를 수정하기 위해 "hello" 스트링을 상수로 옮겨놓는다면 3000번의 할당 대신 한 번의 할당으로도 충분하게 됩니다. 

<br>

**profile-memory**

'?pp=profile-memory'을 url에 붙이면, 몇 번째 줄의 코드가 해당 오브젝트를 할당하고 있는지 알 수 있습니다.

retained 섹션으로 가봅시다. 
리퀘스트가 완료된 후 이러한 객체가 붙습니다. 이는 루비객체 누수의 일반적인 위치입니다.

<br>
### Exceptions are slow

'?pp=trace-exceptions'을 url에 붙인다면 당신의 애플리케이션이 에외를 일으키는 모든 지점을 리턴합니다.

예외는 느리므로 흐름 제어의 형태로 예외를 사용하면 안됩니다. 여기저기서 예외를 일으키는 대신 예외처리를 클래스에 위임한다면, 이 클래스에서 최종사용자에게 가능한한 빨리 예외관련된 무언가를 리턴할 것입니다.

Slow:

catch A -> raise B -> catch B -> raise C -> catch C -> return
Fast:

catch A -> handle exception & return

<br>
### 결론

Rack-mini-profiler에는 Rails와 같은 Rack기반 어플리케이션을 디버깅하고 프로파일링 할 수 있는 다양한 도구들이 있습니다. 이제 각 도구들에 대해 알게 되었으니 성능이 좋지않은 페이지들을 개선할 수 있을 것입니다. 