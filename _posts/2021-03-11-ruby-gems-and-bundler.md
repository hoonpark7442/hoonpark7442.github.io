---
layout: post
title: Ruby Gems, Gemfile & Bundler (번역)
author: sisipark
tags: [Ruby, Ruby on Rails, gem, gemfile, bundler, 루비개발자, 루비, 루비온레일즈]
excerpt_separator: <!--more-->
---

> 해당 포스트는 [Ruby Gems, Gemfile & Bundler (The Ultimate Guide)](https://www.rubyguides.com/2018/09/ruby-gems-gemfiles-bundler/)를 읽고 해석(+의역)한 글입니다.

Ruby gem이란 무엇인가? gem은 당신이 다운로드받거나 설치할 수 있는 패키지이다. 당신이 설치된 gem을 require하면 당신의 루비 프로그램애 추가적인 기능을 더할 수 있다.

<!--more-->
<br>
### gem은 당신에게 아래와 같은 것들을 가능하게 한다
+ 당신의 앱에 로그인 기능을 더한다
+ API와 같은 외부 서비스를 쉽게 적용 가능하다
+ 웹 어플리케이션을 만들 수 있다

이는 수많은 gem들 중 일부분이다.  
<br> 
### 왜 우리는 gem을 사용하는가?

+ 루비에서 라이브러리 및 도구를 공유하는 방법이다
+ gem의 파일 구조와 포맷으로 작동 방식을 쉽게 이해할 수 있다
+ 모든 gem과 함께 제공되는 .spec 파일은 종속성(기타 필요한 gem들)을 설명하므로 코드가 작동해야 하는 모든 것을 갖는다

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