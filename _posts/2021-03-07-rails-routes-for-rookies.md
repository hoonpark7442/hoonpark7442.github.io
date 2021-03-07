---
layout: post
title: Rails Routes for Rookies (번역)
author: sisipark
tags: [Ruby, Ruby on Rails, routes, nested routes, 루비, 루비온레일즈, 라우트]
excerpt_separator: <!--more-->
---

> 해당 포스트는 [Rails Routes for Rookies](https://medium.com/@tafby88/rails-routes-for-rookies-13d5fa2decdd)를 읽고 해석한 글입니다.

최근, Tracoon라 불리는 프로젝트를 완료했는데 해당 프로젝트에서 우리가 자주 사용했던 컨셉 중 하나는 바로 라우트였습니다. 라우트를 레일즈 프로젝트에서 어떻게 사용하는지에 대해 파악할 수 있었지만, 좀 더 깊게 이해하고 싶어졌습니다. 
<!--more-->

## Routes

레일즈에서 라우트의 역할은 HTTP 리퀘스트를 처리해야할 컨트롤러의 액션에 적절히 맵핑해주는 것입니다. 리퀘스트를 받게 되면, 레일즈는 먼저 routes.rb 파일에 정의된 각 경로를 스캔하며 리퀘스트의 verb 및 URL 조합과 일치하는 것이 있는지 확인합니다. 레일즈는 해당 요청을 일치하는 경로로 보내고, 엔드포인트에 경로가 없다면 'no routes match error'를 발생시킵니다. 

{% include aligner.html images="blogs/railsroutes.png" column=1 %}
<center><small style="color:grey">출처 : https://medium.com/@tafby88/rails-routes-for-rookies-13d5fa2decdd</small></center>
<br>


라우트의 또 다른 기능 중 하나는 경로에서 특정 부분을 추출하여 컨트롤러의 param hash에서 사용가능하게끔 해준다는 것이다. 예를 들어, *GET /racoon/:id* 라는 리퀘스트를 받았을 때, id 부분을 컨트롤러의 param hash로 사용 가능하다. 좀 더 구체적인 예로는, *localhost:3000/racoons/6*이라는 리퀘스트를 받았다면 라우트는 라쿤 컨트롤러의 show 액션으로 해당 id를 함깨 보내며 우리에게 id 6번의 라쿤 데이터를 반환해준다. 

```ruby
class RaccoonsController < ApplicationController
  def index
  	@raccoons = Raccoon.all
  end

  def show
  	@raccoon = Raccoon.find(params[:id])
  end
end
```

<br>
## Resources

당신은 직접 당신만의 라우트와 액션을 사용할 수 있지만, RESTful routing을 사용하는 것이 베스트 프랙티스이다. 
REST란 REpresentational State Transfer의 약자이며 일반적으로 가장 많이 쓰이는 API 패턴이다. REST API에는 객체에서 사용해야할 7가지의 메인 타입의 액션이 있다. 아래의 이미지는 그 액션들을 나타낸 표이다. 주목해야할 점은 URL이 동일하지만 HTTP verb가 다르다는 점이다. 각 verb와 URL의 조합으로 해당되는 액션으로 라우팅된다.

{% include aligner.html images="blogs/rest_routes.png" column=1 %}
<center><small style="color:grey">출처 : https://medium.com/@tafby88/rails-routes-for-rookies-13d5fa2decdd</small></center>
<br>

config/routes.rb 파일에서 이러한 방식으로 경로를 입력할 수 있다.

```ruby
# config/routes.rb

get "/raccoons", to: "raccoons#index"
get "/raccoons/:id", to: "raccoons#show"
get "/raccoons/new", to: "raccoons#new"
post "/raccoons", to: "raccoons#create"
get "/raccoons/:id/edit", to: "raccoons#edit"
put "/raccoons/:id", to: "raccoons#update"
delete "/raccoons/:id", to: "raccoons#destroy"
```

레일즈는 이 7가지 라우트를 헬퍼 메소드를 통해 한번에 제공해준다. *resources :raccoons* 라고 적는다면, 위의 7가지 메소드를 직접 적을 필요없이 단 한줄의 코드로 모든 메소드를 사용할 수 있게 된다. 

```ruby
# config/routes.rb

resources :raccoons
```

만약 7가지 메소드 모두 필요하지는 않다면? 아래와 같이 only, 혹은 except 기능을 통해 필요한 메소드만 적용 가능하다.

```ruby
# config/routes.rb

resources :raccoons, only: [:index, :show] # 7개의 라우트 중 index와 show만 적용된다
resources :raccoons, except: [:index] # 7개의 라우트 중 index를 제외한다
```

<br>
## rails routes command

당신이 정의한 라우트를 확인하고 싶다면 간단히 터미널에 *rails route*명령어만 적어주면 된다. 

{% include aligner.html images="blogs/routes_command.png" column=1 %}
<center><small style="color:grey">출처 : https://medium.com/@tafby88/rails-routes-for-rookies-13d5fa2decdd</small></center>
<br>

## Renaming URLs

물론 리네임도 가능하다. 예를 들어, 만약 유저의 로그인을 위해 새로운 세션을 생성하는 컨트롤러가 있다고 해보자. URL은 *localhost:3000/sessions/new* 이러한 형태가 될 것이다. 하지만 이는 직관적이지 않을 수 있다. 만약 URL이 *localhost:3000/login* 이러하다면 어떠한가. 좀 더 그럴싸한 URL로 보인다. 

```ruby
# config/routes.rb

get "/login", to: "sessions#new", as: "login"
```

<br>
## Nested Routes

예를 들어, 라쿤페이지에 댓글을 추가하고 싶다고 해보자. 중첩 라우트를 사용한다면 특정한 라쿤 페이지에 대한 댓글을 찾을 수 있게 해준다. 

```ruby
# config/routes.rb

resources :raccoons do 
	resources :comments
end

```

터미널에서 rails routes 명령어를 입력한다면 아래와 같은 라우트 리스트를 확인할 수 있다.

{% include aligner.html images="blogs/nested_routes.png" column=1 %}
<center><small style="color:grey">출처 : https://medium.com/@tafby88/rails-routes-for-rookies-13d5fa2decdd</small></center>
<br>

raccoon 라우트 아래에 comment가 중첩되어 있는 것을 볼 수 있을 것이다. 이러한 URL을 통해 요청된 객체를 찾기 위해 필요한 두 params, raccoon_id와 id(comment id이다)에 접근할 수 있다. 
하지만 너무 많은 중첩은 프로그램의 복잡성을 높일 수 있으니 최대 2개 정도의 중첩 라우트를 사용하길 권장한다. 

<br>

[^1]: 
    {% include citation.html key="ref1" %}
