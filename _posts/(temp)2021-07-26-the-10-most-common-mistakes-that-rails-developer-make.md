---
layout: post
title: (번역) The 10 Most Common Mistakes That Rails Developers Make
author: sisipark
tags: [Ruby, Ruby on Rails, Rails Developer, 레일즈 개발자, 루비, 루비온레일즈]
excerpt_separator: <!--more-->
---
<span style="color: darkgrey">약 7년전 글이라 현재 트렌드와 맞지 않는 부분이 있을 수 있으나, 그럼에도 현재까지도 유효한 </span>
> 해당 포스트는 [The 10 Most Common Mistakes That Rails Developers Make](https://www.toptal.com/ruby-on-rails/top-10-mistakes-that-rails-programmers-make)를 읽고 해석(+의역)한 글입니다.

Ruby on Rails는 웹개발 프로세스를 간결하고 단순하게 하기 위해 개발된 루비언어를 베이스로 하는 오픈소스 프레임워크입니다.

<!--more-->

레일즈는 convention over configuration의 원칙에 따라 설계됩니다. 간단히 말해 개발자가 best practice 규칙(네이밍, 코드 구조 등)을 준수한다면 세부 사항을 신경쓰지 않아도 auto-magically 작동합니다.

이러한 패러다임은 많은 장점이 있지만, 그에 못지않게 함정도 많습니다. 특히, 이 프레임워크의 뒷단에서 일어나는 일들은 때때로 두통, 혼란, 그리고 "어떻게 돌아가는거야" 등의 문제를 야기할 수 있습니다. 또한 보안과 성능에 관해 바람직하지 않은 영향을 미칠 수도 있습니다.

따라서 레일즈는 사용하기 쉽지만, 그 반면 오용하기도 쉽습니다. 이 튜토리얼은 10가지의 가장 보편적이 레일즈의 문제를 살펴보며 그와 함께 이를 어떻게 피할 수 있는지, 이러한 이슈들의 원인들까지도 설명합니다.

<br>
### Mistake #1. 컨트롤러에 너무 많은 로직을 넣는다.

레일즈는 MVC 아키텍쳐를 기반으로 합니다. 한동안 레일즈 커뮤니티에서 계속하여 레일즈의 팻모델, 스키니 컨트롤러에 대해 얘기했지만, 최근 제가 이어받은 여러 레일즈 앱들은 이러한 원칙을 위반하고 있었습니다. 이 앱들은 모두 너무도 쉽게 뷰 로직(헬퍼에 넣는게 나은), 혹은 도메인/모델 로직을 컨트롤러에 넣는 실수를 범했습니다. 

문제는 컨트롤러가 단일 책임 원칙을 위반하기 시작하면서 향후 코드를 변경하기 어렵고 에러가 발생하기 쉬운 구조로 되었다는 것입니다. 
일반적으로, 컨트롤러에만 있어야 할 로직 유형은 다음과 같습니다.

+ **세션 및 쿠키 처리.** 여기에는 인증/인가 혹은 추가적인 쿠키 처리 프로세스도 포함될 수 있습니다.
+ **모델 선택.** request에서 전달된 파라미터가 지정된 올바른 모델을 찾는 로직입니다. 이상적으로는 나중에 응답을 렌더링하는 데 사용할 인스턴스 변수를 설정하는 단일 find 메소드에 대한 호출이어야 합니다. 
+ **request 파라미터 관리.** request 파라미터를 모으고 해당 파라미터들을 db에 입력할 적절한 모델 메서드를 호출합니다.
+ **렌더링/리다이렉팅.** 결과를 렌더링하거나(html, xml, json 등) 필요에 따라 리다이렉팅 합니다.

이는 여전히 컨트롤러의 단일 책임 원칙에 맞진 않지만, 레일즈 프레임워크가 컨트롤러에 요구하는 최소한의 수준입니다.

<br>
### Mistake #2. 뷰에 너무 많은 로직을 넣는다.

레일즈 템플릿 엔진 ERB는 다양한 컨텐츠로 페이지를 작성할 수 있는 좋은 방법입니다. 하지만 조심하지 않으면, HTML과 루비 코드가 뒤섞인 대용량 파일이 되어 관리/유지하기 어려워집니다. 또한 반복적인 코드가 많아질 수 있는 영역이므로 DRY 원칙을 위반할 수 있습니다. 

이것은 여러 가지 방법으로 나타날 수 있습니다. 하나는 뷰에서 조건부 로직을 남용하는 것입니다. 간단한 예로 current_user 메소드를 생각해 봅시다. 종종 다음과 같은 조건부 로직 구조가 뷰에 있게 됩니다. 

```ruby
<h3>
  Welcome,
  <% if current_user %>
    <%= current_user.name %>
  <% else %>
    Guest
  <% end %>
</h3>
```

이와 같은 작업을 처리하는 더 나은 방법은 current_user에 의해 반환되는 object가 로그인 여부에 관계 없이 항상 설정되어있는지 확인하고 뷰에서 사용되는 메소드에 적절히 응답하는지 확인하는 것입니다(null object라고도 함). 예를 들어 app/controllers/application_controller에 current_user 헬퍼를 다음과 같이 정의할 수 있습니다.

```ruby
require 'ostruct'

helper_method :current_user

def current_user
  @current_user ||= User.find session[:user_id] if session[:user_id]
  if @current_user
    @current_user
  else
    OpenStruct.new(name: 'Guest')
  end
end
```

이렇게 함으로써 이전 뷰 코드를 다음과 같이 간단한 코드 한 줄로 대체할 수 있습니다.

```ruby
<h3>Welcome, <%= current_user.name -%></h3>
```

추가적인 레일즈의 best practice는 아래와 같습니다.

+ 페이지에서 반복되는 부분들을 뷰 레이아웃과 partial을 적절히 사용하여 캡슐화 하세요.
+ Draper gem과 같은 presenters/decorators을 사용하여 루비 객체에서 뷰 빌드 논리를 캡슐화 할 수 있습니다. 이를 사용하여 메소드를 이 객체에 추가하여 뷰 코드에 입력했을 수도 있는 로직 작업을 수행할 수 있습니다.

<br>
### Mistake #3. 모델에 너무 많은 로직을 넣는다.

뷰와 컨트롤러에 들어가는 로직을 최소화 하기 위한 위의 가이드라인에 따르면 MVC 아키텍쳐에서 모든 로직을 넣을 유일한 곳은 모델밖에 없겠네요. 
하지만, 꼭 그렇진 않습니다.

많은 레일즈 개발자들이 실제로 이러한 실수를 저지르며 결국 ActiveRecord 모델 클래스에 모든 것을 집어넣어 단일책임 원칙을 위반할 뿐만 아니라 유지보수 역시 악몽이 되어버립니다. 

이메일 알림 생성, 외부 서비스와의 인터페이스, 다른 데이터 형식으로의 변환 등과 같은 기능은 데이터베이스에서 데이터를 찾고 유지하는 것 이상의 작업을 거의 수행하지 않는  ActiveRecord 모델의 핵심 책임과는 큰 관련이 없습니다. 

뷰에도, 컨트롤러에도 심지어 모델에도 로직이 들어가지 말아야 한다면 우리는 로직을 어디에 넣어야 하는 걸까요?

바로 Plain Old Ruby Objects(POROs)에 넣으면 됩니다. 레일즈와 같은 많은 것을 커버하는 프레임워크를 사용하는 경우 신입 개발자는 종종 프레임워크 외부에 자신만의 클래스를 만드는 것을 꺼려합니다. 그러나 로직을 모델에서 POROs로 옮기는 것은 단지 의사가 지나치게 복잡한 모델을 피하도록 지시한 것에 불과합니다. POROs를 사용하면 이메일 알람이나 API 인터랙션과 같은 내용을 ActiveRecord 모델에 박아넣는 대신 해당 클래스에 캡슐화하는 것이 가능합니다.

따라서 모델에 남아있어야 할 로직은 다음과 같습니다.

+ ActiveRecord 설정 (릴레이션이나 밸리데이션)
+ 몇 개의 attribute들을 업데이트하고 데이터베이스에 저장하는 것을 캡슐화하기 위한 mutation 메소드
+ 내부 모델 정보를 숨기기 위한 액세스 래퍼 (데이터베이스의 first name과 last name 필드를 결합하는 풀네임 메소드)
+ 고급쿼리(단순한 find메소드보다 복잡한 쿼리 등); 일반적으로 where 메소드라든지 다른 query-building 메소드를 모델 클래스 외부에서 사용하면 안됩니다.

<br>
### Mistake #4. 제네릭 헬퍼 클래스를 덤핑그라운드로 사용

이 실수는 위의 3번 실수에 대한 일종의 귀결점입니다. 지금까지 말해온 것과 같이, 레일즈 프레임워크는 MVC 프레임워크의 컴포넌트(모델, 뷰, 컨트롤러)에 중점을 둡니다. 이러한 각 컴포넌트의 클래스에 속하는 각각의 것들에는 나름 명확한 정의가 내려져 있습니다. 하지만 때때로 세가지 컴포넌트 중 어느 것에도 맞지 않는것같은 메소드가 필요할 수도 있습니다. 

레일즈 제너레이터는 우리가 만든 각 리소스와 함께 사용할 헬퍼 디렉토리와 헬퍼 클래스를 편리하게 만들어줍니다. 그러나 모델, 뷰, 컨트롤러에 맞지 않는 기능들을 이러한 헬퍼 클래스에 채우기 시작하는 것은 매우 유혹적입니다. 

레일즈는 확실히 MVC 중심적이지만, 어떠한 것도 사용자가 자신만의 클래스 유형을 만들고 해당 클래스에 대한 코드를 보유하기 위해 적절한 디렉토리를 추가하는 것을 막지는 않습니다. 추가 기능이 있는 경우 어떤 메소드를 함께 그룹화할지 생각하고 해당 메소드를 포함하는 클래스에 적합한 이름을 찾으세요. 레일즈와 같은 포괄적인 프레임워크를 사용하는 것은 좋은 객체지향 설계 모범 사례를 무시하기 위한 핑계가 될 수는 없습니다. 

<br>
### Mistake #5. 너무 많은 Gem을 사용

루비와 레일즈는 개발자가 생각할 수 있는 거의 모든 기능을 제공하는 풍부한 Gem 생태계의 지원을 받고 있습니다. 이는 복잡한 애플리케이션을 신속히 구현하는데에는 효과적이지만 애플리케이션의 Gem파일에 있는 Gem 수가 제공된 기능과 비교할 때 불균형적으로 많은 애플리케이션도 많이 봐왔습니다. 

이는 여러 문제를 일으킵니다. Gem을 과도하게 사용하면 레일즈 프로세스의 크기가 필요 이상으로 커집니다. 이로인해 프로덕션 환경에서 애플리케이션의 성능을 저하시킬 수 있습니다. 사용자의 불만 외에도, 더 큰 서버 메모리 구성과 운영 비용 증가로 이어질 수도 있습니다. 또한 이렇게 커진 레일즈 애플리케이션을 시작하는 데 시간이 더 오래 걸려 개발 속도가 느려지고 자동화된 테스트에 걸리는 시간이 더 길어집니다(일반적으로 느린 테스트는 자주 실행되지 않습니다).

애플리케이션에 설치하는 각 Gem은 다른 Gem에 종속될 수 있으며, 그 Gem이 또 다른 Gem에 종속될 수도 있습니다. 따라서 다른 Gem을 추가하면 복합적인 효과가 있을 수도 있습니다. 예를 들어 rails_admin Gem을 추가하면 총 11개의 Gem이 더 유입되는데 이는 기본 Rails에 설치된 Gem에서 10%이상 증가하는 것입니다. 

새로 설치된 Rails 4.1.0의 Gemfile.lock 파일에는 43개의 표준 Gem이 포함되어 있습니다. 이는 분명 Gemfile에 포함되어 있는 것 이상이며, 소수의 기본 Rails Gem이 종속성으로 가져온 Gem인 것입니다. 

그러므로 각 Gem을 추가할 때는 추가적인 오버헤드가 가치가 있는지 신중하게 고려해야합니다. 예를 들어 레일즈 개발자들은 rails_admin Gem이 사용하기 간편하기 때문에 쉽게 추가할 수 있지만 실제로는 고급 데이터베이스 탐색 도구 그 이상은 아닙니다. 애플리케이션에 어드민 유저가 필요하더라도 해당 유저에게 raw 데이터베이스 액세스 권한을 부여하고 싶지 않을 수 있으며 직접 개발함으로써 더 나은 서비스를 제공할 수도 있습니다. 






RubyGem 덕분에 단지 'gem install' 명령어만으로 유용한 라이브러리들의 풍부한 에코시스템을 누릴 수 있다

주어진 (non-gem)프로젝트에 필요한 gem 목록은 Gemfile이라는 특수한 파일에 리스트되며 번들러가 자동으로 설치할 수 있다. 두가지 개념 모두 본 글의 뒷부분에서 다룰 것이다.

<br>
### 몇가지 gem 예

+ Rails 및 모든 구성요소 (ActiveRecord, ActiveSupport 등) 루비gem으로 배포
+ Pry, irb의 강력한 대체제
+ Nokogiri, 유명한 XML & HTML 파서

대부분의 gem들은 순수 루비로 작성되었으며 몇몇 gem들은 성능향상을 위해 Ruby C 확장 기능이 포함되어 있다.

<br>
## 루비gem을 만드는 방법

bundle gem <name> 을 실행하여 새 gem에 대한 파일을 준비할 수 있다.

### example)

```ruby
bundle gem awsome_gem
```

gem은 아래와 같은 구조로 되어있다.

```ruby
├── awesome_gem.gemspec
├── bin
│   ├── console
│   └── setup
├── Gemfile
├── lib
│   ├── awesome_gem
│   │   └── version.rb
│   └── awesome_gem.rb
├── Rakefile
├── README.md
└── test
    ├── awesome_gem_test.rb
    └── test_helper.rb
```

gem에 관한 모든 정보는 .gemspec 파일에 들어있다.

.gemspec은 아래와 같은 정보를 포함한다.

+ gem 이름
+ gem에 대한 짧은 설명서
+ 제작자 이름
+ 종속성 목록
+ gem에 포함할 파일 목록
+ 옵션 : 제작자의 이메일 주소, 프로젝트 URL 등

gem 버전은 lib/<gem_name>/version.rb 안에 상수로 선언된다.

아래는 gemspec의 예이다.

```ruby
Gem::Specification.new do |spec|
  spec.name          = "awesome_gem"
  spec.version       = AwesomeGem::VERSION
  spec.authors       = ["Jesus Castello"]
  spec.summary       = "Example gem for article about Ruby gems"
  spec.files         = Dir['**/**'].grep_v(/.gem$/)
  spec.require_paths = ["lib"]
  spec.add_development_dependency "bundler", "~> 1.16"
  spec.add_development_dependency "rake", "~> 10.0"
  spec.add_development_dependency "minitest", "~> 5.0"
end
```

require_path배열은 필요할 때 루비가 gem파일들을 찾는 곳이다. 이렇게 하면 코드를 lib/<gem_name>/ 아래에 넣을 수 있으며 require "<gem_name>/<file_name>" 구문으로 요청할 수 있다.


당신이 gem 파일들을 요청하면 Ruby는 require_path의 배열에서 찾게 된다.

### example

파일명 lib/awesome_gem/parser.rb 은 gem내부 어디에서나 require "awesome_gem/parser" 문구로 요청될 것이다.
대부분의 요구 사항은 lib/<gem_name>.rb(/lib의 루트에 있는 유일한 파일)에서 찾을 수 있다.

이것이 당신이 gem을 require할 때 로드되는 파일이다.

<br>

add_development_dependency 라인은 개발 중에 사용할 gem을 정의한다. 이것들은 minitest, RSpec, 또는 pry과 같은 gem이다.

반면 add_dependency는 코드의 일부로 사용하는 gem을 정의한다.
요약 및 설명을 기본값에서 변경한 후에는 bundle gem이 생성하는 bin/console 프로그램을 사용하여 gem으로 irb 세션을 로드할 수 있다.

### example
```ruby
$ bin/console
irb(main):001:0>
irb(main):002:0>
irb(main):003:0> AwesomeGem
=> AwesomeGem
irb(main):004:0>
irb(main):005:0> AwesomeGem::VERSION
=> "0.1.0"
```
그런 다음 gem build <name>.gemspec을 사용하여 gem을 패키징하고 gem push를 사용하여 rubygems.org에 게시할 수 있다.

<br>
## 번들러란 무엇인가

번들러는 종속성 관리를 위한 도구이다.
RubyGems가 이미 처리한 것 아닌가라고 물을 수 있다.
하지만 RubyGems는 gem 자체만을 위해서 종속성 처리를 한다. 
당신의 일반 루비 애플리케이션은 gem으로 만들어진게 아니므로 이 기능을 얻지 못한다. 
이것이 번들러가 존재하는 이유이다.

<br>
## gem 파일 이해

gem파일은 당신의 애플리케이션에서 필요한 젬 목록을 적는 곳이 바로 gem파일이다.
gem파일에 적힌 gem들은 require없이도 로드된다.

<br>
### gem파일 예

```ruby
ruby '2.5.0'
gem 'rails', '~> 5.2.1'
gem 'sqlite3'
gem 'puma', '~> 3.11'
gem 'bootsnap', '>= 1.1.0', require: false
```

Bundler(및 버전 2.0 이후 RubyGems)는 이 파일을 읽고 요청한 버전의 gem을 설치할 수 있다.

bundle install 명령을 실행할 때 다음과 같이 표시된다.

```ruby
Using turbolinks-source 5.1.0
Using turbolinks 5.1.1
Using uglifier 4.1.18
Using web-console 3.6.2
Bundle complete! 18 Gemfile dependencies, 78 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

gem파일의 모든 gem 버전을 선언할 때 사용되는 기호(~>)를 통해 다양한 버전을 요청할 수 있다. 
"나는 버전이 1.2보다 같거나 크지만 2.0보다 작기를 원한다"와 같은 것 역시도 말할 수 있다.


<br>
### Gemfile Options & Gemfile.lock

gem파일에 gem을 require할 때 몇 가지 옵션을 사용할 수 있다.
예를 들어,
github같은 다른 출처에서 gem을 pull해야 할 수도 있다.
이 기능은 아직 rubygems.org에 릴리즈되지 않은 프로젝트라도 최신버전을 사용할 때 유용하다.

```ruby
gem "rails", git: "git@github.com:rails/rails.git"
```
브랜치 옵션을 전달하여 마스터가 아닌 브랜치의 코드를 사용할 수 있다. 아래와 같다.

```ruby
gem "awesome_print", git: "git@github.com:awesome-print/awesome_print.git", branch: "v2"
```

또다른 옵션으로는 require: false가 있다.
이는 번들러가 gem을 auto-require하지 않도록 한다. 즉, 당신이 필요할 때 당신의 코드에서 직접 require해야 한다는 의미이다.


마지막으로 번들러는 Gemfile.lock을 생성한다.
Gemfile.lock은 자동으로 생성되며 설치된 모든 gem의 버전을 정확하게 나타낸다. 
번들러는 이러한 버전을 설치하므로 이 애플리케이션을 프로덕션 환경에 배포하거나 프로젝트를 다른 개발자와 공유할 때 모든 사용자가 동일한 gem세트를 사용하여 작업할 수 있다.


<br>

[^1]: 
    {% include citation.html key="ref1" %}